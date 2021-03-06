# 多层神经网络（Multi-Layer Neural Network）  
##  

考虑一个监督学习问题，即我们有机会得到带标签的训练样本 $(x^{(i)}, y^{(i)})$ 。神经网络给出了一种方式用来定义复杂以及非线性的假设形式 $h_{W,b}(x)$ ，该形式带有参数 $W, b$，这些参数可被用来拟合我们的数据。  

为描述神经网络，我们将开始描述一个最为简单的神经网络 —— 只有一个神经元。我们将用下面这幅图来表示这样的单个神经元：  
<center><img src="./images/SingleNeuron.png" width=300/></center>  

这个神经元是一个可以输入 $x1, x2, x3$ 的计算单元（其中， $+1$ 是截距项），并且它将会输出 $\textstyle h_{W,b}(x) = f(W^Tx) = f(\sum_{i=1}^3 W_{i}x_i +b)$ 。其中， $f : \Re \mapsto \Re$ 被称为“激活函数”。在这些笔记中，我们会选择 $f(\cdot)$ 作为我们的 S 型函数：  

$$
f(z) = \frac{1}{1+\exp(-z)}.
$$  

因此，单个神经元对应的是被定义为逻辑斯特回归的输入-输出映射。  

尽管这些笔记将会用在 $S$ 型函数，但很有必要说明的是，其它常见的 $f$ 的选择可以是双曲正切或正切函数：  

$$
f(z) = \tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}.
$$  

最近的研究发现了一种与众不同的激活函数，整流线性激活函数（ $the \ Rectified \ Linear \ Function$ ），在实际中对于深层神经网络的效果更好。这种激活函数与 $S$ 型函数和双曲正切函数 $tanh$ 不同，因为其上限值不确定的，而且不是连续可微的。下面给出整流线性激活函数：  

$$
f(z) = \max(0,x).
$$  

下面是 $S$ 型函数，双曲正切函数 $tanh$ 以及整流线性函数：  

<center><img src="./images/Activation_functions.png"></center>  

双曲正切函数 $tanh(z)$ 是 $S$ 型函数的一个缩放版本，其输出范围是 $[-1,1]$ ，而不是 $[０,1]$ 。整流线性函数是一个分段的线性函数，当输入 $z$ 值小于 $0$ 时，其函数值为 $
0$ 。  

值得注意的是，不像其它地方（包括开放式的课堂教学视频，以及部分 CS229 的课程），我们不按照惯例 $x_0=1$ ，相反，截距项是由参数 $b$ 单独处理。  

最后我们要说的是在后面将会很有用的一个等式： $f(z) = 1/(1+\exp(-z))$ 是一个 $S$ 型函数，其导数为 $f'(z) = f(z) (1-f(z))$ （如果 $f$ 是双曲函数 $tanh$ ，则其导数为 $f'(z) = 1- (f(z))^2$ ）。您也可以自己通过 $S$ 型函数（或双曲函数 $tanh$）的定义尝试求导。整流线性函数在输入 $z \leq 0$ 时梯度为 $0$ ，其它取值时为 $1$ 。当输入值 $z=0$ 时，梯度是不确定的，但这并不会在实际中引起什么问题，因为在优化过程中，我们是建立在大量的训练样本之上来计算梯度的平均值的。  


## 神经网络模型（Neural Network model）　　

神经网络是通过将众多简单的神经元连接在一起得到的，一个神经元的输出可以是另一个的输入。例如，这里是一个小神经网络：  

<center><img src="./images/Network331.png" width=300 /></center>  

在这幅图中，我们用圆圈来表示网络的输入。在圈里被标为 “+1” 的圆圈称为偏置单元，对应于截距项。网络最左边的那一层称为输入层，而输出层即最右层（在这个例子中，输出层只有一个节点）。介于最左和最右的中间层称为隐藏层，因为它的值是无法在训练集中观察到的。由此，我们可以说，该例子中的神经网络有 3 个输入单元（不把计偏置单元计算在内）， 3 个隐藏单元，和 1 个输出单元。  

$n_l$ 表示网络的层数；因此，在我们的例子中的层数 $n_l = 3$ 。我们把第一层 $l$ 表示为 $L_1$，层 $L_1$ 即输入层，输出层用 $L_{n_l}$ 来表示。我们的神经网络参数 $(W,b) = (W^{(1)}, b^{(1)}, W^{(2)}, b^{(2)})$ ，参数 $W^{(l)}_{ij}$ 表示第 $l$ 层的第 $j$ 个单元与第 $l+1$ 层的第 $i$ 个单元的连接权重。（请注意下标的次序），$b^{(l)}_i$ 是第 $l+1$ 层的第 $i$ 个单元的偏置。因此，在我们的例子中，我们有 $W^{(1)} \in \Re^{3\times 3}$ ， $W^{(2)} \in \Re^{1\times 3}$ 。需要注意的是，偏置单元没有输入或连入它们的连接，它们输出的值总是为 “+1”。我们也用 $s_l$ 表示第 $l$ 层节点单元的数量（不包括偏置单元）。  

