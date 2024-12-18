# GRU—门控循环单元

### GRU模型的发展

GRU模型的发展背景与深度学习技术的快速发展紧密相关，特别是在自然语言处理和理解领域。随着深度学习在机器翻译、情感分析、问答系统等任务中的应用，对于能够有效处理序列数据的模型需求日益增长。

然而，传统的循环神经网络（RNN）在处理长序列数据时常常遇到梯度消失或梯度爆炸的问题，这限制了它们捕捉长期依赖关系的能力。为了解决这些问题，长短期记忆网络（LSTM）被提出，它通过引入门控机制来控制信息的流动，从而有效缓解了梯度消失问题。GRU模型则是在LSTM的基础上进一步简化和发展的，它由Cho等人在2014年提出，旨在简化LSTM的结构，同时保持类似的性能。

GRU通过合并LSTM中的遗忘门和输入门为更新门，并引入重置门，同时合并单元状态和隐藏状态，使得模型更为简洁，训练速度更快。这种结构上的简化并没有牺牲性能，GRU在很多任务中的表现与LSTM相当，甚至在某些情况下更优。因此，GRU因其高效和简洁的特性，在自然语言处理、语音识别、时间序列预测等多个领域得到了广泛的应用。

### GRU模型结构

想要去理解一个模型，了解模型的输入输出是至关重要的一部分。将GRU模型当成一个黑匣子，模型结构图可以化简成下图所示： 

<div align=center>
<img src="https://github.com/gzhuuser/easy-nlp/blob/master/docs/chapter3/images/image-202411021243032.png?raw=true" >
</div>



其中：

**$h^{t-1}$**：表示在前一个时间步的隐藏状态（hidden state）。在GRU中，这个隐藏状态会通过更新门、重置门和候选隐藏状态的计算来更新。

**$h^t$**：这表示在当前时间步t的隐藏状态。是由GRU单元根据当前输入和前一时间步的隐藏状态计算得到的。

**$x^t$**：这表示在当前时间步的输入。在GRU中，每个时间步的输入会被用来计算当前时间步的候选隐藏状态和更新门、重置门。

**$y^t$**：表示在当前时间步的输出，是模型根据当前输入和前一时间步的隐藏状态计算得到的结果。



那么，GRU为什么能够如此高效呢？它到底有什么特别之处呢？下面让我们从它的内部结构进行分析：

#### GRU的内部结构

下面，我们对GRU的结构进行逐步的分解：

<div align=center>
<img src="https://github.com/gzhuuser/easy-nlp/blob/master/docs/chapter3/images/image-202411031049271.png?raw=true" >
</div>

如图所示，一开始GRU定义了两个门：R（重置门）和 Z（更新门），这里我们可以把他简单的理解为与隐藏状态长度的向量，计算方式如下：

​					$R^t = \sigma(X^t W^{xr} + H^{t-1} W^{hr} + b^r)$

​					$Z^t = \sigma(X^t W^{xz} + H^{t-1} W^{hz} + b^z)$

其中， $\sigma$ 为激活函数。

**重置门（Reset Gate）**：

- 重置门是决定网络是否保留或忽略前一个时间步的隐藏状态信息。它通过控制信息的流动来帮助网络忘记不再重要的信息。
- 重置门的输出是一个介于0和1之间的值，这个值乘以前一个时间步的隐藏状态。如果重置门的输出接近0，那么前一个时间步的隐藏状态将被忽略；如果输出接近1，则保留前一个时间步的隐藏状态。

**更新门（Update Gate）**:

- 更新门的主要作用是决定网络是否更新当前时间步的隐藏状态。它控制新信息的加入和旧信息的保留，帮助网络决定哪些信息应该被保留，哪些应该被更新。
- 更新门的输出也是一个介于0和1之间的值，这个值决定了前一个时间步的隐藏状态有多少应该被保留，以及新信息有多少应该被加入到当前的隐藏状态中。



接下来，我们具体看看这两个门是怎么去使用的。

<div align=center>
<img src="https://github.com/gzhuuser/easy-nlp/blob/master/docs/chapter3/images/image-202411031248189.png?raw=true" >
</div>

如图，重置门产生的值 $R^t$ ，经过公式：

​				$$\tilde{H}^{t}=\tanh\left(X^{t} W^{x h}+\left(R^{t}\odot H^{t-1}\right) W^{h h}+b^{h}\right)$$

得到候选隐藏状态 $\tilde{H}^{t}$ 。候选隐藏状态并不是最终的隐藏状态，而是用于生成真正隐藏状态的一个参数。

