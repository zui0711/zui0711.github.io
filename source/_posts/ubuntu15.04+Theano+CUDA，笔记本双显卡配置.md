---
title: ubuntu15.04+Theano+CUDA，笔记本双显卡配置
id: 8
categories:
  - 技
date: 2016-07-19 00:03:32
tags:
---

### **Theano安装**

在命令行输入

```shell
sudo apt-get update
sudo apt-get install python-numpy python-scipy python-dev python-pip python-nose g++ libopenblas-dev git
sudo pip install Theano
```

&nbsp;

测试Theano

NumPy (~30s):

```shell
python -c &quot;import numpy; numpy.test()&quot;
```

SciPy (~1m):

```shell
python -c &quot;import scipy; scipy.test()&quot;
```

Theano (~30m):

```shell
python -c &quot;import theano; theano.test()&quot;
```

如果没有error就说明Theano安装完成

&nbsp;

* * *

### **CUDA安装**

在命令行输入

```shell
sudo apt-get install nvidia-352 nvidia-prime nvidia-settings
sudo apt-get install nvidia-cuda-toolkit
```

测试

```shell
nvidia-smi
```

出现

```shell
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
```

同时使用

```shell
nvidia-settings
```

出现

```shell
** Message: PRIME: No offloading required. Abort
** Message: PRIME: is it supported? no
```

在搜了很多解决方法后发现，原因是笔记本是uefi的主版，需要进入bios禁用secureboot功能，在禁用后nvidia-smi即可正常使用。这个奇葩问题来源：[http://forum.ubuntu.org.cn/viewtopic.php?f=42&amp;t=477917](http://forum.ubuntu.org.cn/viewtopic.php?f=42&amp;t=477917)

* * *

### **设置环境变量及测试**

在命令行输入

```shell
sudo gedit ~/.bashrc
```

在文件最后添加

```shell
export PATH = &quot;/usr/lib/nvidia-cuda-toolkit/bin&quot;:$PATH
export LD_LIBRARY_PATH = &quot;/usr/lib/nvidia-cuda-toolkit/lib&quot;:$LD_LIBRARY_PATH
```

在命令行输入

```shell
sudo gedit ~/.theanorc
```

在文件中写入

```shell
[global]
device = gpu
floatX = float32

[nvcc]
fastmath = True

[cuda]
root = /usr/lib/nvidia-cuda-toolkit

[lib]
cnmem = 0.8

[blas]
ldflags = -lopenblas
```

测试blas加速能否正常工作

```shell
import numpy 
id(numpy.dot) == id(numpy.core.multiarray.dot) 
```

得到结果为False则说明正常加速

创建一个theano_test.py文件，写入

```pyhon
from theano import function, config, shared, sandbox
import theano.tensor as T
import numpy
import time

vlen = 10 * 30 * 768 # 10 x #cores x # threads per core
iters = 1000
rng = numpy.random.RandomState(22)
x = shared(numpy.asarray(rng.rand(vlen), config.floatX))
f = function([], T.exp(x))
print f.maker.fgraph.toposort()

t0 = time.time()
for i in xrange(iters):
    r = f()
t1 = time.time()

print 'Looping %d times took' % iters, t1 - t0, 'seconds'
print 'Result is', r
if numpy.any([isinstance(x.op, T.Elemwise) for x in f.maker.fgraph.toposort()]):
    print 'Used the cpu'
else:
    print 'Used the gpu'
```

&nbsp;

切换到theano_test.py文件所在目录，在命令行输入

```shell
python theano_test.py
```

出现

```shell
WARNING (theano.sandbox.cuda): CUDA is installed, but device gpu is not available  (error: Unable to get the number of gpus available: unknown error)
[Elemwise{exp,no_inplace}(&lt;TensorType(float32, vector)&gt;)]
Looping 1000 times took 4.377858 seconds
Result is [ 1.23178029  1.61879337  1.52278066 ...,  2.20771813  2.29967761 1.62323284]
Used the cpu
```

<div>

此时在命令行输入

```shell
sudo python theano_test.py
```

</div>
可正常运行，出现
<div>

```shell
Using gpu device 0: GeForce 840M (CNMeM is disabled, cuDNN not available)
[GpuElemwise{exp,no_inplace}(&lt;CudaNdarrayType(float32, vector)&gt;), HostFromGpu(GpuElemwise{exp,no_inplace}.0)]
Looping 1000 times took 1.110110 seconds
Result is [ 1.23178029 1.61879349 1.52278066 ..., 2.20771813 2.29967761 1.62323296]
Used the gpu
```

再次输入

```shell
python theano_test.py
```

出现错误（省略了部分语句）

```shell
OSError: [Errno 13] Permission denied: '/home/zui/.theano/compiledir_Linux...'
```

此时利用rm -rf删除文件夹.theano文件夹即可

再次输入

```shell
python theano_test.py
```

可以正常运行

</div>
之后每次重启，需要首先用root权限跑Theano代码（比如上面的theano_test.py），才能正常使用GPU，也可以选择建立一个脚本，具体参考[http://www.johnwittenauer.net/configuring-theano-for-high-performance-deep-learning/](http://www.johnwittenauer.net/configuring-theano-for-high-performance-deep-learning/)最后一点

&nbsp;

参考：

[http://deeplearning.net/software/theano/install.html](http://deeplearning.net/software/theano/install.html)

[http://deeplearning.net/software/theano/install_ubuntu.html](http://deeplearning.net/software/theano/install_ubuntu.html)

[l](http://deeplearning.net/software/theano/install.html)[http://www.cnblogs.com/Ponys/p/3438491.html](http://www.cnblogs.com/Ponys/p/3438491.html)

[http://www.johnwittenauer.net/configuring-theano-for-high-performance-deep-learning/](http://www.johnwittenauer.net/configuring-theano-for-high-performance-deep-learning/)

[https://groups.google.com/forum/#!topic/theano-users/_Zd4kdkYlcQ](https://groups.google.com/forum/#!topic/theano-users/_Zd4kdkYlcQ)

[http://forum.ubuntu.org.cn/viewtopic.php?f=42&amp;t=477917](http://forum.ubuntu.org.cn/viewtopic.php?f=42&amp;t=477917)

其他还搜了很多资料，不一一列举

Tip：

利用

```shell
lspci -k | grep -EA2 'VGA'
```

查看显卡时，由于nvidia有些显卡表示为“3D控制器”，所以需要使用

```shell
lspci -k | grep -EA2 'VGA|3D'
```
