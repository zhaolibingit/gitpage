---
layout: post
title:  "python基础模块介绍"
date:   2016-12-01 13:31:01 +0800
categories: python
tag: python基础
---

* content
{:toc}


#### 此篇介绍python基础模块使用

#### os模块

```python
# -*- coding: utf-8 -*-

import os

#Python 的 os 模块封装了常见的文件和目录操作

path = "/root/"
print os.path.abspath('.')
print os.path.basename('./os_study.py')
print os.path.dirname('/')                  #返回文件路径
print os.path.isdir(path)  #判断路径是否为目录
print os.listdir(path)

# os.mkdir	 创建目录
# os.rmdir	 删除目录
# os.rename	 重命名
# os.remove	删除文件
# os.getcwd	获取当前工作路径
# os.walk	遍历目录
# os.path.join	连接目录与文件名
# os.path.split	分割文件名与目录
# os.path.abspath	获取绝对路径
# os.path.dirname	获取路径
# os.path.basename	获取文件名或文件夹名
# os.path.splitext	分离文件名与扩展名
# os.path.isfile	判断给出的路径是否是一个文件
# os.path.isdir	判断给出的路径是否是一个目录

# # print(os.getcwd())#打印当前目录
# os.getcwd() 获取当前工作目录，即当前python脚本工作的目录路径
# os.chdir("dirname")  改变当前脚本工作目录；相当于shell下cd
# os.curdir  返回当前目录: ('.')
# os.pardir  获取当前目录的父目录字符串名：('..')
# os.makedirs('dirname1/dirname2')    可生成多层递归目录
# os.removedirs('dirname1')    若目录为空，则删除，并递归到上一级目录，如若也为空，则删除，依此类推
# os.mkdir('dirname')    生成单级目录；相当于shell中mkdir dirname
# os.rmdir('dirname')    删除单级空目录，若目录不为空则无法删除，报错；相当于shell中rmdir dirname
# os.listdir('dirname')    列出指定目录下的所有文件和子目录，包括隐藏文件，并以列表方式打印
# os.remove()  删除一个文件
# os.rename("oldname","newname")  重命名文件/目录
# os.stat('path/filename')  获取文件/目录信息
# os.sep    输出操作系统特定的路径分隔符，win下为"\\",Linux下为"/"
# os.linesep    输出当前平台使用的行终止符，win下为"\t\n",Linux下为"\n"
# os.pathsep    输出用于分割文件路径的字符串
# os.name    输出字符串指示当前使用平台。win->'nt'; Linux->'posix'
# os.system("bash command")  运行shell命令，直接显示
# os.environ  获取系统环境变量
# os.path.abspath(path)  返回path规范化的绝对路径
# os.path.split(path)  将path分割成目录和文件名二元组返回
# os.path.dirname(path)  返回path的目录。其实就是os.path.split(path)的第一个元素
# os.path.basename(path)  返回path最后的文件名。如何path以／或\结尾，那么就会返回空值。即os.path.split(path)的第二个元素
# os.path.exists(path)  如果path存在，返回True；如果path不存在，返回False
# os.path.isabs(path)  如果path是绝对路径，返回True
# os.path.isfile(path)  如果path是一个存在的文件，返回True。否则返回False
# os.path.isdir(path)  如果path是一个存在的目录，则返回True。否则返回False
# os.path.join(path1[, path2[, ...]])  将多个路径组合后返回，第一个绝对路径之前的参数将被忽略
# os.path.getatime(path)  返回path所指向的文件或者目录的最后存取时间
# os.path.getmtime(path)  返回path所指向的文件或者目录的最后修改时间
```

#### sys模块
```python
import sys

#sys.argv: 实现从程序外部向程序传递参数。
#sys.exit([arg]): 程序中间的退出，arg=0为正常退出。
# sys.getdefaultencoding(): 获取系统当前编码，一般默认为ascii。
# sys.setdefaultencoding(): 设置系统默认编码，执行dir（sys）时不会看到这个方法，
#       在解释器中执行不通过，可以先执行reload(sys)，在执行 setdefaultencoding('utf8')，此时将系统默认编码设置为utf8。（见设置系统默认编码 ）
# sys.getfilesystemencoding(): 获取文件系统使用编码方式，Windows下返回'mbcs'，mac下返回'utf-8'.
# sys.path: 获取指定模块搜索路径的字符串集合，可以将写好的模块放在得到的某个路径下，就可以在程序中import时正确找到。
# sys.platform: 获取当前系统平台。

# sys.stdin,sys.stdout,sys.stderr: stdin , stdout , 以及stderr 变量包含与标准I/O 流对应的流对象. 如果需要更好地控制输出,
# 而print 不能满足你的要求, 它们就是你所需要的. 你也可以替换它们, 这时候你就可以重定向输出和输入到其它设备( device ), 或者以非标准的方式处理它们
# sys.argv --传参，第一个参数为脚本名称即argv[0]
#
# sys.path --模块搜索路径
#
# sys.moudule --加载模块字典
#
# sys.stdin　　--标准输入
#
# sys.stdout　　--标准输出
#
# sys.stderr　　--错误输出
#
# sys.platform --返回系统平台名称
#
# sys.version　　--查看python版本
#
# sys.maxsize　　--最大的Int值
```


