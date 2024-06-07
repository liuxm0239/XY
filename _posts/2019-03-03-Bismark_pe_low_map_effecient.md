---
layout: post
#标题配置
title:  "Bismark: paired-end low mapping efficiency"
#时间配置
date:   2019-03-03 10:16:27 +0800
#大类配置
categories: [生物信息, 计算方法]
#小类配置
tag: [表观遗传学, Bioinfo]
---

* content
{:toc}
---

测试中发现有一批甲基化数据的比对率非常低，

模拟数据的问题，下载的数据比对正常（SRR5343780） [Bismark: paired-end low mapping efficiency](http://seqanswers.com/forums/showthread.php?t=40496)

1. --min_score L,0,-1.9 可以解决；（-1.9允许比对中存在跟多的 mismatch；--score_min L,0,-0.2 将允许 100bp 读取的 ~3 个错配，--score_min L,0,-0.6 将允许最多 10 个错配和/或 Indel）
2. 排除质量值分布问题（trim-glaore；fastqc正常）；
3. 排除 reads 顺序不一致问题；
4. 排除 reference 用错；
5. 排除 directionial/non-dicret 文库问题
