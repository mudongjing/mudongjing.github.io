# 1 安装
tesseract-ocr支持多平台，针对windows，可在官方指定的地址中下载[tesseract-ocr安装包下载](https://digi.bib.uni-mannheim.de/tesseract/)，安装时可不关心额外的语言包，直接连续下一步安装结束，软件默认支持英文识别，其它语言可在这个网址下载对应的[语言包](https://github.com/tesseract-ocr/tessdoc/blob/master/Data-Files.md#data-files-for-version-400-november-29-2016)，但这些网址通常速度不佳。

- 本文采用的是4.0版本，并添加公式识别，以及简繁中文语言包，[tesseact-ocr](https://089u.com/dir/26365709-41548670-254517)(提取码：899120)。~~ tessdata-master.zip是其所有的语言包~~
- 安装完成后，将语言包移到到对应安装目录的`tessdata`文件夹中
- 此外，在环境变量`path` 中添加安装目录，并创建一个新的系统变量`TESSDATA_PREFIX` ，值为安装目录下的`tessdata`对应的目录。
- 此时安装便结束了。查看版本与加载的语言包，打开`cmd`:
```bash
C:\Users> tesseract -v
tesseract 4.00.00alpha
 leptonica-1.74.1
  libgif 4.1.6(?) : libjpeg 8d (libjpeg-turbo 1.5.0) : libpng 1.6.20 : libtiff 4.0.6 : zlib 1.2.8 : libwebp 0.4.3 : libopenjp2 2.1.0
C:\Users> tesseract --list-langs
List of available languages (5):
chi_sim
chi_tra
eng
equ
osd
```
# 2 图片识别
- 打开 `cmd`，输入 ``tesseract 目标图片 存储结果的文件名 -l 语言包名``
	- 若是识别英文，免去 ``-l 语言包名``

有时会出现 `Warning:Invalid resolution 0 dpi. Using 70 instead`，并无大碍，只是目标图片本身没给出明确的分辨率。

![](https://mudongjing.github.io/gallery/python/ocr/python-icon.png)
为方便使用，我们结合python使用该工具，首先安装对应模块：

```bash
python3 -m pip install pytesseract
```
我们对上述的python图片做识别，运行代码：
```python

import pytesseract

py=Image.open(r'./python.png')					#这里是对应图片的目录，./为当前文件夹
print(pytesseract.image_to_string(py,lang='eng')) #lang指定对应的语言包
												   #« python"
```
结果基本是可以的。但也很明显会对额外的内容做错误的识别，这一点在图片较杂乱时尤为明显。

因此，在很多情况下需要对图片做处理，通过二值化等方式突出主要内容，而这一工作主要交给了Pillow模块处理，在`python2`中这一工作主要由`PIL`处理，但在`python3`中则由其分支`Pillow`代替，安装：
```bash
python3 -m pip install pillow
```
网上对于`Pillow`的操作有不少介绍，也可以通过官方的文档了解[pillow](https://pillow.readthedocs.io/en/latest/reference/index.html)各种方法和类的细节。

这里，简单地给出二值化的代码：
```python
from PIL import Image    		   #由于是继承PIL，因此pillow仍用PIL表示
im=Image.open(r'./python.png')
grey=im.convert('L')
grey.save(r'./grey.png')			#保存图片
```
![](https://mudongjing.github.io/gallery/python/ocr/grey-python.png)
```python
th=Image.open(r'./grey.png')
n=170							#设定的一个阈值，需要在0~256间
table=[0]*n+[1]*(256-n)
out=th.point(table,'1')		  #二值化，阈值之下黑，之上白
out.show()					   #显示结果图像
```
![](https://mudongjing.github.io/gallery/python/ocr/wb-python.png)

- 由于计算机中，颜色采用RGB模式，即通过红绿蓝三原色的混合得出颜色，每个三原色对应的值范围即0~256
- 当R=G=B=n，即三原色比例
- 相同，即得到黑白，n越小，颜色越黑，n=256即得到白色，因此阈值的范围有这样的限制
- 这里方法给出的模式'1'，即二值化，通过指定阈值，将图片中黑白色极端化，