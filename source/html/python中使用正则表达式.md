# 1 基本语法
| 模式 | 作用                                        | 模式 | 作用                                 |
| ---- | ------------------------------------------- | ---- | ------------------------------------ |
| .    | 匹配任意字符，换行符\n除外                  | +    | 前一个字符可出现1次或多次            |
| *    | 前一个字符可出现0次或多次                   | ？   | 前一个字符可出现0次1多次             |
| ( )  | 用以存放正则表达式，如(.\*)匹配任意字符多次 | [  ] | 表示一组字符，如[a-zA-Z]匹配任意字母 |

> 上述的是非常常用的，下面一些比较凌乱，但没什么操作难度

| 模式 | 作用                                                     | 模式 | 作用             |
| ---- | -------------------------------------------------------- | ---- | ---------------- |
| ^    | 匹配字符串开头，也用于非操作，如[^0-9]匹配任何非数字字符 | \s   | 匹配空白字符     |
| $    | 匹配字符串尾部                                           | \S   | 匹配非空白字符   |
| \w   | 匹配字母和数字，等价[a-zA-Z0-9]                          | \W   | 匹配非字母或数字 |
| \d   | 匹配数字，等价[0-9]                                      | \D   | 匹配非数字       |

# 2 re模块
re模块是pyhton自带的，可用于正则表达式操作

- 主要包含三种操作，**match**，**search**，**findall**

## 2.1 match
语法为re.match(pattern,string,flags)，pattern为正则表达式，string为待匹配的字符串，flags用于控制匹配方式，如区分大小写、多行匹配。

 > 从string起始位置开始匹配，若不符则返回None,否则，返回在string上pattern对应的头尾位置

- 最简单的，在字符串'www.baidu.com' 中匹配字符串'baidu'
```python
import re
m=re.match('baidu','www.baidu.com')
print(m)							#结果 None
n=re.match('baidu','baidu.com')
print(n)							#结果：<_sre.SRE_Match object; span=(0, 5), match='baidu'>
print(n.start(),n.end())			#结果：0 5 ，对应上面的span
print(n.span())					 #结果：（0，5）
```
- pattern不是纯粹的字符串，包含前面介绍的.\*？等字符
```python
import re
string='Maybe PHP is the best language in the world'
n=re.match('(.*) is (.*?) language (.*)',string)
print('匹配整句话',n.group(0))
														#匹配整句话 Maybe PHP is the best language
print('匹配的第一个结果',n.group(1))
														#匹配的第一个结果 Maybe PHP
print('匹配的第二个结果',n.group(2))
														#匹配的第二个结果 the best
print('匹配的第三个结果',n.group(3))
														#(.*)会匹配尽可能多的字符
														#匹配的第三个结果 in the world
m=re.match('(.*) is (.*?) language (.*?)',string)
														#(.*?)则匹配尽可能少的字符
print('空的',m.group(3))
														#空的 
print('匹配结果的列表',n.group())
														#匹配的第一个结果 Maybe PHP
```
- 需要匹配字符'\\'，需要用**r** pattern。由于\具有转义作用，**r**表示raw string，即原始字符串，使pattern中\只是字符串中普通的转义符，**r** \ \即转义为\，否则\ \ \ \ =\ \转义为\
```python
import re
n=re.match(r'\\','\\\\')
print(n)
						#<re.Match object; span=(0, 1), match='\\'>
```
 **否则**
```python
import re 
n=re.match('\\\\','\\\\\\')
print(n)
						#<re.Match object; span=(0, 1), match='\\'>
```
## 2.2 search
> 扫描整个字符串，并返回匹配的第一个
> 语法与match相似
```python
import re
n=re.search('baidu','www.baidu.com')
print(n)
							#<re.Match object; span=(4, 9), match='baidu'>
```
很简单，没必要赘述。

## 2.3 findall
> 以列表的形式返回字符串中所有匹配的结果

```python
import re
string='1 woman and 2 men live in Baker Street 221B'
n=re.findall('[0-9]+',string)
print(n)
							#['1', '2', '221']
```

------------


> ~~提高正则表达式能力，可以google一下正则表达式游戏，没事闯闯关~~