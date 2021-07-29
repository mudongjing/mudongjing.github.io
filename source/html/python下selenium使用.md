> 通过使用selenium，可以实现模拟浏览器的效果，针对使用JS的网站可以说是不得已的做法。
### 1 环境基础
但使用前，首先需要安装浏览器并配置对应的驱动，例如chrome浏览器安装后，需要下载对应版本的chromedriver文件，本文介绍的是window环境的python操作，所以对应的驱动文件可直接复制到python对应的文件下。
由于chrome浏览器的驱动与浏览器本身的版本关系太过麻烦，因此作者使用火狐，只需要下载最新版的火狐浏览器，与最新的geckodriver驱动文件即可，同样驱动文件复制到python文件中。
接下来可以使用webdriver实现对浏览器的操作。由于不需要浏览器显示，因此下面的代码中加入了headless参数，使浏览器在后台运行。
```python
from selenium import webdriver
fire=webdriver.FirefoxOptions()
fire.add_argument('--headless')
brower=webdriver.Firefox(options=fire)
#此时浏览已经配置好，可以进行操作了
brower.get('http://www.baidu.com')#浏览器访问百度
```
### 2 网页元素抓取
webdriver中针对不同的网页元素提供了对应的抓取方法，如下图：

![DDx2Ke.png](https://mudongjing.github.io/gallery/spiders/python/webdriver.png)(摘自《python从入门到实践》)

### 3 禁止加载
出于信息爬取效率的考虑，一些网页中的图片等信息并非我们所需，且这类信息的加载相对而言较为耗时。因此，要求禁止浏览器加载这类信息，不同浏览器禁止加载的手段略有不同，针对火狐，下面的代码给出了禁止CSS，图片，JavaScript加载的方法。
```python
from selenium import webdriver
import sys 

def Driver(url):
    option=webdriver.FirefoxOptions()
    #option.add_argument('--headless')
    fp = webdriver.FirefoxProfile()
    fp.set_preference("permissions.default.stylesheet",2)#CSS禁止加载
    fp.set_preference("permissions.default.image",2)#禁止图片加载
    fp.set_preference("javascript.enabled",False)#JS禁止加载
	fp.set_preference('dom.ipc.plugins.enabled.libflashplayer.so', False)#禁flash
    brower=webdriver.Firefox(options=option,firefox_profile=fp)
    brower.get(url)

if __name__ == "__main__":
    url='https://www.douban.com/'
    Driver(url)
```