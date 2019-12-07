# Intro

这本书旨在概述当今使用的最重要的计算机算法，并向越来越多需要了解它们的人传授基本技术。在学生获得基本编程技能和熟悉计算机系统后，它将被用作计算机科学第二门课程的教科书。这本书也可能对自学有用，或作为从事计算机系统或应用程序开发的人的参考，因为它包含有用算法的实现和关于性能特征和客户的详细信息。广阔的视角使这本书成为该领域的恰当介绍。

**算法和数据结构的研究**是任何计算机科学课程的基础，但不仅仅是对程序员而言或者对计算机科学学生而言。每个使用计算机的人都希望它运行得更快或者解决更大的问题。这本书里的算法代表了过去50年来发展起来的一套不可分割的知识。从物理学中的三体模拟问题到分子生物学中的基因测序问题，这里描述的基本方法在科学研究中变得至关重要；从建筑建模系统到飞机仿真，它们已经成为工程中必不可少的工具；从数据库系统到互联网搜索引擎，它们已经成为现代软件系统的重要组成部分。这些只是几个例子——随着计算机应用范围的不断扩大，这里介绍的基本方法的影响也在不断扩大。

在第一章中，我们开发了研究算法的基本方法，包括栈、队列和我们在整本书中使用的其他低级抽象的数据类型。在第2章和第3章中，我们概述了排序和搜索的基本算法；在第4章和第5章中，我们将介绍处理图和字符串的算法。第6章是一个概述，将书中的其余内容放在一个更大的背景下。

这本书的方向是研究可能有实际用途的算法。这本书教授了各种各样的算法和数据结构，并提供了关于它们的足够信息，读者可以放心地实现、调试它们，并使它们在任何计算环境中工作。该方法包括:

- 算法。我们对算法的描述是基于完整的实现和对这些程序在一组一致的例子上的操作的讨论。我们不用伪代码，而是用真正的代码，这样程序就可以很快投入实际使用。我们的程序是用Java编写的，但是其风格使得我们的大部分代码可以被重用来开发其他现代编程语言的实现。
- 数据类型。我们使用基于数据抽象的现代编程风格，以便算法及其数据结构封装在一起。
- 应用程序。每章都详细描述了算法在其中发挥关键作用的应用。从物理和分子生物学的应用，到工程计算机和系统，到熟悉的任务，如数据压缩和网络搜索。
- 一种科学的方法。我们强调开发描述算法性能的数学模型，使用这些模型来开发关于性能的假设，然后通过在现实环境中运行算法来测试这些假设。

- 覆盖范围。我们涵盖基本抽象数据类型、排序算法、搜索算法、图形处理和字符串处理。我们将材料保存在算法文本中，描述数据结构、算法设计范例、简化和问题解决模型。我们涵盖了自20世纪60年代以来传授的经典方法和近年来发明的新方法。

我们的主要目标是向尽可能多的受众介绍当今最重要的算法。这些算法通常都是巧妙的创造，值得注意的是，每种算法都可以用十几行或两行代码来表达。作为一个整体，他们代表着惊人的解决问题的能力。它们使得计算模型的构建、科学问题的解决以及商业应用的开发成为可能，没有它们这些都是不可行的。

这本书打算作为计算机科学第二门课程的教科书。它提供了核心材料的全面覆盖，是学生在编程、定量推理和解决问题方面获得经验和成熟的极好工具。通常，一门计算机科学的课程就足以作为先决条件——这本书是为任何熟悉现代编程语言和现代计算机系统基本特征的人准备的。

算法和数据结构是用Java表达的，但是用一种能被精通其他现代语言的人所理解的风格。我们接受现代Java抽象(包括泛型)，但抵制对语言深奥特性的依赖。

大多数支持分析结果的数学材料都是独立的(或者被标记为超出了本书的范围)，所以本书的大部分内容几乎不需要数学方面的具体准备，尽管数学成熟度肯定是有帮助的。应用程序取自科学的介绍性材料，同样是独立的。

涵盖的材料是任何打算主修计算机科学、电气工程或运筹学的学生的基本背景，对任何对科学、数学或工程感兴趣的学生都很有价值。

这本书旨在遵循我们的介绍性文本《Java程序设计导论:跨学科方法》，这是对该领域的广泛介绍。这两本书合在一起可以支持两到三个学期的计算机科学导论，这将给任何学生必要的背景知识，以便在科学、工程或社会科学的任何选定的研究领域成功地解决计算问题。

书中大部分材料的出发点是 Sedgewick 系列的算法教科书系列。从精神上来说，这本书最接近第一版和第二版，但这本书受益于几十年的教学经验。Sedgewick 目前的 Algorithms in C/C++/Java 第三版更适合作为高级课程的参考或文本；这本书是专为大学一年级或二年级学生的一学期课程设计的教科书，是对基础知识的现代介绍，也是工作程序员使用的参考。