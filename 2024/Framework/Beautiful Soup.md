Beautiful Soup

[[toc]]

---
---

https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#

上面的网址，左边菜单栏不会跟着下去，所以不太好。
https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#


现在已经4.12了，中文只有4.4的，凑合用。

---
# 对象的种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种: Tag , NavigableString , BeautifulSoup , Comment.

## tag
```
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
```

tag中最重要的属性: name和attributes

```
tag.name
# u'b'
```

tag的name可以修改
```
tag.name = "blockquote"
tag
# <blockquote class="boldest">Extremely bold</blockquote>
```

一个tag可能有很多个属性. `<b class="boldest">` 有一个 “class” 的属性,值为 “boldest” . 
tag的属性的操作方法与字典相同:
```
tag['class']
# u'boldest'
```

也可以直接”点”取属性, 比如: .attrs :

```
tag.attrs
# {u'class': u'boldest'}
```

tag的属性可以被添加,删除或修改. 再说一次, tag的属性操作方法与字典一样

多值属性
最常见的多值的属性是 class (一个tag可以有多个CSS的class). 还有一些属性 rel , rev , accept-charset , headers , accesskey . 在Beautiful Soup中多值属性的返回类型是list:

``` Python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]

css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]
```

如果某个属性看起来好像有多个值,但在任何版本的HTML定义中都没有被定义为多值属性,那么Beautiful Soup会将这个属性作为字符串返回

```
id_soup = BeautifulSoup('<p id="my id"></p>')
id_soup.p['id']
# 'my id'
```

将tag转换成字符串时,多值属性会合并为一个值

```
rel_soup = BeautifulSoup('<p>Back to the <a rel="index">homepage</a></p>')
rel_soup.a['rel']
# ['index']
rel_soup.a['rel'] = ['index', 'contents']
print(rel_soup.p)
# <p>Back to the <a rel="index contents">homepage</a></p>
```

如果转换的文档是XML格式,那么tag中不包含多值属性

## 可以遍历的字符串 NavigableString

字符串常被包含在tag内.Beautiful Soup用 NavigableString 类来包装tag中的字符串:
```
tag.string
# u'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>
```

通过 unicode() 方法可以直接将 NavigableString 对象转换成Unicode字符串

```
unicode_string = unicode(tag.string)
unicode_string
# u'Extremely bold'
type(unicode_string)
# <type 'unicode'>
```

tag中包含的字符串不能编辑,但是可以被替换成其它的字符串,用 replace_with() 方法:
```
tag.string.replace_with("No longer bold")
tag
# <blockquote>No longer bold</blockquote>
```

如果想在Beautiful Soup之外使用 NavigableString 对象,需要调用 unicode() 方法,将该对象转换成普通的Unicode字符串,否则就算Beautiful Soup已方法已经执行结束,该对象的输出也会带有对象的引用地址.这样会浪费内存.

## BeautifulSoup

BeautifulSoup 对象表示的是一个文档的全部内容.大部分时候,可以把它当作 Tag 对象,它支持 遍历文档树 和 搜索文档树 中描述的大部分的方法.

由于不是HTML 或 XML 的tag，所以没有name和attribute属性，但是 name 非常方便，所以 BeautifulSoup 对象包含了一个值为 `[document]` 的特殊属性 .name
```
soup.name
# u'[document]'
```

## 注释及特殊字符串

Comment 对象是一个特殊类型的 NavigableString 对象:
```
markup = "<b><!--Hey, buddy. Want to buy a used parser?--></b>"
soup = BeautifulSoup(markup)
comment = soup.b.string
type(comment)
# <class 'bs4.element.Comment'>
```


# 遍历文档树

## tag's name
操作文档树最简单的方法就是告诉它你想获取的tag的name.如果想获取 `<head>` 标签,只要用 soup.head :

```
soup.head
# <head><title>The Dormouse's story</title></head>

soup.title
# <title>The Dormouse's story</title>
```

下面的代码可以获取`<body>`标签中的第一个`<b>`标签:
```
soup.body.b
# <b>The Dormouse's story</b>
```

如果想要得到所有的`<a>`标签,或是通过名字得到比一个tag更多的内容的时候,就需要用到 Searching the tree 中描述的方法,比如: find_all()
```
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```


### .contents 和 .children

tag的 .contents 属性可以将tag的子节点以列表的方式输出:

