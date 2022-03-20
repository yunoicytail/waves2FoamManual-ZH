# 1 简介

## 1.1 来源

waves2Foam工具箱最初由Niels Gjøl Jacobsen在Jørgen Fredsøe教授的指导下在丹麦技术大学开发。渗透模块由Bjarne Jensen 与 Niels Gjøl Jacobsen共同开发。

Niels Gjøl Jacobsen受雇于荷兰德尔塔雷斯（Deltares）（2013至今），在德尔塔雷斯进行维护和开发。

## 1.2 背景与应用

waves2Foam 工具箱是CFD开源软件包OpenFOAM[1,2]的插件。

waves2Foam 在GNU-GPL下开源。waves2Foam最初仅作为自由表面波的生成与吸收工具箱于2011年11月发布(Jacobsen et al., 2012)。2014年，工具箱扩展了自由表面波与可渗透介质（如防波堤、冲刷防护等）之间相互作用的建模可能性(Jensen et al., 2014)。

waves2Foam工具箱在大量文献中被广泛使用，其范围从工具箱以及OpenFOAM的自由表面能力的验证到应用。waves2Foam的使用示例如下：

	1. Jacobsen et al. (2014); Jacobsen and Fredsøe (2014a,b) 应用该模型研究了破碎波作用下两岸形态的演化。这涉及到水动力学、泥沙输移和由此产生的河床水位变化之间的耦合。
	2. Paulsen et al. (2014a,b) 使用此工具箱研究了单桩上的波浪载荷。Paulsen et al. (2014a) 的工作还包括waves2Foam与完全非线性波问题的解之间的耦合，即具有非线性边界条件的拉普拉斯方程；更多细节见第4.3.3节。
	3. 月池内水体共振高程(Moradi, 2015)。
	4. 桥面波浪载荷的研究，并与实验数据进行的对比(Seiffert, 2014; Seiffert et al., 2014; Hayatdavoodi et al., 2014).
	5. Jacobsen et al. (2015)通过常规不规则入射波对waves2Foam进行了大量物理实验验证。结果表明waves2Foam中波浪生成与吸收的松弛区方法在不增加水位的情况下补偿了漂移引起的附加质量通量。
	6. Stahlmann (2013)将waves2Foam中的波浪生成与三脚架钢结构周围沉积物床变形的方法耦合起来；目的是评估这种结构对冲刷模式的影响。
	7. Elsafti和Oumeraci（2013）将waves2Foam与岩土模型耦合，以研究沉箱防波堤下的残余孔隙压力累积和耗散。

