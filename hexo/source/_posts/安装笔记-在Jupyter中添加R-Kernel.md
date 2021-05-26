title: '安装笔记: 在Jupyter中添加R Kernel'
date: 2015-08-16 21:23:41

categories:

- Tech

tags: 

- Python
- R
- Data processing

---

[Jupyter](https://jupyter.org/)(即原来的Ipython Notebook Project)基于Web端的实时交互，拥有着极佳的用户体验。同时，Jupyter体系可直接使用其他语言，用户呢能够在同一界面下调用R内核或Python内核(共支持49种)，功能十分强大。    
在Mac OS下尝试在Jupyter中添加R语言内核，遇到了一些问题，在此记录个人安装过程和诸问题的解决方法。

### STEP1 基本软件准备

* Python和IPython.作为初学者，我直接安装了Python的科学计算发行版[Anaconda](https://store.continuum.io/cshop/anaconda/)，其包含了Python2.7并内置很多有用的Package(numpy, SciPy, Matplotlib)，Jupyter(Ipython Notebook)以及Spyder(个人感觉类似于R studio的集成开发环境)
* R 3.2.1(World-Famous Astronaut)，[官网](https://cran.r-project.org/)下载安装即可。

### STEP2 有关库的安装
主要工作是为R语言添加[IRkernal](https://github.com/IRkernel/IRkernel)这一新的Package, 以实现Jupyter对R语言的支持。主要的步骤如下(依照IRkernal的README文档)：

<!--more-->

#### R console

``` R
## 安装开发工具库
install.packages("devtools")  
## 调用RCurl，可通过github下载有关文件
install.packages('RCurl')
library(devtools)
install_github('armstrtw/rzmq')
install_github("takluyver/IRdisplay")
install_github("takluyver/IRkernel")
IRkernel::installspec()
```


在安装rzmq时出现问题，导致后续步骤无法进行。    
参考网络资源，改用brew工具进行安装   
#### Terminal 

``` bash
$ brew install zmq 
```

下载成功但未能安装，利用`brew doctor`进行诊断         

结果显示，显示    
`/usr/local/lib/pkgconfig is not writable.`     

首先考虑的是MacPort可能存在干扰，将其暂时移除     

``` bash
$ sudo mv /opt/local /macports
```

pkgconfig进行重新安装关联

``` bash
$ sudo chown -R `whoami` /usr/local/lib/pkgconfig
$ brew uninstall pkgconfig  
$ brew install pkgconfig    
$ brew unlink pkg-config && brew link pkg-config 
$ brew link zmq 
```

#### 回到R Console 


``` R
install_github("takluyver/IRdisplay")
install_github("takluyver/IRkernel")
IRkernel::installspec()
```


最后一个步骤再次出现问题：  


```R
Error in IRkernel::installspec(): 
Jupyter or IPython 3.0 has to be installed but could neither run “jupyter” nor “ipython”, “ipython2” or “ipython3”.
(Note that “ipython2” is just IPython for Python 2, but still may be IPython 3.0)
```

安装包的位置出现问题，解决方法如下：
     
``` R
print(system.file("kernelspec", package = "IRkernel"))
```


复制该命令的输出结果，我的结果是：    
`/Library/Frameworks/R.framework/Versions/3.2/Resources/library/IRkernel/kernelspec`)


#### 再回到Terminal(够折腾的)

``` bash
ipython kernelspec install --replace --name ir --user /Library/Frameworks/R.framework/Versions/3.2/Resources/library/IRkernel/kernelspec   
```

__指令中的路径改为个人的输出结果即可__。

### STEP3 检验
Terminal中键入`ipython notebook`或`jupyter notebook`或在Anaconda图形界面下就可以打开Jupyter的网页，如果出现下图的选项，就说明顺利完成了!

[![enter image description here][1]][1]


[1]: https://i.stack.imgur.com/JfEej.png


### 使用中出现的其他问题

使用时突然出现，R kernel 无法连接的问题

jupyter-client has to be installed but “jupyter kernelspec --version” exited with code 127.

此种情况，执行下述命令，可修复
	
```bash
## 换成自己的用户名路径	
sudo ln -s /Users/HYF/anaconda/bin/jupyter /usr/bin/jupyter
```


### 参考

1. [Native R kernel for Jupyter](https://github.com/IRkernel/IRkernel)    
2. [Multi-Language IPython (Jupyter) setup](http://ihrke.github.io/jupyter.html)