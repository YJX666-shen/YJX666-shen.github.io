---

title: Packmol

---
<h1 align="center">Packmol</h1>

### 一、**packmol基础定义**

```mermaid
graph LR
A[用户分子的结构文件，设定约束条件] --> B[各种分子按照设定将指定数目的分子堆积到满足要求的区域中]

```

> [!Note]
>
> 1.程序会尝试各种优化算法，尝试不同的堆积结构
>
> 2.收敛至**原子没有不合理接触**且能**满足所有约束条件**的结构
>
> 3.堆积过程中分子结构**保持刚性**，构象不会变化
>
> 4.Packmol在堆积过程中只考虑空间因素，**完全不考虑能量，电荷分布等问题**



1. 作者在官网提到，用户只需提供每种类型单个分子的坐标、各类型分子的数量，以及每种分子类型必须满足的空间约束条件。
2. 该软件包兼容 PDB、TINKER 和 XYZ 格式的输入文件。



### 二、Packmol在WIN下载方式

**使用julia封装器**

* 首先从https://julialang.org/downloads/下载Julia，支持WIN/LINUX/MAC，可以按照具体需求去官网下载
* 官网:[Packmol - 分子动力学初始构型 --- Packmol - Initial configurations for Molecular Dynamics](https://m3g.github.io/packmol/download.shtml)

![Julia](C:\Users\HP\Desktop\Julia.png)

* 打开Julia并输入`import Pkg; Pkg.add("Packmol")`

![输入程序后等待即可](C:\Users\HP\Desktop\输入程序后等待即可.png)

![安装完成](C:\Users\HP\Desktop\安装完成.png)

install successfully！:smile:

* 在Julia中输入`using Packmol; run_packmol()`来运行Packmol

* 每次运行复制粘贴上述代码即可

> [!Important]
>
> 这将打开一个图形用户界面，可以通过它选择Packmol要使用的输入文件
>
> 注意：每次运行Packmol都要输入`using Packmol; run_packmol()`



###  三、**input文件参数说明**

* #### <font color = orange>**距离容差 tolerance**</font>

> 作者声明：The minimal input file must contain the distance tolerance required (for systems at room temperature and pressure and coordinates in Angstroms, 2.0 Å is a good value [[note\]](https://m3g.github.io/packmol/tolerance-note.shtml)).
>
> 最简可运行文件第一行：tolerance = 2.0
>
> 即：在常温常压下，且坐标为埃的体系中，推荐使用2.0作为参数值，这点同样可以在文献中得到验证：



![文献关于2.0的记载](C:\Users\HP\Desktop\文献关于2.0的记载.png)

![推荐值2.0](C:\Users\HP\Desktop\推荐值2.0.png)

* #### <font color = orange>**声明文件类型filetype**</font>

> input文件需要声明输入文件的类型：
>
> 针对不同的输入形式，直接filetype your_file_type即可

![filetype](C:\Users\HP\Desktop\filetype.png)



* #### <font color = orange>**指定待生成文件output**</font>

> `output test.pdb`这是最简可运行输入文件的第三行，往往也是大多数inp文件的第三行
>
> 所有读到这里，开头的前三行更像是表头，每个input文件都要包含这三行，也就是全局设置

> [!IMPORTANT]
>
> tolerance: the number suit your system(type:float)
>
> filetype: pdb，tinker or xyz
>
> output the_file_name（默认输出到与input文件路径相同的文件夹下）

![第几行第几行](C:\Users\HP\Desktop\第几行第几行.png)

以下是官网说明：

![output](C:\Users\HP\Desktop\output.png)



* #### <font color = orange>**添加分子**</font>

> [!NOTE]
>
> 在input文件中，向体系空间中添加分子是以一个一个下面的模块填充分子的：

```python
structure your_molecule
  number atom_num
  inside cube initial_coordinates(examples:0. 0. 0.) final_coordinates(examples:40.)
end structure
```

> [!tip]
>
> **inside cube**用法参见约束参数设置

> [!WARNING]
>
> 坐标的填充要注意，每写一个坐标值加一个'.'然后空格，也就是起始坐标（0，0，0）输入到Packmol里就是
>
> ```python
> 0. 0. 0.
> ```
>
> 起始坐标和结束坐标空格
>
> 结束坐标也要遵循上述规则（40，40，40）
>
> ```python
> 40. 40. 40
> ```
>
> 

如何理解这个一个一个模块填充：

![填充模块](C:\Users\HP\Desktop\填充模块.png)

可以看到每次填充分子，作者都是使用`structure molenule.pdb···end structure`模块



* #### <font color = orange>**添加分子扩展功能atom**</font>

**扩展功能——atom用法**

atom嵌入在添加分子模块的中间：

![atom](C:\Users\HP\Desktop\atom.png)

它的作用是将指定的原子固定在用户期望的位置，这上述示意中，它的作用就是将分子的9号和10号原子固定在（0，0，15）和（20，20，20）处，这个功能在构建囊泡时非常有用



* #### <font color = orange>**约束参数设置**</font>

> [!TIP]
>
> 以下所有用法全部作用与structure test.pdb...end structure中间
>
> ```python		
> structure test.pdb
>     number = 1
>     function num1. num2. num3. num4. num5. num6.
> end structure
> ```

##### **<font color = green>1. fixed</font>**

用法：fixed x y z a b c

前三个参数xyz代表  **分子相对于坐标文件中位置的平移量**

后三个参数abc代表  **旋转角度**

由于使用fixed时只能设置一个分子，可以的搭配`center`使用，若存在‘center'关键字，则前三个数字代表**质心位置**

```python
structure test.pdb
    number = 1
    center
    fixed 0. 0. 0. 0. 0. 0.
end structure
```

这表示：分子被固定在其中心位于原点并且不发生旋转的位置



##### **<font color = green>2. inside cube</font>**

用法：inside cube x<sub>min</sub>y<sub>min</sub>z<sub>min</sub>d

 x<sub>min</sub>y<sub>min</sub>z<sub>min</sub>d是四个实数，受此选项约束的原子坐标满足：

```python
xmin < x < xmin + d
ymin < y < ymin + d
zmin < z < zmin + d
```



##### **<font color = green>3. outside cube</font>** 

用法：outside cube x<sub>min</sub>y<sub>min</sub>z<sub>min</sub>d

x<sub>min</sub>y<sub>min</sub>z<sub>min</sub>d同样是四个实数，受此约束的原子坐标满足：

```python
x < xmin or x > xmin + d
y < ymin or y > ymin + d
z < zmin or z > zmin + d
```



##### **<font color = green>4. inside box</font>**

用法：inside box x<sub>min</sub> y<sub>min</sub> z<sub>min</sub> x<sub>max</sub> y<sub>max</sub> z<sub>max</sub>

六个实数，受此约束的原子坐标满足：

```python	
xmin < x < xmax
ymin < y < ymax
zmin < z < zmax
```



##### **<font color = green>5. outside box</font>**

用法：outside box x<sub>min</sub> y<sub>min</sub> z<sub>min</sub> x<sub>max</sub> y<sub>max</sub> z<sub>max</sub>

六个实数，受约束的原子坐标满足：

```python
x < xmin or x > xmax
y < ymin or y > ymax
z < zmin or z > zmax
```



**<font color = green>6. inside(or outside) sphere</font>**

球体的定义方程：
$$
(
x
−
a
)
2
+
(
y
−
b
)
2
+
(
z
−
c
)
2
−
d
2
=
0
$$
因此想要设置球体的分子吸附约束条件，需要四个参数a,b,c,d

```python
inside sphere 2.30 3.40 4.50 8.0
```

他代表原子将会满足：
$$
(
x
−
2.30
)
2
+
(
y
−
3.40
)
2
+
(
z
−
4.50
)
2
−
8.0
2
≤
0.0
$$


> [!Note]
>
> `outside sphere a b c d`用法和inside类似，但是它的原子是存在与球体之外的



**<font color = green>7. above(or below) plane</font>**

平面的定义式：
$$
ax+by+cz-d=0
$$
可以限制原子位于平面上方或者下方，具体用法为：

```python
above plane 1 2 3 4
below plane 1 2 3 4
```

其中above plane会使得原子满足：
$$

x
+
2
y
+
3
z
−
4
≥
0
$$
below plane约束原子坐标满足：
$$

x
+
2
y
+
3
z
−
4
≤
0
$$
**<font color = green>8. Constrain rorations</font>**

用法：constrain_roration x 180. 20.

这代表这该原子围绕X轴旋转的角度被限制在180±20°范围之内

> [!Note]
>
> 如果想要使用分子与某个轴平行排列，需要约束相对与另外两个轴的旋转
>
> 比如，想要限制分子沿Z轴的旋转，可以使用以下指令：
>
> ```python
> constrain_rotation x 0. 20.
> constrain_rotation y 0. 20.
> ```
>
> 这些旋转是相对与用户输入的分子的初始朝向，因此，建议预先将分子调整至合理朝向



**<font color = green>9. 周期性边界条件</font>**

如果用户想要设置一个30* 30 * 60埃的周期性边界盒子,那么可以使用：

`pbc 30. 30. 60.`

这种情况下，所有分子都将被限制在又最小坐标和最大坐标（0，0，0）（30，30，60）定义的周期性盒子之内，但是计算原子之间的距离时会在边界处应用周期性边界条件



如果提供六个参数，那么周期性边界条件将会应用于用户给出的最小坐标和最大坐标处。

> [!IMPORTANT]
>
> 该参数为全局参数，他将会作用于除fixed固定住的分子之外的所有分子



### 四、**参考网站**

本次写入了大部分常见的功能，如需更多功能比如圆柱体限制、不同的原子半径不同、自动溶剂化大分子等等可以参考下面网站：

[Packmol - 分子动力学初始构型 --- Packmol - Initial configurations for Molecular Dynamics](https://m3g.github.io/packmol/userguide.shtml)

这里介绍了所有限制条件的设置。



其次，计算化学公社sobereva同样总计了大量参数设置，网址：[分子动力学初始结构构建程序Packmol的使用 - 分子模拟 (Molecular Modeling) - 计算化学公社](http://bbs.keinsci.com/thread-12549-1-1.html)

如图：

![sobereva1](C:\Users\HP\Desktop\sobereva1.png)

![sobereva2](C:\Users\HP\Desktop\sobereva2.png)

![sobereva3](C:\Users\HP\Desktop\sobereva3.png)

以及一些示例：

![示例](C:\Users\HP\Desktop\示例.png)



