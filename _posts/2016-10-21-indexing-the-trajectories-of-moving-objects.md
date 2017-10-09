---
layout: post
title: 【译】移动对象轨迹的索引
tags:
  - program
  - bigdata
  - database
  - index
  - translate
---

本文翻译自 Dieter Pfoser 的「Indexing the Trajectories of Moving Objects」，这篇文章简要介绍了移动对象轨迹索引领域的基础概念、需要解决的问题、现有的索引方法以及未来的研究方向。

---

## 摘要

时空应用提供了一种新类型的数据和查询的宝库。这篇文章的关注点是时空的子领域，也就是移动对象的轨迹问题。我们研究这种类型的数据关于索引的问题并指出当前存在的方法和研究方向。移动的一个重要的方面是它发生的情形。一共有三种不同的情形用于区分各种各样的索引方法，即不受限移动、受限移动和网络中的移动。每种情形赋予我们不同的方式去简化索引或者提高整体的查询处理效率。

## 1 介绍

不少应用领域不断产生了大量不同类型的时空数据。比如，我们现在正在经历着快速的科技发展，个人信息广泛传播。移动性是很多应用和服务关心的。移动性的一方面就是移动，也就位置的改变。本文中的应用包括新兴的基于位置的服务，也包括经典的车队管理和车辆的最佳时空分布。

这些应用促进了移动对象索引的研究。尤其是，我们的关注点在于移动对象的记录（也就是它们的轨迹）和索引（为了后处理，比如数据挖掘）。因此，我们不关心当前位置的索引，也不关心移动对象的预测。对象的大小和形状一般是固定的而且不重要。因此，问题就变成了记录移动对象随着时间的位置变化。对象的移动可以用一条处在三维空间（二维空间和一维时间）中的轨迹来代表。

传统的存取方法只考虑数据和查询。然而对轨迹数据来说我们也要考虑移动对象所受到的限制。特别地，我们可以将移动情形分为三类：不受限移动（比如海上的船）、受限移动（比如行人）和交通网络中的移动（比如火车、汽车等）。 正如我们将会在这篇文章中看到的那样，后两种情形允许我们优化查询处理或使用更简单的存取方法索引数据。

本文大纲如下。第二章定义了基本概念，指出轨迹索引所面临的挑战。第三、四、五章指出了针对三种不同移动情形的索引方法。最后，第六章概括并指出了将来的研究方向。

## 2 轨迹和查询

这部分的中心内容是我们如何减轻移动对象索引的负担。这一章简要描述了数据、相关查询和各自的索引挑战。

### 2.1 轨迹

轨迹是我们通过记录移动对象随着时间的位置改变获取的。考虑下面这些应用。交通优化，尤其是对那些车流量很大容易造成交通拥堵的区域，对交通系统来说是个非常有挑战性的任务。本文中的一个核心应用是车辆管理。装载GPS接收器的车辆把自己的位置上传到服务器，由服务器来处理利用。

为了记录对象的移动，我们需要时刻知道对象的位置。然而，GPS和通讯设备只允许我们采样对象的位置或者获取离散时间位置，比如每几秒钟采样一次。第一种标志移动对象的方法就是简化存储的位置采样点。这意味着我们不能准确的查询到移动对象在采样位置时间之间的那些数据。那为了获取整个移动位置，我们需要插值。最简单的方式是使用线性插值法（和多项式曲线法相反）。采样位置变成了线段的端点，对象的移动由整个三维空间中的折线表示。下图表示对象的移动：

 ![QQ截图20161020190929](\media\files\2016\10\21\QQ截图20161020190929.png)

空间和时间坐标组合到一起形成一个单独的坐标系统。虚线表示移动对象在二维空间平面上的投影。

在经典的空间数据库中只有位置信息是可用的。然而在我们的例子中，我们也有衍生信息，比如速度、加速度、旅行距离等等。这些信息是从时间和空间数据中衍生出来的。进一步来说，我们不仅仅索引线段的集合（也就是轨迹）。这些数据的语义属性反映在对数据的查询中。

### 2.2 查询

对轨迹的查询包含适当的空间查询，比如范围查询（找到在某个给定区域和时间区域内的所有对象）和没有空间对应的查询。在这里，所谓的基于轨迹的查询被分成拓扑查询（涉及整个移动对象，比如进入、离开、穿过、路过）和导航查询（包括衍生信息，比如速度和朝向）。下表总结了查询类型： 

![QQ截图20161020192445](\media\files\2016\10\21\QQ截图20161020192445.png)

「Signature」列表示查询涉及的类型，比如基于坐标的查询使用「inside」操作来找到给定范围内的线段。那个 {segments} 符号指的是一个集合，它不捕获（capture）这个集合是否由一条或多条轨迹组成。抽取和轨迹关联的信息非常重要，比如「哪些是这些移动对象在今天上午七点到八点之间离开图森后，在接下来的一个小时内的轨迹？」。这种查询是复合式查询。

