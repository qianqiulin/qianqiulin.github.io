---
layout:     post
title:      "CentOS 7 安装tesseract-ocr"
subtitle:   "ocr"
date:       2019-09-13 21:58:11
author:     "TyurinTsien"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - Centos
---


# 安装tesseract-ocr
## 环境
- CentOS7.4
- Python3.7
- leptonica-1.74.4
    - 编译安装
- tesseract-3.04
    - 编译安装
## 首先安装依赖的leptonica库：
```
wget http://www.leptonica.com/source/leptonica-1.74.4.tar.gz
tar -xvf leptonica-1.74.4.tar.gz
cd leptonica-1.74.4
./configure && make &&sudo make install
```
## 安装tesseract-ocr 
```
wget https://github.com/tesseract-ocr/tesseract/archive/3.04.zip
tar xzf tesseract-ocr-3.04.tar.gz
cd tesseract-3.04
./autogen.sh
./configure
make
sudo make install
sudo ldconfig
```
## 下载语言库
```
wget https://github.com/tesseract-ocr/tessdata/raw/master/eng.traineddata
wget https://github.com/tesseract-ocr/tessdata/raw/master/chi_sim.traineddata
```
- 将下载后的语言库存放至：/usr/local/share/tessdata/ 下面 
```
cp *.traineddata /usr/local/share/tessdata/  
```
- 至此，已可以进行字符的识别了，
- 利用以下类似命令进行识别：
```
tesseract imagename out -l eng/chi_sim
```

# 安装pytesseract
- tesseract-ocr是c++编写的，默认提供的是c++ libs，如果用python开发，还需要安装pytesseract

## 安装
```
pip install pytesseract
```
## 测试
在python 环境下编写代码：
```
import pytesseract
from PIL import Image
image = Image.open("code.png")
code = pytesseract.image_to_string(image,lang='chi_sim')
print(code)
```

# 报错
```
raise TesseractError(proc.returncode, get_errors(error_string))
pytesseract.pytesseract.TesseractError: (1, 'Tesseract Open Source OCR Engine v3.04.02dev with Leptonica Error in pixReadMemPng: function not present Error in pixReadMem: png: no pix returned Error during processing.')
```
## 问题原因及解决方式
- 缺少依赖包。
- 安装依赖包
```
yum install -y libjpeg-devel libpng-devel libgif-devel
```
- 重新编译leptonica
    - 进入leptonica目录
    - 编译安装
    ```
    ./configure 
    make && make install
    ```
