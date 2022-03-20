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

$$\rho=F \rho_{1}+(1-F) \rho_{0} \quad$$ and $$\quad \mu=F \mu_{1}+(1-F) \mu_{0}$$

下标0表示$F=0$对应的材料性质，下标1表示$F=1$对应的材料性质。其中 `waves2Foam`定义$F=1$表示水，$F=0$表示空气。

### 2.3.1 针对渗透结构的修改

可渗透结构的存在意味着网格中的流体量仅限于可渗透材料之间的空隙。仍然需要有$F \in[0,1]$，因为平流方程有效地进行了这项工作：

$$\frac{\partial F}{\partial t}+\frac{1}{n}\left(\nabla \cdot \mathbf{u} F+\nabla \cdot \mathbf{u}_{r}(1-F) F\right)=0$$

$n$是孔隙率。

## 2.4 湍流模型

waves2Foam中未分发湍流模型。OpenFOAM各种分发中有大量湍流模型可用，但请注意文献中给出的建议（如Mayer and Madsen, 2000; Jacobsen et al., 2012; Brown et al., 2016; Zhou et al., 2017; Devolder et al., 2017）。

## 2.5 边界反射补偿

已知的技术可以直接在边界处进行反射补偿 (Wellens, 2012; Higuera, 2015)，但这种方法不适用于waves2Foam，后者是围绕松弛区的使用而发展起来的。

## 2.6 数值海滩

waves2Foam是为实现数字海滩而准备的。这一实施从未完成。有些教程在松弛区的定义中包含关键字`beachType`，为什么不需要该关键因为为其指定了默认值。

## 2.7 松弛区技术

松弛区技术旨在消除数值模拟中的假反射。松弛区技术基于速度场的计算解和带有目标解的指示场之间的加权。松弛区技术分为显式松弛和隐式松弛两种，其中显式/隐式松弛是指时间积分。

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

<center>代码片段2.1：松弛区的基本定义</center>

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

**relaxationScheme Empty**  这种方法有效地实现了松弛（代码片段2.2）。

<center>代码片段2.2：Empty松弛方案</center>

```c++
relaxationZone
{
	relaxationScheme 	Empty;
}
```

**relaxationScheme Spatial** 空间松弛方案根据公式（2.8）（代码片段2.3）进行松弛。

<center>代码片段2.3：空间松弛方案</center>

```c++
relaxationZone
{
	relaxationScheme 	Spatial;
}
```

#### 2.7.2.2 松弛权重

有三种松弛权值可用，每种权值都可以用全局Courant数进行修正。提供以下权值：

- 指数权重（默认）
- 自由多项式权重
- 三阶多项式权重

这些方法产生一个权重$w_R$，可以根据局部`Courant`数进行校正，如下所示：

$$\tilde{w}_{R}=1-\left(1-w_{R}^{*}\right)^{\mathrm{Co} / \mathrm{Co}_{\max }}$$

其中$w_{R}^{*}=1-w_R$与$w_R$在式2.8中使用。$\mathrm{Co}$是局部Courant数，$\mathrm{Co}_{\max }$是最大Courant数。该方法是根据Seng（2012）的工作实施的。通过在松弛区定义中添加关键字`courantCorrection`激活该效果，参见代码片段2.1。

在所有情况下，权重都是松弛区局部坐标系的函数，因此$w_R (\sigma=1 )=0，w_R (\sigma=0)=1$。这里，$\sigma$是松弛区内的局部坐标，坐标取决于松弛区的形状。

**指数权重**：指数权重分布取自Fuhrman et al. (2006)，其形式如下：

$$w_{R}=1-\frac{\exp \sigma^{p}-1}{\exp 1-1}$$

默认情况下，指数$p$设置为3.5。该方法是默认的，不需要指定它，但是可以在代码片段2.4中显式地包含它：

<center>代码片段2.4：指数权重函数</center>

```c++
relaxationWeight 	Exponential; // Default
exponent 			<scalar>; // Default value = 3.5
```

**自由多项式权重**：根据Engsig-Karup (2006)中的自由多项式权重，权重函数如下所示：

$$w_R=1-\sigma^p$$

其中$p$是整数指数。权重可以在代码片段2.5中指定。指数$p$没有默认值。

<center>代码片段2.5：自由多项式权函数</center>

```c++
relaxationWeight 	FreePolynomial;
exponent 			<scalar>;
```

**三阶多项式权重**：根据Engsig-Karup (2006)中的三阶多项式权重，权重函数如下所示：

$$w_{R}=-2 \tilde{\sigma}^{3}+3 \tilde{\sigma}^{2}$$

其中$\tilde{\sigma}=1-\sigma$. 权重可以在代码片段2.6中指定

<center>代码片段2.6：三阶多项式权函数</center>

```c++
relaxationWeight 	ThirdOrderPolynomial;
```

#### 2.7.2.3 松弛区形状

松弛形状指定在计算域中应用某个松弛区的位置。目前有四种不同的放松形态可供选择。这些是：

- 矩形；
- 半圆柱形；
- 圆柱形；
- 冻结

如果网格正在更改（移动或网格细化），则所有松弛区域都会在每个时间步更新受影响的单元索引。例外情况是冻结松弛区，见下文。每一项的设置如下。在所有情况下，垂直轴都是基于重力方向定义的。

