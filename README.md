# 筹码分布

## 基本概念
“筹码分布”的准确的学术名称应该叫“流通股票持仓成本分布”。
![](https://i.imgur.com/Lrof5o6.png)
右边的哪个柱状图就是筹码分布，体现的现有流通股票再股市中的分布状况。


## 相关概念

### 获利盘
获利盘是股票术语其中之一。获利盘一般是指股票交易中，能够卖出赚钱的那部分股票。每一只股票都有获利盘和套牢盘，套牢盘就是买入的股票亏本。
也就是说
![](https://i.imgur.com/w1mtZBv.png)


### 成本分布
COST(10),表示10%获利盘的价格是多少,即有10%的持仓量在该价格以下,其余90%在该价格以上,为套牢盘，该函数仅对日线分析周期有效。
这个也是根据筹码分布就可以简单计算，cost和winner就像两个相反的东西


总之，筹码分布是形态分布计算的根本

## 如何计算
这里简单讲解两个简单的计算

**基本的一些概念**

- 平均分布：将当日的换手筹码在当日的最高价和最低价之间平均分布。


- 三角形分布：将当日的换手筹码在当日的最高价、最低价和平均价之间三角形分布。

![](https://i.imgur.com/Y7XyZpD.jpg)

- 历史换手衰减系数：它是一个常数参数，我们用来赋予今天换手率，也既是当日被移动的成本的权重。如果今天的换手率是A，衰减系数是n，那么我们计算昨日的被移动的筹码的总量是A*n,如果n取值为1，就是一般意义上理解的今天换手多少，就有多少筹码被从作日的成本分布中被搬移；如果n是2，那么我们就放大了作日被移动的筹码的总量..这样的目的在于突出“离现在越近的筹码分布其含义越明显”。


每日成本算法是一个移动平均过程：
**公式是**：当日成本*（换手率*历史换手衰减系数）+上一日成本分布图*（1-换手率*历史换手衰减系数）


代码：(这里只是部分运算代码， 项目源码放在后面)
```

low = data['low']
high = data['high']
vol = data['volume']
TurnoverRate = data['TurnoverRate']
avg = data['avg']
date = data['date']

Chip ={}
ChipList ={}
Profit =[]


flag = True # start
import time
def calcuJUN(dateT,highT, lowT, volT, TurnoverRateT, A, minD):
#     print(highT, lowT, volT)
    
    x =[]
    
    l = (highT - lowT) / minD
    for i in range(int(l)):
        x.append(round(lowT + i * minD, 2))
    length = len(x)
    eachV = volT/length
    for i in Chip:
        Chip[i] = Chip[i] *(1 -TurnoverRateT * A)
    for i in x:
        if i in Chip:
            Chip[i] += eachV *(TurnoverRateT * A)
        else:
            Chip[i] = eachV *(TurnoverRateT * A)
    import copy
    print(dateT)
    ChipList[dateT] = copy.deepcopy(Chip)
            

    
def calcuSin(dateT,highT, lowT,avgT, volT,TurnoverRateT,minD,A):    
    x =[]
    
    l = (highT - lowT) / minD
    for i in range(int(l)):
        x.append(round(lowT + i * minD, 2))
    
    length = len(x)
    
    
    #计算仅仅今日的筹码分布
    tmpChip = {}
    eachV = volT/length
    
    
    #极限法分割去逼近
    for i in x:
        x1 = i
        x2 = i + minD
        h = 2 / (highT - lowT)
        s= 0
        if i < avgT:
            y1 = h /(avgT - lowT) * (x1 - lowT)
            y2 = h /(avgT - lowT) * (x2 - lowT)
            s = minD *(y1 + y2) /2
            s = s * volT
        else:
            y1 = h /(highT - avgT) *(highT - x1)
            y2 = h /(highT - avgT) *(highT - x2)
            
            s = minD *(y1 + y2) /2
            s = s * volT            
        tmpChip[i] = s
            
    
    for i in Chip:
        Chip[i] = Chip[i] *(1 -TurnoverRateT * A)
        
    for i in tmpChip:
        if i in Chip:
            Chip[i] += tmpChip[i] *(TurnoverRateT * A)
        else:
            Chip[i] = tmpChip[i] *(TurnoverRateT * A)
    import copy
    print(dateT)
    ChipList[dateT] = copy.deepcopy(Chip)
            

def calcu(date,highT, lowT,avgT, volT, TurnoverRateT,minD = 0.01, flag =1, AC=1):  #flag 使用哪个计算方式,    AC 衰减系数
    if flag ==1:
        calcuSin(dateT,highT, lowT,avgT, volT, TurnoverRateT,A=AC, minD=minD)
    elif flag ==2:
        calcuJUN(dateT,highT, lowT, volT, TurnoverRateT, A=AC, minD=minD)

    
for i in range(len(date)):
#     if i < 90:
#         continue
    
    highT = high[i]
    lowT = low[i]
    volT = vol[i]
    TurnoverRateT = TurnoverRate[i]
    avgT = avg[i]
    print(date[i])
    dateT = date[i]
    calcu(dateT,highT, lowT,avgT, volT, TurnoverRateT/100, flag=1)  # 东方财富的小数位要注意，兄弟萌。我不除100懵逼了
    
```

## 源码（实践出真知）
来代码，就很清楚了

首先获取数据， 我这里使用平安银行本地数据
![](https://i.imgur.com/SCbp9GR.png)
格式是
![](https://i.imgur.com/xD0H4Bx.png)

- 首先加载数据
- 然后计算筹码分布
```
    def calcu(self,dateT,highT, lowT,avgT, volT, TurnoverRateT,minD = 0.01, flag=1 , AC=1):
        if flag ==1:
            self.calcuSin(dateT,highT, lowT,avgT, volT, TurnoverRateT,A=AC, minD=minD)
        elif flag ==2:
            self.calcuJUN(dateT,highT, lowT, volT, TurnoverRateT, A=AC, minD=minD)

    def calcuChip(self, flag=1, AC=1):  #flag 使用哪个计算方式,    AC 衰减系数
        low = self.data['low']
        high = self.data['high']
        vol = self.data['volume']
        TurnoverRate = self.data['TurnoverRate']
        avg = self.data['avg']
        date = self.data['date']

        for i in range(len(date)):
        #     if i < 90:
        #         continue

            highT = high[i]
            lowT = low[i]
            volT = vol[i]
            TurnoverRateT = TurnoverRate[i]
            avgT = avg[i]
            print(date[i])
            dateT = date[i]
            self.calcu(dateT,highT, lowT,avgT, volT, TurnoverRateT/100, flag=flag, AC=AC)  # 东方财富的小数位要注意，兄弟萌。我不除100懵逼了
```
- 使用筹码分布计算获利盘
![](https://i.imgur.com/i7tNBPU.png)

![](https://i.imgur.com/Oq8qOHU.png)
这是计算结果，可以看出，图形和通达信基本一致。
然后比对每一天的具体数值后， 基本偏差也不大于10%。和东方财富，同花顺等平台比对后也是基本精准的。

- 三角分布使用极限逼近的方法去逼近，划分成一个个小区
仓库地址：[github](https://github.com/2892211452/ChipDistribution)

## 总结
这里的筹码分布本质上其实也只是一个概率模型，能够较好的反应实际的筹码分布。通过三角分布或者均匀分布去进行运算，实践中可以考虑加入更多trick。

最后，这些指标，都只是参考。。。。