我们用 $a^{(l)}_i$ 来表示第 $l$ 层的第 $i$ 个单元的激活值（也可理解为输出值）。当层数 $l=1$ 时，我们也用 $a^{(1)}_i = x_i$ 来表示第 $i$ 层的输入。当参数 $W, b$　为确定值时，我们的神经网络即定义了一个能输出实数的假设 $h_{W,b}(x)$ 。具体而言，该神经网络表示的计算为：  

$$
\begin{align}
a_1^{(2)} &= f(W_{11}^{(1)}x_1 + W_{12}^{(1)} x_2 + W_{13}^{(1)} x_3 + b_1^{(1)})  \\
a_2^{(2)} &= f(W_{21}^{(1)}x_1 + W_{22}^{(1)} x_2 + W_{23}^{(1)} x_3 + b_2^{(1)})  \\
a_3^{(2)} &= f(W_{31}^{(1)}x_1 + W_{32}^{(1)} x_2 + W_{33}^{(1)} x_3 + b_3^{(1)})  \\
h_{W,b}(x) &= a_1^{(3)} =  f(W_{11}^{(2)}a_1^{(2)} + W_{12}^{(2)} a_2^{(2)} + W_{13}^{(2)} a_3^{(2)} + b_1^{(2)}) 
\end{align}
$$  

在后文中，我们还将用 $z^{(l)}_i$ 表示第 $l$ 层的第 $i$ 个单元输入的总加权求和，其中包含偏置项（例如，第 $2$ 层的第 $i$ 个单元输入的总加权求和值为 $\textstyle z_i^{(2)} = \sum_{j=1}^n W^{(1)}_{ij} x_j + b^{(1)}_i$ ），第 $l$ 层的第 $i$ 个单元（译者注：即激活单元的计算或输出值）为 $a^{(l)}_i = f(z^{(l)}_i)$ 。  

需要注意的是，我们的后一种写法更紧凑。具体来说，如果我们将激活函数 $f(\cdot)$ 的应用扩展到向量中的每一个元素上（例如，$f([z_1, z_2, z_3]) = [f(z_1), f(z_2), f(z_3)]$），那么我们可以把以上方程写成一种更紧凑的形式：  
$$
\begin{align}
z^{(2)} &= W^{(1)} x + b^{(1)} \\
a^{(2)} &= f(z^{(2)}) \\
z^{(3)} &= W^{(2)} a^{(2)} + b^{(2)} \\
h_{W,b}(x) &= a^{(3)} = f(z^{(3)})
\end{align}
$$  

我们称这一步为前向传播（$Forward\ Propagation$）。更普遍的情况是，回顾一下以前，我们也使用 $a^{(1)} = x$ 表示输入层的值，那么，有第 $l$ 层的激活 $a^{(l)}$，我们把计算第 $l+1$ 层的激活 $a^{(l+1)}$ 表示为：  

$$
\begin{align}
z^{(l+1)} &= W^{(l)} a^{(l)} + b^{(l)}   \\
a^{(l+1)} &= f(z^{(l+1)})
\end{align}
$$  

通过组织在矩阵中的参数，并借助矩阵向量操作，我们可以充分利用线性代数进行网络参数的快速计算。  

迄今为止，我们都集中在一个神经网络的例子上，但也可以建立神经网络的其他体系结构（即神经元之间的连接模式），包括多个隐藏层。最常见的选择是一个有 $\textstyle n_l$ 层的网络，其层 $1$ 为输入层，层 $\textstyle n_l$ 为输出层，层 $l$ 到层 $l+1$ 都是密集的连接起来的。在这种情况下，为了计算网络的输出，用上面公式中描述的传播步骤，我们可以依次计算所有的激活层，从层 $\textstyle L_{2}$ ，到层 $\textstyle L_{3}$ 等等，直到层 $\textstyle L_{n_l}$ 。这是一个前馈神经网络（ $Feedforward \ Neural \ Network$ ）的例子，因为连通图没有任何有向环或闭合圈。  

