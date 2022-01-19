[TOC]

#程序#

# 基于 Django REST framework 的图片上传

## 前期准备

###　本文使用环境

系统：debian10(腾讯云服务器)

Python 版本：3.7（虚拟环境）

pip 版本：21.3.1

Django 版本：3.2.9

Pillow 版本：8.4.0

### 开发方式

由 Vs code 的 Remote SSH 插件连接到云服务器端

## 搭建开发环境

### Python

Django 环境基于 Python 语言，所以我们需要安装 Python 开发环境

https://www.python.org/downloads/release/python-3710/
*安装时勾选 Add Python xx.xx to PATH 选项*

安装完成后打开 cmd 或 powershell 窗口（linux 下是 shell）
输入以下命令并回车运行:

```bash
python -V
```

如果安装成功则会显示 Python 当前版本

### Pip

Pip 是 Python 的包管理器，通过 Pip 能非常方便地安装各种依赖包
Pip 的安装与配置的前置为 Python，请安装好 Python 后进行 Pip 的配置或安装
Python 3.4 以上都自带了 Pip。
配置命令如下：

```bash
pip3 config set global.index https://www.python.org/downloads/release/python-3710
# 换源到清华源
```

### 虚拟环境 Venv

输入以下命令将创建一个名字为 dev 的虚拟环境

```bash
python -m venv dev
```

### Pillow

我们的 Pillow 将会被安装到虚拟环境中

```bash
cd dev #进入到dev虚拟环境的目录中
source bin/activate #激活虚拟环境
python -m pip install --upgrade pip 
#必要的更新，pip版本过低将直接导致Pillow安装的失败

pip install pillow #安装Pillow
```

### Django

```bash
cd dev #进入到dev虚拟环境的目录中
source bin/activate #激活虚拟环境

pip install django #django
pip install djangorestframework 
#rest framework
```

## 构建项目

### 进入虚拟环境

```bash
cd dev #此处dev为前文中创建的虚拟环境名
#虚拟环境存在于同名的文件夹中

source bin/activate
```

### 创建项目

```bash
django-admin startproject 项目名 .
# 在当前目录建立Django项目
```

### 创建应用(app)

```bash
cd 项目名 # 进入项目所在目录
django-admin startapp 应用名
```

### 迁移数据库

本次迁移数据库是为了将生成的项目文件中自带的模型迁移到数据库中

```bash
python manage.py migrate
```

### 建立 Django 管理用户

```bash
python manage.py createsuperuser --email aa@bb.com --username admin
# 建立一个超级用户，邮箱为aa@bb.com,用户名为admin
```

命令输入并回车后，将会进入密码设置，此时输入的密码是不可见的，正常输入即可

输入完第一次后会要求重复一次密码，依然正常输入即可

至此项目构建完毕

## 编写代码

### 编写序列化器

在项目目录下找到创建到的应用目录，新建一个文件 `mySer.py`

内容如下:

```python
from django.db import models
from django.db.models import fields
from .models import ImageModel
from rest_framework import serializers

class ImageSerializer(serializers.ModelSerializer):
    class Meta:
        model=ImageModel #模型名
        fields="__all__" #字段
```

### 编写模型文件

打开应用目录下的 `models.py` 文件

内容如下:

```python
from django.db import models

class ImageModel(models.Model):
    name=models.CharField(max_length=50)
    #name字段，字符类型(Char),最大长度为50
    image=models.ImageField()
    #image字段，图片类型
```

### 编写视图文件

在应用目录下找到 `views.py`.

内容如下:

```python
from .models import ImageModel
from .Serializers import ImageSerializer
from rest_framework import viewsets

class ImageUploadViewSet(viewsets.ModelViewSet):
    queryset=ImageModel.objects.all()
    serializer_class=ImageSerializer# 设置序列化器为我们编写的ImageSerializer类
```

### 修改应用配置文件

应用目录下找到 `apps.py`

```python
from django.apps import AppConfig


class MyimageConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    #name='应用名'
    name = '项目名.应用名'
```

这里的 name 属性在 django 3.2 后的格式 为 `项目名.应用名`

### 修改 URL 文件

在项目目录下找到 `url.py` 文件,

输入内容如下:

```python
from django.conf.urls import include
from django.contrib import admin

from django.urls import path

from rest_framework import routers
from .myImage.views import ImageUploadViewSet

router=routers.DefaultRouter()
router.register(r'upload',ImageUploadViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('apis/', include(router.urls)),
]

```

### 修改项目设置文件

项目目录下找到 `setting.py` 文件，找到以下内容并修改

```python
#上面还有其他东西，但无关本次操作
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    #以下为加入的内容，本行注释不要加入文件中！
    'rest_framework',
    '项目名.应用名'
]
#下面还有其他东西，但无关本次操作
```

## 部署项目

### 迁移数据库

我们编写了一个模型类（`models.py` 中),

在模型类定义了图片的模型(name,image)，

为了将这个更改应用到数据库中，我们需要使用数据库迁移命令：

```bash
#激活虚拟环境,此处省略步骤

cd 项目目录 
python manage.py makemigrations
# 生成迁移
python manage.py migrate
# 执行迁移
```

## 运行项目

```python
#激活虚拟环境

cd 项目目录
python manage.py runserver 0.0.0.0:8080
# 开启一个简单的http服务器，端口为8080
#访问方式:http://localhost:8080/
#本项目图片上传地址为:
# ttp://localhost:8080/apis/upload/
```
