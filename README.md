# 数据挖掘期末项目——selenium实战-知网
* 撰写人：黄泽豪
* 最新撰写时间： 2021.6.30
## 项目意义：
* 熟悉selenium操作，熟悉自动化挖掘操作
* 学习如何爬取知网内容
* 爬取知网文章，提高论文下载效率，方便学习工作
* 解决挖取CNKI文章过程中的各种问题，提升挖掘能力
## 项目目标
1. 爬取合适的有价值的query（关键词）下的论文
2. 爬取文章数量大于800篇
3. 爬取相关重要信息
4. 导出refworks文件或endnote文件（.txt）
5. 依次下载PDF原文文件，解决中间处理问题
6. 数据分析（关键词替换）——数据可视化
## 项目过程（步骤）
### 第一步：前期准备（调用需要的模块并使用selenium打开浏览器）

```
import pandas as pd
import numpy as np
from lxml.html import fromstring
import time
import base64
import json
import requests
import os
from random import random
import requests_html
from requests_html import HTMLSession
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities
from PIL import Image, ImageEnhance
```

```
wd = webdriver.Chrome()
wd.get("https://www.baidu.com")
opts = webdriver.ChromeOptions()
opts.add_argument('--no-sandbox')#解决DevToolsActivePort文件不存在的报错
opts.add_argument('window-size=1920x3000') #指定浏览器分辨率
opts.add_argument('--disable-gpu') #谷歌文档提到需要加上一这个属性来规避bug
opts.add_argument('--hide-scrollbars') #隐藏滚动条, 应对些特殊页面
#opts.add_argument('blink-settings=imagesEnabled=false') #不加载图片, 提升速度
#opts.add_argument('--headless') #浏览器不提供可视化页面. linux下如果系统不支持可视化不加这条会启动失败
# opts.binary_location = "C:\portable\PortableApps\IronPortable\App\Iron\chrome.exe"
# opts.binary_location = "C:\Program Files\Google\Chrome\Application\chromedriver.exe" #"H:\_coding_\Gitee\InternetNewMedia\CapstonePrj2016\chromedriver.exe"  


driver = webdriver.Chrome( chrome_options = opts) #desired_capabilities=caps,
```

### 第二步：登录知网
- 由于我是使用校园网登录，会直接登录机构，所以无需按个人账户进行登录，个人账户登录操作过程（代码）便不在此展示。

```
driver.get("https://kns.cnki.net/")
element = driver.find_element_by_xpath('//*[@id="highSearch"]')
element.click()
print (driver.window_handles)
```

```
driver.switch_to_window(driver.window_handles[1]) #定位当前窗口
```

### 第三步：进行检索（挖取一页的文章基本信息）

```
# 点击学术期刊
element = driver.find_element_by_xpath('/html/body/div[3]/div[1]/div/ul[1]/li[1]/a')
element.click()
# 点击专业检索
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/ul/li[4]')
element.click()
#勾选期刊类型
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/div/div[1]/div[1]/div[2]/div[1]/div[3]/div/label[2]/input')
element.click()#SCI来源期刊
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/div/div[1]/div[1]/div[2]/div[1]/div[3]/div/label[4]/input')
element.click()#北大核心
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/div/div[1]/div[1]/div[2]/div[1]/div[3]/div/label[5]/input')
element.click()#CSSCI
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/div/div[1]/div[1]/div[2]/div[1]/div[3]/div/label[6]/input')
element.click()#CSCD
```

```
# 编辑关键词：AI_新媒体_query = '(SU="新媒体" and SU="人工智能") OR (SU="AI" and SU="新媒体")'
AI_大数据_query = '( SU="大数据" AND KY="人工智能")'
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/div/div[1]/div[1]/div[2]/textarea')
element.clear()
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/div/div[1]/div[1]/div[2]/textarea').send_keys(AI_大数据_query)#输入关键词
#点击检索
element = driver.find_element_by_xpath('/html/body/div[2]/div/div[2]/div/div[1]/div[1]/div[2]/div[2]/input')
element.click()

```