神经网络可以有多个输出单元。举个例子，这里是一个有着 $\textstyle L_{2}$ 和 $\textstyle L_{3}$ 两个隐含层，以及在层 $\textstyle L_{4}$ 有两个输出单元的神经网络：  

<center><img src="./images/Network3322.png" width=300 /></center>  

为了训练这个网络，我们需要训练样本 $(x^{(i)}, y^{(i)})$ ，其中 $y^{(i)} \in \Re^2$ 。如果存在您感兴趣预测的多个输出，那么这种网络就是有用的。（例如，在医学诊断中的应用，向量 $x$ 可能给出的是病人的输入特征，并且不同的输出可能表明不同种的疾病是否存在。）  

## 反向传播算法（Backpropagation Algorithm）  

假设我们有一个固定的训练集 $\{ (x^{(1)}, y^{(1)}), \ldots, (x^{(m)}, y^{(m)}) \}$ 。我们可以使用批量梯度下降训练网络。详细说来，对于一个训练样本 $(x,y)$ ，成本函数可以定义为：  

$$
\begin{align}
J(W,b; x,y) = \frac{1}{2} \left\| h_{W,b}(x) - y \right\|^2.
\end{align}
$$  

这是一个（半）平方误差函数。给定一组有着 $m$ 个样本的训练集，成本函数定义为：  

$$
\begin{align}
J(W,b)
&= \left[ \frac{1}{m} \sum_{i=1}^m J(W,b;x^{(i)},y^{(i)}) \right]
                       + \frac{\lambda}{2} \sum_{l=1}^{n_l-1} \; \sum_{i=1}^{s_l} \; \sum_{j=1}^{s_{l+1}} \left( W^{(l)}_{ji} \right)^2
 \\
&= \left[ \frac{1}{m} \sum_{i=1}^m \left( \frac{1}{2} \left\| h_{W,b}(x^{(i)}) - y^{(i)} \right\|^2 \right) \right]
                       + \frac{\lambda}{2} \sum_{l=1}^{n_l-1} \; \sum_{i=1}^{s_l} \; \sum_{j=1}^{s_{l+1}} \left( W^{(l)}_{ji} \right)^2
\end{align}
$$  

在 $J(W,b)$ 定义中的第一项是均方差项。第二项是一个正则化项（也叫权重衰减项），它会降低权重的大小，并有助于防止过拟合。  

（注：权重衰减通常不适用于偏置项 $b_{i}^{(l)}$ ），比如我们 $J(W,b)$ 在定义中就没有使用。一般来说，将权重衰减应用到偏置单元中只会对最终的神经网络产生很小的影响。如果您在斯坦福选修过 CS229 （机器学习）课程，或者在 YouTube 上看过课程视频，您也许会认识到这里的权重衰减，其实是课上提到的贝叶斯正则化方法的变种，在贝叶斯正则化中，我们将高斯先验概率引入到参数中计算 $MAP$ （极大后验）估计（而不是极大似然估计）。  

权重衰减参数 $λ$ 用于控制公式中两项的相对重要性。在此重申一下这两个复杂函数的含义： $J(W,b; x,y)$ 是针对单个样本计算平方误差得到的代价函数；而 $J(W,b)$ 则是对整体样本的代价函数，它包含权重衰减项。  

以上的代价函数通常被用来解决分类和回归问题。在分类问题上，我们分别用 $y=0$ 或 $1$ 来表示这两个类标签（回顾一下 $S$ 型激活函数的输出值介于 $[0,1]$ 之间；如果我们之前使用 $tanh$ 双曲正切激活函数，那么激活函数的输出值将会是$-1$ 和 $+1$，刚好用来代表两类）。在回归问题上，我们首先放缩输出值范围，确保最终其输出值在 $[0,1]$ 上（或者如果我们使用双曲正切激活函数，那么输出值在 $[-1,1]$ 上）。  

我们的目标是最小化把 $W$ 和 $b$ 作为参数的函数 $J(W,b)$ 。为训练神经网络，我们将会初始化每一个 $W_{ij}^{(l)}$ 参数，以及每一个 $b_{i}^{(l)}$ 参数，把它们初始化为一个接近 $0$ 且小的随机值（比如说，使用正态分布 $\textstyle {Normal}(0,\epsilon^2)$ 生成的随机值，其中 $\textstyle \epsilon$ 设置为 $\textstyle 0.01$），之后再对目标函数应用如批量梯度下降法（ $Batch \ Gradient \ Descent$ ）来进行参数优化。因为 $J(W, b)$ 是一个非凸函数，梯度下降有可能使我们的函数值到局部最优；然而，实际中，梯度下降通常效果还不错。最后需要注意的是，对参数进行随机初始化是很重要的，而不是全把它们初始设置为 $0$ 。如果所有的参数都起始于一个相同的值，那么所有的隐含层单元将会学习到一样的函数（对于所有的 $i$ 值， $W^{(1)}_{ij}$ 将总是相同的，即对任意输入 $x$ ，有 $a^{(2)}_1 = a^{(2)}_2 = a^{(2)}_3 = \ldots$ ）。随机初始化的目的是使**对称失效（ $Symmetry \ Breaking$ ）**。  

