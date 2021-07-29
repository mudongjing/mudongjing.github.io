> 日常的码农生活中，通常需要编辑各种语言的代码，Code Runner可以方便地切换语言编译。

# 1 安装
VS Code的插件安装非常方便，在侧边栏的小方块图标处点击搜索即可安装对应插件。
# 2 配置
首先，针对目标语言配置其环境，使得在终端输入命令可运行。

> 通常此时，插件就可以编译文件了。如若不行，在setting中修改其命令：

1. 多数情况下，我们需要更改一下相关语言的设置，为了方便，还是选择在`json`文件中修改。因此在`setting`中搜索`code runner`找到对应的位置，就可发现许多<u> **Edit in settings.json** </u>的选项。
	- 找到选项`Code-runner: Executor Map`，点击<u> **Edit in settings.json** </u>，就可以发现添加了各个语言的编译命令。
	2.上述工作后可以直接编辑。若不小心退出，可以通过下述方法进入文件。 按`F1`输入settings，点击Open Settings(JSON)，或者点击左下方的设置中的setting，再点击右上角的![](http://p2pworker.xyz/wp-content/uploads/2020/12/wp_editor_md_9d5859cd8dad79dff9fb9fa23b7e75f0.jpg)。
3. 进入JSON后，可以发现Code Runner的配置，部分内容如下($\$ \$ $之间的内容）：

~~代码块里面显示不了$符号，估计是插件问题，搞不懂，只能凑活~~
```json
$
"code-runner.executorMap": {
      "javascript": "node",
      "java": "cd $dir && javac $fileName && java $fileNameWithoutExt",\\
      "c": "cd $dir && gcc $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
      "cpp": "cd $dir && g++ $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
      "objective-c": "cd $dir && gcc -framework Cocoa $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
      "php": "php",
      "python": "python3 -u $fileName",
		//set PYTHONIOENCODING=utf8 指定编译时的编码，可处理中文乱码，需要时，可将其添加在python3前面，不要忘了加上 &&
      "perl": "perl",
      "perl6": "perl6",
      "ruby": "ruby",
      "go": "go run",
    }
$
```

插件已经给出了各种语言的编译运行命令，与终端命令基本一致，因此对于c/c++，java等语言环境配置完成即可在VS Code中编译代码。其它语言，或编译失败时，查看一下对应的命令是否正确，如添加$fileName指定编译当前文件。

其次，该插件默认结果输出在OUTPUT中，但我们希望显示在终端，可以在上述内容的上方添加下面的代码。

- 第一条指运行在终端，第二条要求编译时跳到文件所处的目录。
```json
	"code-runner.runInTerminal":true,
    "code-runner.fileDirectoryAsCwd": true,
```