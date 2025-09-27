---
title: Using TanStack DB & ElectricSQL to build real-time apps with optimistic updates
date: 2025-6-30 15:00:00
tags:
   - Realtime
   - TanStackDB
   - ElectricSQL
cover: /img/cover/tanstackdb_with_electricsql.png
mermaid: true
---

An Integration Guide to TanStack DB & ElectricSQL: Building Real-Time Apps with Optimistic Updates

## Abstract

This document is an in-depth technical guide providing a clear, reusable example of how to combine TanStack DB and ElectricSQL to build modern web applications with **real-time data synchronization**, **secure optimistic updates**, and **transactional consistency**. We will explore the core architecture and best practices through a Q&A application centered around a `messages` table.

> This guide was written based on the following key dependency versions:

> - `@electric-sql/client`: `^1.0.4`
> - `@electric-sql/experimental`: `^1.0.4`
> - `@electric-sql/react`: `^1.0.4`
> - `@tanstack/db`: `^0.4.1`
> - `@tanstack/react-db`: `^0.1.23`
> - `@tanstack/electric-db-collection`: `^0.1.25`

## Architecture Overview

Our Q&A application architecture is divided into a client and a server, communicating via API calls and a real-time data stream. The interaction model of the core components is as follows:

```mermaid
graph TB
    subgraph "Client: App"
        Component[React Component] --> Action[TanStack DB Action]
        Action --> |Optimistic Update| Collection[DB Collection: Messages]
        Collection --> |Shape Request| Auth[Shape Auth Proxy]
    end
    
    subgraph "Server: API & Gatekeeper"
        Auth --> |Validated Request| ShapeProxy[Shape Proxy Endpoint]
        ShapeProxy --> Electric[ElectricSQL Service]
        
        Action --> |API Call: /questions/send| APIEndpoint[API Endpoint]
        APIEndpoint --> |DB Transaction| Postgres[PostgreSQL Database]
    end
    
    Postgres --> |Replication| Electric
    Electric --> |Live Data Stream| Collection

    style Auth fill:#f9f,stroke:#333,stroke-width:2px
    style APIEndpoint fill:#ccf,stroke:#333,stroke-width:2px
    style Action fill:#cfc,stroke:#333,stroke-width:2px
```

## 1. Foundation: The Shape Gatekeeper Security Model

To securely sync the database to the client in real-time, we cannot expose it directly. ElectricSQL uses the concept of "Shapes" to define subsets of data that a client can subscribe to. Our architecture adds a "Shape Gatekeeper" layer to validate and authorize these data subscription requests.

### 1.1. Dual JWT Authentication

Our authentication system employs a dual JWT model for flexible and secure data access control:

1.  **Auth Token**: A standard JWT used to verify the user's identity, issued by the main authentication service.
2.  **Shape Token**: A short-lived, special-purpose JWT. Before subscribing to a specific data table (a Shape), the client must exchange its valid **Auth Token** for a **Shape Token** from the backend. The claims of this token contain the exact table and query conditions (e.g., `WHERE conversation_id = 'conv-abc'`) it is allowed to access, thus enabling row-level security.

### 1.2. Step 1: Shape Authorization Endpoint (`/shapes/auth-token`)

The client exchanges tokens by calling the backend's `/shapes/auth-token` endpoint.

```typescript
// File path: apps/api/src/routes/main.ts
// A Hono-based backend route implementation

mainApp.post(
  "/shapes/auth-token",
  jwtAuth, // Middleware to validate the user's Auth Token
  zValidator("json", z.object({ table: z.string(), params: z.any() })),
  async (c) => {
    const { table, params } = c.req.valid("json");
    const user = c.get("user"); // User info parsed from the Auth Token

    let whereClause: string | undefined;

    // Dynamically generate the SQL WHERE clause based on the requested table and user identity
    if (table === "messages") {
      const conversationId = params?.conversationId;
      if (!conversationId) {
        // Disallow access to any messages if conversationId is not provided
        whereClause = `conversation_id = ''`;
      } else {
        // Business logic to verify user's ownership of the `conversationId` should be here.
        // For example, query the database to confirm the `conversationId` belongs to the current `user.userId`.
        const hasPermission = await checkConversationPermission(user.userId, conversationId);
        if (!hasPermission) {
          throw new ForbiddenError("No permission to access messages of this conversation");
        }

        // Authorize access to all messages in this conversation
        whereClause = `conversation_id = '${conversationId}'`;
      }
    } else {
      throw new ForbiddenError(`Access to table not allowed: ${table}`);
    }
    
    // Encode the authorization info (table name and WHERE clause) into a new Shape Token
    const shapeToken = await JWTService.generateShapeToken({ table, where: whereClause });

    return c.json({ token: shapeToken });
  }
);
```

