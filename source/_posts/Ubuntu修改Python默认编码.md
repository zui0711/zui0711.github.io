---
title: Ubuntu修改Python默认编码
id: 80
categories:
  - 技
date: 2016-11-13 15:17:58
tags:
---

在安装Python包或者进行文件读取等操作的时候，可能会出现类似的编码问题：
```shell
UnicodeDecodeError: ‘ascii’ codec can’t decode byte 0xe5 in position 108: ordinal not in range(128)
```

在cmd中进入Python环境，输入：

```python
import sys
sys.getdefaultencoding()
```

得到：

```shell
'ascii'
```

可以看见Python默认编码格式为ascii，当读入的文字在ascii字符集之外时就会出现这样的问题，此时需要修改Python默认编码，改为utf-8。下面具体讲解。

### **查找site-packages目录**

在cmd中进入Python环境，输入：

```python
import site
site.getsitepackages()
```

得到：

```shell
['/usr/local/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages']
```

具体哪一个为目录，我不是很确定，大概跟安装Python的方法有关，我建议两个都建个sitecustomize.py试试。

### **建立sitecustomize.py文件**

在cmd进入上面得到的文件夹下，建立sitecustomize.py文件并输入（注意启用root权限）：

```python
import sys
sys.setdefaultencoding('utf-8')
```

### **测试**

在cmd中重新进入Python环境，输入：

```python
import sys
sys.getdefaultencoding()
```

得到：

```shell
'utf8'
```

可以看到Python默认编码已经改为utf-8。