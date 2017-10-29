# A Professional Quant Equity Workflow

# 一个专业化的股权量化工作流程


---


By Jonathan Larkin, Chief Investment Officer at Quantopian


[原文链接](https://blog.quantopian.com/a-professional-quant-equity-workflow/)


在前面的[文章](https://blog.quantopian.com/the-foundation-of-algo-success/)中,我阐述了一个高夏普比率(Sharpe ratio)<img src="http://latex.codecogs.com/gif.latex?$^{[1]}$" />的量化投资策略的哲学基础.


今天我们将更加细致了解当下流行的投资流程,深入到量化投资的世界：cross-sectional equity investing,也被称作**股权统计套利**或者**股权市场中性投资**.
这种股权投资通常就是：在一个有高度风控的投资组合里持有数百只股票的多头和空头头寸,其目标是捕捉瞬间的市场异常,这种异常和市场的方向或者主要的风险因素没有相关性.


如果你想知道全世界最大的对冲基金里宽客精英是如何度过他们每一天的,请继续读下去吧：


所有的cross-sectional策略,可以被提炼成6个阶段：
- Data
- Universe Definition
- Alpha Discovery
- Alpha Combination
- Portfolio Construction
- Trading

每个阶段的成功是策略成功的必要条件,但不是充分条件.只有在每个阶段中审慎和周到做好决策,才能使得策略能良好的运作.流程图如下：
![image](https://github.com/kite8/Quantopian-lectures-notebook-translation/blob/master/image/workflow.jpg)

## Data (数据) ##


宽客都是数据驱动和data-informed类型的投资者.所有的量化投资都始于数据.幸运的是,Quantopian已经帮你完成了跑腿的粗活和和已经清洗过,有标记,symbol mapped, joined across vendors, and constructed, where possible, point-in-time数据集. 第一步,你需要回答一个简单的问题:"包含哪些信息的数据集能对预估未来收益更有帮助?"如果你要去挖金子,你得知道从哪开始挖.


## Universe Definition (证券池的设计) ##

今天Quantopian的历史价格数据已经涵盖了约8000只美国在交易所上市的证券.在推动策略这方面事情之前(做策略相关的事情更具吸引力),首先你必须要筛掉一部分上市证券,得到你所满意并用于交易的证券,例如,你的交易证券池.Q500US和Q1500US刚刚发布,您可以使用其中之一.或利用这些中的底层机制来帮助你风格生带有自己风格的自定义证券池(universe).

你也许会问,"为什么要做这些自我限制?难道不是使用所有可用数据这样更好吗?在你前面的文章中不是讨论过,要尽量利用"广度"吗?"

首先我们要从实际的情况出发,比如筛掉流动性不足的股票.对于筛选这件事,这里有一个不那么明显但是很关键的理由,成功的cross-sectional策略会让证券池的证券价格走势不要太相似,也不要差太远,保持一定的平衡.为了理性的对证券排名,cross-sectional策略需要衡量证券的相对价值,对用于排名维度数据需要做标准化处理.(归一化,特征缩放)

在设计证券池时,你必须有自己的投资理念.想象一下接下来两个例子的场景.如果你的策略是基于股票隔夜信息内容来进行日内交易<img src="http://latex.codecogs.com/gif.latex?$^{[2]}$" />,对于ADRs这类证券必须清理掉.当信息扩散到ADR原发行地时(存在时差),我们的策略是依赖于美国证券交易所交易价格的投资者行为,这点在逻辑是不一致的.第二个例子,如果你的策略是基于财报数据,例如应计异象<img src="http://latex.codecogs.com/gif.latex?$^{[3]}$" />,你必须剔除掉不适用这类标准的股票(在这个例子中,比如银行股).

## Alpha Discovery (Alpha挖掘/因子挖掘) ##
alpha可以理解为,当证券池中股票用cross-section策略进行交易时,以每只股票的相对回报组成的一个实数向量.一个alpha可以从一个线性序列中构建,也可以是没有维度的向量构建.Alphas也被叫做因子,在Quantopian中这两种叫法都是可以的.[Pipeline API](https://www.quantopian.com/tutorials/pipeline)将带你走进alpha建模的世界.在这一阶段,不要去考虑真实世界中的因素,比如交易,佣金或者风险.创建关于投资者行为,市场结构,信息不对称或市场低效率的任何其他潜在原因的假设,并看看该假设是否具有预测能力.需要一些点子?去Google或者[SSRN search](https://papers.ssrn.com/sol3/DisplayAbstractSearch.cfm)搜"equity market anomalies"吧.

Alpha研究是艺术和科学的结合,也往往会发生神奇的事情.Alpha研究是一个不断迭代的过程：提出假设,检验假设,分析问题,修正假设.我们近期发布了一个新的开源计划,目前正在Quantopian Research进行测试,被称为[`alphaens`](https://www.quantopian.com/posts/alphalens-a-new-tool-for-analyzing-alpha-factors).你可以用`Pipeline API`去表达你的alpha,用`alphalens`去评估其效用.

## Alpha Combination (Alpha聚合/因子聚合) ##
在今天的市场,很难用单因子(alpha)模型去撑起一个投资策略.一个成功的策略通常会有许多个独立的因子(alpha)；如果这些因子(alpha)够厉害,只需少量的因子就足矣.本阶段的目标是将多个正则化的因子(alpha)通过加权的方式,最终得到一个单因子,这个因子比之前最好的单因子预测能力还要强.加权的方式可以很简单：有时候通过加一个排名或者对因子(alpha)进行求平均,就是一个不错的解决方案.事实上,一个很流行的模型只是把[两个因子(alpha)](https://www.amazon.com/Little-Book-Still-Beats-Market/dp/0470624159/ref=sr_1_1?ie=UTF8&qid=1471483267&sr=8-1&keywords=the+little+book+that+beats+the+market)进行聚合.随着复杂度的增加,一些经典的投资组合理论能帮上忙；比如,在因子最终聚合加权方式上可以选择[马科维茨的均值一方差组合模型(lowest possible variance)](https://www.quantopian.com/posts/the-efficient-frontier-markowitz-portfolio-optimization-in-python-using-cvxopt). 最后(敲黑板！),现代的机器学习技术(深度学习&强化学习?)能帮助我们捕捉因子间复杂的关系.将你的因子转换成特征值,然后用机器学习算法去做分类,是当前很流行的一个研究方向<img src="http://latex.codecogs.com/gif.latex?$^{[4,5]}$" />.

## Portfolio Construction (创建投资组合) ##
到这个阶段之前,我们都只是在做研究,没有进行实操.之前的步骤都是在研究,这项工作最好在非结构化的Quantopian [Research environment](https://www.quantopian.com/research),因为这里可以快速验证你的想法.此时问题发生了变化：我们已经有了一个最终的alpha向量,我们需要把它用于结构化的真实世界中的投资组合进行交易,获取利润.我们在每次迭代中重复如下步骤：我们计算alphas,并聚合成一个最终alpha,在前一次迭代中的投资组合基础上,用我们最终alpha去计算一个理想化的投资组合,生成一个换仓列表,将前一时刻的投资组合逐步替换成理想化的投资组合.

如何定义一个理想化的投资组合创建步骤,有许多疑问需要解答：你要如何认知你的风险(例如,你的风险模型是什么)? 在创建投资组合这一步中,你的目标函数是什么?哪些投资组合是受限的?

这三个问题总是被要求去回答的.我们今天只使用最简单的技术：根据最终alpha向量的分位数构建投资组合；比如,你多头的仓位等于权重最顶部的1/5,空头仓位等于权重最底部的1/5,通过权重的设置的,就能让投资组合的多头和空头达到一定数额的投资额,并且,多头和空头的价值是相等的.

随着投资组合的复杂度提高,回答上述三个问题也会变得更复杂.

## Trading (交易) ##
上述步骤的输出的是理想化的投资组合和将当前投资组合转换到理想化投资组合的换仓列表.交易,指的是在市场上进行交易的阶段.从你迄今为止所做的每一个选择的特点,您需要回答一些实际的操作问题：我需要怎样的交易频次?alpha的预测能力衰减周期是多少?在市场中,我们是更加被动的做的更耐心点,还是很积极且迅速的去执行?你可以通过`alphalens`查看最终alpha的持久性分析和营业额,和在[`pyfolio`](https://quantopian.github.io/pyfolio/notebooks/round_trip_example/)中查看完整策略的往返分析.如果选股策略是价格差异与自相似性之间的平衡,投资组合的构建是风险与收益之间的平衡,那么交易就会alpha衰减,显性成本,隐性成本和信息泄漏之间取得平衡.

---

你可能会问,"我必须严格按照这个流程才能获得成功吗?或者为我的算法在Quantopian上拿到一个配额(allocation)?"不一定的.我们在找寻能在回测之外依旧表现稳定的高夏普比率的策略；没有哪种流程可以完美解决这个问题.所有可能的策略构建的空间是如此之大,在此文中,我为大家勾勒出一个框架,并且为大家展示了世界上最大最成功的量化机构是如何解决这个问题的.

*"To follow the path, look to the master, follow the master, walk with the master, see through the master, become the master."*

这个结构,能让你自由发挥,创造属于你自己的框架.

---

[1] Sharpe Ratio is a statistical measurement of the risk adjusted performance of a portfolio, and is calculated by dividing a portfolio’s average return by the standard deviation of its returns. It shows a portfolio’s reward per unit of risk and is useful when comparing two similar portfolios. As the Sharpe Ratio increases, the better its performance.

 

[2] Aboody, David and Even Tov, Omri and Lehavy, Reuven and Trueman, Brett, Overnight Returns and Firm-Specific Investor Sentiment (April 11, 2016). Available at SSRN: https://ssrn.com/abstract=2554010 or https://dx.doi.org/10.2139/ssrn.2554010

 

[3] Dechow, Patricia M. and Khimich, Natalya V. and Sloan, Richard G., The Accrual Anomaly (March 22, 2011). Available at SSRN: https://ssrn.com/abstract=1793364 or https://papers.ssrn.com/sol3/papers.cfm?abstract_id=1793364

 

[4] Creamer, Germán G. and Freund, Yoav, Automated Trading with Boosting and Expert Weighting (April 1, 2010). Quantitative Finance, Vol. 4, No. 10, pp. 401–420. Available at SSRN: https://ssrn.com/abstract=937847

 

[5] Huerta, Ramon and Elkan, Charles and Corbacho, Fernando, Nonlinear Support Vector Machines Can Systematically Identify Stocks with High and Low Future Returns (September 6, 2012). Algorithmic Finance (2013), 2:1, 45-58. Available at SSRN: https://ssrn.com/abstract=1930709 or https://dx.doi.org/10.2139/ssrn.1930709
