# SRE: Debugging Simple Memory Leaks in Go
# 调试Go程序的简单内存泄漏问题

Memory leaks are a class of bugs where memory is not released even after it is no longer needed. They are often explicit, and highly visible, which makes them a great candidate to begin learning debugging. Go is a language particularly well suited to identifying memory leaks because of its powerful toolchain, which ships with amazingly capable tools (pprof) which make pinpointing memory usage easy.

[内存泄漏](https://en.wikipedia.org/wiki/Memory_leak)是一类程序bug，指内存虽然不用了，但没有释放掉。这类问题通常很明确，很容易可视化，这使得它们成为开始学习调试程序的一大候选。Go是一门尤其适合定位内存泄漏问题的语言，因为它有强大的工具链，具备非常优秀的工具(pprof), 使得定位内存使用情况变得简单。

I’m hoping this post will illustrate how to visually identify memory, narrow it down to a specific process, correlate the process leak with work, and finally find the source of the memory leak using pprof. This post is contrived in order to allow for the simple identification of the root cause of the memory leak. The pprof overview is intentionally brief and aims to illustrate what it is capable of and isn’t an exhaustive overview of its features.

我希望本文能讲清楚如何可视化的定位内存问题，并限制到某个特定的进程，协同分析进程的内存泄漏情况，最终利用pprof找到泄漏根源。本文的目标是简单定位内存泄漏问题的根源。pprof 概况简单，为了了解它的能力，而不是费力了解它的所有特性。

The service used to generate the data in this post is available here.

在[这儿](https://github.com/dm03514/grokking-go/tree/master/simple-memory-leak)可以找到本文用来产生数据的服务。

## What is a memory leak?
## 什么是内存泄漏？

If memory grows unbounded and never reaches a steady state then there is probably a leak. The key here is that memory grows without ever reaching a steady state, and eventually causes problems through explicit crashes or by impacting system performance.

如果内存无限制地增长，并且没法到达一个稳定状态，那么程序就可能存在泄漏的风险。关键是内存持续增长从不稳定，最终会因明确的程序崩溃或隐含的性能下降导致问题。

Memory leaks can happen for any number of reasons. There can be logical leaks where data-structures grow unbounded, leaks from complexities of poor object reference handling, or just for any number of other reasons. Regardless of the source, many memory leaks elicit a visually noticeable pattern: the “sawtooth”.

内存泄漏可能因各种原因发生。可能是逻辑泄漏，数据结构所占内存无限增加，或对象应用错误处理导致，或仅仅因为其他原因。不管什么原因，很多内存泄漏会导致一个明显可见的模式：内存“锯齿”图

## Debug Process
## 调试过程

This blog post is focused on exploring how to identify and pinpoint root cause for a go memory leak. We’ll be focusing primarily on characteristics of memory leaks, how to identify them, and how to determine their root cause using go. Because of this our actual debug process will be relatively superficial and informal.

本文专注于探索如何确认和定位一个Go程序的内存泄漏发生的原因。我们将把注意力重点放在内存泄漏的特征上，如何确认它们，以及如何用Go的工具链分析泄漏原因。 因为这点，我们实际的调试过程可能相对浅显，不那么正式。

The goal of our analysis is to progressively narrow scope of the problem by widdling away possibilities until we have enough information to form and propose a hypothesis. After we have enough data and reasonable scope of the cause, we should form a hypothesis and try to invalidate it with data.

我们的分析目标是通过排除可能性渐渐缩小问题范围，直到我们有足够的信息可以形成并提出一个假设。在我们有了足够的数据和合理的推测后，我们可以形成一个假设，并用数据推翻它。

Each step will try to either pinpoint a cause of an issue or invalidate a non-cause. Along the way we’ll be forming a series of hypotheses, they will be necessarily general at first then progressively more specific. This is loosely based on the scientific method. Brandon Gregg does an amazing job of covering different methodologies for system investigation (primarily focused on performance).

每一步我们会试图定位一个问题的原因或者否定其他原因。一路下去我们会形成一系列假设，一开始她们比较宽泛，慢慢地会更具体。这大致基于科学的方法。Brandon Gregg 总结了各种不同的[方法论](http://www.brendangregg.com/methodology.html)用于系统研究(首要目标是系统性能)。

Just to reiterate we’ll try to:

Ask a question
Form a Hypothesis
Analyze the hypothesis
Repeat until the root cause is found

我们将不断迭代以下过程：

- 提出一个问题
- 形成一条假设
- 分析上述假设
- 重复直到找到根本原因


----------------

via: https://medium.com/dm03514-tech-blog/sre-debugging-simple-memory-leaks-in-go-e0a9e6d63d4d

作者：[dm03514](https://medium.com/@dm03514)
译者：[mbyd916](https://github.com/mbyd916)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出