### 2.3 索引基本原理

轨迹是三维的而且能用空间存取方式来索引。然而，还是有很多困难。轨迹被分解成一段段的线段，然后被索引。R-Tree 的使用在下图中展示：

![QQ截图20161020194826](\media\files\2016\10\21\QQ截图20161020194826.png)

R-Tree 用最小边界盒子（MBBs）近似估计数据对象。使用 MBBs估计线段被证明是低效的。从上图中我们也能看到大量的「dead space」没有被使用，被轨迹本身占用的实际空间很小。这导致高重叠并且因此导致索引结构的小识别能力。

其他的轨迹索引问题包括轨迹保存和偏斜数据增长。关于第一个问题，空间索引趋向于根据空间临近度把线段分成组存进节点。然而，对轨迹来说，如果索引可以保存轨迹对某些查询是有益的，比如根据轨迹空间邻近度把它们分成组。第二个问题和轨迹数据在时间维度上的增长有关，空间维度是固定的，比如范围限制在一个城市。研究数据的这个特性可以进一步提高查询效率。

### 2.4 移动情形

移动对象是受限制的。特别地，我们可以将移动情形分为三类：不受限移动（比如海上的船）、受限移动（比如行人）和交通网络中的移动（比如火车、汽车等）。 不受限移动在时空存取方法中经常被提到，但是它很难代表现实情况。受限移动和交通网络中的移动代表了相似的情形。前者假设存在空间物体限制移动，比如当考虑到汽车、房屋、湖泊、公园等的移动的时候。交通网络中的移动仅仅是受限移动的一个抽象。在这里，只关注物体相对网络的位置而不关注二维参考系统。比如，我们可能希望许多应用关注汽车相对路网的位置而不是它们的绝对坐标位置。移动有效地发生在与前两种情形不同的空间中。

本章阐述了轨迹索引的挑战。对辅助信息的研究，比如基础设施和交通网络，我们可以提高查询效率，简化索引。接下来的三章探究了对三种移动情形的索引和查询处理方法。

## 3 不受限移动的索引

不受限移动从概念上来讲是轨迹索引中最简单的情形。接下来，我们描述了几种适应数据和查询需要的存取方法。

### 3.1 TB-Tree

TB-Tree 是一种考虑到轨迹数据特殊性（也就是轨迹保存和时间维度上的增长）的存取方法，旨在有效地处理相关类型的查询。

对于 R-Tree 一个潜在的假设是所有插入的几何体都是独立的。在本文中也就是说所有的线段都是独立的。然而，线段是轨迹的一部分，R-Tree 只隐式地说明了这点。对于 TB-Tree，我们旨在寻求一种能够严格地保存轨迹的存取方法。因此，索引中每个叶节点应该只包含属于同一条轨迹的线段。这样索引就变成了一个轨迹束。这种方法只有通过对最重要的R-tree属性（即节点重叠或空间区分）作出一些让步才有可能。作为缺点，来自不同轨迹在空间上临近的线段将会被存储进不同的节点。这又增加了节点重叠，减少了空间区分，并且因此增加了经典范围查询成本。所以，TB-Tree 是以空间分离度换轨迹保存。

TB-Tree 结构由一系列叶节点组成，每个包含轨迹的一部分，组织成层级结构。对于查询处理来说，通过轨迹标识获取线段是有必要的。因此，包含相同轨迹线段的叶节点由一个双链表链接在一起。这种方式可以保存轨迹演化过程，又从整体提升了基于轨迹的查询和复合查询的性能。下图是一个 TB-Tree 结构的一部分和一条轨迹：

![QQ截图20161020204206](\media\files\2016\10\21\QQ截图20161020204206.png)

为了清楚起见，轨迹画成了带状而不是一条线。灰色表示的轨迹被分成六段，穿过六个节点，C1、C3 等等。在 T B-Tree 中这些叶节点由双链表链接。

下图展示了 R-Tree 和 TB-Tree 中非叶结点层的边界盒子的排布：

 ![QQ截图20161020204718](\media\files\2016\10\21\QQ截图20161020204718.png)

R-Tree 分组纯粹依靠空间特性，比如邻近度，忽视了数据在时间维度上的增长。TB-Tree 结构是根据轨迹保存来控制的。由于数据是在不断增长的时间层上插入的，MBBs 呈现出时间维度上的聚集。

TB-Tree 结构不是轨迹索引的唯一方法。接下来的一章简要介绍了一些存在的其他方法。

### 3.2 其他轨迹索引方法

