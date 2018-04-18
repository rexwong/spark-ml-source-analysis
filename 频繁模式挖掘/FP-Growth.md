# FP-growth

关联规则挖掘的一个典型例子是购物篮分析。关联规则研究有助于发现交易数据库中不同商品（项）之间的联系，找出顾客购买行为模式，如购买了某一商品对购买其他商品的影响，分析结果可以应用于商品货架布局、货存安排以及根据购买模式对用户进行分类。

关联规则的相关术语如下：

1. **项与项集**

   这是一个集合的概念，在一篮子商品中的一件消费品即为一项（Item），则若干项的集合为项集，如$\left \{ 尿布,啤酒 \right \}$构成一个二元项集。

2. **关联规则**

   一般记为的形式，$X$为先决条件，$Y$为相应的关联结果，用于表示数据内隐含的关联性。如：表示购买了尿布的消费者往往也会购买啤酒。

   关联性强度如何，由三个概念—**支持度、置信度、提升度**来控制和评价。

   *例：有10000个消费者购买了商品，其中购买尿布1000个，购买啤酒2000个，购买面包500个，同时购买尿布和面包800个，同时购买尿布和面包100个。*

   * **支持度（Support）**

     **支持度是指在所有项集中$\left \{ X,Y \right \}$出现的可能性，即项集中同时含有X和Y的概率**：

     该指标作为建立强关联规则的第一个门槛，衡量了所考察关联规则在“量”上的多少。通过设定最小阈值(minsup)，剔除“出镜率”较低的无意义规则，保留出现较为频繁的项集所隐含的规则。

     设定最小阈值为5%，由于$\left \{ 尿布,啤酒 \right \}$的支持度为$\frac{800}{10000}=8\%$，满足基本输了要求，成为频繁项集，保留规则；而$\left \{ 尿布,啤酒 \right \}$的支持度为$\frac{100}{10000}=1\%$，被剔除。

   * **置信度（Confidence）**

     置信度表示在先决条件$X$发生的条件下，关联结果$Y$发生的概率：

     这是生成强关联规则的第二个门槛，衡量了所考察的关联规则在“质”上的可靠性。相似的，我们需要对置信度设定最小阈值（mincon）来实现进一步筛选。

     具体的，当设定置信度的最小阈值为70%时，置信度为$\frac{800}{1000}=80\%$，而的置信度为$$\frac{800}{2000}=40\%$$，被剔除。

   * **提升度（lift）**

     提升度表示在含有$X$的条件下同时含有$Y$的可能性与没有$X$这个条件下项集中含有$Y$的可能性之比：公式为$confidence(artichok => cracker)/support(cracker) = \frac{80\%}{50\%} = 1.6$。该指标与置信度同样衡量规则的可靠性，可以看作是置信度的一种互补指标。

## FP-Tree的构造：

>| TID  | Items                   |
>| ---- | ----------------------- |
>| 001  | Cola, Egg, Ham          |
>| 002  | Cola, Diaper, Beer      |
>| 003  | Cola, Diaper, Beer, Ham |
>| 004  | Diaper, Beer            |
>
>TID代表交易流水号，Items代表一次交易的商品。
>
>我们对这个数据集进行关联分析，可以找出关联规则{Diaper}→{Beer}。 
>它代表的意义是：购买了Diaper的顾客会购买Beer。这个关系不是必然的，但是可能性很大，这就已经足够用来辅助商家调整Diaper和Beer的摆放位置了，例如摆放在相近的位置，进行捆绑促销来提高销售量。

1. **`事务`**：每一条交易称为一个事务，例如示例1中的数据集就包含四个事务。 
2. **`项`**：交易的每一个物品称为一个项，例如Cola、Egg等。 
3. **`项集`**：包含零个或多个项的集合叫做项集，例如$\left \{ Cola,Egg,Ham \right \}$。 
4. **`k−项集`**：包含k个项的项集叫做k-项集，例如$\left \{ Cola \right \}$叫做1-项集，$\left \{ Cola,Egg \right \}$叫做2-项集。 
5. **`支持度计数`**：一个项集出现在几个事务当中，它的支持度计数就是几。例如$\left \{ Diaper,Beer \right \}$出现在事务002、003和004中，所以它的支持度计数是3。 
6. **`支持度`**：支持度计数除于总的事务数。例如上例中总的事务数为4，$\left \{ Diaper,Beer \right \}$的支持度计数为3，所以它的支持度是$\frac{3}{4}=75\%$，说明有75%的人同时买了Diaper和Beer。 
7. **`频繁项集`**：支持度大于或等于某个阈值的项集就叫做频繁项集。例如阈值设为50%时，因为{Diaper, Beer}的支持度是75%，所以它是频繁项集。 
8. **`前件和后件`**：对于规则{Diaper}→{Beer}，{Diaper}叫做前件，{Beer}叫做后件。 
9. **`置信度`**：对于规则{Diaper}→{Beer}，{Diaper, Beer}的支持度计数除于{Diaper}的支持度计数，为这个规则的置信度。例如规则{Diaper}→{Beer}的置信度为$\frac{3}{3}=100\%$。说明买了Diaper的人100%也买了Beer。 
10. **`强关联规则`**：大于或等于最小支持度阈值和最小置信度阈值的规则叫做强关联规则。关联分析的最终目标就是要找出强关联规则。


1. FP-Growth算法的任务是找出数据集中的频繁项集。 
2. FP-Growth算法的步骤，大体上可以分成两步：
   1. FP-Tree的构建； 
   2. FP-Tree上频繁项集的挖掘。

