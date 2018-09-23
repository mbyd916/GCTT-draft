# SRE: Debugging Simple Memory Leaks in Go
# 调试Go程序的简单内存泄漏问题

Memory leaks are a class of bugs where memory is not released even after it is no longer needed. They are often explicit, and highly visible, which makes them a great candidate to begin learning debugging. Go is a language particularly well suited to identifying memory leaks because of its powerful toolchain, which ships with amazingly capable tools (pprof) which make pinpointing memory usage easy.

内存泄漏是一类程序bug，指内存虽然不用了，但没有释放掉。这类问题通常很明确，很容易可视化，这使得它们成为开始学习调试程序的一大候选。Go是一门尤其适合定位内存泄漏问题的语言，因为它有强大的工具链，具备非常优秀的工具(pprof), 使得定位内存使用情况变得简单。

I’m hoping this post will illustrate how to visually identify memory, narrow it down to a specific process, correlate the process leak with work, and finally find the source of the memory leak using pprof. This post is contrived in order to allow for the simple identification of the root cause of the memory leak. The pprof overview is intentionally brief and aims to illustrate what it is capable of and isn’t an exhaustive overview of its features.

我希望本文能讲清楚如何可视化的定位内存问题，并限制到某个特定的进程，协同分析进程的内存泄漏情况，最终利用pprof找到泄漏根源。本文的目标是简单定位内存泄漏问题的根源。pprof 概况简单，为了了解它的能力，而不是费力了解它的所有特性。

The service used to generate the data in this post is available here.

在[这儿](https://github.com/dm03514/grokking-go/tree/master/simple-memory-leak)可以找到本文用来产生数据的服务。


----------------

via: https://medium.com/dm03514-tech-blog/sre-debugging-simple-memory-leaks-in-go-e0a9e6d63d4d

作者：[dm03514](https://medium.com/@dm03514)
译者：[mbyd916](https://github.com/mbyd916)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [GCTT](https://github.com/studygolang/GCTT) 原创编译，[Go 中文网](https://studygolang.com/) 荣誉推出
