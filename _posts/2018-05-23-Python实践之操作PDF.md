---
layout:     post                    	# 使用的布局（不需要改）
title:      Python实践之操作PDF               # 标题 
subtitle:   	#副标题
date:       2018-05-23              # 时间
author:     iceman                      # 作者
header-img: img/post-bg-kuaidi.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Python	
    - PDF
---


今天和大家分享一个平时大家都可能会遇到的如下这些场景：

- 拷贝PDF中文本信息
- 给PDF文档加个水印
- 只需要PDF中某几页(可能是交叉页面)
- 手头有一大堆PDF文档，需要给每个文档都加个密码

上面几个场景，看起来简单，但是当真需要处理的时候，发现手头如果没有一个比较好的PDF的编辑器，很难完成任务。同时当对于一大堆文档的时候，即使有个好的PDF编辑器，也要做重复且大量的工作，才能完成任务

今天就和大家分享一个好东西，那就是Python这个强大的脚本语言，当处理如上问题，那叫一个简单。废话就不多说，我们赶快进入正题，在进入正题前，我们先了解几个基本概念，这样方便不太清楚的小伙伴也能先有个正题理解


## 前奏

### 什么是PDF文档

老规矩，维基百科给PDF定义如下：

> **可移植文档格式**（**Portable Document Format**，简称**PDF**）是一种用独立于[应用程序](https://zh.wikipedia.org/wiki/%E6%87%89%E7%94%%E7%A8%8B%E5%BC%8F)、[硬件](https://zh.wikipedia.org/wiki/%E7%A1%AC%E4%BB%B6)、[操作系统](https://zh.wikipedia.org/wiki/%E4%BD%9C%E6%A5%AD%E7%B3%BB%E7%B5%B1)的方式呈现[文档](https://zh.wikipedia.org/wiki/%E6%96%87%E6%A1%A3)的[文件格式](https://zh.wikipedia.org/wiki/%E6%AA%94%E6%A1%88%E6%A0%BC%E5%BC%8F)。每个PDF文件包含固定布局的平面文档的完整描述，包括文本、字形、图形及其他需要显示的信息。 

PDF文档以**.pdf**作为文件扩展名，由于PDF是可移植的文档格式，跨平台、格式固定，这样我发个文档给你，你打开时就不会因为当前系统安装的软件版本不一样打不开、格式布局发生变法，导致你看到的不是我想给你看到的样式。基于这样那样的问题，目前使用场景很多

### 准备环境

既然我们要用Python操作PDF，那边我们首先得有Python环境和处理PDF的Python模块。

#### 安装Python3

首先我们需要在本机准备一个Python的环境，目前有很多中安装Python的方法和软件。此处我强烈推荐大家安装**Anaconda**，这是一个学习Python的利器

Python易用，但用好却不易，其中比较头疼的就是包管理和Python不同版本的问题，特别是当你使用Windows的时候。为了解决这些问题，有不少发行版的Python，比如WinPython、Anaconda、pycharm等，这些发行版将Python和许多常用的package打包，方便pythoners直接使用

这些坑我也都踩过，当时心血来潮要学学数据分析，发现清一色的使用**Anaconda**，小白嘛，所以也就从众一回，自从用上了**Anaconda**，我觉觉得应该这就是我想要的

**Anaconda**是专注于数据分析的Python发行版本，包含了conda、Python等190多个科学包及其依赖项。对于**Anaconda**的优点别人的经典总结：**省时省心、分析利器 **

- 省时省心： Anaconda通过管理工具包、开发环境、Python版本，大大简化了你的工作流程。不仅可以方便地安装、更新、卸载工具包，而且安装时能自动安装相应的依赖包，同时还能使用不同的虚拟环境隔离不同要求的项目
- 分析利器： 在 Anaconda 官网中是这么宣传自己的：适用于企业级大数据分析的Python工具。其包含了720多个数据科学相关的开源包，在数据可视化、机器学习、深度学习等多方面都有涉及。不仅可以做数据分析，甚至可以用在大数据和人工智能领域

既然说了这么多，关键的是下面如何安装：

[**Anaconda**](https://www.anaconda.com/download/)的下载页参见官网，Linux、Mac、Windows均支持 ,建议安装基于Python3的版本，目前基本上都是使用Python3了(如果使用Python2可能本文示例存在一些不兼容)

建议大家先huag花个时间去学习下**Anaconda**，目前网上很多教程，有两个地方可以按照自己需求选择学习

- Jupyter Notebook
- Spyder

阅读本文可以先略过这些操作，如果自己要亲自实践一定要先好好学习下它。

#### 安装PyPDF2模块

完成了**Anaconda**的安装后，下一步我们就要安装模块**PyPDF2**了，要安装它，首先打开其命令行

![打开Anacoda命令行](http://ww1.sinaimg.cn/large/665db722gy1frjqth1x3jj205t03q0sn.jpg)

在命令行输入如下命令：

````bash
pip install PyPDF2
````

![安装PyPDF2](http://ww1.sinaimg.cn/large/665db722gy1frjqujfvfgj20it0awq37.jpg)

注意：这个模块名称是区分大小写的，所以要确保 y 是小写，其他字母都是大写**PyPDF2**，安装完成后在项目中导入*import PyPDF2*即可

完成了安装后，打开IDE(上上图中的**Spyder**)

![Spyder3](http://ww1.sinaimg.cn/large/665db722gy1frjqz46cq3j21hc0scmzr.jpg)


### 提醒

本文主要使用**PyPDF2**库进行pdf文档操作，通过使用发现该库有些缺点，如果无法满足你的需求，下面的内容仅供参考

缺点

- 提取文本时，对中文支持不好
- PyPDF2 不能将任意文本写入 PDF，PyPDF2 写入能力，仅限于从其他 PDF 中拷贝页面、旋转页面、重叠页面和加密文件
- 模块不允许直接编辑 PDF。必须创建一个新的 PDF，然后从已有的文档拷贝内容

## PDF文档操作

下面进入今天的正题，开始前，首先应该去看看PyPDF2的[官方文档](https://pythonhosted.org/PyPDF2/index.html)，主要内容如下图：

![PyPDF2](http://ww1.sinaimg.cn/large/665db722gy1frkazb8k3vj208q09fjrm.jpg)

首先列出主函数和相应全局变量

```python
# -*- coding: utf-8 -*-

import PyPDF2
import os

filename = 'The Zen of Python.pdf'
password = 'iceman'
watermarkpdf = os.path.splitext(filename)[0]+'_with watermark' + os.path.splitext(filename)[1]
tempdir = 'tempdir'

def main(filename):
    if os.path.exists(filename):
        with open(filename, 'rb') as fileObj:
            pdfReader= PyPDF2.PdfFileReader(fileObj)
            print('the pdf info: %s\n' % pdfReader.documentInfo)
            
            #获取文档内容
            getPdfFileText(pdfReader)
            #添加水印
            addWatermark(pdfReader)
            #创建加密pdf，再解密打开
            decryptPdf(watermarkpdf)
            #拆分pdf和合并pdf
            splitPdfAndMergePdf(pdfReader)
    else:
        print('the %s not exist' % filename)
        
    
if __name__ == '__main__':
    main(filename)
```

### PDF文档内容提取

提取页面内容相对较简单，通过**PdfFileReader**对象获取指定页面对象PageObject，通过其方法*extractText()*即可返回文本内容，目前发送中文不太友好，同时也没有找到相对应的解决方案

```python
def getPdfFileText(pdfReader):
    #获取第一页
    pdfPage = pdfReader.getPage(0)
    #获取页面内容
    content = pdfPage.extractText()
    print('The content: %s' % content)
```

结果如下：

```latex
The Zen of Python, by Tim Peters
 
Beautiful is better than ugly.
 
Explicit is better than implicit.
 
Simple is better than complex.
 
Complex is better than complicated.
 
Flat is better than nested.
 
....
 
let's do more of those!
```



### PDF添加水印

对于给PDF文档添加水印其主要步骤有如下几步：

- 将水印放入pdf文件‘watermark.pdf’中，通过PdfFileReader加载
- 加载需要添加水印的pdf文件'The Zen of Python.pdf'
- 通过PdfFileWriter创建输出文件
- 遍历页面依次通过mergePage合并页面，再通过addPage将页面进行保存

```python
def addWatermark(pdfReader):
    #打印水印文件
    pdfWmReader = PyPDF2.PdfFileReader(open('watermark.pdf', 'rb'))
    #创建新的用于保存添加水印后的Pdf
    pdfWriter = PyPDF2.PdfFileWriter()
    tempPage = pdfWmReader.getPage(0)
    #遍历页面添加水印
    for pageNum in range(0, pdfReader.numPages): 
        #对每页调用合并
        pdfReader.getPage(pageNum).mergePage(tempPage)
        #把加了水印的页面添加到最终pdf中
        pdfWriter.addPage(pdfReader.getPage(pageNum))
        
    #保存
    savePdfFile = open(watermarkpdf, 'wb')
    #为下一个步骤做准备
    pdfWriter.encrypt(password)
    pdfWriter.write(savePdfFile)
    savePdfFile.close()
    print('==> add water mark finished')
```

结果形如：

!![水印](http://ww1.sinaimg.cn/large/665db722gy1frki3pk4dwj20rs0qmwg8.jpg)

### PDF加解密

PDF的加解密主要就是三个方法的使用

- encrypt 加密
- isEncrypted 判断文件是否加密
- decrypt 解密文档

```python
def decryptPdf(filename):
    if os.path.exists(filename):
        with open(filename, 'rb') as fileObj:
            pdfReader= PyPDF2.PdfFileReader(fileObj)
            if pdfReader.isEncrypted:
                print('==> this pdf is encryped...')
                pdfReader.decrypt(password)
                print(pdfReader.documentInfo)
    else:
        print(filename + 'is not exist')
```



### PDF页面拆分与合并

页面拆分和合并主要还是通过对页面的遍历，再对页面对象的一些操作。

```python
def createPdf(filename, pageObj):
    pdfWriter = PyPDF2.PdfFileWriter()
    pdfWriter.addPage(pageObj)
    savePdfFile = open(filename, 'wb')
    pdfWriter.write(savePdfFile)
    savePdfFile.close()
    
def splitPdfAndMergePdf(pdfReader):
    #首先将拆分的pdf放入临时目录
    if not os.path.exists(tempdir):
        os.mkdir(tempdir)
    #遍历源文档，按每页拆分
    for pageNum in range(0, pdfReader.numPages):
        createPdf(tempdir + os.path.sep + 'temp_' + str(pageNum)+'.pdf',
                  pdfReader.getPage(pageNum))
    print('==>split file finshed, then merge file')
    
    pdfWriter = PyPDF2.PdfFileWriter()
    #遍历目录，如果是以pdf结尾的就合并
    for file in os.listdir(tempdir):
        if os.path.splitext(file)[1] == '.pdf':
            pdfTempReader = PyPDF2.PdfFileReader(open(tempdir + os.path.sep + file, 'rb'))
            pdfWriter.addPage(pdfTempReader.getPage(0))       
    savePdfFile = open('MergePdf.pdf', 'wb')
    pdfWriter.write(savePdfFile)
    savePdfFile.close()
```

结果形如：

![](http://ww1.sinaimg.cn/large/665db722gy1frki14et1nj20cy07174j.jpg)

该文中涉及的到的代码和文档上传至github中[**step_by_step_learn_python**](https://github.com/wqliceman/step_by_step_learn_python)

欢迎关注交流共同进步
![奔跑阿甘](http://ww1.sinaimg.cn/large/665db722gy1frf76owwqjj2076076q3e.jpg)