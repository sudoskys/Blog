---
title: JSUOP在Androlua中的初级应用
date: 2021-7-25 11:09:37
tags: Android
cover: false
---

## Jsoup概述

jsoup是java里的一个普遍使用的html解析器，其的逻辑简单，语法容易，且功能强大

直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。

我们在爬虫采集网页领域，主要作用是用HttpClient获取到网页后，具体的网页提取需要的信息的时候 ，就用到Jsoup，Jsoup可以使用强大的类似Jquery，css选择器，来获取需要的数据。

<br />但Androlua中并没有自带。

我们需要将其导入

##### Dex库下载链接：[Jsoup)](https://lua.dianas.cyou/pages/ffda64)

使用方法：工程目录下创建libs目录，将jsoup开头的dex文件放入,在程序开头加入 import'org.jsoup.*'即可

##### JAVA教程详见：[JSoup快速入门 - JSoup教程™ (yiibai.com)](https://www.yiibai.com/jsoup/jsoup-quick-start.html#article-start)

<br />

## Jsoup API

#### 从String解析文档

```lua
html = "<html><head><title>First parse</title></head>"
+ "<body><p>Parsed HTML into a doc.</p></body></html>"
doc = Jsoup.parse(html)
```

#### 解析一个网页碎片

```lua
html = "<div><p>Lorem ipsum.</p>"
doc = Jsoup.parseBodyFragment(html)
body = doc.body()

```

#### 从URL加载文档

```lua
doc = Jsoup.connect("http://example.com/").get()
title = doc.title()
```

#### 高级url请求

```lua
doc = Jsoup.connect("http://example.com")
.data("query", "Java")
.userAgent("Mozilla")
.cookie("auth", "token")
.timeout(3000)
.post()
```

#### 从文件加载文档

```lua
input = new File("/tmp/input.html")
doc = Jsoup.parse(input, "UTF-8", "http://example.com/")
```

#### DOM方法导航文档

```lua
--寻找元素
getElementById(String id)
getElementsByTag(String tag)
getElementsByClass(String className)
getElementsByAttribute(String key) （及相关方法）
--元素的兄弟姐妹：siblingElements()，firstElementSibling()，lastElementSibling()，nextElementSibling()，previousElementSibling()
--图：parent()，children()，child(int index)

--元素数据
attr(String key)获取和attr(String key, String value)设置属性
attributes() 获得所有属性
id()，className()和classNames()
text()获取和text(String value)设置文本内容
html()获取和html(String value)设置内部HTML内容
outerHtml() 获取外部HTML值
data()获取数据内容（例如script和style标签）
tag() 和 tagName()

--处理HTML和文本
append(String html)， prepend(String html)
appendText(String text)， prependText(String text)
appendElement(String tagName)， prependElement(String tagName)
html(String value)
```

#### Selector-syntax查找元素

##### 选择器概述

# 使用选择器语法来查找元素

## 问题

你想使用类似于CSS或jQuery的语法来查找和操作元素。

## 方法