梯度下降法中每一次迭代都按照如下公式对参数 $\textstyle W$ 和 $\textstyle b$ 进行更新：  

$$
\begin{align}
W_{ij}^{(l)} &= W_{ij}^{(l)} - \alpha \frac{\partial}{\partial W_{ij}^{(l)}} J(W,b) \\
b_{i}^{(l)} &= b_{i}^{(l)} - \alpha \frac{\partial}{\partial b_{i}^{(l)}} J(W,b)
\end{align}
$$  

其中， $\alpha$ 是学习率，以上公式中一个关键步骤是计算偏导数。我们现在来讲一下**反向传播**（ $Backpropagation$ ）算法，它是一种计算偏导数的高效方法。  

我们首先讲一下反向传播是如何计算 $\textstyle \frac{\partial}{\partial W_{ij}^{(l)}} J(W,b; x, y)$ 和 $\textstyle \frac{\partial}{\partial b_{i}^{(l)}} J(W,b; x, y)$ 的，以及针对一个样本 $(x,y)$ 的代价函数 $J(W,b;x,y)$ 的偏导数。一旦我们把这些计算出来了，我们将会看到计算所有样本的代价函数 $J(W,b)$ 的偏导数：  

$$
\begin{align}
\frac{\partial}{\partial W_{ij}^{(l)}} J(W,b) &=
\left[ \frac{1}{m} \sum_{i=1}^m \frac{\partial}{\partial W_{ij}^{(l)}} J(W,b; x^{(i)}, y^{(i)}) \right] + \lambda W_{ij}^{(l)} \\
\frac{\partial}{\partial b_{i}^{(l)}} J(W,b) &=
\frac{1}{m}\sum_{i=1}^m \frac{\partial}{\partial b_{i}^{(l)}} J(W,b; x^{(i)}, y^{(i)})
\end{align}
$$  

上述两个公式略有不同，这是因为权重衰减应用在了参数 $W$ 上，而不是参数 $b$ 。  

反向传播算法背后的过程如下：给定一个训练样本 $(x, y)$ ，我们首先执行前向传递，以计算整个网络的激活，这其中包含假设 $h_{W,b}(x)$ 的输出值。然后，对每层 $l$ 的每个节点 $i$ ，我们将会计算一个误差项（$error \ term$）$\delta^{(n_l)}_i$ ，该误差项计算了当前节点对我们网络的输出误差的贡献度。  

对一个输出节点而言，我们可以直接计算出网络激活值和真实目标值的差距，并用该值来定义误差项 $\delta^{(n_l)}_i$ （第 $n_l$ 层是输出层）。  

那有关隐单元呢？对着那些隐含单元来说，我们将会基于误差项的加权平均，即把 $a^{(l)}_i$ 作为输入的节点的误差项的加权平均，来计算 $\delta^{(l)}_i$ ，详细说来，反向传播算法如下：  

1. 执行前馈传递，其中计算层 $L_2, L_3$ 到输出层 $L_n$ 的激活函数值。  
2. 对第 $L_n$ 层的每个第 $i$ 个输出单元，设定
$$
\begin{align}
\delta^{(n_l)}_i
= \frac{\partial}{\partial z^{(n_l)}_i} \;\;
\frac{1}{2} \left\|y - h_{W,b}(x)\right\|^2 = - (y_i - a^{(n_l)}_i) \cdot f'(z^{(n_l)}_i)
\end{align}
$$  
3. 循环层数。 $For \ l = n_l-1, n_l-2, n_l-3, \ldots, 2$  
	 - 循环当前层（第 $l$ 层）节点（第 $i$ 个节点），设定  
	 $$
     \delta^{(l)}_i = \left( \sum_{j=1}^{s_{l+1}} W^{(l)}_{ji} \delta^{(l+1)}_j \right) f'(z^{(l)}_i)
     $$
