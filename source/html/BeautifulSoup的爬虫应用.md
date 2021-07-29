>BeautifulSoup也是一种解析工具，用于从HTML和XML文件中提取数据

# 1 安装
使用环境**python3**，python2已停止维护，尽量不用它。
```bash
python -m pip install bs4
```
如果**python**的安装中，对应的*python.exe*文件名为*python3.exe*，上述的命令中的**python**则用**python3**代替。
(bs4之前还有bs3，只不过停止开发了)

# 2 使用
我们使用该工具从网页的代码中爬取相关数据，下面是陶哲轩的博客[What's new](https://terrytao.wordpress.com/ "What's new")的部分HTML代码。其中`<h2>` 标签对应的是文章标题，我们将提取网页中的`<h2>`标签中的文章标题。

```html
<div class="entry post-12099 post type-post status-publish format-standard hentry category-mathco category-update tag-arithmetic-regularity-lemma tag-ben-green tag-counting-lemma tag-daniel-altman tag-gowers-uniformity-norms">
		<div class="post-meta">
			<h2 class="post-title" id="post-12099">
				<a href="https://terrytao.wordpress.com/2020/11/26/a-correction-to-an-arithmetic-regularity-lemma-an-associated-counting-lemma-and-applications/" rel="bookmark">
				A correction to “An arithmetic regularity lemma, an associated counting lemma, and applications”
				</a>
			</h2></div>
</div>
```
针对上面的页面，我们只需要提取形如`<h2 class="post-title" id="**">`的标签即可，其中id是一个变量，对应的python代码如下：

``` python
import requests
from bs4 import BeautifulSoup
url='https://terrytao.wordpress.com'
header={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.67 Safari/537.36 Edg/87.0.664.47'}
n=requests.get(url,headers=header)
soup=BeautifulSoup(n.text,'html.parser')
							#将得到的页面转化为BeautifuSoup对象
							#soup.prettify()可优化代码结构，输出更优美的页面代码
title_1=soup.find('h2',class_='post-title').a.text
							#提取第一个符合的标题，a即提取其中的<a> <\a>的标签内容，text则提取其中的文本
print(title_1)
							#A correction to “An arithmetic regularity lemma, an associated counting lemma, and applications”
							#---------------------------------
							#如果得到的标题头尾有一些无用的信息，如博客名字或其它，可在text后加上.strip(),可在字符串头尾删除指定内容，具体可自行搜索
title_list=soup.find_all('h2',class_='post-title')
							#提取当前页面的所有标题
for i in title_list:
    print(i.a.text)
							#A correction to “An arithmetic regularity lemma, an associated counting lemma, and applications”
							#Climbing the cosmic distance ladder (book announcement)
							#The structure of translational tilings in Z^d
							#Foundational aspects of uncountable measure theory: Gelfand duality, Riesz representation, canonical models, and canonical disintegration
							#An uncountable Mackey-Zimmer theorem
							#Exploring the toolkit of Jean Bourgain
							#Course announcement: Math 246A, complex analysis
							#Vaughan Jones
							#Zarankiewicz’s problem for semilinear hypergraphs
							#Fractional free convolution powers
```

另外如果要查看网页代码，除了上述的通过requests返回的结果，或浏览器中按`F12`，还可以在地址栏中输入`view-source:url`，便可直接显示网页的代码，url为对应的网址。
## 2.1 文档树
HTML代码中的各个标签都相当于树结构中的节点，大标签下包含着多个小标签，而在这里把它们看作父子关系。
仍然截取[What's new](https://terrytao.wordpress.com/ "What's new")的部分HTML代码
```html
<head profile="http://gmpg.org/xfn/11">
	<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
	<title>
		What's new | Updates on my research and expository papers, discussion of open problems, and other maths-related topics.  By Terence Tao
	</title>
	<link rel="pingback" 
		  href="https://terrytao.wordpress.com/xmlrpc.php" />
	<meta name="google-site-verification" 
		  content="YnmTzeF3F_x_iP1LlMorwk_-PaRMR7FRkMea24K_cnY" />
	<link rel='dns-prefetch' href='//s0.wp.com' />
	<script id='wpcom-actionbar-placeholder-js-extra'>
		var actionbardata = {"**"}};
	</script>
```
此时`<head>`作为一个父节点，下面包含着`<title>`,`<link>`,`<script>`等子节点。获取对应节点的代码如下：


```python
import requests
from bs4 import BeautifulSoup
url='https://terrytao.wordpress.com'
header={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.67 Safari/537.36 Edg/87.0.664.47'}
n=requests.get(url,headers=header)
soup=BeautifulSoup(n.text,'html.parser')
print(soup.head.find_all('link'))				 #获取所有目标标签
lis=soup.head.contents							#获取head下所有子节点
												  #通过contents获得的列表，在标签之间存在'\n'，因此需要去除这些无用数据
soup_lis=[i for i in lis if i!='\n']
print(soup_lis[2])								 #head下第二个节点
print(soup.head.descendants)					   #获取head下的所有节点，无论节点往下几级
```
- 其它的还可用children，parent表示节点的子标签和父节点。
- 其中find_all可以与正则表达式结合，即find_all(re.compile('**'))，.compile用于编译内部的正则表达式
- 另外还可以利用select()方法
	- select('head div')：返回head下所有的div标签
	- select('head > div')：返回head下子节点级别中的div标签
	- 此外，可用select_one()返回结果中的第一个

## 2.2 lxml模块
- 安装
```dos
python -m pip install lxml
```
> lxml与BeautifulSoup功能相似，lxml额外提供了xpath选择器方法，且速度更快一些

使用的重点在于熟悉xpath的语法，或者在网页使用`F12`后，在控制台中选中要搜索的目标右键，`Copy`中有`Copy XPath`选项。

```python
import requests
from lxml import etree
#省略url,head
n=requests.get(url,headers=header)
html=etree.HTML(n.text)
title=html.xpath('//h2[@class="post-title"]/text()')
```