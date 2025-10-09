---

title: anaconda

---
<h1>Anaconda</h1>

***

* 查看已安装的环境

```PYTHON
conda env list
```

***

* 创建虚拟环境

```python
conda creat -n my_env python==3.10 moudule_name
```

> 创建名为myenv的虚拟环境并且python版本3.10 下载pandas第三方库

```python
conda creat -n my_env python==3.10 #大多数时候这个就可以
conda creat -n my_env python==3.10 -y #-y就是所有的y/n选项都填yes
```

***

* 激活虚拟环境

```python
conda activate name_of_env
```

***

* 退出虚拟环境

```python
conda deactivate
```

***

* 分享开发使用的环境

```python
conda env export > environment.yaml
```

利用yaml文件创建环境

```python
conda env create -f environment.yaml
```



***

* 删除一个环境及其安装包

```python
conda remove --name myenv --all
```

***

* conda管理包

```python
conda list
```

> 查看指定环境中的包

```python
conda list -n env_name
```

***

* 在库中搜索包

```python
conda search numpy
conda search mathplotlib
conda search numpy==1.12
```

安装包到环境

```python
conda install numpy
```

安装包到指定环境

```python
conda install -n name_env numpy
```

同时安装多个包到当前环境

```python
conda install numpy mathplotlib
```

安装指定版本包到指定环境

```python
conda install -n my_env numpy==1.2.2
```

安装包更新

```python
conda update numpy
```

移除安装包

```python
conda remove -n env numpy
```



