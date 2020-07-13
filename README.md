# python-crawler
#### 安装Firefox浏览器，并更新至最新版
#### 安装python3.x，并测试环境
#### 下载selenium的Firefox对应版本驱动[geckodriver.exe]，拷贝驱动至python安装目录
#### 运行程序
```
#!/usr/bin/env python
# coding=utf-8
import csv
import re

import selenium.webdriver.support.ui as ui
from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.common.exceptions import TimeoutException
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC

# 爬取页数限制
pagesize = 10

# 爬取网站
website = 'http://www.gxast.org.cn'

# 配置不显示图片
options = webdriver.FirefoxProfile()
options.set_preference('permissions.default.image', 2)
# 创建driver
browser = webdriver.Firefox(options)

# 已爬取连接
hold_pool = []
article_urls = []
# 声明一个全局列表，用来存储字典
data_list = []


def is_visible():
    try:
        ui.WebDriverWait(browser, 10).until(EC.visibility_of_element_located((By.XPATH, '/html/body/div[2]/div[2]')))
        return True
    except TimeoutException:
        return False


def crawl_article_url(web_url):
    print("解析连接：" + web_url)
    # 请求网站
    browser.get(web_url)
    # 等待页面渲染
    is_visible()
    # page = browser.page_source
    # url_list = re.findall('href=\"(.*?)\"', page, re.S)

    url_list = browser.find_elements_by_xpath("//a")
    for alink in url_list:
        try:
            url = alink.get_attribute("href")
            # 已爬取页和当前页/首页连接，进行下一次循环
            if hold_pool.__contains__(url) or url == '#':
                continue
            hold_pool.append(url)

            # 相对路径连接，则拼接网站前缀
            if not (url.startswith("http://") or url.startswith("https://")):
                url = website + url

            # 非本网站的连接跳过
            elif not url.startswith(website):
                continue

            # 判断是否为详情页
            pattern = re.compile(r'\S*/details_\d+\.html$')
            match = re.match(pattern, url)
            if match:
                article_urls.append(url)
                print('文章详情连接：' + url)
                continue
            # 递归爬取
            crawl_article_url(url)
        except Exception as e:
            print(e)


def main():
    hold_pool.append("/bsfw/xzzq/")
    hold_pool.append("http://xh.cast.org.cn/")
    crawl_article_url(website)
    # 全部爬取结束后退出浏览器
    browser.quit()

    # 将数据写入csv文件
    with open('data_csv.csv', 'w', encoding='utf-8', newline='') as f:
        # 表头
        title = data_list[0].keys()
        # 声明writer对象
        writer = csv.DictWriter(f, title)
        # 写入表头
        writer.writeheader()
        # 批量写入数据
        writer.writerows(data_list)
    print('csv文件写入完成')


if __name__ == '__main__':
    main()

```