#### date和datetime模块

```python

import time,datetime

print time.altzone
#返回与utc时间的时间差,以秒计算

print time.asctime()
#返回时间格式
#Thu Apr 25 11:15:14 2019

print time.localtime()
# >>>time.struct_time(tm_year=2016, tm_mon=8, tm_mday=22, tm_hour=9, tm_min=32, tm_sec=28, tm_wday=0, tm_yday=235, tm_isdst=0)
#tm_year(年)，tm_mon(月),tm_mday(日),tm_hour(时),tm_min(分),tm_sec(秒),tm_wday（星期，从0到6,0代表周一，一次类推）,tm_yday(这一年中的地几天),tm_isdst(夏令时间标志)

print time.time()
# #(获取时间戳)
# a = time.time()

# b = a/3600/24/365
# print(b)
# >>>46.67149458666888
#  2016-1970
#  >>>46
#时间是村1970年开始算，为什么时间要从1970年开始算呢
#1970年1月1日 算 UNIX 和 C语言 生日. 由于主流计算机和操作系统都用它,其他仪器,手机等也就用它了.


print time.gmtime(time.time()-800000)
#返回utc时间的struc时间对象格式
# >>>time.struct_time(tm_year=2016, tm_mon=8, tm_mday=12, tm_hour=20, tm_min=9, tm_sec=14, tm_wday=4, tm_yday=225, tm_isdst=0)

print time.asctime(time.localtime())

# >>>Mon Aug 22 10:23:52 2016
# print(time.ctime())
# >>>Mon Aug 22 10:24:25 2016


string_2_struct = time.strptime("2016/05/22","%Y/%m/%d")#将时间字符串转换成struct时间对象格式
print string_2_struct
# >>>time.struct_time(tm_year=2016, tm_mon=5, tm_mday=22, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=6, tm_yday=143, tm_isdst=-1)


string_3_struct = time.strptime("2016/05/22","%Y/%m/%d")
struct_2_stamp = time.mktime(string_3_struct) #将struct时间对象转成时间戳
print struct_2_stamp
# >>>1463846400.0


print time.gmtime(time.time()-86640)##将utc时间戳转换成struct_time格式
#>>>time.struct_time(tm_year=2016, tm_mon=8, tm_mday=21, tm_hour=2, tm_min=25, tm_sec=14, tm_wday=6, tm_yday=234, tm_isdst=0)

a = time.timezone
print(a/3600)
# >>>-8.0东八区

# time.sleep()#睡几秒

print time.localtime()#当前本地时间


#strftime()#重要！
# a = time.localtime()#将本地时间格式化
# time = time.strftime("%Y/%m/%d %H:%M:%S",a)
# print(time)
# >>>2016/08/22 10:31:07
# 前
# 前后刚好相反
# 后
# # time.strptime()#前后一定要位置一一对应(注意符号空格)
# time1 = time.strptime("2016-08:20 14:31:52","%Y-%m:%d %H:%M:%S")
# print(time1)


#####datetime（时间的加减）

# print(datetime.datetime.now()) #返回 2016-08-22 10:33:53.290793

# print(datetime.date.fromtimestamp(time.time()) ) #2016-08-22

# print(datetime.datetime.now() )#2016-08-22 10:34:37.885578当前时间一般使用这个

# print(datetime.datetime.now() + datetime.timedelta(3)) #当前时间+3天#2016-08-25 10:35:38.554216

# print(datetime.datetime.now() + datetime.timedelta(-3)) #当前时间-3天#2016-08-19 10:35:58.064103

# print(datetime.datetime.now() + datetime.timedelta(hours=3)) #当前时间+3小时#2016-08-22 13:36:16.540940

# print(datetime.datetime.now() + datetime.timedelta(minutes=30)) #当前时间+30分#2016-08-22 11:06:34.837476

# c_time  = datetime.datetime.now()
# print(c_time.replace(minute=3,hour=2)) #时间替换
```
#### re模块