**矩形**：矩形形状是最有用的形状，因为它同样适用于二维和三维模拟。它是基于矩形的两个对角点的坐标和一个侧面的方向（以及松弛方向）来定义的。松弛形状的格式在代码片段2.7中给出

<center>代码片段2.7：矩形松弛形状</center>

```c++
relaxationShape 	Rectangular;
relaxType 			<word>;
startX 				<point>;
endX 				<point>;
orientation 		<vector>;
```

关键字`relaxType`采用两个词中的任意一个，即INLET和OUTLET。前者用于定义矢量方向的方向描述弛豫方向，而出口的使用意味着弛豫方向与方向规定的方向相反，详见第3.1.3节。
所有点必须在同一水平面上。

**半圆柱形**：可以根据代码片段2.8中的规范定义半圆柱形。这种方法也许不是特别有用，但它展示了如何定义可选形状的示例。

<center>代码片段2.8：半圆柱形松弛形状</center>

```c++
relaxationShape 	SemiCylindrical;
centre 				<point>;
rInner 				<scalar>;
rOuter 				<scalar>;
zeroAngleDirection 	<vector>;
angleStart 			<scalar>; // [degrees]
angleEnd 			<scalar>; // [degrees]
```

**圆柱形**：圆柱形松弛形状定义了一个具有内部和外部辐射的松弛区（代码片段2.9）。目标解在圆柱形松弛区的外缘处强制执行。到目前为止，这种松弛区的唯一用途似乎是Arrighi et al. (2015)的工作。

<center>代码片段2.9：圆柱形松弛形状</center>

```c++
relaxationShape 	Cylindrical;
centre 				<point>;
rInner 				<scalar>;
rOuter 				<scalar>;
```

**冻结**：冻结松弛形状在模仿任何其他松弛形状的方式上是特殊的，除了一个方面：*如果计算网格正在移动，它不会更新松弛区域的单元索引*。该方法不适用于具有拓扑变化的网格。松弛形状的定义见代码片段2.10

<center>代码片段2.10：冻结型松弛形状</center>

```c++
relaxationShape 		Frozen;
actualRelaxationShape 	<word>;
// Additional lines for the actual relaxation shape
```

## 2.8 渗透结构阻力规范

waves2Foam中的阻力为(式2.4):

$$\frac{\mathbf{F}_{p}}{\rho}=a \mathbf{u}+b \mathbf{u}\|\mathbf{u}\|_{2}$$

其中$a$和$b$是阻力系数。除此之外，附加质量系数$C_m$如下所示：

$$C_{m}=\gamma_{p} \frac{1-n}{n}$$

这里，$\gamma_{p}$是闭合系数，取0.34。

### 2.8.1 阻力表达方式

在waves2Foam中实现了三种阻力公式。以下小节详细说明了每种方法所需的系数

#### 2.8.1.1 OpenFOAM原生

原生公式读取两个参数$d$和$f$（代码片段2.11）。

<center>代码片段2.11：原生松弛区</center>

```c++
d d [0 -2 0 0 0 0 0] 	<vector>;
f f [0 -1 0 0 0 0 0] 	<vector>;
porosity 				<scalar>;
gammaAddedMass 			<scalar>;
```

#### 2.8.1.2 Engelund (1953)

在Engelund (1953)的公式中，阻力系数$a$和$b$采用以下形式：

$$a=\alpha \frac{(1-n)^{3}}{n^{2}} \frac{\nu}{d_{50}^{2}} \quad, \quad b=\beta \frac{1-n}{n^{3}} \frac{1}{d_{50}}$$

这里，$\nu$是运动粘度，$d_{50}$是颗粒材料的标称中值直径。输入参数在代码片段2.12中指定。

<center>代码片段2.12：根据Engelund (1953)的阻力公式</center>

```c++
resistanceFormulation 				engelund1953;

porosity 							<scalar>;
gammaAddedMass 						<scalar>;

d50 	d50 	[0 1 0 0 0 0 0] 	<scalar>;
alpha 	alpha 	[0 0 0 0 0 0 0] 	<scalar>;
beta 	beta 	[0 0 0 0 0 0 0] 	<scalar>;
```

#### 2.8.1.3 Van Gent (1995)

在Van Gent (1995)的公式中，阻力系数$a$和$b$采用以下形式：

$$a=\alpha \frac{(1-n)^{2}}{n^{3}} \frac{\nu}{d_{50}^{2}} \quad, \quad b=\beta\left(1+\frac{7.5}{K C}\right) \frac{1-n}{n^{3}} \frac{1}{d_{50}}$$

这里，$\nu$是运动粘度，$d_{50}$是颗粒材料的标称中值直径。输入参数在代码片段2.13中指定。

<center>代码片段2.13：根据an Gent (1995)的阻力公式</center>

```c++
resistanceFormulation 			vanGent1995;
porosity 						<scalar>;
gammaAddedMass 					<scalar>;
d50		d50 	[0 1 0 0 0 0 0] <scalar>;
alpha 	alpha 	[0 0 0 0 0 0 0] <scalar>;
beta 	beta 	[0 0 0 0 0 0 0] <scalar>;
KC 		KC 		[0 0 0 0 0 0 0] <scalar>; // Default: 10000
```

插入的值只是示例。关于$\alpha, \beta, KC$的选择，参见Jensen等人（2014年）；Jacobsen等人（2015年）；Losada等人（2016年）。此外，请注意，Jacobsen等人（准备中）正在讨论KC数的影响。