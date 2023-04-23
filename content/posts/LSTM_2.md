---
title: 了解LSTM神经网络
subtitle:
date: 2023-04-24T01:44:51+08:00
draft: false
author:
  name: waka
  link:
  email:
  avatar: /images/avatar.jpg
description: 了解LSTM神经网络
keywords: LSTM, 神经网络, 机器学习
license:
comment: false
weight: 0
tags:
  - LSTM
  - 神经网络
  - 机器学习
categories:
  - 机器学习
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: false
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---
## 循环神经网络

人类不会每一秒都从头开始思考。当你阅读这篇文章时，你会根据你对前面单词的理解来理解每个单词。你不会扔掉所有东西，然后重新开始思考。你的思想有坚持。

传统的神经网络无法做到这一点，这似乎是一个主要缺点。例如，假设您想对电影中每一点发生的事件类型进行分类。目前尚不清楚传统的神经网络如何利用其对电影中先前事件的推理来通知后来的事件。

递归神经网络解决了这个问题。它们是带有循环的网络，允许信息持续存在。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-rolled.png)

**递归神经网络有循环。**

在上图中，一大块神经网络，$A$, 查看一些输入$x_t$并输出一个值$h_t$. 循环允许信息从网络的一个步骤传递到下一个步骤。

这些循环使递归神经网络看起来有点神秘。然而，如果你多想想，就会发现它们与普通的神经网络并没有什么不同。循环神经网络可以被认为是同一网络的多个副本，每个副本都将一条消息传递给后继者。考虑一下如果我们展开循环会发生什么：

![展开的递归神经网络。](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-unrolled.png)

**展开的递归神经网络。**

这种链状性质表明循环神经网络与序列和列表密切相关。它们是用于此类数据的神经网络的自然架构。

他们肯定被使用了！在过去的几年里，将 RNN 应用于各种问题取得了令人难以置信的成功：语音识别、语言建模、翻译、图像字幕……这个例子不胜枚举。关于 RNN 可以实现的惊人壮举，我将在 Andrej Karpathy 的优秀博客文章[The Unreasonable Effectiveness of Recurrent Neural Networks](http://karpathy.github.io/2015/05/21/rnn-effectiveness/)中进行讨论。但他们真的很了不起。

这些成功的关键是使用“LSTM”，这是一种非常特殊的递归神经网络，对于许多任务，它比标准版本要好得多。几乎所有基于递归神经网络的令人兴奋的结果都是用它们实现的。本文将探讨这些 LSTM。

## 长期依赖的问题

RNN 的吸引力之一是它们可能能够将以前的信息连接到当前的任务，例如使用以前的视频帧可能会告知对当前帧的理解。如果 RNN 可以做到这一点，它们将非常有用。但他们可以吗？这取决于。

有时，我们只需要查看最近的信息即可执行当前任务。例如，考虑一个语言模型试图根据前面的单词预测下一个单词。如果我们试图预测“the clouds are in the _sky_ ”中的最后一个词，我们不需要任何进一步的上下文——很明显下一个词将是 sky。在这种情况下，相关信息和需要它的地方之间的差距很小，RNN 可以学习使用过去的信息。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-shorttermdepdencies.png)

但在某些情况下，我们需要更多上下文。考虑尝试预测文本“我在法国长大……我说一口流利的_法语_”中的最后一个词。最近的信息表明，下一个词可能是一种语言的名称，但如果我们想缩小哪种语言的范围，我们需要法国的上下文，从更远的地方。相关信息和需要它的点之间的差距变得非常大是完全有可能的。

不幸的是，随着差距的扩大，RNN 变得无法学习连接信息。

![神经网络与长期依赖性作斗争。](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/RNN-longtermdependencies.png)

