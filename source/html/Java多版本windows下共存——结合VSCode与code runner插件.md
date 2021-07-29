虽然jdk版本已经到15了，但由于spring等框架并没有支持地那么及时，jdk8仍旧是主力。
但自己在学习中还是希望尝试新版，本文主要探讨在windows下VS Code如何实现多版本jdk共存。

> 网络上已经有不少关于jdk共存的方法，例如在环境变量中添加新的JAVA_HOME，基于此还有给出批处理文件方便快速切换的方法。

### 下文主要是在我看来觉得挺方便的方法。

- 首先jdk8但正常方式配置完之后，不需要理会了。之后下载新版本的jdk，尽量是最新的，因为那样直接将bin目录放在`path`中就OK了。
- 由于Linu系统常自带**OpenJDK**，因此这里考虑的是下载**OpenJDK**的新版本，我这里是jdk15。
- windows下下载`zip`文件，直接解压即可，复制`bin`目录到`path`即可。
- 接下来，模仿`Python`2和3共存的方法，将`bin`目录中的主要程序文件，如`java`,`javac`的文件复制一份，在名字后加上对应的版本号（当然加上什么都是随意的），我这里就是改成`java15`。
效果如下：
![](https://mudongjing.github.io/gallery/java/jdk/window/java15.png)

#### 接下来就是处置VS Code的时间了。

>- 首先对本人而言，安装了插件 `Language Support for Java(TM) by Red Hat redhat.java`，之后总是提醒我JDK版本低。
>  - 现在有了OpenJDK的最新版，告诉VS Code：“ 我有了！”，就行了
>- 首先在setting的`json`文件中，有个`java.home`的值，在里面填上新JDK的安装目录即可


再接下来，就是安装插件`Code Runner`，这个在之前的[文章](http://p2pworker.xyz/?p=174)提及过。

- 利用`Code Runner`编译对应语言的代码，只要能够在终端运行对应的命令就行
- 现在我们已经实现这一非常基础的条件，按照`Java`编译命令的dos命令格式写进去即可，只不过，对应的`javac`,`java`可能要换成`javac15`,`java15`。

> 此时我们实现了修改Code Runner 配置即可更换`Java`的编译版本

但这样稍微有点费劲，有点像来回修改环境变量的方法。

- 在Code Runner的配置文件中，除了常用的各种语言，我们可以随意添加其它的，比如仿照上面的`java`的配置，添加一个`java8`:

~~请无视上下两个$~~
```json
$
"java8": "cd $dir && javac -d . $fileName && java $fileNameWithoutExt"
$
```
*原文件中可能没有`-d .`的内容，这里是因为方便编译带包的代码*
平常的编译，只会调用纯种的`java`，当我们需要换个版本时，按`F1`，输入`run by language`，即可选择编译环境，或按`ctrl`+`alt`+`j`，同样的功能。
![](https://mudongjing.github.io/gallery/java/jdk/window/java8.png)

>可以看见多了个java8的选项，对应jdk1.8。利用此方法，估计可以随意添加各种版本的编译环境。