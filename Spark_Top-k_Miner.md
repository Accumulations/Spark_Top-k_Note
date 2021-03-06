#基于Spark的Top-k对比序列模式挖掘
##对比序列模式
######定义
指在目标类序列集合中频繁出现，而在非目标类序列集合中不频繁出现的序列。
######作用
能够识别不同类别序列样本集合间的差异，并描述各类别样本集合的特征。

比如：

- 在医学领域分析阳性肿瘤与阴性肿瘤的DNA序列，识别对比序列模式，有助于提高临床肿瘤预测和诊断的精度

##对比序列模式挖掘相关概念
######间隔约束
- 定义：是一个由两个非负整数确定的区间，表示序列模式中两个相邻元素间允许间隔的元素数目的最小值和最大值
- 目的：让模式的匹配更加灵活

######支持度阙值
- 目的：通过支持度计算，找出满足支持度阙值的序列

##传统的基于支持度阙值的对比序列模式挖掘算法
######要求
传统序列模式挖掘算法要求用户设置正例支持度阙值a、负例支持度阙值b和间隔约束r。
######目标
挖掘出正例支持度大于等于a,并且负例支持度小于等于b的最小化模式
######缺点
1. 在不具备足够的先验知识的情况下，用户难以设置恰当的支持度阙值，这可能导致挖掘不到有用的结果
2. 使用最小化约束进行剪枝，虽然减少了搜索空间，但可能会丢失一些对比显著的模式
3. 若反复调整支持度阙值，必然增加用户使用算法难度

##带有间隔约束的基于top-k对比序列模式挖掘算法：kDSP-Miner算法
######要求
给定需挖掘对比模式个数k，间隔约束r
######目的
挖掘在全部对比模式中支持度对比最显著的k个对比序列模式
######缺点
随着数据采集技术的发展，各领域获取数据的规模越来越大，kDSP-Miner算法基于单台计算机设计，难以满足大规模数据挖掘的需求。

##基于支持度阙值和基于top-k算法区别
######挖掘目标不同
带间隔约束的top-k对比序列模式是指支持度变化最为显著的k个对比序列模式，而带间隔约束的满足支持度阙值的对比模式指的是满足支持度阙值的对比序列模式
######挖掘过程不同
带间隔约束的top-k对比序列模式不会使用最小化约束来进行剪枝，保证了支持度变化显著的对比序列模式不会由于其子序列满足条件而被剪掉
######应用难度不同：
在挖掘带间隔约束的top-k对比序列模式时，用户仅需设定期望得到的模式的数目，而不需要设定具体的正例支持度阙值和负例支持度阙值。一般来说，设定k值比较直观，因而不会由于没有足够的先验知识，设定不合适的支持度阙值，导致丢失一些对比显著的模式

##基于Spark的Top-k对比序列模式：SP-kDSP-Miner算法
######目的
采用并行计算技术，设计并行挖掘算法，实现从大规模序列数据中发现top-k对比序列模式
######挑战
1. 候选对比序列模式生成
2. 并行计算方法
3. 剪枝策略

##本次论文的主要工作


##问题定义
######某些符号概念
- ∑: 序列元素的集合
- S: 序列集中的序列 S=e1e2...en
    - S的最大前缀: `pre(S)=e1e2...e[n-1]`
    - S的最大后缀: `suf(S)=e2e3...en`
- |S|: 序列的长度
- S[i],S[j]之间元素的间隔: `gap(S,i,j)=j-i-1`
- γ: 间隔约束  `整数区间γ=[γ.min,γ.max]`
- S子序列: `S‘=S[k1]S[k2]...S[km]`，其中1<=k1<k2<...<km<=|S|
    - 若k[i] - k[i-1] -1 =0,则称S‘是序列S的连续子序列
    - 便于描述，将S‘用S的下标表示为<k1,k2,...,km>,称是S’ 在S中的一个实例