可以使用 [Element (jsoup Java HTML Parser 1.14.1 API)](https://jsoup.org/apidocs/org/jsoup/nodes/Element.html#select(java.lang.String))和[Elements (jsoup Java HTML Parser 1.14.1 API)](https://jsoup.org/apidocs/org/jsoup/select/Elements.html#select(java.lang.String))方法实现：

```lua
File input = new File("/tmp/input.html");
Document doc = Jsoup.parse(input, "UTF-8", "http://example.com/");
 
Elements links = doc.select("a[href]"); //带有href属性的a元素
Elements pngs = doc.select("img[src$=.png]");
  //扩展名为.png的图片
 
Element masthead = doc.select("div.masthead").first();
  //class等于masthead的div标签
 
Elements resultLinks = doc.select("h3.r > a"); //在h3元素之后的a元素
```

jsoup elements对象支持类似于[CSS](http://www.w3.org/TR/2009/PR-css3-selectors-20091215/) (或[jquery](http://jquery.com/))的选择器语法，来实现非常强大和灵活的查找功能。.

这个`select` 方法在

[Document (jsoup Java HTML Parser 1.14.1 API)](https://jsoup.org/apidocs/org/jsoup/nodes/Document.html)

[Element (jsoup Java HTML Parser 1.14.1 API)](https://jsoup.org/apidocs/org/jsoup/nodes/Element.html)

[Elements (jsoup Java HTML Parser 1.14.1 API)](https://jsoup.org/apidocs/org/jsoup/select/Elements.html)

对象中都可以使用。且是上下文相关的，因此可实现指定元素的过滤，或者链式选择访问。

Select方法将返回一个[Elements (jsoup Java HTML Parser 1.14.1 API)](https://jsoup.org/apidocs/org/jsoup/select/Elements.html)集合，并提供一组方法来抽取和处理结果

### Selector选择器概述

* `tagname`: 通过标签查找元素，比如：`a`
* `ns|tag`: 通过标签在命名空间查找元素，比如：可以用 `fb|name` 语法来查找 `<fb:name>` 元素
* `#id`: 通过ID查找元素，比如：`#logo`
* `.class`: 通过class名称查找元素，比如：`.masthead`
* `[attribute]`: 利用属性查找元素，比如：`[href]`
* `[^attr]`: 利用属性名前缀来查找元素，比如：可以用`[^data-]` 来查找带有HTML5 Dataset属性的元素
* `[attr=value]`: 利用属性值来查找元素，比如：`[width=500]`
* `[attr^=value]`, `[attr$=value]`, `[attr*=value]`: 利用匹配属性值开头、结尾或包含属性值来查找元素，比如：`[href*=/path/]`
* `[attr~=regex]`: 利用属性值匹配正则表达式来查找元素，比如： `img[src~=(?i)\.(png|jpe?g)]`
* `*`: 这个符号将匹配所有元素

### Selector选择器组合使用

* `el#id`: 元素+ID，比如： `div#logo`
* `el.class`: 元素+class，比如： `div.masthead`
* `el[attr]`: 元素+class，比如： `a[href]`
* 任意组合，比如：`a[href].highlight`
* `ancestor child`: 查找某个元素下子元素，比如：可以用`.body p` 查找在"body"元素下的所有 `p`元素
* `parent > child`: 查找某个父元素下的直接子元素，比如：可以用`div.content > p` 查找 `p` 元素，也可以用`body > *` 查找body标签下所有直接子元素
* `siblingA + siblingB`: 查找在A元素之前第一个同级元素B，比如：`div.head + div`
* `siblingA ~ siblingX`: 查找A元素之前的同级X元素，比如：`h1 ~ p`
* `el, el, el`:多个选择器组合，查找匹配任一选择器的唯一元素，例如：`div.masthead, div.logo`

### 伪选择器selectors

* `:lt(n)`: 查找哪些元素的同级索引值（它的位置在DOM树中是相对于它的父节点）小于n，比如：`td:lt(3)` 表示小于三列的元素
* `:gt(n)`:查找哪些元素的同级索引值大于`n``，比如`： `div p:gt(2)`表示哪些div中有包含2个以上的p元素
* `:eq(n)`: 查找哪些元素的同级索引值与`n`相等，比如：`form input:eq(1)`表示包含一个input标签的Form元素
* `:has(seletor)`: 查找匹配选择器包含元素的元素，比如：`div:has(p)`表示哪些div包含了p元素
* `:not(selector)`: 查找与选择器不匹配的元素，比如： `div:not(.logo)` 表示不包含 class="logo" 元素的所有 div 列表
* `:contains(text)`: 查找包含给定文本的元素，搜索不区分大不写，比如： `p:contains(jsoup)`
* `:containsOwn(text)`: 查找直接包含给定文本的元素
* `:matches(regex)`: 查找哪些元素的文本匹配指定的正则表达式，比如：`div:matches((?i)login)`
* `:matchesOwn(regex)`: 查找自身包含文本匹配指定正则表达式的元素
* 注意：上述伪选择器索引是从0开始的，也就是说第一个元素索引值为0，第二个元素index为1等

可以查看

[Selector (jsoup Java HTML Parser 1.14.1 API)](https://jsoup.org/apidocs/org/jsoup/select/Selector.html)

 API参考来了解更详细的内容。

### 在AndroluaBox中关于selector的解释

```lua
------------
tagname：按标签查找元素，例如 a
ns|tag：在命名空间中按标记fb|name查找<fb:name>元素，例如查找元素
#id：按ID查找元素，例如 #logo
.class：按类名查找元素，例如 .masthead
[attribute]：具有属性的元素，例如 [href]
[^attr]：具有属性名称前缀的[^data-]元素，例如查找具有HTML5数据集属性的元素
[attr=value]：具有属性值的元素，例如[width=500]（也是可引用的[data-name='launch sequence']）
[attr^=value]，[attr$=value]，[attr*=value]：用与启动属性，以结束，或包含所述的值，例如元素[href*=/path/]
[attr~=regex]：具有与正则表达式匹配的属性值的元素; 例如img[src~=(?i)\.(png|jpe?g)]
*：所有元素，例如 *

---------------------------
选择器组合
el#id：具有ID的元素，例如 div#logo
el.class：带有类的元素，例如 div.masthead
el[attr]：具有属性的元素，例如 a[href]
任何组合，例如 a[href].highlight
ancestor child：从祖先下降的子元素，例如在类“body”的块下的任何位置.body p查找p元素
parent > child：直接从父级下降的子元素，例如div.content > p查找p元素; 并body > *找到body标签的直接子节点
siblingA + siblingB：找到兄弟B元素之后紧接着兄弟A，例如 div.head + div
siblingA ~ siblingX：找到兄弟A前面的兄弟X元素，例如 h1 ~ p
el, el, el：对多个选择器进行分组，找到与任何选择器匹配的唯一元素; 例如div.masthead, div.logo
伪选择器
:lt(n)：找到其兄弟索引（即它在DOM树中相对于其父节点的位置）小于的元素n; 例如td:lt(3)
:gt(n)：查找兄弟索引大于的元素n; 例如div p:gt(2)
:eq(n)：查找兄弟索引等于的元素n; 例如form input:eq(1)
:has(selector)：查找包含与选择器匹配的元素的元素; 例如div:has(p)
:not(selector)：查找与选择器不匹配的元素; 例如div:not(.logo)
:contains(text)：查找包含给定文本的元素。搜索不区分大小写; 例如p:contains(jsoup)
:containsOwn(text)：查找直接包含给定文本的元素
:matches(regex)：查找文本与指定正则表达式匹配的元素; 例如div:matches((?i)login)
:matchesOwn(regex)：查找自己的文本与指定正则表达式匹配的元素
注意，上面的索引伪选择器是基于0的，即第一个元素是索引0，第二个元素是1

从元素中提取属性，文本和HTML
要获取属性的值，请使用该Node.attr(String key)方法
对于元素（及其组合子元素）上的文本，请使用 Element.text()
对于HTML，使用Element.html()或Node.outerHtml()
上述方法是元素数据访问方法的核心。还有其他人：
Element.id()
Element.tagName()
Element.className() 和 Element.hasClass(String className)
所有这些访问器方法都有相应的setter方法来更改数据。



```

##### 操作HTML

```lua
--设置属性值
使用属性setter方法Element.attr(String key, String value)，和Elements.attr(String key, String value)。
如果需要修改class元素的属性，请使用Element.addClass(String className)和Element.removeClass(String className)方法。

--设置元素的HTML
div = doc.select("div").first()
div.html("<p>lorem ipsum</p>")
div.prepend("<p>First</p>")
div.append("<p>Last</p>")

Element span = doc.select("span").first()
span.wrap("<li><a href='http://example.com/'></a></li>")

--设置元素的文本内容
Element div = doc.select("div").first()
div.text("five > four")
div.prepend("First ")
div.append(" Last")
```

设置元素的文本内容
Element div = doc.select("div").first()
div.text("five > four")
div.prepend("First ")
div.append(" Last")

## 应用示例

##### 使用基本API

```lua
--导入jsoup库
import'org.jsoup.*'
--给出链接
local url = 'https://gitee.com/luo-ying2020/studyuseto'
--这里使用Http方法先获取到网页源码再解析。
--注意：jsoup自带的connect方法是同步加载，会影响程序加载速度
Http.get(url,function(code,content)
--回调
  if code==200 then--状态码200通信正常
    doc = Jsoup.parse(content)
    --使用jsoup解析网页
    print(doc.title())--使用jsoup方法获取到网页标题
  
  else
    print('无法访问')
  end
end)
```

##### DOM分类查询

```lua
--导入jsoup库
import'org.jsoup.*'
--link
url = 'https://gitee.com/luo-ying2020/studyuseto'
--request 
Http.get(url,function(code,content)
--if 200 , the website is ok
if code==200 then
doc = Jsoup.parse(content)--use jsoup解析
classification = doc.getElementsByClass('text-gray')--通过class查找为class名text-gray的网页元素
--上一行看不懂的去百度HTML基础

classification = luajava.astable(classification)--将其转换成table表
--luajava换成可以循环的table表

--for循环
--这里的v指的是table里面的每一条，k是第几个，for循环开始后，跑到最后的k的值即数据条数
for k,v in pairs(classification) do--循环打印输出
print(v.text())--.text()获取此条的text文本属性内容
end

else
print('无法访问')
end
end)
```

##### 函数实战展示

```lua
function GETHOME() 
  local home ='https://music.163.com/#/song?id=1491356576'--赋值 
  --导入jsoup库 
  require "import "  
--定义请求头  
header={ 
    [ "User-Agent "]=  "Mozilla/5.0 (Linux; U; Android 7.3.7; en-us; Nexus One Build/FRF91) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1 ", 
  } 
  --请求头设置结束 
--androlua自带HTTP
  Http.get(home,nil,"utf-8",header,function(code,content) --这里可以使用gb2312
    --这里使用Http方法先获取到网页源码再解析。 
    --因为jsoup自带的connect方法是同步加载，会影响程序加载速度 
    if code==200 then--判断网站状态 
      --content=content:gsub( "" ", " ") or content  --替换获取到的内容，没什么必要 
      doc = Jsoup.parse(content)--使用jsoup解析
      --   classification = doc.select( "am-gallery am-avg-sm-3   am-avg-md-3 am-avg-lg-4 am-gallery-default am-no-layout ")--查找到所有class为text-gray的网页元素 
      classification = doc.getElementsByClass('am-gallery-item')--查找到所有class为text-gray的网页元素 
      --           print(classification) 
   
 classification = luajava.astable(classification)--将其转换成table表   


      for k,v in pairs(classification) do--循环打印输出 
        title=v.text() 
--是的这里支持:match正则处理
        websiteto=v.html():match('href= "(.-) "') 
        website=home..websiteto --连接
        cover=v.html():match([[nal= "(.-) "]]) 

--选择器
        --      picup=v.getElementsByClass( "media media-4x3 p-0 ") 
        --      picupi=v.getElementsByTag( "a ") 

--插入table表，这个table在源文件中是绑定着适配器的，所以这里完成就显示在列表之中了
        adphome.add{home1=title,home2=website,homecover1=loadbitmap(cover)} 
------------- 
      end 

    else 
      print('请求失败'..code) 
    end 
  end) 
end

```

这个函数中CSS选择器

```lua
doc.select("a[href]")
```

这个选择器有人应该不陌生（大概，像jquery选择器一样，可以获取文章元素

还有像js一样的：

```lua
--      picup=v.getElementsByClass( "media media-4x3 p-0 ") 
 --      picupi=v.getElementsByTag( "a ")
```

这里调用Document类的getElementsByClass() 方法，getElementById()方法和Element类的getElementsByTag()方法

不清楚的，去看上面的Jsoup API

### Jsoup的不足

只能处理静态页面，对于动态页面或者渲染后的页面不顶事儿，需要模拟的ajax请求

（我不会）

Jsoup是个人项目，后期的维护方面存在一定的不确定性，如果需要用应用在一些大型，长久性的项目中要慎重。

Jsoup在操作十分便捷，但在性能上，尽管jsoup本身够轻量，但它依然需要解析整个HTML，再进行进一步的搜索，它的底层还是通过正则做匹配，对简单的HTML是牛刀杀鸡，如果HTML复杂，Jsoup派的上用场。

## 参考/引用 文章/应用

[用Java优雅爬虫——jsoup - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/47868644)

[Jsoup解析Html中文文档 - 超超boy - 博客园 (cnblogs.com)](https://www.cnblogs.com/jycboy/p/jsoupdoc.html)

代码手册APP--AndroluaBox