```
#选择每页50篇
element = driver.find_element_by_xpath('/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/div[2]/div/div/div/i')
element.click()
element = driver.find_element_by_xpath('/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/div[2]/div/div/ul/li[3]/a')
element.click()
# 查看页面信息
element = driver.find_element_by_id('gridTable')
page_html = element.get_attribute('innerHTML')
page_html
pd.read_html(page_html)[0]
```
- 此时已经可以获取一页数据信息，信息包括论文篇名、作者、刊名、发表时间、被引和下载次数。

### 第四步：导出refworks和endnote文件
> 这里我们需要导出的refworks文件或endnote文件可用来进行数据可视化分析。
#### endnote：分每页50篇各自循环导出
```
# 将循环时需要点击的元素做成列表
piliang_list = []
piliang_list.append('//*[@id="gridTable"]/div[1]/div[2]/div[1]/a')
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/div[1]/label/input")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/i")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/ul/li[1]/a")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/ul/li[1]/ul/li[9]/a")
```

```
for page in range(0,26):# 因为最后一页没有下一页的按键，所以在第一行代码的页面循环中只需填写到最后一页的上一页页数即可，如此次爬取的全部为27页，则填写（0,26），循环到第26页的最后一个下一页按键即可。
    for i in piliang_list:#循环点击元素的列表，依次点击
        element = driver.find_element_by_xpath(i)
        element.click()
        time.sleep(2)
    driver.switch_to.window(driver.window_handles[2])#定位新窗口
    time.sleep(10)
    driver.find_element_by_xpath('//*[@id="litotxt"]/a').click()#点击导出
    time.sleep(10)
    driver.close()#关闭新窗口
    time.sleep(4)
    driver.switch_to.window(driver.window_handles[1])#定位回原窗口
    driver.find_element_by_xpath('//*[@id="PageNext"]').click()#点击下一页
    time.sleep(12)
```
#### refworks：分每页50篇各自循环导出

```
# 将循环时需要点击的元素做成列表
piliang_list = []
piliang_list.append('//*[@id="gridTable"]/div[1]/div[2]/div[1]/a')
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/div[1]/label/input")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/i")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/ul/li[1]/a")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/ul/li[1]/ul/li[8]/a")
```

```
for page in range(0,26):# 因为最后一页没有下一页的按键，所以在第一行代码的页面循环中只需填写到最后一页的上一页页数即可，如此次爬取的全部为27页，则填写（0,26），循环到第26页的最后一个下一页按键即可。
    for i in piliang_list:#循环点击元素的列表，依次点击
        element = driver.find_element_by_xpath(i)
        element.click()
        time.sleep(2)
    driver.switch_to.window(driver.window_handles[2])#定位新窗口
    time.sleep(10)
    driver.find_element_by_xpath('//*[@id="litotxt"]/a').click()#点击导出
    time.sleep(10)
    driver.close()#关闭新窗口
    time.sleep(4)
    driver.switch_to.window(driver.window_handles[1])#定位回原窗口
    driver.find_element_by_xpath('//*[@id="PageNext"]').click()#点击下一页
    time.sleep(12)
```
> 此时便可以将我们所需要的endnote文件导出到本地文档中，我们也可将这些文件用于进行下一步的数据可视化分析了。

### 第五步：PDF原文文件下载（依次下载每篇文章的PDF原文文件，将所有需要的文章下载下来）
- 在此之前我们应该梳理一下挖取时的逻辑顺序，经过几次尝试，先基本摸清可能出现的一些问题，再将这些问题分情况进行判断解决，使用较好的逻辑顺序去挖取我们所需要的文件及其数据。
#### 封装挖取时所需的各操作步骤（方便使用、简洁代码）
- 通过几次尝试之后，我已基本摸清挖取时可能出现的问题，也对解决这些问题进行了一定的操作，最后直接将其中反复出现的许多代码先进行各自不同的封装，这样在后面反复出现这些代码时便可直接调用，方便使用也简洁了整个代码，减少代码量。

 **封装判断验证码函数** 