4. 计算所需部分的偏微分，如下已给出：  
    $$
    \begin{align}
    \frac{\partial}{\partial W_{ij}^{(l)}} J(W,b; x, y) &= a^{(l)}_j \delta_i^{(l+1)} \\
    \frac{\partial}{\partial b_{i}^{(l)}} J(W,b; x, y) &= \delta_i^{(l+1)}.
    \end{align}
    $$

最后，我们可以用矩阵和向量的符号标记来重写算法。我们使用 $\textstyle \bullet$ 来表示逐个元素地点乘操作（在 Matlab 或 Octave 中，我们使用 .* 来表示，这也被称作 $Hadamard$ 乘积）。如果有 $\textstyle a = b \bullet c$ ，即 $\textstyle a_i = b_ic_i$ 。与我们将 $\textstyle f(\cdot)$ 的定义从逐个元素扩展应用到向量上一样，我们也可以对 $\textstyle f'(\cdot)$ 做同样的操作（那么有，$\textstyle f'([z_1, z_2, z_3]) = [f'(z_1), f'(z_2), f'(z_3)]$）。  

现在，算法流程可以写为：  

1. 执行前馈传递，使用定义前馈传递流程的方程来计算第二，三层 $L_2, L_3$ 直到输出层 $L_n$ 的激活函数值。  
2. 对输出层（即第 $n_l$ 层），设定
$$
\begin{align} \delta^{(n_l)} = - (y - a^{(n_l)}) \bullet f'(z^{(n_l)}) \end{align}
$$  
3. 循环层数 $l$ 。 $For \ l = n_l-1, n_l-2, n_l-3, \ldots, 2$，设定  
	 $$
    \begin{align} \delta^{(l)} = \left((W^{(l)})^T \delta^{(l+1)}\right) \bullet f'(z^{(l)}) \end{align}
     $$
4. 计算所需部分的偏微分，如下已给出：  
    $$
\begin{align}
\nabla_{W^{(l)}} J(W,b;x,y) &= \delta^{(l+1)} (a^{(l)})^T, \\
\nabla_{b^{(l)}} J(W,b;x,y) &= \delta^{(l+1)}.
\end{align}
    $$  

**实现须知**：在以上所述的第2，3步中，我们需要为每个节点 $i$ 计算偏导数 $\textstyle f'(z^{(l)}_i)$ 。假设 $\textstyle f(z)$ 是一个 S 型激活函数，先前我们已经将第 $l$ 层的第 $i$ 个节点的激活函数值存储在了网络中。因此，通过我们先前得出的 $\textstyle f'(z)$ 的表达式，我们可以计算第 $l$ 层的第 $i$ 个节点的激活函数的偏导数 $\textstyle f'(z^{(l)}_i) = a^{(l)}_i (1- a^{(l)}_i)$ 。  

最后，我们便可以描述整个梯度下降算法了。下面是伪代码， $\textstyle \Delta W^{(l)}$ 是一个矩阵（与 $\textstyle W^{(l)}$ 同一维度），$\textstyle \Delta b^{(l)}$ 是一个向量（与 $\textstyle b^{(l)}$ 同一维度）。注意这个符号，$\textstyle \Delta W^{(l)}$ 是一个矩阵，尤其要说明的是，它并不是 $\textstyle \Delta$ 乘以 $\textstyle W^{(l)}$ ，我们实现一次梯度下降的过程如下：  

1. 对所有层 $l$ ，设定矩阵或向量中元素值均为 $0$ ： $\textstyle \Delta W^{(l)} := 0$ ， $\textstyle \Delta b^{(l)} := 0$ 。  
2. $For \ i=1 \ to \ m$,
	1. 使用反向传播计算 $\textstyle \nabla_{W^{(l)}} J(W,b;x,y)$ 和 $
\textstyle \nabla_{b^{(l)}} J(W,b;x,y)$ 。
	2. 设定 $\textstyle \Delta W^{(l)} := \Delta W^{(l)} + \nabla_{W^{(l)}} J(W,b;x,y)$ 。
	3. 设定 $\textstyle \Delta b^{(l)} := \Delta b^{(l)} + \nabla_{b^{(l)}} J(W,b;x,y)$
3. 更新参数：
$$
\begin{align}
W^{(l)} &= W^{(l)} - \alpha \left[ \left(\frac{1}{m} \Delta W^{(l)} \right) + \lambda W^{(l)}\right] \\
b^{(l)} &= b^{(l)} - \alpha \left[\frac{1}{m} \Delta b^{(l)}\right]
\end{align}
$$

为训练神经网络，我们现在可以通过重复执行梯度下降的步骤的方法，减少成本函数 $\textstyle J(W,b)$ 的值。