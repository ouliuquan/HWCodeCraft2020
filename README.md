# HWCodeCraft2020
2020华为软件精英挑战赛，杭夏赛区-小白兔奶糖

### 成绩

- 复赛A2：2.4388 （总榜Rank13）
- 复赛B： 2.8213（总榜Rank3）
- 决赛A：324.3789 （Rank9）
- 决赛B：316.9556 （Rank13）

### 初赛/复赛 

#### 题意

在有向图中找到所有指定长度的环，详见我写的[Baseline](https://zhuanlan.zhihu.com/p/125764650)

#### 初赛/复赛A

初赛与复赛A榜几乎没有区别，就是数据规模不一样。

具体的思路和其他组都差不多：

- 利用前向星方式存图并重建
- 二级存储保存反向3层路径
- 4+3组合递归/for套娃搜索
- 路径组合的方式有两种：1234,4+1,4+2,4+3 或 0+3,1+3,2+3,3+3,4+3

#### 复赛B

复赛B榜临时修改了两个需求，最大环长修改为8，整数金额变为最多两位小数。

因为时间只有两个半小时，所以草草的修改了4+3方案为4+4，实现可能有很多不太合理的地方



距离复赛结束已经有一段时间了，代码中出现的trick基本已经在大佬们的分析中出现过，在此就不再赘述。



### 决赛

#### 题意

计算有向图中每个点的中介中心性

#### 基础思路

参考2001年的论文，**A Faster Algorithm for Betweenness Centrality**，存在一种使用线性空间，计算结点数次单源最短路的求解方法。

#### 数据特性

决赛的数据一共有三组：

1. 菊花图（无标度网络）：存在大量入度为0，出度为1的结点
2. 随机图 ：所有结点组成一个SCC
3. 第一组+第二组：图中一共两个WCC

此外，转账金额普遍较小，所有数据的转账金额小于100（包括样例与AB榜在线评测数据，可能天地银行在之前的比赛被掏空了吧），利用数据的特性，可以设计一些神奇的数据结构和计算方法，从而将算法的时间由Baseline的700+提升到可怕的200+。

一些已知但是我们组并没有直接使用的结论，其中最后两条也是最后成绩和前排相差巨大的主要原因：

- B榜当天所有路径长度不超过65535，故short可过，不需要类型适配
- float的精度足以应付B榜在线的数据
- 最短路DAG中，所有点的前驱数量不会超过5
- 第二组随机图，90%以上的点只有一个前驱

#### 改进思路-A榜

- 参考论文**Fast exact computation of betweenness centrality in social networks**可以对入度为0，出度为1的结点做特殊处理，对图1的提升巨大。这个流程可以是递归的，但是实际上只删除一层是最好的 （从而得到720+的Baseline）
- 根据转账金额较小导致路径长度较为集中这一点，可以只在优先队列中保存距离而不保存结点，将具体的结点映射为一个结点列表，从而大大减小堆的规模（提升260+）
- 根据转账金额较小的这一点，可以使用动态的数据类型，如自动切换的uint_64,uint_32,uint_16（提升100+）
- 根据一定的顺序重排图结构（如BFS序），对于局部稠密的图有非常好的效果（图1）（提升25+）

A榜的实现和IOT_TEAM的比较类似，相较而言他们的实现具有更好的Cache性能（故在B榜的成绩要好很多），具体请参考[IOT_TAEM主代码手的Github](https://github.com/Chadriy/CodeCraft2020)

#### 改进思路-B榜

- A榜代码得分339
- 利用转账金额不会超过100，最短路中最小值单调递增的特性，可以设计一个数据结构从而抛弃优先队列（**感谢Claris**），具体而言，即将A榜的优先队列+结点保存修改为枚举距离+结点保存的方案，由于距离范围较大，故对距离按照后7位进行哈希，由于金额不超过100，遍历当前点时不会出现当前点所在列表的结点更新。详情参见代码（在线提升10+）
- 使用结构体或ull数组代替原先的双数组方案来存储路径数和前驱下标，可以有更好的Cache性能（提升10+）

此外，参照鸽鸽鸽提出的数据特性（上述最后两条），构建DAG所使用的前驱结点列表可以使用二维前驱数组，或配合特殊处理前继数为1，记录父节点的方案，从而大大降低Cache miss，性能得到较大提升（可惜在比赛的过程中没有想到这一点，提升参考鸽鸽鸽的最终成绩）。

后来按照鸽鸽鸽的思路进行了复现（写的比较挫），本地第一组数据3.6s，第二组数据68s，第三组数据37s（详情参见代码）



### 总结

因为起了这个队名，快要被小白兔打死了（逃



Cache的重要性有些时候甚至大于算法的复杂度



两位队友都很忙，起到的作用比较有限，很多时候还不如其他队伍的同学偶尔给与的一点帮助（还是竞争对手），因此比赛全程基本为队长一个人在solo，承担了写程序调Bug交流思路造数据水群等各项工作，到比赛后期（主要是决赛阶段）渐渐力不从心了，以致于代码的问题还是在和其他队伍的交流过程中发现的（也算是后期乏力的一个主要原因），最后没能在全国总决赛中取得较好的成绩，感觉有些可惜。明年如果还有足够的时间和精力，还会继续参加。



通过这次比赛认识到了很多优秀的同学，也学到了不少（可能没啥用）的神奇魔法，收获颇丰。在此特别感谢以下队伍在比赛过程中提供的无私帮助：

- 成渝赛区：IOT_TEAM
- 上合赛区：好想去冰岛看极光呀
- 江山赛区：我是一条大锦鲤，大锦鲤！
- 杭夏赛区：#507，鸽鸽鸽（Claris）

感谢在比赛过程中认识的每一位同学。