理论上，RNN 绝对有能力处理这种“长期依赖”。人类可以仔细地为他们挑选参数来解决这种形式的玩具问题。遗憾的是，在实践中，RNN 似乎无法学习它们。[Hochreiter (1991) \[German\]](http://people.idsia.ch/~juergen/SeppHochreiter1991ThesisAdvisorSchmidhuber.pdf)和[Bengio 等人](http://www-dsi.ing.unifi.it/~paolo/ps/tnn-94-gradient.pdf)深入探讨了这个问题。[(1994)](http://www-dsi.ing.unifi.it/~paolo/ps/tnn-94-gradient.pdf)，他发现了一些非常根本的原因，为什么这可能很困难。

值得庆幸的是，LSTM 没有这个问题！

## 长短期记忆网络

长短期记忆网络——通常简称为“LSTM”——是一种特殊的 RNN，能够学习长期依赖关系。[它们由Hochreiter & Schmidhuber (1997)](http://www.bioinf.jku.at/publications/older/2604.pdf)引入，并在后续工作中被许多人提炼和推广。[<sup><span><span>1</span></span></sup>](http://colah.github.io/posts/2015-08-Understanding-LSTMs/#fn1)它们在处理大量问题时表现出色，现在已被广泛使用。

LSTM 明确设计用于避免长期依赖问题。长时间记住信息实际上是他们的默认行为，而不是他们努力学习的东西！

所有循环神经网络都具有神经网络重复模块链的形式。在标准 RNN 中，这个重复模块将具有非常简单的结构，例如单个 tanh 层。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-SimpleRNN.png)

**标准 RNN 中的重复模块包含单层。**

LSTM 也有这种链状结构，但重复模块有不同的结构。不是只有一个神经网络层，而是有四个，以一种非常特殊的方式进行交互。

![LSTM 神经网络。](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-chain.png)

**LSTM 中的重复模块包含四个交互层。**

不要担心正在发生的事情的细节。稍后我们将逐步介绍 LSTM 图。现在，让我们试着熟悉我们将要使用的符号。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM2-notation.png)

在上图中，每条线都带有一个完整的向量，从一个节点的输出到其他节点的输入。粉色圆圈代表逐点操作，如向量加法，而黄色框是学习的神经网络层。行合并表示串联，而行分叉表示其内容被复制并且副本转到不同的位置。

## LSTM 背后的核心思想

LSTM 的关键是细胞状态，即贯穿图表顶部的水平线。

细胞状态有点像传送带。它直接沿着整个链条运行，只有一些次要的线性交互。信息很容易原封不动地沿着它流动。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-C-line.png)

LSTM 确实有能力删除或添加信息到细胞状态，由称为门的结构仔细调节。

门是一种有选择地让信息通过的方式。它们由一个 sigmoid 神经网络层和一个逐点乘法运算组成。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-gate.png)

sigmoid 层输出介于 0 和 1 之间的数字，描述应该让多少成分通过。值为零表示“不让任何东西通过”，而值为 1 表示“让一切都通过！”

LSTM 具有三个这样的门，以保护和控制单元状态。

## 循序渐进的 LSTM 演练

LSTM 的第一步是决定我们要从细胞状态中丢弃哪些信息。该决定由称为“遗忘门层”的 S 形层做出。它看着$h_{t-1}$和$x_t$, 并输出一个介于$0$和$1$对于细胞状态中的每个数字$C_{t-1}$. A$1$代表“完全保留这个”，而一个$0$代表“彻底摆脱这个”。

让我们回到我们的语言模型示例，它试图根据所有先前的单词预测下一个单词。在这样的问题中，细胞状态可能包括当前主体的性别，以便可以使用正确的代词。当我们看到一个新主题时，我们想忘记旧主题的性别。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-f.png)

下一步是决定我们要在细胞状态中存储哪些新信息。这有两个部分。首先，称为“输入门层”的 sigmoid 层决定我们将更新哪些值。接下来，一个 tanh 层创建一个新候选值的向量，$\tilde{C}_t$，可以将其添加到状态。在下一步中，我们将结合这两者来创建对状态的更新。

在我们的语言模型示例中，我们想要将新主题的性别添加到单元格状态，以替换我们忘记的旧主题。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-i.png)

现在是更新旧细胞状态的时候了，$C_{t-1}$, 进入新的细胞状态$C_t$. 前面的步骤已经决定了要做什么，我们只需要实际去做。

我们将旧状态乘以$f_t$，忘记了我们早些时候决定忘记的事情。然后我们添加$i_t*\tilde{C}_t$. 这是新的候选值，根据我们决定更新每个状态值的程度进行缩放。

在语言模型的情况下，这是我们实际上要删除有关旧主题性别的信息并添加新信息的地方，正如我们在前面的步骤中所决定的那样。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-C.png)

最后，我们需要决定要输出什么。此输出将基于我们的细胞状态，但将是过滤后的版本。首先，我们运行一个 sigmoid 层，它决定我们要输出细胞状态的哪些部分。然后，我们将细胞状态通过$\tanh$（将值推到介于$-1$和$1$) 并将其乘以 sigmoid 门的输出，这样我们就只输出我们决定输出的部分。

