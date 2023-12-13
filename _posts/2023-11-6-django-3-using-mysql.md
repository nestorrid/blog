---
title: 3. 在Django项目中集成MySql
date: 2023-11-6 01:59:44 +08000
categories: [笔记, Django]
tags: [python, django, backend]
---

## 安装mysqlclient

```bash
pipenv install mysqlclient
```

> 在使用`pipenv install mysqlclient`时报错, 本来以为是mysqlclient或者pipenv的问题, 反复尝试无果
>
> 又随便找了几个其他的包尝试安装, 包括`requests`,`pydoc`等
>
> 发现依然会出现同样的问题,但是像`django`等一些包就安装很顺利
>
> 最后找到一些解决方案, 需要安装一个名为`pkg-connfig`的包,
>
> ```zsh
> sudo apt-get install pkg-config
> ```
>
> 但是Mac 并不支持`apt-get`
>
> 于是只能换一种思路, 使用`brew`来安装`pkg-config`
>
> 经尝试
>
> ```zsh
> brew install pkg-config
> ```
>
> 运行完成之后`mysqlclient`便可以通过pipenv正常安装了
>
> > PS: brew安装:
> >
> > ```bash
> > /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
> > ```
>

## 修改settings.py配置文件

vscode快捷键: `cmd+T`/ `cmd+shift+O`

修改`DATABASES`设置

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'storefront',
        'HOST': 'localhost',
        "USER": 'root',
        "PASSWORD": '000000'
    }
}
```

## 运行自定义SQL

1. 生成一个空的migration文件:

   ```dash
   # sotre是子app的名字
   > python manage.py makemigrations store --empty
   
   Migrations for 'store':
     store/migrations/0002_auto_20231107_0610.py
   ```

2. 可以修改该文件的名字, 如`0002_insert_test_data.py`

3. 在文件中编辑SQL语句

   ```python
   class Migration(migrations.Migration):
   
       dependencies = [
           ('store', '0001_initial'),
       ]
   
       operations = [
            # 第一条SQL用于部署
            # 第二条SQL用于还原
            # 具体执行那一条根据python manage.py migrate来进行操作
           migrations.RunSQL("""
               INSERT INTO store_collection (title)
               VALUES ('collection1')
           """, """
               DELETE FROM store_collection
               WHERE title = 'collection1'
           """)
       ]
   
   ```

4. 部署SQL

   ```bash
   python manage.py migrate
   ```

5. 还原

   ```bash
   python manage.py migrate store 0001
   ```

## 添加测试数据

测试数据在线工具: [https://mockaroo.com](https://mockaroo.com)

在线编辑字段, 选择类型, 选择sql, 下载SQL文件

在datagrip中执行即可

或者使用`Faker`,`DataFactory`等python库进行数据生成