### 1.3. Step 2: Secure Shape Proxy Endpoint (`/shape`)

After obtaining a short-lived `Shape Token`, the client does not request the ElectricSQL service directly. Instead, it requests our custom `/shape` proxy endpoint. This endpoint's responsibility is to **validate the Shape Token** and securely forward the request to the actual ElectricSQL service.

```typescript
// File path: apps/api/src/routes/main.ts
import { proxy } from 'hono/proxy'; // Ensure proxy is imported

mainApp.get(
  "/shape", 
  shapeAuth, // Key middleware: validates the Shape Token
  async (c) => {
    // 1. Get authorization parameters from the JWT payload.
    // The `shapeAuth` middleware has already validated the JWT and placed its payload
    // in the request context. This is the source of truth, not the client's URL query params.
    const { table, where } = c.get("shapePayload");
    
    const electricServiceUrl = new URL(`${process.env.ELECTRIC_API_URL}/v1/shape`);

    // 2. Forward only non-security-related client query parameters (e.g., `offset`).
    const clientRequestUrl = new URL(c.req.url);
    clientRequestUrl.searchParams.forEach((value, key) => {
      if (key !== "table" && key !== "where") {
        electricServiceUrl.searchParams.set(key, value);
      }
    });

    // 3. Construct the final request using the validated parameters from the JWT.
    // This prevents the client from tampering with the request to access unauthorized tables or rows.
    electricServiceUrl.searchParams.set("table", table);
    if (where) {
      electricServiceUrl.searchParams.set("where", where);
    }

    // 4. Proxy the well-formed, secure request to the ElectricSQL service.
    // The proxy function is from the 'hono/proxy' library.
    return proxy(electricServiceUrl.toString(), {
      ...c.req,
      headers: {
        ...c.req.headers,
      },
    });
});
```

### 1.4. Client-side Collection Definition

On the client, we create a `createMessagesCollection` factory function that encapsulates all the logic for obtaining a Shape Token and configuring ElectricSQL's data synchronization.

```typescript
// File path: apps/web/src/hooks/data/collections.ts
import { createCollection } from '@tanstack/react-db';
import { electricCollectionOptions } from '@tanstack/electric-db-collection';
import { selectMessageSchema, type Message } from '@/db/schema'; // Zod Schema
import { createShapeConfig } from './shape-config'; // Import the Shape config factory
import { getApiBaseUrl } from './api'; // Import the API base URL getter

// Internal function to create the actual collection
function _createMessagesCollection(conversationId: string) {
  // Optional: specify columns to sync for performance optimization
  const columnsToSync = [
    'id',
    'conversation_id',
    'content',
    'role',
    'status',
    'created_at',
    'updated_at',
  ].join(',');

  // `createShapeConfig` is a helper function encapsulating auth and error handling logic
  const shapeConfig = createShapeConfig('messages', { conversationId }, undefined, columnsToSync);

  return createCollection(
    electricCollectionOptions({
      id: `messages-${conversationId}`,
      schema: selectMessageSchema,
      shapeOptions: {
        url: shapeConfig.url || `${getApiBaseUrl()}/shape`,
        params: shapeConfig.params,
        headers: shapeConfig.headers,
        onError: shapeConfig.onError,
        parser: shapeConfig.parser, // Optional custom parser for data transformation
      },
      getKey: (item: Message) => item.id,
    }),
  );
}

// Type alias for better type safety
type MessagesCollectionType = ReturnType<typeof _createMessagesCollection>;

// Global cache to avoid creating duplicate Collection instances
const messagesCollectionsCache = new Map<string, MessagesCollectionType>();

/**
 * Creates a messages collection with caching
 *
 * @example
 * // ✅ Correct usage
 * const messagesCollection = createMessagesCollection(conversationId);
 *
 * // ❌ Wrong usage - DO NOT wrap in useMemo
 * const messagesCollection = useMemo(() => createMessagesCollection(conversationId), [conversationId]);
 *
 * @param conversationId - The conversation ID to filter messages
 * @returns Cached collection instance
 */
export function createMessagesCollection(conversationId: string) {
  const cacheKey = `messages-${conversationId}`;

  if (messagesCollectionsCache.has(cacheKey)) {
    return messagesCollectionsCache.get(cacheKey)!;
  }

  const collection = _createMessagesCollection(conversationId);
  messagesCollectionsCache.set(cacheKey, collection);
  return collection;
}
```