对于语言模型示例，由于它刚刚看到一个主语，它可能想要输出与动词相关的信息，以防接下来会发生什么。例如，它可能会输出主语是单数还是复数，以便我们知道如果接下来是动词应该变位的形式。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-focus-o.png)

## 长短期记忆的变体

到目前为止，我所描述的是一个非常普通的 LSTM。但并不是所有的 LSTM 都和上面的一样。事实上，似乎几乎每篇涉及 LSTM 的论文都使用略有不同的版本。差异很小，但值得一提的是其中的一些差异。

[由Gers & Schmidhuber (2000)](ftp://ftp.idsia.ch/pub/juergen/TimeCount-IJCNN2000.pdf)引入的一种流行的 LSTM 变体正在添加“窥孔连接”。这意味着我们让门层查看单元状态。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-var-peepholes.png)

上图给所有的门都加了窥视孔，但是很多论文会给一些窥视孔，其他的就不给了。

另一种变体是使用耦合的遗忘门和输入门。我们不是单独决定要忘记什么以及我们应该向什么添加新信息，而是一起做出这些决定。我们只会忘记什么时候要输入一些东西来代替它。当我们忘记旧的东西时，我们只会向状态输入新值。

![](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-var-tied.png)

LSTM 的一个稍微更显着的变化是门控循环单元 (Gated Recurrent Unit, GRU)，由[Cho 等人引入。(2014)](http://arxiv.org/pdf/1406.1078v3.pdf)。它将遗忘门和输入门组合成一个“更新门”。它还合并了细胞状态和隐藏状态，并做了一些其他的改变。生成的模型比标准 LSTM 模型更简单，并且越来越受欢迎。

![门控循环单元神经网络。](http://colah.github.io/posts/2015-08-Understanding-LSTMs/img/LSTM3-var-GRU.png)

这些只是一些最著名的 LSTM 变体。还有很多其他的，比如[Yao 等人的 Depth Gated RNNs。(2015)](http://arxiv.org/pdf/1508.03790v2.pdf)。还有一些完全不同的方法来解决长期依赖关系，比如[Koutnik 等人的 Clockwork RNNs。(2014)](http://arxiv.org/pdf/1402.3511v1.pdf)。

这些变体中哪个最好？差异重要吗？[格雷夫等人。(2015)](http://arxiv.org/pdf/1503.04069.pdf)对流行的变体进行了很好的比较，发现它们都差不多。[Jozefowicz 等人。(2015)](http://jmlr.org/proceedings/papers/v37/jozefowicz15.pdf)测试了超过一万个 RNN 架构，发现其中一些在某些任务上比 LSTM 更有效。

## 结论

早些时候，我提到了人们使用 RNN 取得的显著成果。基本上所有这些都是使用 LSTM 实现的。对于大多数任务，它们确实工作得更好！

写成一组方程式，LSTM 看起来相当吓人。希望在本文中逐步介绍它们可以使它们更容易理解。

LSTM 是我们使用 RNN 可以完成的一大步。人们很自然地会想：是否又迈出了一大步？研究人员的共同意见是：“是的！还有下一步，就是注意！这个想法是让 RNN 的每一步都从一些更大的信息集合中挑选信息来查看。例如，如果您使用 RNN 创建描述图像的标题，它可能会选择图像的一部分来查看它输出的每个单词。事实上，[徐_等人。_(2015)](http://arxiv.org/pdf/1502.03044v2.pdf)正是这样做的——如果你想探索注意力，这可能是一个有趣的起点！使用注意力已经取得了许多非常令人兴奋的结果，而且似乎还有更多的结果……

注意力并不是 RNN 研究中唯一令人兴奋的话题。例如，[Kalchbrenner等_人的 Grid LSTM。_(2015)](http://arxiv.org/pdf/1507.01526v1.pdf)似乎非常有前途。在生成模型中使用 RNN——例如[Gregor_等人。_(2015)](http://arxiv.org/pdf/1502.04623.pdf) ,[钟_等人。_(2015)](http://arxiv.org/pdf/1506.02216v3.pdf)或[Bayer & Osendorfer (2015)](http://arxiv.org/pdf/1411.7610v3.pdf) – 似乎也很有趣。过去几年对于递归神经网络来说是一个激动人心的时期，而即将到来的只会更加精彩！

<!--more-->