``` Python
head_tag = soup.head
head_tag
# <head><title>The Dormouse's story</title></head>

head_tag.contents
[<title>The Dormouse's story</title>]

title_tag = head_tag.contents[0]
title_tag
# <title>The Dormouse's story</title>
title_tag.contents
# [u'The Dormouse's story']
```


BeautifulSoup 对象本身一定会包含子节点,也就是说`<html>`标签也是 BeautifulSoup 对象的子节点:
```
len(soup.contents)
# 1
soup.contents[0].name
# u'html'
```

字符串没有 .contents 属性,因为字符串没有子节点:

通过tag的 .children 生成器,可以对tag的子节点进行循环:
```
for child in title_tag.children:
    print(child)
    # The Dormouse's story
```


### .descendants
.contents 和 .children 属性仅包含tag的直接子节点

.descendants 属性可以对所有tag的==子孙==节点进行递归循环

```
for child in head_tag.descendants:
    print(child)
    # <title>The Dormouse's story</title>
    # The Dormouse's story
```


### .string
如果tag只有一个 NavigableString 类型子节点,那么这个tag可以使用 .string 得到子节点:

```
title_tag.string
# u'The Dormouse's story'
```

如果一个tag仅有一个子节点,那么这个tag也可以使用 .string 方法,输出结果与当前唯一子节点的 .string 结果相同:
```
head_tag.contents
# [<title>The Dormouse's story</title>]

head_tag.string
# u'The Dormouse's story'
```


如果tag包含了==多个==子节点,tag就无法确定 .string 方法应该调用哪个子节点的内容, .string 的输出结果是 None
```
print(soup.html.string)
# None
```


### .strings 和 stripped_strings

```
for string in soup.strings:
    print(repr(string))

# u"The Dormouse's story"
# u'\n\n'
```

输出的字符串中可能包含了很多空格或空行,使用 .stripped_strings 可以去除多余空白内容:
```
for string in soup.stripped_strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u"The Dormouse's story"
```


### .parent

```
title_tag = soup.title
title_tag
# <title>The Dormouse's story</title>
title_tag.parent
# <head><title>The Dormouse's story</title></head>
```

文档的顶层节点比如`<html>`的父节点是 BeautifulSoup 对象:

BeautifulSoup 对象的 .parent 是None:

### .parents

通过元素的 .parents 属性可以递归得到元素的所有父辈节点,下面的例子使用了 .parents 方法遍历了`<a>`标签到根节点的所有节点.


### .next_sibling 和 .previous_sibling 兄弟结点

```
sibling_soup.b.next_sibling
# <c>text2</c>

sibling_soup.c.previous_sibling
# <b>text1</b>
```

`<b>`标签有 .next_sibling 属性,但是没有 .previous_sibling 属性,因为`<b>`标签在同级节点中是第一个.同理,`<c>`标签有 .previous_sibling 属性,却没有 .next_sibling 属性:
```
print(sibling_soup.b.previous_sibling)
# None
print(sibling_soup.c.next_sibling)
# None
```


例子中的字符串“text1”和“text2”不是兄弟节点,因为它们的==父节点不同==:

实际文档中的tag的 .next_sibling 和 .previous_sibling 属性==通常是字符串或空白==. 看看“爱丽丝”文档:

### .next_siblings 和 .previous_siblings


### .next_element 和 .previous_element

.next_element 属性指向解析过程中下一个被解析的对象(字符串或tag),结果可能与 .next_sibling 相同,但通常是不一样的.

### .next_elements 和 .previous_elements



# 搜索文档树

Beautiful Soup定义了很多搜索方法,这里着重介绍2个: find() 和 find_all() .其它方法的参数和用法类似,请读者举一反三.

使用 find_all() 类似的方法可以查找到想要查找的文档内容

## 过滤器

介绍 find_all() 方法前,先介绍一下过滤器的类型,这些过滤器贯穿整个搜索的API.过滤器可以被用在tag的name中,节点的属性中,字符串中或他们的混合中.

### 字符串

最简单的过滤器是字符串.在搜索方法中传入一个字符串参数,Beautiful Soup会查找与字符串完整匹配的内容,下面的例子用于==查找文档中所有的`<b>`标签==:

```
soup.find_all('b')
# [<b>The Dormouse's story</b>]
```
如果传入字节码参数,Beautiful Soup会当作UTF-8编码,可以传入一段Unicode 编码来避免Beautiful Soup解析编码出错

