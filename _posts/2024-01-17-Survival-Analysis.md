---
layout: post
#标题配置
title:  生存分析
#时间配置
date:   2024-01-17 11:18:45 +0800
#大类配置
categories: 统计学
#小类配置
tag: 教程
---

* content
{:toc}

----

2021 年的时候,有段时间专门研究了下生存分析.当时想了很久,怎么用直观的例子来类比生存分析,后来用了不同放射物下的“薛定谔的猫猫”做类比,就容易理解了一些.

## 生存分析的步骤
1. 首先做单因素cox挑选显著相关特征
2. 然后用挑选出的多个特征，做多因素cox


# 单因素和多因素cox回归分析

前面我们讲过一个R函数搞定风险评估散点图，热图，其中LASSO模型的输入就是单因素cox分析得到的显著与生存相关的基因。今天我们就来探讨一下如何使用R来做单因素和多因素cox回归分析。

我们用R的survival包自带的一套肺癌的数据来举例
```
#安装下面两个R包
install.packages(c("survival", "survminer"))
​
#加载这两个R包
library("survival")
library("survminer")
​
#加载肺癌这套数据
data("lung")
#显示前6行
head(lung)
```
这里每一行是一个样本，从第三列开始每一列是一个特征
![](https://pic1.zhimg.com/80/v2-277d2fc984d795b2e0d73c5c32e4385c_720w.jpg)

1. 单因素cox回归分析

对单个特征进行cox回归分析，看它是否与样本的生存显著相关
```
#单因素cox回归分析，这里看性别sex这个特征
res.cox <- coxph(Surv(time, status) ~ sex, data = lung)
res.cox
```
可以看到这里算出来的p值是0.00149，是显著的
![](https://pic2.zhimg.com/80/v2-dc61f7f11d859a5e58b8982c97f56ce9_720w.jpg)
我们在来看一下summary

`summary(res.cox)`
这里的exp(coef)就是HR（hazard ratio，风险率），lower .95和upper .95为95%的置信区间
![](https://pic1.zhimg.com/80/v2-800b0cd3b963d69ad2e56ddde0180b44_720w.jpg)

2. 批量单因素cox回归分析

一般我们的关注的特征都比较多，用上面的代码一个一个来做单因素cox回归分析效率太低了，下面我们来看看如何批量做单因素cox回归分析。
```
#假设我们要对如下5个特征做单因素cox回归分析
covariates <- c("age", "sex",  "ph.karno", "ph.ecog", "wt.loss")
#分别对每一个变量，构建生存分析的公式
univ_formulas <- sapply(covariates,
                        function(x) as.formula(paste('Surv(time, status)~', x)))
```
![](https://pic2.zhimg.com/80/v2-64471772fb6f57be769fc7ad2ad1a2e9_720w.jpg)
```
#循环对每一个特征做cox回归分析
univ_models <- lapply( univ_formulas, function(x){coxph(x, data = lung)})
```
![](https://pic4.zhimg.com/80/v2-d760382d4783bcf4ea75bc5810bb3287_720w.jpg)

```
#提取HR，95%置信区间和p值
univ_results <- lapply(univ_models,
                       function(x){
                         x <- summary(x)
                         #获取p值
                         p.value<-signif(x$wald["pvalue"], digits=2)
                         #获取HR
                         HR <-signif(x$coef[2], digits=2);
                         #获取95%置信区间
                         HR.confint.lower <- signif(x$conf.int[,"lower .95"], 2)
                         HR.confint.upper <- signif(x$conf.int[,"upper .95"],2)
                         HR <- paste0(HR, " (",
                                      HR.confint.lower, "-", HR.confint.upper, ")")
                         res<-c(p.value,HR)
                         names(res)<-c("p.value","HR (95% CI for HR)")
                         return(res)
                       })
#转换成数据框，并转置
res <- t(as.data.frame(univ_results, check.names = FALSE))
as.data.frame(res)
write.table(file="univariate_cox_result.txt",as.data.frame(res),quote=F,sep="\t")
```
得到的结果如下，你会发现对于sex这个特征来说，结果跟前面单独做得到的结果是一样的。
![](https://pic2.zhimg.com/80/v2-281c81b8a2a57c7c7c38cdb086c92441_720w.jpg)



3. 多因素cox回归分析

<mark>**前面是单独看每一个特征是否跟生存相关，而多因素cox回归是同时检测多个特征是否与生存相关。
一般先通过单因素cox回归分析找出与生存显著相关的特征，然后基于这些特征再去做多因素cox回归分析，或者做LASSO分析。**

```
res.cox <- coxph(Surv(time, status) ~ age + sex + ph.karno + ph.ecog, data =  lung)
x <- summary(res.cox)
pvalue=signif(as.matrix(x$coefficients)[,5],2)
HR=signif(as.matrix(x$coefficients)[,2],2)
low=signif(x$conf.int[,3],2)
high=signif(x$conf.int[,4],2)
multi_res=data.frame(p.value=pvalue,
                     HR=paste(HR," (",low,"-",high,")",sep=""),
                     stringsAsFactors = F
)
multi_res
write.table(file="multivariate_cox_result.txt",multi_res,quote=F,sep="\t")
```
得到的结果如下
![](https://pic1.zhimg.com/80/v2-26a88269585d94abe8895154afb2d3c8_720w.jpg)




完整代码参考

[raw](http://www.sthda.com/english/wiki/cox-proportional-hazards-model)
[wechat link](https://mp.weixin.qq.com/s?__biz=MzI4ODE0NTE3OA==&mid=2649207612&idx=1&sn=af4480273740bd1c397bf1e7a050497c&chksm=f3d1f949c4a6705f6ff777bae3ef4a11e4a61693c55d59fa7131eb236da253a4b07157067254&token=1784749928&lang=zh_CN#rd)
​