- 依次下载PDF文件时可能会出现验证码页面，这就需要我们先进行一次验证码出现与否的判断了，代码如下：

```
# 封装判断验证码
def img_have():
    try:
        driver.find_element_by_xpath('//*[@id="vImg"]')#验证码图片
        return True#为真，即有验证码图片存在
    except:
        return False#反之为空则没有验证码，返回False
```

 **封装截图函数** 
- 我们可以直接通过xpath获取每张验证码的链接，再将验证码链接请求到API中，但这样可能会出现一种问题，即获取链接时，可能因为点击了验证码图片导致图片更新，而获取的链接为上一张图片的链接，这样即使识别出来的验证码也不能被正确输入使用，所以为了避免这种情况的发生，我选择将验证码通过截图方式截取下来，并保存在本地文件中，再将其请求到API上，这样便能获取当前正确的验证码，而且这样做还可以通过PIL模块对截取图片进行优化，例如降噪、提升饱和感等，更有利于API进行识别。代码如下：

```
#封装截图
def img():
    driver.find_element_by_xpath('//*[@id="vImg"]').click()
    time.sleep(2)
    screenImg = r"D:\课程资料\大二下课程总\web数据挖掘\期末验证码\screenImg.png"#本地保存位置
    driver.get_screenshot_as_file(screenImg)
    location = driver.find_element_by_xpath('//*[@id="vImg"]').location#验证码位置
    size = driver.find_element_by_xpath('//*[@id="vImg"]').size#截图大小
    left = location['x']
    top = location['y']
    right = location['x'] + size['width']
    bottom = location['y'] + size['height']
    img = Image.open(screenImg).crop((left, top, right, bottom))
    # 优化图片
    img = img.convert('RGBA')
    img = img.convert('L')
    img = ImageEnhance.Contrast(img)
    img = img.enhance(2.0)
    img.save(screenImg)
    img = Image.open(screenImg)
    return img
```

 **封装图鉴API** 
