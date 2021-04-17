# 2 数学描述



## 2.1 坐标系统

坐标系由文件`constant/g`中给出的重力向量的方向定义。这意味着垂直坐标可以是任何方向；理论上。只有沿y轴或z轴的重力矢量的定义得到了广泛的检验。

## 2.2 动量方程与连续性方程

动量和连续性方程遵循OpenFOAM中流体体积（VOF）类型模拟的标准实现。动量方程写在剩余压力中，剩余压力定义为超过静水压力的压力。动量方程如下：

$$\frac{\partial \rho \mathbf{u}}{\partial t}+\nabla \cdot \rho \mathbf{u u}^{T}=-\nabla p^{*}+\mathbf{g} \cdot\left(\mathbf{x}-\mathbf{x}_{r}\right) \nabla \rho+\nabla \cdot \mu_{t o t} \nabla \mathbf{u}$$

式中，$\rho$表示流体密度，$\mathbf{u}$是笛卡尔坐标系下的速度矢量，$t$是时间，$\nabla=(\partial / \partial x, \partial / \partial y, \partial / \partial z)$是微分算子，$p^*$代表剩余压力，$\mathbf{g}$是重力引起的加速度矢量，$\mathbf{x}=(x,y,z)$是笛卡尔坐标矢量。$\mathbf{x_r}$是参考位置（定义为海平面，见第3.1.2节），$\mu_{tot}$是总动力粘度，见第2.4节。

不可压缩连续性方程：

$$\nabla \cdot \mathbf{u}=0$$

剩余压力定义为$p^*=p-pg$，其中$p$是总压力。Rusche (2002)概述了离散化方程的求解过程。

### 2.2.1 针对渗透结构的修改

对于可渗透结构，如防波堤、冲刷防护、可渗透海底等，速度定义为过滤速度。过滤速度与通过孔隙率`n`的孔隙速度有关，如下所示：

$$\mathbf{u}=n\mathbf{u}_p$$

$\mathbf{u}_p$是孔隙速度。Jensen et al. (2014)介绍了渗透层校正实现的推导。连续性方程与式（2.2）相同，而动量方程采用以下修正形式：

$$\left(1+C_{m}\right) \frac{\partial}{\partial t} \frac{\rho \mathbf{u}}{n}+\frac{1}{n} \nabla \cdot \frac{\rho}{n} \mathbf{u} \mathbf{u}^{T}=-\nabla p^{*}+\mathbf{g} \cdot\left(\mathbf{x}-\mathbf{x}_{r}\right) \nabla \rho+\frac{1}{n} \nabla \cdot \mu_{t o t} \nabla \mathbf{u}-\mathbf{F}_{p}$$

动量方程的这种形式与Higuera et al. (2014)中提出的形式不同，然而，在随后的工作中，主要作者(Higuera, 2015) 赞同公式（2.4）中的公式。第2.8节讨论了$C_m$和$\mathbf{F}_p$的取值。连续性方程仍然由式（2.2）给出。

该建模框架已成功应用于研究波浪与透水结构之间的相互作用(Jensen et al., 2014; Jacobsen et al., 2015; Van Gent et al., 2015)。这些工作验证了反射、波浪耗散、波浪漫顶和波浪引起的压力以及渗透结构内部的设置等数量。

## 2.3 指定场对流(Advection of the indicator field)

VOF场的对流F如下：

$$\frac{\partial F}{\partial t}+\nabla \cdot \mathbf{u} F+\nabla \cdot \mathbf{u}_{r}(1-F) F=0$$

式中，$\mathbf{u}_r$为相对速度，详见Berberovi¢ et al. (2009)。F用于评估密度和粘度的空间变化，如下所示：

$$\rho=F \rho_{1}+(1-F) \rho_{0} \quad$ and $\quad \mu=F \mu_{1}+(1-F) \mu_{0}$$

下标0表示$F=0$对应的材料性质，下标1表示$F=1$对应的材料性质。其中 `waves2Foam`定义$F=1$表示水，$F=0$表示空气。

### 2.3.1 针对渗透结构的修改

可渗透结构的存在意味着网格中的流体量仅限于可渗透材料之间的空隙。仍然需要有$F \in[0,1]$，因为平流方程有效地进行了这项工作：

$$\frac{\partial F}{\partial t}+\frac{1}{n}\left(\nabla \cdot \mathbf{u} F+\nabla \cdot \mathbf{u}_{r}(1-F) F\right)=0$$

n是孔隙率。

## 2.4 湍流模型

waves2Foam中未分发湍流模型。OpenFOAM各种分发中有大量湍流模型可用，但请注意文献中给出的建议（如Mayer and Madsen, 2000; Jacobsen et al., 2012; Brown et al., 2016; Zhou et al., 2017; Devolder et al., 2017）。

## 2.5 边界反射补偿

已知的技术可以直接在边界处进行反射补偿 (Wellens, 2012; Higuera, 2015)，但这种方法不适用于waves2Foam，后者是围绕松弛区的使用而发展起来的。

## 2.6 数值海滩

waves2Foam是为实现数字海滩而准备的。这一实施从未完成。有些教程在松弛区的定义中包含关键字`beachType`，为什么不需要该关键因为为其指定了默认值。

## 2.7 松弛区技术

弛豫区技术旨在消除数值模拟中的假反射。松弛区技术基于速度场的计算解和带有目标解的指示场之间的加权。松弛区技术分为显式松弛和隐式松弛两种，其中显式/隐式松弛是指时间积分。

此外，还对计算域中松弛区的权重和位置进行了描述。

### 2.7.1 松弛区特点

#### 2.7.1.1 显示松弛

松弛区的显式方法如下简单地给出：

$$\phi=\left(1-w_{R}\right) \phi_{\text {target }}+w_{R} \phi_{\text {computed }}$$

这里，权重函数$w \in{0,1}$可以用各种方式来定义，如下节所述。

在求解压力-速度耦合之前，该方法根据公式（2.8）明确校正每个时间步的$\alpha$和$\mathbf{u}$场。该方法最初在Jacobsen et al.（2012）中针对OpenFoam进行了描述，但对于Boussinesq型波建模或全拉普拉斯问题的解决方案来说，这是一种相当常见的方法（参见Fuhrman et al.，2006；Engsig-Karup et al.，2009）。

#### 2.7.1.2 隐式松弛

Vukcevic et al. (2016a,b)提出了一种隐式松弛区实现方法，尽管该方法目前在waves2Foam中不可用。

### 2.7.2 松弛区的指定

松弛区在`waveProperties.input`文件里子词典中指定，请参阅第3.1.2节了解更多详细信息。松弛区的定义见代码片段2.1。以下各节分别对这些控件进行了描述。

代码片段2.1：松弛区的基本定义

```c++
relaxationZone
{
	relaxationScheme 	<word>;
	relaxationWeight 	<word>;
	relaxationShape 	<word>;
	courantCorrection 	<bool>; // Default: false
}
```

#### 2.7.2.1 松弛方案

松弛方案本质上是一种方式，松弛是已实现了的。

**relaxationSchemeEmpty**  这种方法有效地扭转了松弛（代码片段2.2）。

代码片段2.2：Empty松弛方案

```c++
relaxationZone
{
	relaxationScheme 	Empty;
}
```

