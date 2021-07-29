> 首先，如果电脑配置挺好的，也没有运行太多程序的，建议还是使用完善的IDE编写带包的Java代码，省心！

因为我电脑上跑着各种程序，打开IEDA这个庞然大物有时候比较难受，因此，喜欢用VS Code一点点写一些不用太多依赖的代码。当然了，如果需要各种Maven，spring等等，还是别在编辑器上太多时间。
#### 准备
- 因为不是拿VScode做主力，因此即使是编辑器也不希望安装太多插件，因此编译java只要个code runner就行了。
- 但是，code runner配置的编译命令只适合没有包的代码，即编译完了，class文件和java文件在同一个目录里
- 而code runner实际是利用系统自带的终端，cmd或powershell运行原本的编译命令，因此，只要自己终端能够完成的，将命令配置到code runner就可以完成。

> 这里我采用了非常笨的方法(~~我也想不出其它的了~~)，利用cmd中的命令提取java文件中我指定的包
#### 修改json文件
- 因为我将vscode中指定的终端设置为powershell，下面是配置的代码：
```bash
"java": "cd $dir && javac -d . $fileName && CMD /C 'for /f 'tokens=1,2 ' %i in ($fileName)  do if %i==package  java %j.$fileNameWithoutExt &exit'",
```
原本的命令就是cmd中的，因此用`CMD /C " **"`的方式在powershell中运行。
这里主要是利用批处理命令做的，只不过要在终端中直接运行，会略微有区别。大家可以详细地了解一下[cmd命令](https://www.jb51.net/help/cmd.htm)。

> 这里解释一下上面的命令，以便适当修改

- 首先前面的cd $dir之类的就是原本的一些操作
- $filename之类的含义可以看一下插件的描述，里面有介绍

![](https://mudongjing.github.io/gallery/vscode/command.jpg)

- 最后，便是CMD /C 中的命令，
	- for /f in %i in (filename) do :是从一个名为filename的文件里按行提取数据并打算做点什么的命令。
	do后面可以跟一些操作，%i也可以随便写%a之类的，当然不要乱写个%~I之类的，搞不好有什么其它作用
	- “tokens=1,2 ”,记得要在2后面留个空格，不然可能无法提取到包的值。实际在cmd里面是用的双引号，只不过，在插件的配置中，外面已经有双引号，所以上面的命令用的是单引号。
		- tokens=x,y:指定文件的每一行中第x个值和第y个值要传递上去，其中第x个值就是传递给%i，而第y个值，就是传递给%j，这里j就是按字母表自动排下来的，如果是tokens=x,y,z ，z就传递给%k。
		- 其次，一行中凭什么分出第几个值的，默认是按空格，你要不服，可以用delims指定，比如按逗号划分，取前两个值，就是 `for /f "tokens=1,2 delims=,"`
	- 根据上面的意思，%i就是获取每行第一个值，自然第一行一个值就是`package`,当然你可能写一些其它乱起八糟的，那样我可能无能为力。 do后面跟着的if命令就是我们熟悉的那个，不需要额外介绍了。
	- 最后，加个&exit，就是找到第一个package完成编译就退出，否则，它还会往后遍历各行查找package。

> 写带各种包的，还是用IDE吧。时间还是多折腾在代码上。