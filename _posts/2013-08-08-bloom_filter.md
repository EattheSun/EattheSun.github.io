---
layout: post
title: Bloom Filter 的一种实现
---


# 关于Bloom Filter

又叫作[布隆过滤器](http://zh.wikipedia.org/wiki/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8)。

## 作用

布隆过滤器可以用于检索一个元素是否在一个集合中。通常用在数据量很大的场景，因为在这种场景下，传统的数据结构比如树、链表、数组等效率会比较低下。

## 原理

当一个元素被加入集合时，通过K个Hash函数将这个元素映射成一个位阵列（Bit array）中的K个点，把它们置为1。检索时，我们只要看看这些点是不是都是1就（大约）知道集合中有没有它了：

	如果这些点有任何一个0，则被检索元素一定不在；如果都是1，则被检索元素很可能在。

这就是布隆过滤器的基本思想。

## 优点

相比于其它的数据结构，布隆过滤器在空间和时间方面都有巨大的优势:

1. 布隆过滤器存储空间和插入/查询时间都是常数（O(k)）
2. Hash函数相互之间没有关系，方便由硬件并行实现
3. 布隆过滤器不需要存储元素本身，在某些对保密要求非常严格的场合有优势

## 缺点

1. 随着存入的元素数量增加， **误算率随之增加** 
2. 一般情况下不能从布隆过滤器中 **删除元素**

# 实现

这段实现源自rs库，原本是一个利用 [libevent](http://www.wangafu.net/~nickm/libevent-book/) 实现的 http 服务器。

这里只扣出其中最关键的BloomFilter算法部分。

## 数据结构

A. BloomFile，BloomFile是一个包含了集合中所有元素映射后结果的一个文件。 BloomFile 结构上分为 BloomFilterHeader 和 BitBody 两部分：

	+--------------------------------------------------------+
	|      32bytes         | unit | unit | unit | ... | unit |
	+--------------------------------------------------------+
	|                      |                                 |
	|>  BloomFileHeader   <|>             BitBody           <|

- BloomFileHeader 和通常其他文件格式的头部一样，包含了一些必要的信息。

- BitBody 是文件BloomFile中除了头部之外的所有内容。

- unit 在这里是 uint32_t 类型的数值，当然也可以是其他的数值类型。用来进行位映射计算的。


大体算法如下(TODO)：

1. BloomFilter 程序将 BloomFile 文件读入内存 ( 利用 mmap 进行内存映射 )
2. 如果要计算一个元素是否存在这个已知的集合中，则：

- 首先利用某种挑选算法（这里使用的简单的模余运算 % ），将这个元素映射到 BitBody 中的某个 unit 上
- 然后利用既定的hash函数进行运算，查看该unit中相应的bit位是否为1

结构体定义：

	#define BloomFileTag		0xfa9734ed
	#define BloomFileVer		0x0100
	#define BloomFileHeaderBytes	32

	typedef struct tagBloomFileHeader {
		uint32_t tag;		// 固定标识，用于文件识别 -> 0xfa9734ed
		uint32_t ver;		// 版本号 -> 0x0100
		uint64_t len;		// BitBody 中用于位映射运算的 unit 个数
		uint32_t k;		//
		uint32_t unitLen;	// 每个 unit 的长度
		uint64_t reserved;	// 保留字段
	} BloomFileHeader;


所谓 unit 即 BitBody 中包含的内容单元。


判断一个元素是否在集合中：

	#define bloomUnitBytes		sizeof(uint32_t)
	#define bloomHashBytes	 	sizeof(uint64_t)
	#define halfBloomHashBits	(bloomHashBytes * 4)

	uint32_t BloomFilter_Has(BloomFilter* self, uint64_t h) {
		size_t i, j;
		size_t k = self->k;
		size_t n = self->n;
		uint32_t* array = self->array;

		uint64_t delta = (h >> (halfBloomHashBits + 1)) | (h << (halfBloomHashBits - 1));
		uint32_t has = -1;

		for (j = 0; j < k; j++) {
			i = (size_t)(h % n);
			has &= array[i];
			if (has == 0) {
				break;
			}
			h += delta;
		}
		return has;
	}