- S'⊆γS: 若序列S的子序列S'存在实例<k1,k2,...,km>，满足`γ.min<=k[i]-k[i-1]<=γ.max，则称序列S的子序列S'满足间隔约束γ
- 序列P在D中的支持度: `Sup(P,D,γ)=|{S∈D|P⊆γS}| / |D|`
    - D: 给定序列样本集合
- 序列P在D+与D-间的对比度：`CR(P,D+,D-,γ)=Sup(P,D+,γ)-Sup(P,D-,γ)`
    - D+: 给定正序列集
    - D-: 负序列集
- 简化`CR(P,D+,D-)=CR(P,D+,D-,γ)`，由于γ不会变化

######定义
1. 对比序列模式: 若`CR(P,D+,D-)>0`，则P为一个带间隔约束γ的对比序列模式
2. top-k对比序列模式： 对比度最大的k个对比序列模式
3. 注意：若有多个模式的对比度相同，则按2条规则排序对比度
    - 优先选择长度更短的模式
    - 若长度相同，则按正例序列集合中元素出现顺序选择排序靠前的模式
    - 用户也可根据实际情况修改规则以适应具体要求

##SP-kDSP-Miner算法设计
######算法要点
1. Spark架构下候选对比序列模式生成
2. 利用Spark集群计算候选对比序列模式的对比度
3. Spark架构下的候选模式剪枝策略

######算法流程图
![SP-kDSP-Miner算法流程图](/Users/fang/Documents/github-file/Spark_Top-k_Note/Flow chart of SP-kDSP-Miner.jpg)

####候选对比序列模式生成
设计使用于Spark架构的集合枚举树遍历方式和剪枝策略是算法执行效率的基础

> 引理1：  若Sup(P,D+)<=σ(σ>0) , 则CR(P,D+,D-)<=σ

> 定理1：  若Sup(P,D+)=0 , 则P不是对比序列

> ==>剪枝策略1： 若Sup(P,D+)=0，那么剪去集合枚举树中P的所有子节点

> 定理2： P'是P的连续子序列，若Sup(P',D+)<σ，则P的正例支持度Sup(P,D+)<=Sup(P',D+)<σ

> ==>剪枝策略2： 令R为当前对比度最大的k个对比序列模式集合，CRmin = min{CR(P'',D+,D-)|P''∈R}。若Sup(P,D+)<CRmin，那么剪去集合枚举树中P的所有子节点

######Spark并行计算的开销
1次作业的开销主要包括：作业调度开销、作业通信开销、作业计算开销。所以我们应该尽可能产生相互不影响（没有剪枝关系）的候选对比序列模式，进而在1次Spark作业中可以并行处理多个候选模式，充分利用Spark集群的计算能力，提高计算效率

######广度优先遍历策略

> 目的: 在生成长度为l的候选对比序列模式之前会先生成所有长度小于l的候选模式

> 注意： 长度相等的候选模式，它们之间不存在子序列关系，因此不会出现相互剪枝的情况，而1次Spark作业中可以并行处理所有长度为l的候选对比序列模式。

######基于广度优先遍历集合枚举树策略，设计候选模式生成方法
给定个候选对比序列模式P,Q(|P|=|Q|=l)，如果pre(P)=suf(Q),则有P,Q可生成第l+1层的对比候选序列模式，表示为P⊕Q = Q[1]P[1]P[2]...P[l]=Q[1]Q[2]...Q[l]P[l]

![集合枚举树示例](/Users/fang/Documents/github-file/Spark_Top-k_Note/An example of a set enumeration tree.jpg)

> SP-kDSP-Miner生成候选模式算法的伪代码

> 算法1. GENERATE(C[l])算法

> 输入：长度为l的候选对比序列模式的集合C[l]、当前对比度最大的k个对比序列模式中最小的对比度CRmin

> 输出：长度为l+1的候选对比序列模式集合C[l+1]
![SP-kDSP-Miner生成候选模式算法的伪代码](/Users/fang/Documents/github-file/Spark_Top-k_Note/SP-kDSP-Miner code.jpg)

####对比度并行计算

![Contrast calculation process in SP-kDSP-Miner](/Users/fang/Documents/github-file/Spark_Top-k_Note/Contrast calculation process in SP-kDSP-Miner.jpg)
![Contrast calculation process](/Users/fang/Documents/github-file/Spark_Top-k_Note/Contrast calculation process.jpg)


> 剪枝策略3：在对比度计算过程中，对于候选模式P，若Sup(P,D+)<CRmin，则把模式P从C[l]中移除--令C[l]是长度为l的候选对比序列模式集合

######剪枝策略3与剪枝策略2不同点
虽然两者都基于定理2，但它们的执行时间与剪枝对象不同

- 剪枝策略3在对比度计算过程中生效，即在剪枝跳槽(CRmin)变化前执行。这是因为计算对比度时使用并行方法，为了降低通信开销，只能先计算正例支持度，这样剪枝策略3可以避免不必要的负例支持度计算，而完成对比度计算之后，会更新全局的结果集R，剪枝条件（CRmin）也会随之更新
- 在生成新的候选对比序列模式时剪枝策略2生效，避免生成无意义的候选模式

####复杂度分析及负载均衡
######SP-kDSP-Miner伪代码
![SP-kDSP-Miner伪代码](/Users/fang/Documents/github-file/Spark_Top-k_Note/SP-kDSP-Miner伪代码.jpg)

######负载均衡
基于Spark框架的分布式算法的运行时间取决于计算时间最长的计算节点，即符合木桶效应。因此，实现各计算节点的负载均衡有利于降低算法总执行时间。

方法：为保证计算节点负载均衡，各计算节点载入的序列数据大小应该尽量相同。因此，在对D+,D-进行分片时，我们采用均分策略，即将数据集分片为与计算节点数量相同的等份载入节点

下一步工作：考虑计算节点的负载动态均衡，即允许节点的负载在算法执行过程中动态调整

##实验