### ==RE==
Beautiful Soup会通过正则表达式的 match() 来匹配内容

下面例子中找出==所有以b开头的标签==,这表示`<body>`和`<b>`标签都应该被找到:
```
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b
```

下面代码找出所有名字中包含”t”的标签:

```
for tag in soup.find_all(re.compile("t")):
    print(tag.name)
# html
# title
```

### 列表
如果传入列表参数,Beautiful Soup会将与列表中任一元素匹配的内容返回.下面代码找到文档中所有`<a>`标签和`<b>`标签:
```
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```


### True

True 可以匹配任何值,下面代码查找到所有的tag,但是不会返回字符串节点

```
for tag in soup.find_all(True):
    print(tag.name)
```

### 方法
如果没有合适过滤器,那么还可以定义一个方法,方法只接受一个元素参数,如果这个方法返回 True 表示当前元素匹配并且被找到,如果不是则反回 False

``` Python
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')

soup.find_all(has_class_but_no_id)
# [<p class="title"><b>The Dormouse's story</b></p>,
#  <p class="story">Once upon a time there were...</p>,
#  <p class="story">...</p>]
```


通过一个方法来过滤一类标签属性的时候, 这个方法的参数是要被过滤的属性的值, 而不是这个标签. 下面的例子是找出 href 属性不符合指定正则的 a 标签.

```
def not_lacie(href):
        return href and not re.compile("lacie").search(href)
soup.find_all(href=not_lacie)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```


## find_all()

==find_all( name , attrs , recursive , string , **kwargs )==

find_all() 方法搜索当前tag的所有tag子节点,并判断是否符合过滤器的条件.这里有几个例子:

```
soup.find_all("title")
# [<title>The Dormouse's story</title>]

soup.find_all("p", "title")
# [<p class="title"><b>The Dormouse's story</b></p>]

soup.find_all("a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find_all(id="link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

import re
soup.find(string=re.compile("sisters"))
# u'Once upon a time there were three little sisters; and their names were\n'
```


### name 参数

name 参数可以查找所有名字为 name 的==tag==,字符串对象会被自动忽略掉
搜索 name 参数的值可以使任一类型的 过滤器 ,字符窜,正则表达式,列表,方法或是 True 
```
soup.find_all("title")
# [<title>The Dormouse's story</title>]
```

### keyword 参数

如果一个指定名字的参数不是搜索内置的参数名,搜索时会把该参数当作指定名字tag的属性来搜索,如果包含一个名字为 id 的参数,Beautiful Soup会搜索每个tag的”id”属性.

```
soup.find_all(id='link2')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

如果传入 href 参数,Beautiful Soup会搜索每个tag的”href”属性:
```
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

搜索指定名字的属性时可以使用的参数值包括 字符串 , 正则表达式 , 列表, True .

下面的例子在文档树中查找所有包含 id 属性的tag,无论 id 的值是什么:

```
soup.find_all(id=True)
```

使用多个指定名字的参数可以同时过滤tag的多个属性:
```
soup.find_all(href=re.compile("elsie"), id='link1')
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]
```


有些tag属性在搜索不能使用,比如HTML5中的 data-* 属性:

```
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')
data_soup.find_all(data-foo="value")
# SyntaxError: keyword can't be an expression
```

但是可以通过 find_all() 方法的 attrs 参数定义一个字典参数来搜索包含特殊属性的tag:

```
data_soup.find_all(attrs={"data-foo": "value"})
# [<div data-foo="value">foo!</div>]
```



### 按CSS搜索

CSS类名的关键字 class 在Python中是保留字
4.1.1版本开始,可以通过 class_ 参数搜索有指定CSS类名的tag:
```
soup.find_all("a", class_="sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

class_ 参数同样接受不同类型的 过滤器 ,字符串,正则表达式,方法或 True :

ag的 class 属性是 多值属性 .按照CSS类名搜索tag时,可以分别搜索tag中的每个CSS类名:
```
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.find_all("p", class_="strikeout")
# [<p class="body strikeout"></p>]