### 1.5. Deep Dive: The `createShapeConfig` Factory for Resilient Connections

The core value of `createShapeConfig` is its `onError` callback, which provides an elegant, automated **connection recovery mechanism** for handling expired Shape Tokens. The implementation details are provided in the appendix.

### 1.6. Appendix: Client-side Helper Function Implementations

To make this document self-contained, simplified versions of key helper functions are provided below.

```typescript
// File path: apps/web/src/lib/api.ts

// In a real application, this URL might be dynamic (e.g., from .env or a Zustand store)
export function getApiBaseUrl(): string {
  return 'https://your-api.example.com'; 
}

// In a real application, API call functions would use fetch or axios
export async function sendQuestionApi(params: any): Promise<{ txid: string }> {
  const response = await fetch(`${getApiBaseUrl()}/questions/send`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${getAuthToken()}`, // Function to get the user's Auth Token
    },
    body: JSON.stringify(params),
  });
  if (!response.ok) throw new Error('API request failed');
  return response.json();
}
```

```typescript
// File path: apps/web/src/stores/shape-token-store.ts
// A Shape Token cache implemented with Zustand

import { create } from 'zustand';

interface ShapeTokenState {
  tokenCache: Map<string, string>; // Key: `${table}|${JSON.stringify(params)}`
  getToken: (key: string) => string | undefined;
  setToken: (key: string, token: string) => void;
  removeToken: (key: string) => void;
}

export const useShapeTokenStore = create<ShapeTokenState>((set, get) => ({
  tokenCache: new Map(),
  getToken: key => get().tokenCache.get(key),
  setToken: (key, token) => set(state => ({ tokenCache: new Map(state.tokenCache).set(key, token) })),
  removeToken: (key) => {
    set((state) => {
      const newCache = new Map(state.tokenCache);
      newCache.delete(key);
      return { tokenCache: newCache };
    });
  },
}));
```

```typescript
// File path: apps/web/src/hooks/data/shape-config.ts
import { type ShapeOptions } from '@electric-sql/react';
import { FetchError } from '@electric-sql/client';
import { useShapeTokenStore } from '@/stores/shape-token-store';
import { getApiBaseUrl } from '@/lib/api';

