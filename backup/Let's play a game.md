按照前面职业规格的情况，35w年收入是能接受的。就当这个数据是自己的目前身价吧，我们拿职业生涯来做个游戏。

游戏规则如下：

---

1.0版本

1. 身价随着年龄会下跌，具体下跌数值按照公式计算：count(loss) = (age-35)^ageLossFactor；年龄增长在市场中的评估损失，且年龄越大，损失比重越大，损失因子ageLossFactor目前按照2计算。
2. 经验随着工作会增长，但随着年龄增长，越来越少，沉淀是不少的，count(exper) = log<sub>experienceAddFactor</sub>(age-30)。然后experienceAddFactor目前设置为2，体现IT行业经验增长较快。计算沉淀，用定积分做计算，得到log(x+1)，然后x=(age-30)*daysCount，使用 https://zh.numberempire.com/definiteintegralcalculator.php 计算得到数据。经验积累假设30岁之前的都已经在当前身价里面了。
3. 然后平时的玩乐性质的体验，总结一下的，可以加50。
4. 平时学习的体验总结，可以加100-20000左右吧，都可以调整，目前就是1.0版本。
5. 平时工作的体验总结，可以加200-50000左右，因为更有实际意义。

得到目前的每日身价：

$$
value = 350000 + \int_0^Dln(x+1)dx-(age-35)^2*D
$$

$$
D = (age-30)*daysCount
$$

让我们开始这个游戏~