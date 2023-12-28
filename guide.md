本文档为 [Topas3.7 英文文档](https://topas.readthedocs.io/en/latest/index.html) 的翻译版，仅选择其中重要的点进行翻译，如要详细了解请阅读原文。

速查表：
[Built-in Default Params](https://topas.readthedocs.io/en/latest/parameters/defaults.html)
[Geant4 Material Database](https://geant4-userdoc.web.cern.ch/UsersGuides/ForApplicationDeveloper/html/Appendix/materialNames.html#g4matrdb) 

[toc]

# Parameter System

Topas 的参数文件执行严格的类型检查，其中每个参数必须具有特定的声明类型（字符串、布尔值、整数等），并检查提供的值以确保它们适合给定类型。参数文件的每一行的语法形式如下：

```Topas
Paramerter_Type:Prameter_Name = Parameter_Value
```

*Parameter_Name* 几乎可以是任何字符串，但我们有前缀约定来保持清晰：

* `Ma/` Material 材料 
* `El/` Element 元素
* `Is/` Isotopes 同位素
* `Ge/` Geometry 几何设置
* `So/` Source 粒子源
* `Ph/` Physics 物理过程
* `Vr/` Variance Reduction 方差降低
* `Sc/` Score 计数
* `Gr/` Graphics 图形界面
* `Tf/` Time feature 时间特征
* `Ts/` Overall Control 全局控制

*Parameter_Type* 告诉 TOPAS 此参数中的数据类型，值和参数类型严格对应：

* `d` 有单位的双浮点数，并在赋值最后给出单位
* `u` 无单位的双浮点数
* `i` 32位整数，$0-2^{32}$
* `b` 布尔值，`"True" "t" "1"`，`"False" "f" "0"`
* `s` 字符串，必须使用双引号括起来
* `dv` 有单位浮点数向量，赋值第一位是向量的元素数目，后续是元素；`uv sv iv` 含义相同

参数文件中 `#` 表示注释，空行会被忽略，大小写不敏感。对于加/减/乘运算以及赋值符号前后必须添加空格以提高可读性。两个有单位的数无法相乘，可能会产出新的单位。Topas 中不存在除法，防止出现 0 为除数。

对于参数文件中可允许的语法格式具体参见 *examples/Basic/AllParameterForms.txt*

# Hierarchical Control

待完成

# Materials

## Basic

Topas 内部根据 NIST 数据集定义了自然界丰度下的元素，也包括 Geant4 内部定义的材料，通常用户只需要自己定义材料，但是元素、核素在某些情况下也可以自定义。自定义材料时注意不要和其他文件中的材料名冲突，也不要以 *G4_* 开头。原则上首先使用内部定义好的材料，预定义材料有电离模型的特定校正来提供额外的好处。

使用内置元素定义空气的语法如下：

```
sv:Ma/Air/Components=4 "Carbon" "Nitrogen" "Oxygen" "Argon" # 元素名称
uv:Ma/Air/Fractions=4 0.000124 0.755268 0.231781 0.012827 # 元素质量分数
d:Ma/Air/Density=1.2048 mg/cm3
d:Ma/Air/MeanExcitationEnergy=85.7 eV # 平均激发能
s:Ma/Air/DefaultColor="lightblue"

b:Ma/Air/NormalizeFractions = "True" # 质量分数归一化，可选，默认 "False"
```

*Fractions* 表示材料中的元素质量分数，默认情况下不执行归一化，用户可自行决定。

*MeanExcitationEnergy* 表示材料的平均激发能，物理意义是 Bethe 公式中的 `I` 项。

使用内置材料定义新材料的语法如下，添加额外的参数 *BuildFromMaterials*，并且材料组分使用内置材料名称而不是元素名称：

```
b:Ma/MyMixture/BuildFromMaterials = "True"
sv:Ma/MyMixture/Components = 2 "G4_WATER" "Air" # Geant4 的内置材料以 G4_ 开头
uv:Ma/MyMixture/Fractions = 2 .5 .5
d:Ma/MyMixture/Density = .5 g/cm3
```

如果同种材料需要不同的縻都督，可以使用 *VariableDensityBins*：

```
i:Ma/MyMaterial/VariableDensityBins = 100
u:Ma/MyMaterial/VariableDensityMin = .1
u:Ma/MyMaterial/VariableDensityMax = 10.
```

这样定义了 100 个密度范围为 $[0.1\times \rm{Density}, 10\times \rm{Density}]$ 的相同组成的材料，材料的名称为：
```
MyMaterial_VariableDensityBin_0
MyMaterial_VariableDensityBin_1
...
MyMaterial_VariableDensityBin_99
```

## Elements and Isotopes

定义核素语法如下：

```
i:Is/U235/Z = 92 # 质子数
i:Is/U235/N = 235 # 核子数
d:Is/U235/A = 235.01 g/mole # 质量数
i:Is/U238/Z = 92
i:Is/U238/N = 238
d:Is/U238/A = 238.03 g/mole
```

根据不同丰度的核素组成元素：

```
s:El/MyEIU/Symbol = "MyElU"
sv:El/MyElU/IsotopeNames = 2 "U235" "U238"
uv:El/MyElU/IsotopeAbundances = 2 90. 10. # 丰度
```

详细例子：*examples/Basic/Isotope.txt*

# Geometry