// A simplified fetchShapeToken function
async function fetchShapeToken(table: string, params: Record<string, any>): Promise<string> {
   const response = await fetch(`${getApiBaseUrl()}/shapes/auth-token`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${getAuthToken()}`, // Function to get the user's Auth Token
      },
      body: JSON.stringify({ table, params }),
    });
    if (!response.ok) throw new Error('Failed to fetch shape token');
    const data = await response.json();
    return data.token;
}


export function createShapeConfig<T extends Row>(
  table: string,
  params: Record<string, any>,
): ShapeOptions<T> {
  const cacheKey = `${table}|${JSON.stringify(params)}`;

  const getAuthHeader = async () => {
    // Check cache first
    const cachedToken = useShapeTokenStore.getState().getToken(cacheKey);
    if (cachedToken) return `Bearer ${cachedToken}`;

    // On cache miss, fetch a new token
    const newShapeToken = await fetchShapeToken(table, params);
    useShapeTokenStore.getState().setToken(cacheKey, newShapeToken);
    return `Bearer ${newShapeToken}`;
  };

  const onErrorFunc = async (error: Error) => {
    // Handle only 401 errors, which usually mean an expired token
    if (error instanceof FetchError && error.status === 401) {
      console.warn(`[Shape Auth] 401 Unauthorized. Invalidating token and retrying...`);
      // Key step: remove the expired token from the cache
      useShapeTokenStore.getState().removeToken(cacheKey);
      // Return an empty object to signal to ElectricSQL that this is a recoverable error, and it should retry automatically
      return {}; 
    }
    throw error;
  };

  return {
    params: { table, ...params },
    headers: { Authorization: getAuthHeader },
    onError: onErrorFunc,
  };
}
```

## 2. Core: Atomic Actions & The `txid` Bridge

When a user submits a new question, we need to update the client UI, call the backend API, and ensure eventual data consistency. This is where TanStack DB's `createTransaction` and ElectricSQL's `txid` mechanism come into play.

### 2.1. Transactional Action (`sendQuestionAction`)

We encapsulate the operation of sending a new question into an atomic, asynchronous Action function.

```typescript
// File path: apps/web/src/hooks/data/actions.ts
import { createTransaction } from '@tanstack/react-db';
import { createMessagesCollection } from './collections';
import { sendQuestionApi } from '@/lib/api'; // API call function
import { createQuestionPlaceholder, createAnswerPlaceholder } from './placeholders'; // Placeholder factories

export async function sendQuestionAction(params: {
  conversationId: string;
  questionContent: string;
  questionId: string;  // Pre-generated on the client
  answerId: string;    // Pre-generated on the client
}) {
  const { conversationId, questionContent, questionId, answerId } = params;
  const messagesCollection = createMessagesCollection(conversationId);

  const transaction = createTransaction({
    autoCommit: false,
    mutationFn: async () => {
      // 1. Call the backend API
      const response = await sendQuestionApi({
        conversationId,
        questionId,
        answerId,
        content: questionContent,
      });
      if (!response.txid) throw new Error('API did not return a transaction ID');

      // 2. Wait for ElectricSQL to sync this transaction back to the client.
      // This is the key to the seamless "optimistic -> real data" transition.
      // IMPORTANT: awaitTxId must be awaited, otherwise optimistic updates will be ignored.
      await messagesCollection.utils.awaitTxId(response.txid);
    },
  });

  // Optimistic Update
  // This part executes synchronously and immediately, making the UI feel instant.
  transaction.mutate(() => {
    const questionMessage = createQuestionPlaceholder({
      id: questionId,
      conversation_id: conversationId,
      content: questionContent,
    });
    const answerMessage = createAnswerPlaceholder({
      id: answerId,
      conversation_id: conversationId,
    });
    messagesCollection.insert([questionMessage, answerMessage]);
  });

  // Commit the transaction, which triggers the execution of `mutationFn`
  await transaction.commit();
}
```

### 2.1.1. Appendix: Placeholder Object Creation

The `createQuestionPlaceholder` and `createAnswerPlaceholder` functions used in the optimistic update are simple factories for creating objects that match the local database `Message` schema.

```typescript
// File path: apps/web/src/hooks/data/placeholders.ts
import type { Message } from '@repo/shared/types';

export function createQuestionPlaceholder(data: Partial<Message>): Message {
  const now = new Date();
  return {
    id: data.id!,
    conversation_id: data.conversation_id!,
    content: data.content || '',
    role: 'user', // or 'question'
    status: 'completed',
    created_at: now,
    updated_at: now,
    // ... other schema-required fields should have default values
  };
}

export function createAnswerPlaceholder(data: Partial<Message>): Message {
  const now = new Date();
  return {
    id: data.id!,
    conversation_id: data.conversation_id!,
    content: 'Generating answer...',
    role: 'assistant', // or 'answer'
    status: 'pending',
    created_at: new Date(now.getTime() + 1), // Ensure sequence
    updated_at: new Date(now.getTime() + 1),
    // ... other schema-required fields should have default values
  };
}
```

### 2.2. Backend API & `txid` Generation

The backend API's core responsibilities are:
1.  Execute all necessary write operations (inserting the question and the answer placeholder) within a **single database transaction**.
2.  **Obtain the transaction ID (`txid`)** from the database operation.
3.  Immediately return the `txid` to the client.
4.  Asynchronously trigger the subsequent answer generation task.

```typescript
// File path: apps/api/src/routes/main.ts
mainApp.post("/questions/send", jwtAuth, async (c) => {
  const { conversationId, questionId, answerId, content } = c.req.valid("json");
  const userId = c.get("user").userId;

  // 1. Create the question and answer placeholder in a single transaction
  const { txid } = await messageService.createQuestionAndAnswerPlaceholder({
    conversationId,
    userId,
    questionId,
    questionContent: content,
    answerId,
  });

  // 2. Asynchronously trigger answer generation
  setImmediate(() => {
    generateAnswer(conversationId, answerId, content)
      .catch(err => console.error("Answer generation failed:", err));
  });
  
  // 3. Immediately return the transaction ID
  return c.json({ success: true, txid });
});
```

### 2.2.1. Deep Dive: How the Backend Gets the `txid`

The `txid` is the bridge between the client's action and the backend's data synchronization. In our backend's `message.service.ts`, this ID is obtained via a transaction helper function that wraps all database write operations. Its core relies on a built-in PostgreSQL function, `pg_current_xact_id()`.

```typescript
// File path: apps/api/src/services/message.service.ts

export class MessageService {
  // ... other methods

  /**
   * A transaction wrapper that ensures all operations run within a transaction and returns the transaction ID.
   */
  private async withTransaction<T>(
    callback: (tx: DrizzleTransaction, txid: number) => Promise<T>,
  ): Promise<T> {
    // Use Drizzle's transaction method to start a database transaction
    return await db.transaction(async (tx) => {
      // Inside the transaction, first get the current transaction ID
      const txid = await this.getTxId(tx);
      // Then, execute the callback containing all database operations
      return await callback(tx, txid);
    });
  }

  /**
   * Gets the current transaction's ID.
   * Internal function used by various transaction wrappers
   *
   * Uses pg_current_xact_id()::xid::text to get the 32-bit internal xid,
   * which matches the ID sent in PostgreSQL logical replication streams,
   * and is the ID we use for matching in Electric
   */
  private async getTxId(tx: DrizzleTransaction): Promise<number> {
    try {
      // Use pg_current_xact_id()::xid::text to get the 32-bit internal xid
      // ::xid conversion removes the epoch, giving the raw 32-bit value
      // This matches the value PostgreSQL sends in logical replication streams
      const result = await tx.execute(sql`SELECT pg_current_xact_id()::xid::text as txid`);
      const txid = (result.rows[0] as { txid: string })?.txid;

      if (txid === undefined) {
        throw new Error("Failed to get transaction ID from pg_current_xact_id");
      }

      return Number.parseInt(txid, 10);
    }
    catch (error) {
      // Fallback to deprecated txid_current()
      console.warn("[Transaction] pg_current_xact_id() failed, falling back to txid_current():", error);
      try {
        const result = await tx.execute(sql`SELECT txid_current() as txid`);
        const txid = (result.rows[0] as { txid: bigint })?.txid;

        if (txid === undefined) {
          throw new Error("Failed to get transaction ID from txid_current");
        }

        return Number(txid);
      }
      catch (fallbackError) {
        console.error("[Transaction] Both txid methods failed:", fallbackError);
        throw new Error("Failed to get transaction ID from all available methods");
      }
    }
  }

  /**
   * Creates the question and answer placeholders in a single transaction.
   */
  async createQuestionAndAnswerPlaceholder(params: ...): Promise<{ txid: number }> {
    // Execute operations via withTransaction to ensure atomicity and get the txid
    return this.withTransaction(async (tx, txid) => {
      // 1. Insert question message...
      await tx.insert(messages).values({ ... });

      // 2. Insert answer placeholder...
      await tx.insert(messages).values({ ... });
      
      // 3. Return the txid obtained from the wrapper
      return { txid };
    });
  }
}
```
The beauty of this design is that the `withTransaction` method encapsulates the entire flow: "start transaction, get ID, execute operations, commit transaction." Any method needing to write to the database atomically and requiring a `txid` for ElectricSQL sync can simply use this helper.


### 2.3. How `awaitTxId` Works

The `txid` is the key that bridges the gap between the frontend's optimistic update and the backend's true data. The `await collection.utils.awaitTxId(txid)` function works as follows:

1.  **Listen to the Replication Stream**: It registers a temporary listener with ElectricSQL's client-side runtime.
2.  **Match the Transaction ID**: ElectricSQL receives the data replication stream from PostgreSQL. This stream includes metadata for each transaction, including its `txid`.
3.  **Resolve the Promise**: When the client-side runtime receives a data change that matches the `txid` it is waiting for, it knows that this specific transaction has been successfully synced from the server to the client's local database. At this point, the `awaitTxId` promise is resolved.

This mechanism elegantly guarantees that when the `commit()` function returns, our local database not only contains the optimistic placeholders but that these placeholders have already been overwritten by the persisted, true data from the server.

> **Critical Requirement**: `awaitTxId` **must** be awaited. If you don't use `await`, the UI will flicker and optimistic updates won't work properly.

## 3. The Payoff: Real-time UI with `useLiveQuery`

With a secure data channel and reliable data operations in place, we can now reap the benefits in the UI layer. TanStack DB's `useLiveQuery` hook allows us to effortlessly subscribe to changes in a Collection. The latest version of TanStack DB now supports loading states, making it easier to handle initial data synchronization and empty states.

```typescript
// File path: apps/web/src/components/question-answer-list.tsx
import { useLiveQuery } from '@tanstack/react-db';
import { createMessagesCollection } from '@/hooks/data/collections';

function QuestionAnswerList({ conversationId }: { conversationId: string }) {
  // Subscribe to the message collection for a specific conversation
  const messagesCollection = useMemo(() => createMessagesCollection(conversationId), [conversationId]);

  // The new useLiveQuery syntax with destructured data and loading state
  const {
    data: messages,
    isLoading,
  } = useLiveQuery(q =>
    q.from({ messagesCollection })
      .orderBy(({ messagesCollection }) => messagesCollection.created_at, 'desc')
);

  // Handle loading state - when there's no data, it stays in loading state
  if (isLoading) {
    return <div className="loading">Loading messages...</div>;
  }

  return (
    <div className="message-container">
      {messages?.map(msg => (
        <div key={msg.id} className={`message role-${msg.role} status-${msg.status}`}>
          <p>{msg.content}</p>
        </div>
      ))}
    </div>
  );
}
```

When a user submits a new question, `sendQuestionAction`'s `transaction.mutate()` immediately inserts the placeholders into the local database. `useLiveQuery` detects this change, and the `QuestionAnswerList` component re-renders instantly, showing the new question and "Generating answer...", achieving a perfect optimistic update. When the backend's answer generation task completes and updates the database, ElectricSQL syncs the change back to the client. `useLiveQuery` triggers another re-render, updating the answer placeholder with the real response.

## 4. End-to-End Flow

Let's summarize the entire process with a sequence diagram:
```mermaid
sequenceDiagram
    participant User
    participant Component as React Component
    participant Action as sendQuestionAction
    participant LocalDB as TanStack/Electric DB
    participant API as Backend API
    participant Postgres
    participant Electric as ElectricSQL Replication

    User->>Component: Clicks "Send" button
    Component->>Action: sendQuestionAction({ content: 'Hello!' })

    Action->>LocalDB: Optimistically insert Question & Answer placeholders
    Note right of LocalDB: UI instantly shows question and "Generating answer..."

    Action->>API: POST /questions/send
    API->>Postgres: BEGIN TRANSACTION
    API->>Postgres: INSERT question message
    API->>Postgres: INSERT answer placeholder
    API->>Postgres: COMMIT
    Postgres-->>API: Returns txid
    API-->>Action: Returns { success: true, txid: '...' }

    Action->>LocalDB: await collection.utils.awaitTxId(txid)
    Note right of LocalDB: Action pauses, listening for replication

    Postgres-->>Electric: Replicates committed transaction
    Electric-->>LocalDB: Pushes changes to client
    LocalDB-->>Action: awaitTxId promise resolves

    par Background Answer Generation
        API->>API: Generate Answer
        API->>Postgres: UPDATE answer message SET content = '...'
    end

    Postgres-->>Electric: Replicates answer
    Electric-->>LocalDB: Pushes final reply to client
    Note right of LocalDB: UI automatically updates to show the final reply
```
With this architecture, we have successfully built a powerful real-time application in a declarative, fault-tolerant, and efficient manner. This pattern elegantly encapsulates the complexities of UI updates, data persistence, and real-time synchronization, allowing developers to focus on implementing business logic.

## 5. Troubleshooting

### 5.1. JSONB Fields and TypeScript Types

When working with PostgreSQL JSONB fields in ElectricSQL, you'll encounter some important limitations and performance considerations:

#### Performance Impact

**⚠️ WARNING**: Avoid using JSONB fields for UI-critical data that is frequently accessed or rendered. JSONB fields can severely impact UI performance because:
- Each access requires parsing the JSON structure
- React re-renders are triggered for the entire JSONB object even when only a nested property changes
- Large JSONB structures can cause significant memory overhead

**Best Practice**: Use JSONB only for supplementary data that is:
- Rarely accessed in the UI
- Not part of the core rendering logic
- Used for metadata, configuration, or audit trails

#### TypeScript Type Issues

ElectricSQL's client SDK converts all JSONB fields to the generic `Json` type, which loses all type information. There is currently no automatic way to preserve types, requiring manual type assertions.

**Problem Example**:
```typescript
// Database schema definition with Drizzle ORM
const messages = pgTable("messages", {
  id: text("id").primaryKey(),
  content: text("content").notNull(),
  metadata: jsonb("metadata").$type<{
    author: { name: string; id: string };
    tags: string[];
    priority: number;
  }>(),
});

// Type inference from Drizzle
type Message = typeof messages.$inferSelect;
// Message.metadata type is: { author: ..., tags: ..., priority: ... } | null

// What ElectricSQL returns after synchronization
const selectMessageSchema = createSelectSchema(messages);
type ElectricMessage = z.infer<typeof selectMessageSchema>;
// ElectricMessage.metadata type is: Json | null (type information lost!)
```

**Solution - Transform Functions with Type Assertions**:
```typescript
// Using Drizzle-generated types and Zod schemas
const messages = pgTable("messages", {
  id: text("id").primaryKey(),
  content: text("content").notNull(),
  metadata: jsonb("metadata").$type<MessageMetadata>(),
  log_data: jsonb("log_data").$type<LogType1 | LogType2 | LogType3>(),
});

type Message = typeof messages.$inferSelect;
const selectMessageSchema = createSelectSchema(messages);

// Transform function to restore proper types after ElectricSQL sync
function transformElectricMessagesToTyped(
  electricMessages: Array<z.infer<typeof selectMessageSchema>>
): Message[] {
  return electricMessages.map(msg => ({
    ...msg,
    // Type assertions required for JSONB fields
    metadata: msg.metadata as Message['metadata'],
    log_data: msg.log_data as Message['log_data'],
  }));
}

// Usage in collection with automatic transformation
export function createMessagesCollection(conversationId: string) {
  const collection = createCollection(
    electricCollectionOptions({
      id: `messages-${conversationId}`,
      schema: selectMessageSchema,
      shapeOptions: { /* ... */ },
      getKey: item => item.id,
    }),
  );

  // Wrap collection methods to include transformation
  return {
    ...collection,
    findMany: async (filter?: any) => {
      const results = await collection.findMany(filter);
      return transformElectricMessagesToTyped(results);
    },
    // Add other wrapped methods as needed
  };
}
```

**Recommended Approach**:
1. **Flatten your data structure when possible** - use separate columns instead of JSONB to maintain type safety
2. **Use Drizzle's type inference with transform functions** - Define types in your Drizzle schema and create transform functions
3. **Implement proper collection caching pattern** to avoid duplicate instances:

```typescript
// Real-world example: Collection factory with proper caching
import { pgTable, text, jsonb, timestamp } from 'drizzle-orm/pg-core';
import { createSelectSchema } from 'drizzle-zod';
import type { ShapeOptions } from '@electric-sql/react';

// Define your table with typed JSONB
const documents = pgTable("documents", {
  id: text("id").primaryKey(),
  status: text("status").notNull(),
  metadata: jsonb("metadata").$type<{
    processor: string;
    version: string;
    tags: string[];
  }>(),
  created_at: timestamp("created_at").defaultNow(),
  updated_at: timestamp("updated_at").defaultNow(),
});

// Get proper TypeScript types
type Document = typeof documents.$inferSelect;
const selectDocumentSchema = createSelectSchema(documents);

// Internal collection factory (not exported)
function _createDocumentsCollection() {
  // Optimize by selecting specific columns
  const columnsToSync = [
    'id',
    'status',
    'metadata',
    'created_at',
    'updated_at',
  ].join(',');

  // Custom parser for date handling and type restoration
  const parser = {
    timestamp: (date: string) => new Date(date),
  } as unknown as ShapeOptions['parser'];

  const shapeConfig = createShapeConfig('documents', {}, parser, columnsToSync);

  return createCollection(
    electricCollectionOptions({
      id: 'documents',
      schema: selectDocumentSchema,
      shapeOptions: {
        url: shapeConfig.url,
        params: shapeConfig.params,
        headers: shapeConfig.headers,
        onError: shapeConfig.onError,
        parser: shapeConfig.parser,
      },
      getKey: (item: Document) => item.id,
    }),
  );
}

// Type alias for the collection
type DocumentsCollectionType = ReturnType<typeof _createDocumentsCollection>;

// Cache instance
const documentsCollectionsCache = new Map<string, DocumentsCollectionType>();

/**
 * Creates a documents collection with caching
 *
 * @example
 * // ✅ Correct usage - direct call
 * const docsCollection = createDocumentsCollection();
 *
 * // ❌ Wrong usage - DO NOT wrap in useMemo
 * const docsCollection = useMemo(() => createDocumentsCollection(), []);
 *
 * @returns Cached collection instance
 */
export function createDocumentsCollection() {
  const cacheKey = 'documents';

  if (documentsCollectionsCache.has(cacheKey)) {
    return documentsCollectionsCache.get(cacheKey)!;
  }

  const collection = _createDocumentsCollection();
  documentsCollectionsCache.set(cacheKey, collection);
  return collection;
}
```

**Key Points**:
- **Separate internal factory from public API**: Use `_createCollection` internally
- **Type-safe caching**: Use `ReturnType<typeof _createCollection>` for cache types
- **Never use `useMemo`**: Collections already handle their own caching
- **Column selection**: Optimize sync performance by specifying columns
- **Custom parsers**: Handle date conversions and other transformations

### 5.2. Simplified Authentication with Subqueries

ElectricSQL now supports subqueries (https://github.com/electric-sql/electric/discussions/2931), which can significantly simplify your authentication and authorization logic.

**Before (Complex Permission Checks)**:
```typescript
// Multiple database queries to check permissions
const hasPermission = await checkConversationPermission(user.userId, conversationId);
if (!hasPermission) {
  throw new ForbiddenError("No permission");
}
whereClause = `conversation_id = '${conversationId}'`;
```

**After (Using Subqueries)**:
```typescript
// Single WHERE clause with subquery handles everything
whereClause = `conversation_id IN (
  SELECT c.id FROM conversations c
  JOIN conversation_members cm ON c.id = cm.conversation_id
  WHERE cm.user_id = '${user.userId}'
    AND cm.role IN ('owner', 'member')
)`;
```

This approach:
- Reduces round trips to the database
- Moves authorization logic to the database layer
- Simplifies your backend code
- Ensures consistency between permission checks 