Nasciemento 等人采用了 2.1 节中介绍的轨迹模型并且研究了多维访问方法对轨迹数据的适用性。。它们比较了不同 R-Tree 版本在不同范围查询下的性能表现。

Hadjileftheriou 等人定义了一个方法「artificial object updates」减少前文所述的「dead space」。他们有效地操纵将轨迹划分成段。部分持久性树结构用于索引数据。 这种方法概括了以前的工作，其中假定对象以时间的线性函数移动，而在更复杂的函数中是允许的。

Porkaew 等人对比了本地空间和参数空间的轨迹索引。在参数空间中，轨迹线段是用一个位置和一个运动向量来表示的。在他们的实验评估中，作者使用R-Tree 结构作为两种表示的索引。

其他的工作进一步提出一些方法来索引移动空间形状，不单单是对轨迹进行索引了。然而这些方法并不总是考虑连续，而是只考虑离散改变。比如 Tzouramanis 等人利用重叠的四叉树来索引空间对象离散的改变。

Tao 和 Papadias 提出了一种方法叫做 MV3R-Tree，用它来索引过去的移动形状的位置。他们的方法兼并了多版本的 B-Tree 和 3D R-Tree 来处理时间戳和区间查询。

## 4 受限移动

我们说对象的移动是受一些叫做基础设施的东西限制的。

在数据方面，基础设施表示相对移动对象的「black-out」区域，因此，在有基础设施的地方是没有对象和轨迹的。然而，在索引里面，我们使用近似的数据，叫做「dead space」。索引那些被基础设施覆盖的区域不是空的，在索引里这会导致不必要的搜索负担，同时也会返回一些错误的数据，这个问题必须被消除。两者都会导致额外的 I/O 操作。

为了消除额外的 I/O 操作，我们可以在一个预处理步骤中使用基础设施，这个想法是不搜索没有任何对象的设施。我们选择的策略是查询基础设施以保存查询轨迹数据。总的来说，这将是有利的，因为与轨迹数据相比，可以假定基础设施元件的数量非常小。此外，轨迹数据随时间增长，而基础设施数据或多或少保持恒定。一般原则是基于包含在其中的基础设施来分解给定的查询窗口。直觉是将没有被基础设施占据的查询窗口的部分分割成井状的矩形，即尽可能切成正方形的。在下图中，几个比较大的基础设施元件被表示为黑色矩形，这种分割过程的可能结果被表示为白色矩形：

![QQ截图20161021092925](\media\files\2016\10\21\QQ截图20161021092925.png)

随后使用从该分割得到的查询窗口（而不是基础设施上的大查询窗口）来查询轨迹数据索引。

## 5 网络中的移动

在许多应用中，移动发生在网络中。 当处理网络受限运动时，人们不关心空间范围，例如道路的厚度或物体在其坐标上的绝对位置，而是其相对于网络的位置，例如距 101 公路的 21 公里处。

由网络定义的空间与网络嵌入的欧几里得空间完全不同。 直观上，网络受限空间的维度低于其嵌入空间的维度。

相对于网络的建模运动简化了获得的轨迹数据。 两个空间维度基本上减少到一个。下图通过在三维和二维空间中展示相同的轨迹来阐述该原理。 将二维网络简化为一系列一维片段，并且相应地映射轨迹数据：

![QQ截图20161021094332](\media\files\2016\10\21\QQ截图20161021094332.png)

降低数据的维度大大降低了索引挑战。 现有的数据库管理系统通常不提供三维索引，因此不需要面对轨迹处理。 虽然希望为新类型的数据设计新的访问方法，但是它在短期或中期可能是有吸引力的。例如，在 R 树发现进入一些商业数据库产品之前需要十几年。根据数据的类型，通过转换数据使用现有的访问方法可能是有益的。 考虑网络中的移动是这样的变换。 我们可以使用简单的访问方法，例如二维 R-Tree，这反过来允许新类型的数据容易地集成到商业数据库管理系统中。

## 6 总结和未来工作

时空数据来自广泛的应用。 在这项工作中，我们提出了用于轨迹（一种来源于记录点对象的运动的数据）索引的一种方法。现有的方法分为三种移动情形，受限移动，不受限移动和网络中的移动。每一个情形都可以以不同的方式辅助空间时间查询的处理。不受限移动是定义新的访问方法的典型展示。 受限移动允许我们减少查询窗口的范围。 网络中的移动降低了数据的维数，从而降低了索引的维数。

未来工作的方向可以是定义更有效或者更专门的访问方法，或者满足从实际应用产生的现有需求。这可以通过利用我们现有的知识处理时空数据并结合现有的数据库管理系统方法来实现。

---

*原文链接：[http://www.dieter.pfoser.org/publications/pfoser_debull02.pdf](http://www.dieter.pfoser.org/publications/pfoser_debull02.pdf)* 