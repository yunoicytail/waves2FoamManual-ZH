# 3. 工具

本章讨论了预处理和后处理实用程序的范围。

## 3.1 前处理工具

### 3.1.1 浪高仪

该实用程序允许定义探头和浪高仪。定义如下所述。

#### 3.1.1.1 输入

文件`constant/probeDefinitions`用于指定浪高仪和探头的位置。通用设置在代码片段3.1中定义。

<center>代码片段3.1：浪高仪和探头定义的通用设置</center>

```c++
<name of wave gauges>
{
	type waveGauge; 		// Definition of the type
	add 	(0 0 1); 		// Length and orientation of gauge
	axis 	z; 				// Vertical axis
	// Insert point distribution
}

<name of probes>
{
	type probeGauge;
	outputInterval <label>; 	// The frequency of outputting
	fields (<name0> <name1>); 	// The fields to sample
	// Insert point distribution
}
```

要提供的点分布（代码片段3.1）定义如下。

 ><font color=red>警告3.1：浪高仪的定义
 >必须定义所有的浪高仪，使它们至少部分位于计算域内。如果一个或多个波计完全超出计算域，则不检查浪高仪相对于计算域的位置，并且计算将在没有任何解释的情况下崩溃。</font>

**circularDistribution**：定义围绕给定中心具有一定半径的圆形分布。点分布的定义见代码片段3.2。

<center>代码片段3.2：圆形点分布</center>

```c++
pointDistribution 	circularDistribution;
N 					4;
centre 				(0 0 0);
radius 				1;
```

**lineDistribution**：定义沿由起点和终点定义的直线的线性分布。有可能使分布沿直线延伸；导致非等距点分布（代码片段3.3）。

<center>代码片段3.3：线性点分布</center>

```c++
pointDistribution 	lineDitribution;
N 					4;
linestart 			(0 0 0);
lineend 			(1 0 0);
stretch 			1.;
```
**quadrilateralDistribution**：定义一个四边形的点分布，即可以定义一个矩形的浪高仪分布，例如3*3=9个浪高仪（代码片段3.4）。

<center>代码片段3.4：四边形点分布</center>

```c++
pointDistribution 	quadrilateralDistribution;
N0 					10;
N1 					11;
linestart0 			(0 0 0);
lineend0 			(1 0 0);
lineend1 			(1 1 0);
stretch0 			0.9; 		// Default = 1.0
stretch1 			1.1; 		// Default = 1.0
```
**userDefinedDistribution**: 在物理实验中，当浪高仪应按位置分布时（代码片段3.5），可以方便地使用用户定义的分布。

<center>代码片段3.5：用户定义的点分布</center>

```c++
pointDistribution 	userDefinedDistribution;
N 					4;
xValues 			nonuniform List<scalar> 4(0 1 2 3);
yValues 			nonuniform List<scalar> 4(0.1 0.2 0.3 0.4);
zValues 			uniform 0;
```
所有位置（`xValues`、`yValues`和`zValues`）都可以根据非均匀格式或均匀格式定义。

#### 3.1.1.2 输出

在`constant/probeDefinitions`文件中，每个子字典有两个（探针）或四个（浪高仪）来自`waveGaugesNProbes`的输出。所有输出都写入case文件夹中的文件夹`waveGaugesNProbes`。文件详情如下：