应用的列表在稳步增加中，最新的应用可在[google学术](https://scholar.google.com)或[ResearchGate](www.researchgate.net)上查看。

## 1.3 兼容性

### 1.3.1 分支

OpenFOAM有三个主要的分支：

* OpenFOAM(OF)通过www.openfoam.org分发；
* OpenFOAM(OF+)通过www.openfoam.com分发；
* foam-extend(FE)通过foam-extend社区发布。

本教程中使用括号中的简写。

### 1.3.2 命名术语

OpenFOAM的所有三个分支及许多版本均支持waves2Foam。多年来，各个分支均修改了命名术语。例如，孔隙率F在某些版本中称为alpha1,在其他版本中称为alpha.water。类似地，在某些版本中，剩余压力被称为pd，在其他版本中为p_rgh。waves2Foam通过类<waves2Foam_src>/waves2Foam/include/crossVersionCompatibility.[C,H]来支持这些差异。

### 1.3.3 支持版本

waves2Foam在以下版本中均可编译，但请注意警告1.1.

* OF-3.0，OF-4.0
* OF+-3.0，OF+-1606，OF+-1612
* FE-3.1, FE-3.2, FE-4.0 

> <font color=red>警告1.1: 数值浪高仪不能在最新版的OpenFOAM版本(OF和OF+)上编译。液面取样工具（数值浪高仪）是OpenFOAM模拟自由表面波的重要部分。由于代码重组，浪高仪不适用于OF-4.0（及更新版本）和OF+-1612（及更新版本）。这个问题将在未来更新中解决。（译者注：OF+-1612以上版本问题已解决）</font>

## 1.4 安装

### 1.4.1 第三方依赖

安装waves2Foam需要以下第三方依赖项。除了编译OpenFoam本身所需的依赖项之外，还需要这些依赖项。

* gfortran：用于编译OceanWave3D和Fenton代码（用于流函数）。
* git是访问OceanWave3D源代码所必需的。
* subversion（SVN）是访问waves2Foam源代码所必需的。
* GNU Scientific Library (GSL)用于编译waves2Foam。GSL提供了OpenFoam不支持的各种数学函数。

有关这些第三方软件包的安装，请参阅Linux/UNIX版本上的软件包管理系统。如果缺少这些包中的任何一个，则编译waves2Foam失败。

### 1.4.2 下载

下载和安装说明针对默认安装位置（代码片段1.1）。如果安装在非默认位置，请参阅第1.4.3节了解$./Allwmake之前的其他操作。

<center>代码片段1.1：默认安装目录的下载和安装说明（$号表示命令）。</center>

``` shell
## First, source your OpenFoam version
$ mkdir -p $FOAM_RUN/../applications/utilities
$ cd $FOAM_RUN/../applications/utilities
svn co http://svn.code.sf.net/p/openfoam-extend/svn/trunk/Breeder_1.6/other/waves2Foam
$ cd waves2Foam
$ ./Allwmake
```

### 1.4.3 `bashrc`文件

waves2Foam默认给出一个控制文件`bashrc.org`，位于目录`bin`中。在默认安装的情况下，此文件将自动复制到bin/bashrc。该文件控制到GNU Scientific库的链接以及库和可执行文件的目标目录。
如果需要非标准安装，则可以修改以下用户定义变量集：

* **WAVES_DIR**: 这是waves2Foam安装的完整路径。
* **WAVES_APPBIN**: 这是可执行文件的位置。默认值是用户目录（$FOAM_USER_APPBIN），但另一个选项是$FOAM_APPBIN，这对于多用户的群集安装非常有用。
* **WAVES_LIBBIN**: 这是库的位置。默认值是用户目录（$FOAM_USER_LINBIN），但另一个选项是$FOAM_LIBBIN，这对于多用户的群集安装非常有用。
* **WAVES_GSL_INCLUDE**:  GNU Scientific library的头文件（GSL在编译期间追加）。
* **WAVES_GSL_LIB**: GNU Scientific library的路径。

建议不要直接修改`bashrc.org`文件。复制并修改此文件。修改`bashrc.org`文件后更新SVN存储库时可能会导致问题。

### 1.4.4 如何获得waves2Foam的更新

对waves2Foam的更新是通过更新SVN存储库获得的，然后是重新编译。
代码片段1.2概述了这一点。

<center>代码片段1.2：更新SVN存储库并重新编译waves2Foam：</center>

```shell
$ svn update
$ ./Allwmake
```

SVN存储库的更新通常通过CFD在线发布在名为“Release of aWave Generation and Absorption Toolbox for OF”的线程中。最简单的方法是订阅线程并接收电子邮件通知。

附录C中详细说明了waves2Foam的所有修订。

> Nota Bene 1.1：更新后的bashrc
> 请注意文件`bashrc.org`文件可能会更新新版本的waves2Foam。如果发生这种情况，建议使用最新版本的`bashrc.org`文件，参见第1.4.3节中的说明。

### 1.4.5 如何重新编译部分waves2Foam

有时，如果进行了修改或添加，则需要重新编译部分代码。由于`bin/bashrc`包含了环境变量的临时定义，因此不可能只重新编译部分代码。因此，waves2Foam应该始终使用`Allwmake`脚本进行编译。

## 1.5 引用

请用户通过提供支持各种版本的学术工作的适当引用来支持waves2Foam工具箱中的工作：

waves2Foam的任何使用：请引用代码片段1.3中规定的原始工作(Jacobsen et al., 2012)。

><center>代码片段1.3：参考waves2Foam发布时附带的原始文件:</center>
>
>@article{jacobsenFuhrmanFredsoe2012,
>Author = {Jacobsen, N G and Fuhrman, D R and Freds\o{}e, J},
>Title = {{A Wave Generation Toolbox for the Open-Source CFD Library:
>OpenFoam\textregistered}},
>Journal = {{International Journal for Numerical Methods in Fluids}},
>Year = {{2012}},
>Volume = {{70}},
>Number = {{9}},
>Pages = {{1073-1088}},
>DOI = {{10.1002/fld.2726}},
>}

孔隙度模块的使用：请引用代码片段1.4中规定的渗透结构过滤速度(Jensen et al., 2014)的Navier-Stokes方程的推导。

> <center>代码片段1.4：参考waves2Foam中孔隙度实现的论文</center>
>
> @article{jensenJacobsenChristensen2014,
> Author = {Jensen, B and Jacobsen, N G and Christensen, E D},
> Title = {{Investigations on the porous media equations and resistance
> coefficients for coastal structures}},
> Journal = {{Coastal Engineering}},
> Year = {{2014}},
> Volume = {{84}},
> Pages = {{56-72}},
> }

与OceanWave3D的耦合：当使用waves2Foam和OceanWave3D之间的耦合时，请引用Paulsen et al. (2014a)了解耦合（代码片段1.5），引用Engsig-Karup et al. (2009)了解OceanWave3D的开发（代码片段1.6）。
耦合后来被简化到waves2Foam框架中，并作为外部源安装到框架中，见第4.2.8节）。

> <center>代码片段1.5：引用waves2Foam和Ocean-Wave3D之间的耦合。</center>
> @article{ paulsenBredmoseBingham2014,
> Author = {Paulsen, B. T. and Bredmose, H. and Bingham, H. B.},
> Title = {{An efficient domain decomposition strategy for wave loads on
> surface piercing circular cylinders}},
> Journal = {{Coastal Engineering}},
> Year = {{2014}},
> Volume = {{86}},
> Pages = {{57-76}},
> }

><center>代码片段t 1.6: 引用 OceanWave3D.</center>
>@article{ engsigKarupBinghamLindberg2009,
>Author = {Engsig-Karup, A. P. and Bingham, H. B. and Lindberg, O.},
>Title = {{An efficient flexible-order model for 3D nonlinear water waves}},
>Journal = {{Journal of Computational Physics}},
>Year = {{2009}},
>Volume = {{228}},
>Number = {{6}},
>Pages = {{2100-2118}},
>DOI = {{10.1016/j.jcp.2008.11.028}},
>}