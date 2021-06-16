# selenium实战-知网
* 撰写人：黄泽豪
* 最新撰写时间： 2021.6.16
## 项目意义：
* 熟悉selenium操作，熟悉自动化挖掘操作
* 学习如何爬取知网内容
* 爬取知网文章，提高论文下载效率，方便学习工作
## 项目目的
1. 爬取合适的有价值的query（关键词）下的论文
2. 爬取文章数量大于800篇
3. 爬取相关重要信息
4. 导出refworks文件（.txt）
5. 下载原文
## 项目结果
* 成功爬取合适的有价值的query（大数据和人工智能）下的论文总计26页的所有文章，总计1293篇文章。爬取内容包括论文篇名、作者、刊名、发表时间、下载次数以及导出refworks文件（.txt），并成功保存在本地的文件夹中，而原文下载由于需要验证码，且百度文字图片识别的API一直无法准确识别出，因此原文下载失败，没有完成。
## 所遇问题以及解决方案
1. 导出refworks文件时，一次性最多选择500篇文章，若想继续导出其他文章则需清除以及选择其他文章，在自动化循环爬取时，我们可以加上while判断语句进行判断，当然，我们也可以变换思路，只爬取每50篇的refworks文件。代码如下：

```
piliang_list = []
piliang_list.append('//*[@id="gridTable"]/div[1]/div[2]/div[1]/a')
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/div[1]/label/input")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/i")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/ul/li[1]/a")
piliang_list.append("/html/body/div[3]/div[2]/div[2]/div[2]/form/div/div[1]/div[2]/ul[1]/li[2]/ul/li[1]/ul/li[8]/a")
piliang_list
```

```
for page in range(0,26):
    for i in piliang_list:
        element = driver.find_element_by_xpath(i)
        element.click()
        time.sleep(2)
    driver.switch_to.window(driver.window_handles[2])
    time.sleep(10)
    driver.find_element_by_xpath('//*[@id="litotxt"]/a').click()
    time.sleep(10)
    driver.close()
    time.sleep(4)
    driver.switch_to.window(driver.window_handles[1])
    driver.find_element_by_xpath('//*[@id="PageNext"]').click()
    time.sleep(12)
# 忘了加上while进行一个终止判断，代码循环运行到第26页后后面没有页数，则没有下一页键无法自动化点击，出现报错。
```


2.在爬取中遇到了许多操作问题都是因为对selenium的不熟悉而造成的，我的解决方案是去浏览selenium的操作指南[selenium操作指南](https://www.selenium.dev/documentation/zh-cn/webdriver/browser_manipulation/)


3.若有多个页面，要先定位好需要操作的页面后才能继续自动化挖取，不然会定位不到而无法挖取

```
driver.switch_to_window(driver.window_handles[1])
```

```
driver.switch_to_window(driver.window_handles[2])
```

## 总结
* 在本次项目实践中，我学习并逐渐了解了selenium的操作以及知网文章爬取，并加深了对数据挖掘这门课的理解。感谢智超老师的细心指导以及解答，同时感谢在项目完成过程中对我有帮助的同学们。