css_soup.find_all("p", class_="body")
# [<p class="body strikeout"></p>]
```

搜索 class 属性时也可以通过CSS值完全匹配:

```
css_soup.find_all("p", class_="body strikeout")
# [<p class="body strikeout"></p>]
```

完全匹配 class 的值时,如果CSS类名的顺序与实际不符,将搜索不到结果


### string 参数

通过 string 参数可以搜搜文档中的字符串内容.与 name 参数的可选值一样, string 参数接受 字符串 , 正则表达式 , 列表, True

```
soup.find_all(string="Elsie")
# [u'Elsie']

soup.find_all(string=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']

soup.find_all(string=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]

def is_the_only_string_within_a_tag(s):
    ""Return True if this string is the only child of its parent tag.""
    return (s == s.parent.string)

soup.find_all(string=is_the_only_string_within_a_tag)
# [u"The Dormouse's story", u"The Dormouse's story", u'Elsie', u'Lacie', u'Tillie', u'...']
```

虽然 string 参数用于搜索字符串,还可以与其它参数混合使用来过滤tag.Beautiful Soup会找到 .string 方法与 string 参数值相符的tag.下面代码用来搜索内容里面包含“Elsie”的`<a>`标签:

```
soup.find_all("a", string="Elsie")
# [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]
```


### limit 参数
文档树很大那么搜索会很慢.如果我们不需要全部结果,使用 limit 参数限制返回结果的数量
效果与SQL中的limit关键字类似,当搜索到的结果数量达到 limit 的限制时,就停止搜索返回结果.

### recursive 参数
调用tag的 find_all() 方法时,Beautiful Soup会检索当前tag的所有==子孙节点==,如果只想搜索tag的直接子节点,可以使用参数 `recursive=False`

Beautiful Soup 提供了多种DOM树搜索方法. 这些方法都使用了类似的参数定义. 比如这些方法: find_all(): name, attrs, text, limit. 但是只有 find_all() 和 find() 支持 recursive 参数.

## 像调用 find_all() 一样调用tag
find_all() 几乎是Beautiful Soup中最常用的搜索方法,所以我们定义了它的简写方法. **BeautifulSoup 对象和 tag 对象可以被当作一个方法来使用**,这个方法的执行结果与调用这个对象的 find_all() 方法相同,下面两行代码是==等价的==:

```
soup.find_all("a")
soup("a")
```

```
soup.title.find_all(string=True)
soup.title(string=True)
```



## find()

find( name , attrs , recursive , string , **kwargs )

使用 find_all 方法并设置 limit=1 参数不如直接使用 find() 方法.下面两行代码是**等价**的:

```
soup.find_all('title', limit=1)
# [<title>The Dormouse's story</title>]

soup.find('title')
# <title>The Dormouse's story</title>
```

唯一的区别是 find_all() 方法的返回结果是值包含一个元素的列表,而 find() 方法直接返回结果.

find_all() 方法没有找到目标是返回空列表, find() 方法找不到目标时,**返回 None** .


soup.head.title 是 tag的名字 方法的简写.这个简写的原理就是多次调用当前tag的 find() 方法:

```
soup.head.title
# <title>The Dormouse's story</title>

soup.find("head").find("title")
# <title>The Dormouse's story</title>
```



## find_parents() 和 find_parent()

find_parents( name , attrs , recursive , string , **kwargs )

find_parent( name , attrs , recursive , string , **kwargs )



## find_next_siblings() 和 find_next_sibling()

find_next_siblings( name , attrs , recursive , string , **kwargs )

find_next_sibling( name , attrs , recursive , string , **kwargs )


## find_previous_siblings() 和 find_previous_sibling()

find_previous_siblings( name , attrs , recursive , string , **kwargs )

find_previous_sibling( name , attrs , recursive , string , **kwargs )


## find_all_next() 和 find_next()

find_all_next( name , attrs , recursive , string , **kwargs )

find_next( name , attrs , recursive , string , **kwargs )


## find_all_previous() 和 find_previous()

find_all_previous( name , attrs , recursive , string , **kwargs )

find_previous( name , attrs , recursive , string , **kwargs )


## CSS选择器

在 Tag 或 BeautifulSoup 对象的 ==.select()== 方法中传入字符串参数, 即可使用CSS选择器的语法找到tag:

```
soup.select("title")
# [<title>The Dormouse's story</title>]

soup.select("p nth-of-type(3)")
# [<p class="story">...</p>]
```

### 通过tag标签逐层查找

```
soup.select("body a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("html head title")
# [<title>The Dormouse's story</title>]
```

### 找到某个tag标签下的直接子标签

```
soup.select("head > title")
# [<title>The Dormouse's story</title>]

soup.select("p > a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("p > a:nth-of-type(2)")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

soup.select("p > #link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("body > a")
# []
```


### 找到兄弟节点标签

```
soup.select("#link1 ~ .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

soup.select("#link1 + .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```


### 通过CSS的类名查找

```
soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

### 通过tag的id查找
```
soup.select("#link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("a#link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

### 同时用多种CSS选择器查询元素
```
soup.select("#link1,#link2")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

### 通过是否存在某个属性来查找
```
soup.select('a[href]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```


### 通过属性的值来查找

```
soup.select('a[href="http://example.com/elsie"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select('a[href^="http://example.com/"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="tillie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```


### 通过语言设置来查找:
```
multilingual_markup = """
 <p lang="en">Hello</p>
 <p lang="en-us">Howdy, y'all</p>
 <p lang="en-gb">Pip-pip, old fruit</p>
 <p lang="fr">Bonjour mes amis</p>
"""
multilingual_soup = BeautifulSoup(multilingual_markup)
multilingual_soup.select('p[lang|=en]')
# [<p lang="en">Hello</p>,
#  <p lang="en-us">Howdy, y'all</p>,
#  <p lang="en-gb">Pip-pip, old fruit</p>]
```

### 返回查找到的元素的第一个(select_one)

```
soup.select_one(".sister")
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```

对于熟悉CSS选择器语法的人来说这是个非常方便的方法.Beautiful Soup也支持CSS选择器API, 如果你仅仅需要CSS选择器的功能,那么直接使用 lxml 也可以, 而且速度更快,支持更多的CSS选择器语法,但Beautiful Soup整合了CSS选择器的语法和自身方便使用API.


# 修改文档树

## 修改tag的名称和属性

```
tag.name = "blockquote"
tag['class'] = 'verybold'
tag['id'] = 1
```

## 修改 .string
```
tag = soup.a
tag.string = "New link text."
```

## append()
Tag.append() 方法想tag中添加内容,就好像Python的列表的 .append() 方法

## NavigableString() 和 .new_tag()
如果想添加一段文本内容到文档中也没问题,可以调用Python的 append() 方法 或调用 NavigableString 的构造方法:
创建一个tag最好的方法是调用工厂方法 BeautifulSoup.new_tag() 

## insert()
Tag.insert() 方法与 Tag.append() 方法类似,区别是不会把新元素添加到父节点 .contents 属性的最后,而是把元素插入到指定的位置.

## insert_before() 和 insert_after()
insert_before() 方法在当前tag或文本节点前插入内容:
insert_after() 方法在当前tag或文本节点后插入内容:

## clear()
Tag.clear() 方法移除当前tag的内容

## extract()
PageElement.extract() 方法将当前tag移除文档树,并作为方法结果返回:
这个方法实际上产生了2个文档树: 一个是用来解析原始文档的 BeautifulSoup 对象,另一个是被移除并且返回的tag.被移除并返回的tag可以继续调用 extract 方法:

## decompose()
Tag.decompose() 方法将当前节点移除文档树并完全销毁:


## replace_with()
PageElement.replace_with() 方法移除文档树中的某段内容,并用新tag或文本节点替代它:

## wrap()
PageElement.wrap() 方法可以对指定的tag元素进行包装,并返回包装后的结果

## unwrap()
Tag.unwrap() 方法与 wrap() 方法相反.将移除tag内的所有tag标签,该方法常被用来进行标记的解包:



# 输出

## 格式化输出

==prettify()== 方法将Beautiful Soup的文档树格式化后以Unicode编码输出,每个XML/HTML标签都独占一行

BeautifulSoup 对象和它的tag节点都可以调用 prettify() 方法:


## 压缩输出
只想得到结果字符串,不重视格式

对一个 BeautifulSoup 对象或 Tag 对象使用Python的 ==unicode() 或 str()== 方法:

str() 方法返回UTF-8编码的字符串,可以指定 编码 的设置.
还可以调用 encode() 方法获得字节码或调用 decode() 方法获得Unicode.


## 输出格式

Beautiful Soup输出是会将HTML中的特殊字符转换成Unicode,比如“&lquot;”:


## get_text()

如果只想得到tag中包含的文本内容,那么可以调用 get_text() 方法,这个方法获取到tag中包含的所有文版内容包括子孙tag中的内容,并将结果作为Unicode字符串返回:
```
markup = '<a href="http://example.com/">\nI linked to <i>example.com</i>\n</a>'
soup = BeautifulSoup(markup)

soup.get_text()
u'\nI linked to example.com\n'
soup.i.get_text()
u'example.com'
```
可以通过参数指定tag的文本内容的分隔符:
```
# soup.get_text("|")
u'\nI linked to |example.com|\n'
```
还可以去除获得文本内容的前后空白:
```
# soup.get_text("|", strip=True)
u'I linked to|example.com'
```
或者使用 .stripped_strings 生成器,获得文本列表后手动处理列表:
```
[text for text in soup.stripped_strings]
# [u'I linked to', u'example.com']
```


# 指定文档解析器

要解析的文档是什么类型: 目前支持, “html”, “xml”, 和 “html5”
指定使用哪种解析器: 目前支持, “lxml”, “html5lib”, 和 “html.parser”


## 解析器之间的区别
。。

# 编码

任何HTML或XML文档都有自己的编码方式,比如ASCII 或 UTF-8,但是使用Beautiful Soup解析后,文档都被转换成了Unicode:

通过传入 from_encoding 参数来指定编码方式:

```
soup = BeautifulSoup(markup, from_encoding="iso-8859-8")
soup.h1
<h1>םולש</h1>
soup.original_encoding
'iso8859-8'
```

如果仅知道文档采用了Unicode编码, 但不知道具体编码. 可以先自己猜测, 猜测错误(依旧是乱码)时, 可以把错误编码作为 exclude_encodings 参数
在没有指定编码的情况下, BS会自己猜测编码, 把不正确的编码排除掉, BS就更容易猜到正确编码.

```
soup = BeautifulSoup(markup, exclude_encodings=["ISO-8859-7"])
soup.h1
<h1>םולש</h1>
soup.original_encoding
'WINDOWS-1255'
```


## 输出编码
输出编码均为UTF-8编码



## 智能引号

使用Unicode时,Beautiful Soup还会智能的把引号 转换成HTML或XML中的特殊字符

也可以把引号转换为ASCII码:

默认情况下,Beautiful Soup把引号转换成Unicode:



# 比较对象是否相同

两个 NavigableString 或 Tag 对象具有相同的HTML或XML结构时, Beautiful Soup就判断这两个对象相同.

如果想判断两个对象是否严格的指向同一个对象可以通过 ==is== 来判断


# 复制Beautiful Soup对象

copy.copy() 方法可以复制任意 Tag 或 NavigableString 对象

```
import copy
p_copy = copy.copy(soup.p)
print p_copy
# <p>I want <b>pizza</b> and more <b>pizza</b>!</p>
```

复制后的对象跟与对象是相等的, 但指向不同的内存地址


# 解析部分文档

如果仅仅因为想要查找文档中的`<a>`标签而将整片文档进行解析,实在是浪费内存和时间.最快的方法是从一开始就把`<a>`标签以外的东西都忽略掉. 
SoupStrainer 类可以定义文档的某段内容,这样搜索文档时就不必先解析整篇文档,只会解析在 SoupStrainer 中定义过的文档. 创建一个 SoupStrainer 对象并作为 parse_only 参数给 BeautifulSoup 的构造方法即可.


## SoupStrainer
SoupStrainer 类接受与典型搜索方法相同的参数：name , attrs , recursive , string , `**kwargs` 。下面举例说明三种 SoupStrainer 对象：

```
from bs4 import SoupStrainer

only_a_tags = SoupStrainer("a")

only_tags_with_id_link2 = SoupStrainer(id="link2")

def is_short_string(string):
    return len(string) < 10

only_short_strings = SoupStrainer(string=is_short_string)
```


还可以将 SoupStrainer 作为参数传入 搜索文档树 中提到的方法.这可能不是个常用用法,所以还是提一下:

```
soup = BeautifulSoup(html_doc)
soup.find_all(only_short_strings)
# [u'\n\n', u'\n\n', u'Elsie', u',\n', u'Lacie', u' and\n', u'Tillie',
#  u'\n\n', u'...', u'\n']
```




# 使用

tag.text 约等于 tag.string，并且可以无视内部的 span
























































































































































