- **\<name of set>_controlDict**:此文件包含应复制到`controlDict`的部分，以便使用波高计进行运行时采样，有关说明，请参见[第3.1.1.3节](#3113-运行时采样)。
- **\<name of set>_sets** (仅包含波高计) ：此文件包含波高计的定义，基本上是OpenFoam中`sample`实用程序标准格式的垂直线定义。
- **\<name of set>surfaceElevationDict** (仅包含波高计) ：此文件用于作为后处理步骤对波高计进行采样。只需将文件复制到系统文件夹并执行命令`surfaceElevation`（[第3.3.1节]()）。
- **\<name of set>.vtk**: 可以使用`ParaView/paraFoam`打开此文件，以便可视化探头和波高计的位置。

#### 3.1.1.3 运行时采样

运行时采样以波高计为例。类似的方法适用于探头压力计。
作为后处理步骤或运行时评估，可以使用预定义的波计对表面高程进行采样。后者在这里讨论，前者在第3.3.1节中描述。`waveGaugesNProbes`输出的一部分是名为`<name of set>_controlDict`的文件（代码片段3.6）。

<center>代码片段3.6：表面高程采样控制示例。</center>

```c++
surfaceElevation
{
	type 			surfaceElevation;
	functionObjectLibs 	( "libwaves2Foam.so" );
	outputControl 		timeStep; // Alternative: outputTime
	outputInterval 		1;
	//Additional output controls in waves2Foam
	//samplingStartTime -1;
	//surfaceSampleDeltaT 0.025;
	setFormat 		raw;
	interpolationScheme 	cellPointFace;
	fields (alpha1);
	#includeIfPresent "../waveGaugesNProbes/surfaceElevationAnyName_sets";
}
```

注意，指示符字段alpha1的名称自动调整为给定版本OpenFoam的默认命名。
普通的`outputControls`适用于此实用程序，但也可以以近似等距的时间步长对表面高程进行采样，该时间步长远小于字段的输出时间（请参见代码片段3.6中的注释行）。注意，不考虑`surfaceSampleDeltaT`的值，采样实用程序仅基于标准OpenFoam控件（`outputControl`和`outputInterval`）访问。建议使用默认设置。

### 3.1.2 setWaveParameters

实用程序`setWaveParameters`是一种预处理实用程序，计算所有必要的有物理意义波浪参数，例如`setWaveParameters`将水深和波周期的信息转换为一阶斯托克斯波理论的波数。

所有的输入参数都在`<casePath>/constant/waveProperties.input`文件中，处理后的数据输出到`<casePath>/constant/waveProperties`中。这样分为两个文件的做法源于历史原因。

输入文件`waveProperties.input`包含全局信息以及与每个波浪边界和/或松弛区相关的信息。以下两节给出了边界条件和/或松弛区域的全局参数和设置。（此处的`和/或`表明松弛区设置可以没有-译者注）

<center>代码片段3.7：waveProperties.input中的全局设置</center>

```c++
// Only relevant for porousWaveFoam
porosityModel jensenJacobsenChristensen2014;
// Global definition of still water level
seaLevel <scalar>;
sealevelAsReference <boolean>;
// Names of relaxation zones. The wave theories are defined in the
// subdictionaries name0Coeffs, name1Coeffs, etc.
relaxationNames (name0 name1 ...);
// Wave theory used to initiate the simulation.
initializationName outlet;
// Not necessary. Default value is 'emptyExternal'.
externalForcing emptyExternal;
```

#### 3.1.2.1 全局参数

全局参数的在代码片段3.7中指定。松弛区域的数量是通过列表`relaxationNames`定义的。这也意味着松弛区域可以在子词典中定义，例如，`name0Coeffs`，但是如果字符串name0没有被写在列表`relaxationNames`中，那么该松弛区域未被激活。
外部强迫是一种为模拟提供外部波数据的选项，有关这些选项的说明，请参见第4.3节。

#### 3.1.2.2 子字典信息

每组波浪参数都在一个子词典中给出，这个子词典称为`<name>+ “Coeffs”`。
代码片段3.8中给出了一个松弛区域名为`name0`的子词典的例子：

<center>代码片段3.8：waveProperties.input中子字典全览</center>
```c++
name0Coeffs
{
	// Insert all required data for any given wave type
	// Insert all required data for any given relaxation zone definition
}
```

有关波浪类型的详细信息，请参阅第4节。关于松弛区及其定义的细节可以在2.7节中找到。

#### 3.1.2.3 例子
代码片段3.9中展示了一个具有两个松弛区域的计算域的示例。最左边的松弛区用来产生二阶波浪，最右边的松弛区用来吸收波浪。
求解器为weavFoam。

### 3.1.3 relaxationZoneLayout

程序`relaxationZoneLayout`可用于研究松弛区的布局。该程序输出三个场到0文件夹下。

* `relaxationZoneLayout`:该场数据展示了松弛区的空间布置。值为`-1`表示其位于所有松弛区之外。第一个松弛区赋值为`0`，第二个赋值为`1`，以此类推。
* `relaxationZoneSigmaValue`：这是松弛区$\sigma$-权重的空间分布。
* `relaxationZoneWeightOnComputed`：展示了用于计算场的结果权重，即值`1`表示不更改计算场。

<center>代码片段3.9：waveProperties.input的例子</center>

```c++
FoamFile
{
	version 2.0;
	format 	ascii;
	class 	dictionary;
	object 	waveProperties.input;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

//porosityModel jensenJacobsenChristensen2014;

seaLevel 	0.00;

relaxationNames (inlet outlet);

initializationName outlet;

inletCoeffs
{
	waveType 	stokesSecond;
	height 		0.1;
	period 		3.0;
	depth 		1.0;
	phi 		0.0;
	direction 	(1.0 0.0 0.0);
	debug 		off;
	Tsoft 		2.0;
	relaxationZone
	{
		relaxationScheme 	Spatial;
		relaxationShape 	Rectangular;
		relaxType 		INLET;
		startX 			(-15.0 0.0 -1.0);
		endX 			(-10.0 0.0 1.0);
		orientation 		( 1.0 0.0 0.0);
	}
};

outletCoeffs
{
	waveType 	potentialCurrent;
	U 		(0 0 0);
	Tsoft 		2;
	relaxationZone
	{
		relaxationScheme 	Spatial;
		relaxationShape 	Rectangular;
		relaxType 		OUTLET;
		startX 			( 10.0 0.0 -1.0);
		endX 			( 15.0 0.0 1.0);
		orientation 		( 1.0 0.0 0.0);
	}
};
```

### 3.1.4 setWaveField
程序`setWaveField`用于根据用户定义的波浪理论设置初始条件。后者通过`waveProperties.input`中的关键词`initializationName`定义，请参考[第3.1.2.1节](#3121-全局参数)。