## 构建步骤

1. 扫描一遍数据库，找出频繁项的列表 $L$，按照支持度计数递减排序。即：

   $L =\left \{ (Cola:3), (Diaper:3), (Beer:3), (Ham:2) \right \} $

2. 再次扫描数据库，由每个事务不断构建FP-Tree： 

   1. FP-Tree的根节点为 null 。
   2. 从数据库中取出事务，按照 $L$排序，然后把每个项逐个添加到FP-Tree的分枝上去。例如，事务001排序后为{Cola, Ham}，在根节点上加一棵子树Cola-Ham。事务002排序后为{Cola, Diaper, Beer}，因为根节点上已经有个子树节点“Cola”，所以可以共用该节点，在Cola节点上加一棵子树Diaper-Beer，同时Cola的计数加1。事务003可与树共用节点Cola-Diaper-Beer，所以只需在Beer后面加个子树节点“Ham”，同时把Cola、Diaper、Beer的计数加 1 即可。·········

3. FP-Tree还有一样东西：头结点表。作用是将所有相同的项链接起来，这样比较容易遍历。 

**构造FP-Tree的伪代码如下**：

```
算法：FP-Tree构造算法 
输入：事务数据集 D，最小支持度阈值 min_sup 
输出：FP-Tree 
(1) 　　扫描事务数据集 D 一次，获得频繁项的集合 F 和其中每个频繁项的支持度。对 F 中的所有频繁项按其支持度进行降序排序，结果为频繁项表 L ; 
(2) 　　创建一个 FP-Tree 的根节点 T，标记为“null”; 
(3) 　　for 事务数据集 D 中每个事务 Trans do 
(4) 　　　　对 Trans 中的所有频繁项按照 L 中的次序排序; 
(5) 　　　　对排序后的频繁项表以 [p|P] 格式表示，其中 p 是第一个元素，而 P 是频繁项表中除去 p 后剩余元素组成的项表; 
(6) 　　　　调用函数 insert_tree( [p|P], T ); 
(7) 　　end for 

insert_tree( [p|P], root) 
(1) 　　if root 有孩子节点 N and N.item-name=p.item-name then 
(2) 　　　　N.count++; 
(3) 　　Else 
(4) 　　　　创建新节点 N; 
(5) 　　　　N.item-name=p.item-name; 
(6) 　　　　N.count++; 
(7) 　　　　p.parent=root; 
(8) 　　　　将 N.node-link 指向树中与它同项目名的节点; 
(9) 　　end if 
(10)　　if P 非空 then 
(11)　　　　把 P 的第一项目赋值给 p，并把它从 P 中删除; 
(12)　　　　递归调用 insert_tree( [p|P], N); 
(13)　　end if
```



```python
#FP树
class treeNode:
    def __init__(self, nameValue, numOccur, parentNode):
        self.name = nameValue
        self.count = numOccur#计数器
        self.nodeLink = None#用于链接相似的元素项
        self.parent = parentNode#指向父节点，需要被更新
        self.children = {} #存放子节点
    
    def inc(self, numOccur):
        self.count += numOccur
        
    def disp(self, ind=1):
        print '  '*ind, self.name, ' ', self.count
        for child in self.children.values():
            child.disp(ind+1)
```



```python
def createTree(dataSet, minSup=1): #数据集，最小支持度
    headerTable = {}
    #遍历数据集两次
    for trans in dataSet:#统计
        for item in trans:
            headerTable[item] = headerTable.get(item, 0) + dataSet[trans]
    for k in headerTable.keys():  #移除不满足最小支持度的元素项
        if headerTable[k] < minSup: 
            del(headerTable[k])
    freqItemSet = set(headerTable.keys())
    #print 'freqItemSet: ',freqItemSet
    if len(freqItemSet) == 0: return None, None  #都不满足，则退出
    for k in headerTable:
        headerTable[k] = [headerTable[k], None] #修改为下一步准备
    #print 'headerTable: ',headerTable
    retTree = treeNode('Null Set', 1, None) #根节点
    for tranSet, count in dataSet.items():  #第二次遍历
        localD = {}
        for item in tranSet:  #排序
            if item in freqItemSet:
                localD[item] = headerTable[item][0]
        if len(localD) > 0:
            orderedItems = [v[0] for v in sorted(localD.items(), key=lambda p: p[1], reverse=True)]
            updateTree(orderedItems, retTree, headerTable, count)#用排序后的频率项进行填充
    return retTree, headerTable 

def updateTree(items, inTree, headerTable, count):
    if items[0] in inTree.children:#检查第一个元素是否作为子节点存在
        inTree.children[items[0]].inc(count) #更新计数
    else:   #新建一个子节点添加到树中
        inTree.children[items[0]] = treeNode(items[0], count, inTree)
        if headerTable[items[0]][1] == None: 
            headerTable[items[0]][1] = inTree.children[items[0]]
        else:
            updateHeader(headerTable[items[0]][1], inTree.children[items[0]])
    if len(items) > 1:#对剩下的元素项迭代调用，每一次奥调用去掉第一个元素
        updateTree(items[1::], inTree.children[items[0]], headerTable, count)
        
def updateHeader(nodeToTest, targetNode):   #确保节点链接指向树中该元素项的每一个实例
    while (nodeToTest.nodeLink != None):    #直达链尾
        nodeToTest = nodeToTest.nodeLink
    nodeToTest.nodeLink = targetNode
```