```python
import re

#1.去除以下html文件中的标签，只显示文本信息。
html_data = '''
<div>
<p>岗位职责：</p>
<p>完成推荐算法、数据统计、接口、后台等服务器端相关工作</p>
<p><br></p>
<p>必备要求：</p>
<p>良好的自我驱动力和职业素养，工作积极主动、结果导向</p>
<p>&nbsp;<br></p>
<p>技术要求：</p>
<p>1、一年以上 Python 开发经验，掌握面向对象分析和设计，了解设计模式</p>
<p>2、掌握HTTP协议，熟悉MVC、MVVM等概念以及相关WEB开发框架</p>
<p>3、掌握关系数据库开发设计，掌握 SQL，熟练使用 MySQL/PostgreSQL 中的一种<br></p>
<p>4、掌握NoSQL、MQ，熟练使用对应技术解决方案</p>
<p>5、熟悉 Javascript/CSS/HTML5，JQuery、React、Vue.js</p>
<p>&nbsp;<br></p>
<p>加分项：</p>
<p>大数据，数理统计，机器学习，sklearn，高性能，大并发。</p>
</div> 
'''

# #方法1：
# print re.sub('<div>|</div>|<p>|</p>|<br>|&nbsp;','',html_data)
# #方法2：
# #匹配/ 0次或者一次，匹配任意字符n次  ？代表0次或一次，+代表任意次
# print re.sub('</?\w+>|&nbsp;','',html_data)
# #方法3：
# p = re.compile('</?\w+>|&nbsp;')
# print re.sub(p,'',html_data)


#2.将以下网址提取出域名：
url_list ='''
http://www.interoem.com/messageinfo.asp?id=35`
http://3995503.com/class/class09/news_show.asp?id=14
http://lib.wzmc.edu.cn/news/onews.asp?id=7
http://www.zy-ls.com/alfx.asp?newsid=377&id=6
http://www.fincm.com/newslist.asp?id=415
'''

#print re.sub('http://|/.*','',url_list)

#3.匹配ip地址

re.findall(r'(?<![\.\d])(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)(?![\.\d])','102.112.121.112')

## 正则参考 http://www.runoob.com/python/python-reg-expressions.html

# (?: re)	类似 (...), 但是不表示一个组
# \w	匹配字母数字及下划线

#### Python中re的match、search、findall、finditer区别？？？

# 1、match
#
#         re.match(pattern, string[, flags])
# 从首字母开始开始匹配，string如果包含pattern子串，则匹配成功，返回Match对象，失败则返回None，若要完全匹配，pattern要以$结尾。
#
# 2、search
#
#         re.search(pattern, string[, flags])
# 若string中包含pattern子串，则返回Match对象，否则返回None，注意，如果string中存在多个pattern子串，只返回第一个。
#
# 3、findall
#
#         re.findall(pattern, string[, flags])
# 返回string中所有与pattern相匹配的全部字串，返回形式为数组。
#
# 4、finditer
#
#         re.finditer(pattern, string[, flags])
# 返回string中所有与pattern相匹配的全部字串，返回形式为迭代器。
```
####  random模块
```python

import random

#0到1的随机数，是一个float浮点数
print random.random()

print random.randint(1,10)

print random.randrange(1,20)
```
#### logging 模块

```python

import logging

logging.basicConfig(filename='log.log',
                    format='%(asctime)s - %(name)s - %(levelname)s:  %(message)s',
                    datefmt='%Y-%m-%d %H:%M:%S %p',
                    level=10)

logging.debug('debug')
logging.info('info')
logging.warning('warning')
logging.error('error')
logging.critical('critical')
logging.log(10, 'log')


# %(name)s：Logger的名字
#
# %(levelno)s：数字形式的日志级别
#
# %(levelname)s：文本形式的日志级别
#
# %(pathname)s：调用日志输出函数的模块的完整路径名，可能没有
#
# %(filename)s：调用日志输出函数的模块的文件名
#
# %(module)s：调用日志输出函数的模块名
#
# %(funcName)s：调用日志输出函数的函数名
#
# %(lineno)d：调用日志输出函数的语句所在的代码行
#
# %(created)f：当前时间，用UNIX标准的表示时间的浮 点数表示
#
# %(relativeCreated)d：输出日志信息时的，自Logger创建以 来的毫秒数
#
# %(asctime)s：字符串形式的当前时间。默认格式是 “2003-07-08 16:49:45,896”。逗号后面的是毫秒
#
# %(thread)d：线程ID。可能没有
#
# %(threadName)s：线程名。可能没有
#
# %(process)d：进程ID。可能没有
#
# %(message)s：用户输出的消息
```