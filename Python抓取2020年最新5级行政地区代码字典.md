
# Python抓取2020年最新5级行政地区代码字典 #

Python抓取2020年最新5级行政地区代码字典

最近要更新地区字典，于是通过网上查找的资料，进行整理出来了一段代码，用于抓取数据

设好数据库地址后，可直接插入数据库

地区代码字典数据来源：http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2020/index.html

以下代码仅供学习参考使用

```
#!/usr/bin/python
# -*- coding: UTF-8 -*-
#   功能：  获取省市县行政地区代码
import importlib
import sys
import pymysql
 
importlib.reload(sys)
import requests
import lxml.etree as etree
import os
 
 
class chinese_city():
    # 初始化函数
    def __init__(self):
        self.baseUrl = 'http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2020/index.html'
        self.base = 'http://www.stats.gov.cn/tjsj/tjbz/tjyqhdmhcxhfdm/2020/'
        self.conn=pymysql.connect(host = "", user = "", passwd = "", db = "test", charset = "utf8")
        self.cur = self.conn.cursor()
        self.trdic = {
            1: '//tr[@class="provincetr"]',
            2: '//tr[@class="citytr"]',
            3: '//tr[@class="countytr"]',
            4: '//tr[@class="towntr"]',
            5: '//tr[@class="villagetr"]'
        }
 
    def __del__(self):
        if self.cur:
            self.cur.close()
        if self.conn:
            self.conn.close()
 
    def crawl_page(self, url):
        ''' 爬行政区划代码公布页 '''
        # print(f"crawling...{url}")
        headers = {'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:71.0) Gecko/20100101 Firefox/71.0',
                   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'}
        i = 0
        #超时自动重复执行
        while i < 10:
            try:
                html = requests.get(url, headers=headers, timeout=20)
                html.encoding = 'gbk'  # 这里添加一行
                # print(html.status_code)
                text = html.text
                return text
            except requests.exceptions.RequestException:
                i += 1
                print('超时' + url)
 
    # 解析省页，返回list
    def parseProvince(self):
        html = self.crawl_page(self.baseUrl)
        tree = etree.HTML(html, parser=etree.HTMLParser(encoding='gbk'))
        nodes = tree.xpath('//tr[@class="provincetr"]')
        id = 1
 
        values = []
        for node in nodes:
            items = node.xpath('./td')
            for item in items:
                value = {}
                nexturl = item.xpath('./a/@href')
                province = item.xpath('./a/text()')
                code = ''.join(nexturl).replace('.html','',1)+''.join('0000')
                # print(code)
                value['url'] = self.base + "".join(nexturl)
                value['name'] = "".join(province)
                value['zonecode'] = code
                value['pid'] = code
                value['id'] = id
                value['level'] = 1
                # print(repr(value['name']))
                id = id + 1
                last_id = self.insert_to_db(value)
                value['id'] = last_id
                values.append(value)
                print(value)
        return values
 
 
 
 
    # 根据trid 解析子页
    def parse(self, trid, pid, url):
        if url.strip() == '':
            return None
        # url_prefix+url
        html = self.crawl_page(url)
        tree = etree.HTML(html, parser=etree.HTMLParser(encoding='gbk'))
 
        if trid == 3:
            nodes = tree.xpath(self.trdic.get(trid))
            if len(nodes) == 0:
                nodes = tree.xpath(self.trdic.get(4))
                print('有镇的市：' + url)
        else:
            nodes = tree.xpath(self.trdic.get(trid))
 
        path = os.path.basename(url)
        base_url = url.replace(path, '')
        id = 1
        values = []
        # 多个城市
        for node in nodes:
            value = {}
            nexturl = node.xpath('./td[1]/a/@href')
            if len(nexturl) == 0:
                nexturl = ''
            tempcode = node.xpath('./td[1]/a/text()')
            if len(tempcode) == 0:
                tempcode = node.xpath('./td[1]/text()')
            name = node.xpath('./td[2]/a/text()')
            if trid==2:
                code =''.join(tempcode).replace('000000','',1)
            if trid == 3:
                code = ''.join(tempcode).replace('000000', '', 1)
            elif trid==4:
                code=''.join(tempcode).replace('000','',1)
            elif trid==5:
                code=''.join(tempcode).replace('000','',1)
            if len(name) == 0:
                name = node.xpath('./td[2]/text()')
            value['zonecode'] = "".join(code)
            urltemp = "".join(nexturl)
            if len(urltemp) != 0:
                value['url'] = base_url + "".join(nexturl)
            else:
                value['url'] = ''
            value['name'] = "".join(name)
            print(repr(value['name']))
            print(value['url'])
            value['id'] = id
            value['pid'] = pid
            value['level'] = trid   #行政等级
            id = id + 1
            last_id = self.insert_to_db(value)
            value['id'] = last_id
            values.append(value)
            print(value)
        return values
 
    # 解析社区页
    def parseVillager(self, trid, pid, url):
        html = self.crawl_page(url)
        tree = etree.HTML(html, parser=etree.HTMLParser(encoding='gbk'))
        nodes = tree.xpath(self.trdic.get(trid))
        id = 1
        values = []
        # 多个城市
        for node in nodes:
            value = {}
            nexturl = node.xpath('./td[1]/a/@href')
            code = node.xpath('./td[1]/text()')
            vcode = node.xpath('./td[2]/text()')
            name = node.xpath('./td[3]/text()')
            value['zonecode'] = "".join(code)
            value['url'] = "".join(nexturl)
            value['name'] = "".join(name)
            print(repr(value['name']))
            value['id'] = id
            value['pid'] = pid
            value['level'] = trid
            values.append(value)
            id = id + 1
            last_id = self.insert_to_db(value)
            value['id'] = last_id
            values.append(value)
            print(value)
 
        return values
 
    # 插入数据库
    def insert_to_db(self, taobao):
        # return 0
        param = []
        lastid = 0
        try:
            sql = 'INSERT INTO tab_citys (id,parent_id,city_name_zh,city_level,city_code) VALUES(%s,%s,%s,%s,%s)'
            param = (0,taobao.get("pid"), taobao.get("name"),taobao.get("level"), taobao.get("zonecode"))
            self.cur.execute(sql, param)
            lastid = self.cur.lastrowid
            self.conn.commit()
        except Exception as e:
            print(e)
            self.conn.rollback()
        return lastid
 
    # 从头执行解析
    def parseChineseCity(self):
        values = self.parseProvince()
        for value in values:
            citys = self.parse(2, value['zonecode'], value['url'])
            if not citys is None:
                for city in citys:
                    countys = self.parse(3, city['zonecode'], city['url'])
                    # 获取 乡镇和居委会数据，不要的可以注释掉
                    if not countys is None:
                        for county in countys:
                            towns = self.parse(4, county['zonecode'], county['url'])
                            # if towns is not None:
                            #     for town in towns:
                            #         villagers = self.parseVillager(5, town['zonecode'], town['url'])
 
 
if __name__ == '__main__':
    chinese_city = chinese_city()
    chinese_city.parseChineseCity()

```