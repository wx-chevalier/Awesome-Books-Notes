> 参考地址：https://ngte.cowtransfer.com/s/6bae58d4a68442

# A History of Clojure

Clojure 被设计为一种通用的实用功能语言，适合其宿主语言（例如 Java）在任何地方的专业人员使用。Clojure 最初于 2005 年设计，于 2007 年发布，它是 Lisp 的方言，但不是任何以前的 Lisp 的直接后代。它通过并发安全状态管理结构补充了不可变数据纯函数的编程功能，该结构支持编写正确的多线程程序，而无需使用互斥锁。Clojure 是面向托管设计的，因为它可以编译到另一种语言（例如 JVM）并在其运行时运行。这不仅仅是一个实施策略；众多功能确保用 Clojure 编写的程序可以直接有效地利用宿主语言的库并与之互操作。

尽管结合了（当时）两个不太受欢迎的想法，函数式编程和 Lisp，但 Clojure 至今已在金融，气候科学，零售，数据库，分析，出版，医疗保健，广告和基因组学等行业得到广泛采用。全球的咨询公司和初创公司，这令其作者的职业生涯大为惊讶。Clojure 中的大多数思想都不是新颖的，但是它们的结合使 Clojure 在语言设计（功能性，托管性，Lisp）中处于独特的位置。本文阐述了 Clojure 最初开发背后的动机以及各种设计决策和语言构造的基本原理。然后涵盖发行和采用后的演变。

# INTRODUCTION

Clojure 的目标可以最简洁地概括为：我希望一种语言可以像 Java 或 C＃ 那样受欢迎，但要支持一种更为简单的编程模型，以用于我一直在专业进行的各种信息系统开发中。

# BACKGROUND AND MOTIVATION

I would broadly characterize my work, and the work most commonly done by professional pro-grammers, as information systems programming. Most developers are primarily engaged in making systems that **acquire, extract, transform, maintain, analyze, transmit and render information—facts about the world**. Most often, this information documents some human activity, be that of customers, suppliers, advertisers, travelers, voters, members, students, patients etc and must deal with all the irregularity thereof. This is in stark contrast to **artificial systems**, e.g., programming language compilers, which make up their own rules, in fully enumerated spaces, can eliminate irregularity and can reject anything which does not conform.

Information system programmers have the thankless task of attempting to superimpose some-what regular models over information and real-world activity that refuses to comply. For instance, in music scheduling, trying to decide: whether artists have songs or songs have artists. Or optimizing a scheduler to spread out the plays of songs by each artist, except on ‘twofer Tuesdays’ when songs by each artist must be played in adjacent pairs. In information systems programming, twofer Tuesdays are everywhere.