- 经由同学推荐，在这里我们使用[图鉴API](http://www.ttshitu.com/)进行验证码图片识别，一元可享有500次调用量，亲测有效，十分准确，一识一个准，且其代码已封装好，可直接使用。代码如下：

```
# 封装图鉴API
uname = '*****'#个人隐私，不便展示，可自己注册购买使用
pwd = '*******'#个人隐私，不便展示，可自己注册购买使用
typeid  = "3"
def base64_api(uname, pwd, img, typeid):
    with open('D:\课程资料\大二下课程总\web数据挖掘\期末验证码\screenImg.png', 'rb') as f:    
        base64_data = base64.b64encode(f.read())
        img = base64_data.decode()
    data = {"username": uname, "password": pwd, "typeid": typeid, "image": img}
    time.sleep(1)
    result = json.loads(requests.post("http://api.ttshitu.com/predict", json=data).text)
    if result['success']:
        return result["data"]["result"]
    else:
        return result["message"]
    return ""
```

 **封装验证码输入和正确情况判断** 
- 识别到的验证码需要进行输入，而且我们会经常需要定位窗口，所以我们将输入验证码和页面切换进行封装，以及当我们输入正确也下载了PDF原文后对这种正确情况进行一个封装，让其返回原窗口进行下一篇文章挖掘，代码如下：

```
#封装输入验证码
def shuru():
    yanzhengma = base64_api(uname, pwd, img, typeid)
    driver.find_element_by_xpath('//*[@id="vcode"]').clear()
    element = driver.find_element_by_xpath('//*[@id="vcode"]').send_keys(yanzhengma)
```

```
# 页面切换
def qiehuan():
    all_window=driver.window_handles
    new_page = len(all_window)
    driver.switch_to.window(driver.window_handles[new_page-1])
```

```
#封装正确情况判断
def true():
    qiehuan()
    driver.close()
    driver.switch_to.window(driver.window_handles[2])
    time.sleep(1)
    qiehuan()
    driver.close()
    driver.switch_to.window(driver.window_handles[1])
    time.sleep(1)
```

 **封装检查下载文件数** 
- 当我们输入验证码后，若是出现输入错误，验证码窗口会进行提示，我们可以依据这个来判断是否输入正确，但这其中会出现一种情况，即当你输入3个字符或中文字符时错误提示才会出现，若填入的是4个数字或英文的字符，会直接进行刷新，出现新的验证码，没有错误提示，这样就不利于我们进行判断，所以，我们选择检查下载文件数量进行验证码输入正确的判断标准，只有验证码输入正确且下载成功了，文件数量才会增加，若是文件数量没有增加，则代表验证码输入错误，并没有进行下载。代码如下：

```
#封装检查下载文件数
def wenjian1():
    path = r'D:\课程资料\大二下课程总\web数据挖掘\期末验证码\PDF文件'#（需在selenium浏览器更改下载文件存储的地址）
    file = int(len([lists for lists in os.listdir(path) if os.path.isfile(os.path.join(path, lists))]))#通过len和int将验证码输入前的文件夹里的文件数量进行检测，获取数量为未输入验证码的文件量，需调用os模块
    return file
```

```
def wenjian2():
    path = r'D:\课程资料\大二下课程总\web数据挖掘\期末验证码\PDF文件'#（需在selenium浏览器更改下载文件存储的地址）
    file2 = int(len([lists for lists in os.listdir(path) if os.path.isfile(os.path.join(path, lists))]))#通过len和int将验证码输入后的文件夹里的文件数量进行检测，获取数量为输入验证码后文件夹里的文件量，需调用os模块
    return file2
```

```
#判断有无下载
def wenjian3():
    if file2 > file:#如果输入验证码后的文件夹里文件数量增加，也就是数量大于输入前文件数量，则代表下载成功，验证码输入正确，反之则失败，输入错误，没有下载，文件数量没有大于前一次。
        return True
    else:
        return False
```

#### 自动循环PDF原文文件下载
- 当我们将所有用到的功能封装好后，我们便可以开始进行PDF原文文件的自动循环下载了，代码如下：

```
for page in range(1,4): #先进行三页循环，可自行选择爬取页数
    for link in range(1,50):#可选择从一页中的第几篇文章开始挖取到第几篇文章结束挖取
        pdf_xpath = '//*[@id="gridTable"]/table/tbody/tr[{}]/td[2]/a'.format(link)
        driver.find_element_by_xpath(pdf_xpath).click()#点击文章进入文章详情页
        time.sleep(1)
        driver.switch_to.window(driver.window_handles[2])#定位新窗口
        driver.find_element_by_xpath('//*[@id="pdfDown"]').click()#点击下载
        time.sleep(3)
        qiehuan()#切换到最新页面
        time.sleep(4)
        img_have()#判断是否弹出验证码窗口
        abc = img_have()
        if abc is True:#如果有验证码窗口
            img()#爬取验证码
            time.sleep(2)
            base64_api(uname, pwd, img, typeid)#调用图鉴api识别验证码内容
            shuru()#填写识别出来的验证码内容
            wenjian1()#检测下载文件夹内的文件数量
            file = wenjian1()
            time.sleep(1)
            driver.find_element_by_xpath('/html/body/div/form/dl/dd/button').click()#点击验证码提交
            wenjian2()#检测点击下载后文件夹内的文件数量
            file2 = wenjian2()
            time.sleep(1)
            wenjian3()#判断下载文件夹内的文件数量是否变化
            abc2 = wenjian3()
            while abc2 == False:#如果数量没有增加，则返回False，判断等于False，则重新进行验证码爬取识别以及输入并再次检测
                img()#爬取验证码
                base64_api(uname, pwd, img, typeid)#调用图鉴api识别验证码内容
                shuru()#填写识别出来的验证码内容
                wenjian1()#检测下载文件夹内的文件数量
                file = wenjian1()
                driver.find_element_by_xpath('/html/body/div/form/dl/dd/button').click()#点击验证码提交
                wenjian2()#检测点击下载后文件夹内的文件数量
                file2 = wenjian2()
                wenjian3()#判断下载文件夹内的文件数量是否变化
                abc2 = wenjian3()
                if abc2 == True:#如果判断等于True，则代表输入正确下载成功
                    break
            true()#下载成功，情况正确，关闭窗口，返回原窗口，进行下一篇文章PDF原文文件下载   
        else:#没有验证码窗口，则直接下载原文文件，然后关闭详情页，返回文章集合页，继续下一篇PDF原文文件下载
            qiehuan()
            driver.close()
            driver.switch_to.window(driver.window_handles[1])
            time.sleep(1)
    driver.find_element_by_xpath('//*[@id="PageNext"]').click()#一页循环挖取结束，点击进入下一页
    time.sleep(2)
```
- 这样就成功爬取出了我们需要的全部PDF原文文件，而其中出现的问题也得到了一并解决。

### 第六步：数据分析
- 将爬取到的endnote文件先制作成ris文件再通过VOSviewer对这些文章的关键词进行数据可视化处理，获取关键词及它们之间的数据结果关系，处理分析结果如下：
![数据可视化分析图](https://gitee.com/huang_zehao/selenium-real-combat---hownet/raw/master/%E6%95%B0%E6%8D%AE%E5%8F%AF%E8%A7%86%E5%8C%96%E5%88%86%E6%9E%90%E5%9B%BE%E7%89%87.png)
> 在人工智能和大数据的主题下所爬取到的所有文章中，关键词都有深度学习、数据挖掘、物联网、区块链、云计算、数字经济、金融科技、智慧教育、教育大数据、大数据分析、大数据时代、互联网、智能制造、算法歧视、算法、数字孪生、数据驱动、知识服务、媒体融合、智能化、5G、图书馆、知识图谱等。其中深度学习、区块链、物联网、云计算等是高频重点关键词，也就是说在人工智能和大数据的主题下，这几个关键词是大体的研究重点；而金融经济类的数字科技，如金融科技和数字经济比起教育类的智慧教育、教育大数据等关键词出现次数多，也就是说在人工智能和大数据的主题下，相较于教育类更多人研究金融经济类；媒体融合、5G、智能化等关键词频次较少，但与金融经济类和教育类以及区块链、物联网等关键词联系更为紧密，也就是说它们出现在同一篇文章中的次数多，这也就意味着现实生活中这些关键词的联系是人工智能和大数据主题下的一大热门课题；而数字孪生、数据驱动、知识服务、算法歧视、算法等关键词出现的频次和与其他关键词的联系就相对少了很多，也就是说这些关键词更相对的属于更加专业化、深度化的课题下，它们较为小众，但同样重要，只是研究其课题程度的人较少。

## 项目结果
* 成功爬取合适的有价值的query（大数据和人工智能）下的论文总计27页的所有文章，总计1307篇文章。爬取内容包括论文篇名、作者、刊名、发表时间、下载次数以及导出refworks文件和endnote文件（.txt），并成功保存在本地的文件夹中，也成功依次自动化循环下载PDF原文文件，成功处理了中间产生的各种问题，最后进行了数据分析——数据可视化。

## 总结
* 在本次期末项目实践中，我学习并逐渐了解学习了selenium和其他模块的操作以及知网文章爬取，并加深了对数据挖掘这门课的理解。感谢智超老师的细心指导以及解答，同时感谢在项目完成过程中对我有帮助的同学们，希望大家互帮互助，共同进步，谢谢！