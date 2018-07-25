# 2.4 增量式实现

<link href="../../../css/style.css" rel="stylesheet"></link>
我们至今讨论的动作值方法全都使用观察到的奖赏的样本均值来估计动作值. 我们现在转向这些平均值怎样应该以高效的方式计算出来这一问题, 更具体地说, 怎样以常数的内存使用与常数的每一时步的计算时间计算出来. 

为了简化标记我们将专注于单个动作上. 让$$R_i$$表示在第$$i$$次选择该动作后接收到的奖赏, 并以$$Q_n$$表示在选择了$$n - 1$$次该动作后其估计值, 现在我们可以简单地将$$Q_n$$写作

$$Q_n \doteq \frac{R_1 + R_2 + \dots + R_{n - 1}}{n - 1}.$$

最显而易见的实现方式就是维持对所有接收到的奖赏的记录, 然后每当需要估计值的时候就计算上方的等式. 然而, 以这种方式实现的话, 对内存与计算时间的需求会随着越来越多的奖赏被观察到而逐步增大. 每一个新观察到的奖赏都会需要额外的内存来存储, 需要额外的计算时间来计算分子中的和.

就像你怀疑的那样, 这事实上并不是必需的. 很简单就能发明增量式地更新均值的公式, 且只需要极小的、常数的计算时间来处理每个新奖赏. 给定$$Q_n$$和第$$n$$次的奖赏$$R_n$$, 所有$$n$$个奖赏的新的均值可以使用下式计算

$$
\begin{align}
Q_{n + 1} &= \frac{1}{n} \sum_{i = 1}^n R_i \\
&= \frac{1}{n} \left( R_n + \sum_{i = 1}^{n - 1}R_i \right) \\
&= \frac{1}{n} \left( R_n + (n - 1) \frac{1}{n - 1} \sum_{i = 1}^{n - 1}R_i \right) \\
&= \frac{1}{n} \left( R_n + (n - 1) Q_n \right) \\
&= \frac{1}{n} \left( R_n +n Q_n - Q_n \right) \\
&= Q_n + \frac{1}{n} \left[ R_n - Q_n \right],
\tag{2.3}
\end{align}
$$
上述的等式甚至在$$n = 1$$时也成立, 即$$Q_2 = R_1$$, 对任意$$Q_1$$均成立. 这一实现方式只需要存储$$Q_n$$和$$n$$的内存, 以及对每个新奖赏使用[(2.3)](to equation2.3)的极小的计算时间.

更新规则[(2.3)](to equation2.3)属于一种在全书中经常出现的更新形式. 这一一般形式为
$$
\begin{align}
新估计值 &\leftarrow 旧估计值 + 步长 [目标 - 旧估计值] \\
(NewEstimate &\leftarrow OldEstimate + StepSize [Target - OldEstimate])
\end{align}
\tag{2.4}
$$
表达式$$ [目标 - 旧估计值]$$是估计中*误差*&lt;error&gt;. 通过向"目标"靠近来减小该误差. 假定上目标预示着希望的移动方向, 虽然这可能是有噪声的. 例如在前述的案例中, 目标就是第$$n$$次奖赏.

注意在增量式方法[(2.3)](to equation2.3)中步长参数会随时间变化. 在处理对动作$$a$$的第$$n$$次奖赏时, 该方法使用的步长参数为$$\frac{1}{n}$$. 在本书中我们将步长参数记为$$\alpha$$, 或更为一般的, 记为$$\alpha_t(a)$$.

完整的使用增量式计算样本均值与$$\varepsilon$$-贪心动作选择的赌博机算法的伪代码展示在了下方的方框内. 函数$$bandit(a)$$假定为以动作为参数, 并返回响应奖赏的函数.

<div class="pc_box">
<h4>一个简单的赌博机算法</h4>
<pre>
初始化, for $$a = 1$$ to $$k$$:
	$$Q(a) \leftarrow 0$$ 
	$$N(a) \leftarrow 0$$

Loop forever:
	$$A \leftarrow \begin{cases} \operatorname{argmax}_a Q(a) &\text{以}1-\varepsilon\text{的概率(若多个最值, 随意选择)}\\ \text{一个随机动作} & \text{以}\varepsilon的概率 \end{cases}$$
	$$R \leftarrow bandit(A)$$
	$$N(A) \leftarrow N(A) + 1$$
	$$Q(A) \leftarrow Q(A) + \frac{1}{N(A)}[R - Q(A)]$$
</pre>
</div>