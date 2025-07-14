---
type: source-note
title: Bloom Filters by Example
id: 20250711110725
created: 2025-07-11T11:52:25
source:
  - web
url: https://llimllib.github.io/bloomfilter-tutorial/zh_CN/
tags:
  - source-note
  - bloom-filter
processed: false
archived: false
---
[English](https://llimllib.github.io/bloomfilter-tutorial/)

Bloom filter 是一个数据结构，它可以用来判断某个元素是否在集合内，具有运行快速，内存占用小的特点。

而高效插入和查询的代价就是，Bloom Filter 是一个 **基于概率的数据结构** ：它只能告诉我们一个元素 *绝对不* 在集合内或 *可能* 在集合内

Bloom filter 的基础数据结构是一个 **比特向量** 。 下面是一个简单的示例：

|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 |

表中的每一个空格表示一个比特, 空格下面的数字表示它的索引。只需要简单的对输入进行多次哈希操作，并把对应于其结果的比特置为1，就可以向 Bloom filter 添加一个元素。

相较于解释，直观的看到它的变化总是更利于我们加深理解的，所以你可以输入一些字符串然后观察上方的向量变化。其中使用了 Fnv 和 Murmur 这两个简单的哈希函数：

当你往集合里添加一个字符串的时候, 你可以检查应用对应哈希函数的位置是否为1。这里我用了绿色表示最新添加的元素对应位置，但是实际上你要知道，表格里的不同颜色都只代表了值为 1。

你可以简单的通过对字符串应用同样的哈希函数，然后看比特向量里对应的位置是否为1的方式来判断一个元素是否在集合里。如果是，你只知道元素 *可能* 在里面, 因为这些对应位置有可能恰巧是由其他元素或者其他元素的组合所引起的。

测试:

fnv:  
murmur:

这个元素在集合中吗？ 否

误判的概率: 0%

这些就是 Bloom filter 全部的基础内容了。

## 高级话题

在写下更多关于 Bloom filter 的内容之前，我需要声明一点：我从未在生产环境使用过 Bloom filter。所以不要不假思索的相信下面的内容，我想做的只是给你一个概括式的介绍，同时告诉你可以去哪里寻找更多内容。

在下面的内容里, 我们假设在 Bloom filter 里面有 *k* 个哈希函数, *m* 个比特, 以及 *n* 个已插入元素。

### 哈希函数

Bloom filter 里的哈希函数需要是 **[彼此独立](http://en.wiktionary.org/wiki/independent_function)** 且 **[均匀分布](http://en.wikipedia.org/wiki/Uniform_distribution_\(discrete\))** 。同时，它们也需要尽可能的快 (尽管 sha1 之类的加密哈希算法被广泛应用，但是在这一点上考虑并不是一个很好的选择).

这些都是快速，简单且彼此独立的哈希函数的例子： <sup><a href="https://llimllib.github.io/bloomfilter-tutorial/zh_CN/#footnote3">3</a></sup> 包括 [murmur](https://sites.google.com/site/murmurhash/), [fnv](http://isthe.com/chongo/tech/comp/fnv/) 族哈希函数, 以及 [HashMix](https://web.archive.org/web/20061030103559/http://www.concentric.net/~Ttwang/tech/inthash.htm).

如果你希望了解一个比加密哈希函数快的哈希函数可以达到什么程度，可以参考 [这个故事](https://github.com/bitly/dablooms/pull/19) 。当把 bloom filter 的实现从 md5 切换到 murmur 时，速度提升了 800%。

一个关于 Bloom filter 实现方式的简单调查:

- [Chromium](https://chromium.googlesource.com/chromium/chromium/+/refs/heads/main/chrome/browser/safe_browsing/bloom_filter.cc) 使用 [HashMix](https://web.archive.org/web/20061030103559/http://www.concentric.net/~Ttwang/tech/inthash.htm). (同时, [这里](https://web.archive.org/web/20160306232658/http://blog.alexyakunin.com/2010/03/nice-bloom-filter-application.html) 是一个简短说明他们如何使用 bloom filter）
- [python-bloomfilter](https://github.com/jaybaird/python-bloomfilter/blob/master/pybloom/pybloom.py) 使用加密哈希算法。
- [Plan9](https://plan9.io/sources/plan9/sys/src/cmd/venti/srv/bloom.c) 使用一种简单的哈希算法， [Mitzenmacher 2005](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.152.579&rank=1)
- [Sdroege Bloom filter](https://github.com/sdroege/snippets/blob/master/snippets/bloomfilter.c) 使用 fnv1a (把这个加进来是希望展示有人使用 fnv 算法)
- [Squid](https://github.com/squid-cache/squid/blob/master/src/store_key_md5.cc) 使用 MD5

### Bloom filter 应该设计为多大？

Bloom filter 的一个优良特性就是可以修改过滤器的错误率。一个大的过滤器会拥有比一个小的过滤器更低的错误率。

错误率会近似于 *(1-e <sup>-kn/m</sup>) <sup>k</sup>*, 所以你只需要先确定可能插入的数据集的容量大小 *n*, 然后再调整 *k* 和 *m* 来为你的应用配置过滤器。 <sup><a href="https://llimllib.github.io/bloomfilter-tutorial/zh_CN/#footnote2">2</a></sup>

而这带来了一个显而易见的问题:

### 应该使用多少个哈希函数?

Bloom filter 使用的哈希函数越多运行速度就会越慢。但是如果哈希函数过少，又会遇到误判率高的问题。所以这个问题上需要认真考虑。

在创建一个 Bloom filter 的时候需要确定 *k* 的值，也就是说你需要提前圈定 *n* 的变动范围。而一旦你这样做了，你依然需要确定 *m* （总比特数）和 *k* (哈希函数的个数）的值。

似乎这是一个十分困难的优化问题，但幸运的是，对于给定的 *m* 和 *n* ，有一个函数可以帮我们确定最优的 *k* 值: *(m/n)ln(2)* <sup><a href="https://llimllib.github.io/bloomfilter-tutorial/zh_CN/#footnote2">2</a>, <a href="https://llimllib.github.io/bloomfilter-tutorial/zh_CN/#footnote3">3</a></sup>

所以可以通过以下的步骤来确定 Bloom filter 的大小:

1. 确定 *n* 的变动范围
2. 选定 *m* 的值
3. 计算 *k* 的最优值
4. 对于给定的 *n*, *m*, and *k* 计算错误率。如果这个错误率不能接收，那么回到第二步，否则结束

### Bloom filter 的时间复杂度和空间复杂度?

对于一个 *m* 和 *k* 值确定的 Bloom filter，插入和测试操作的时间复杂度都是 O(k)。这意味着每次你想要插入一个元素或者查询一个元素是否在集合中，只需要使用 *k* 个哈希函数对这个元素求值，然后将对应的比特位标记或者检查对应的比特位。

相比之下，Bloom filter 的空间复杂度更难以概述，它取决于你可以忍受的错误率。同时也取决于输入元素的范围，如果这个范围是有限的，那么一个确定的比特向量就可以很好的解决问题。如果你甚至不能很好的估计输入元素的范围，那么你最好选择一个哈希表或者一个可拓展的 Bloom filter。 <sup><a href="https://llimllib.github.io/bloomfilter-tutorial/zh_CN/#footnote4">4</a></sup>.

### 可以用 Bloom filter 来做什么?

我会将你引向 [wiki](http://en.wikipedia.org/wiki/Bloom_filter#Examples) 而不是将它们的内容拷贝过来。 [C. Titus Brown](https://archive.org/details/pyvideo_402___handling-ridiculous-amounts-of-data-with-probabilistic-data-structures) 的演讲很好的阐述了 Bloom filter 在生物信息学中的应用。

### 参考文献

1: [Network Applications of Bloom Filters: A Survey](http://citeseer.ist.psu.edu/viewdoc/download;jsessionid=6CA79DD1A90B3EFD3D62ACE5523B99E7?doi=10.1.1.127.9672&rep=rep1&type=pdf), Broder and Mitzenmacher. An excellent overview.

2: [Wikipedia](http://en.wikipedia.org/wiki/Bloom_filter#Probability_of_false_positives), which has an excellent and comprehensive page on Bloom filters

3: [Less Hashing, Same Performance](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.152.579&rank=1), Kirsch and Mitzenmacher

4: [Scalable Bloom Filters](http://gsd.di.uminho.pt/members/cbm/ps/dbloom.pdf), Almeida et al