如果我们忽略 $R^t$ （即假设 $R^t$ 全部为1），那么这个公式就简化为RNN中计算隐藏状态的公式。在这个公式中， $R^t$ 是一个与长度相 $H^{t-1}$ 同的向量。在 $R^t$ 和 $H^{t-1}$ 进行逐元素相乘的过程中， $R^t$ 的值介于0到1之间。如果  $R^t$ 中的值接近0，那么与 $H^{t-1}$ 对应元素相乘的结果也会接近0，这意味着几乎完全忽略前面的信息，当前时间步的候选隐藏状态主要来自 $X^t$ 。相反，如果 $R^t$ 中的值接近1，那么意味着不选择遗忘前一时间步的隐藏状态，当前时间步的候选隐藏状态将同时来源于 $X^t$ 和 $H^{t-1}$ 。在实际中， $R^t$ 是一个可学习参数，它会根据前面的信息去学习哪些该去遗忘，哪些需要遗留。



最后，我们来看看真正的隐藏状态是如何产生的：

<div align=center>
<img src="https://github.com/gzhuuser/easy-nlp/blob/master/docs/chapter3/images/image-202411041156317.png?raw=true" >
</div>

如图所示，真正的隐藏状态 $H^t$ 是由候选隐藏状态、更新门和前一时间步隐藏状态所决定的。具体公式如下：

​					$H^{t} = Z^{t} \odot H^{t-1} + (1 - Z^{t}) \odot \tilde{H}^{t}$

与 $R^t$ 类似， $Z^t$ 也是一个介于0到1之间的控制单元。同样的，倘若 $Z_t$ 都等于1， $H^t$ 就直接等于 $H^{t-1}$ ，也就是不使用 $X^t$ 来更新当前的隐藏状态；倘若 $Z_t$ 都等于0， $H^t$ 主要是由 $X^t$ 进行更新，但并不意味着不通过前一时间步的隐藏状态更新，因为 $\tilde{H}^{t}$ 的公式中含有 $H^{t-1}$ 。



### 总结

GRU引入两个额外的门（重置门R 和更新门Z），这两个门都是输入0到1之间值的控制单元。重置门是在计算候选隐藏状态时控制前一时间步隐藏状态信息的多少，而更新门是在计算真正隐藏状态时控制当前时间步输入X信息的多少。

想象一下，GRU就像是一个有选择性记忆的智能笔记本。当你阅读一本书时，更新门就像是你决定是否保留你之前读到的信息。重置门则像是你决定是否要回顾之前的内容，以便更好地理解当前的章节。候选隐藏状态是你尝试将新信息和旧信息结合起来的过程。最后，隐藏状态更新就像是你最终决定在笔记本上写下的内容。

**公式：**

​					$R^t = \sigma(X^t W^{xr} + H^{t-1} W^{hr} + b^r)$

​					$Z^t = \sigma(X^t W^{xz} + H^{t-1} W^{hz} + b^z)$

​				  $$\tilde{H}^{t}=\tanh\left(X^{t} W^{x h}+\left(R^{t}\odot H^{t-1}\right) W^{h h}+b^{h}\right)$$

​					$H^{t} = Z^{t} \odot H^{t-1} + (1 - Z^{t}) \odot \tilde{H}^{t}$


极端情况：

Z里面全为0、R里面全为1时，就会回到RNN的结构；

Z里面全为0、R里面全为0时， $H^t$ 就只会看到 $X^t$ 的信息；

Z里面全为1时， $H^t$ 就只会看到 $H^{t-1}$ 的信息，而忽略 $X^t$ 的信息。

通常情况下，会在这几个极端情况之间进行可学习的调整，使得模型可以去区别两者信息的重要性，并赋予相应的权重。



### GRU 与 LSTM 的对比

GRU 在 LSTM 的基础上主要做出了两点改变：

**门的数量减少了**：

- 在LSTM中，有三个门来控制信息的流动：一个决定哪些信息应该被遗忘（遗忘门），一个决定哪些新信息应该被添加（输入门），还有一个决定哪些信息应该输出（输出门）。
- GRU简化了这个过程，它把遗忘门和输入门合并成了一个叫做更新门的东西。更新门的作用是决定我们应该保留多少旧信息以及添加多少新信息。

**记忆单元的处理方式变了**：

- 在LSTM中，有一个特殊的部分叫做记忆单元，它负责长期存储信息。LSTM通过一系列复杂的门控机制来更新这个记忆单元。
- GRU没有专门的记忆单元，它直接在隐藏状态中处理信息。GRU使用另一个门，叫做重置门，来决定我们应该忽略多少旧信息，以便为新信息腾出空间。



简单来说，GRU就像是一个更简洁的LSTM。它通过减少门的数量和简化信息处理的方式来尝试做到和LSTM一样的事情，但是用更少的计算资源。这样，GRU在某些情况下可以更快地学习，并且可能更容易训练。不过，这也意味着在需要处理非常长的依赖关系时，GRU可能不如LSTM那么强大。
