---
title: 最前沿：Meta Learning to Learn, 到底我们要学会学习什么？
date: 2023-02-01 19:05:21
tags:
	
	- 论文
---







**1 前言**


在本文之前，智能专栏上已经有多篇介绍 Meta Learning 的文章：

Kay Ke：概要：NIPS 2017 Deep Learning for Robotics Keynote

Flood Sung：最前沿：百家争鸣的 Meta Learning/Learning to learn

Flood Sung：机器人革命与学会学习 Learning to Learn

Flood Sung：学会学习 Learning to Learn：让 AI 拥有核心价值观从而实现快速学习

那么在这篇博文中，我打算更深入的谈谈我对 Meta Learning 的理解，也算是全新的视角。

2 到底什么是 Meta 的？


-------------------

我不知道是谁最先把 Meta Learning 翻译成元学习的，但从中文的角度问你 “元学习是什么？” 你要怎么回答？老实说我也是懵逼的。特意查一下元在字典中的意思，主要是有初始，根源的意思。但 Meta Learning 不是初始呀，这里的 Meta 更应该是指更高 level 的东西。Learning to Learn 这个词看起来应该更好理解，直接翻译就对了：学会学习。所以 Meta Learning 可以理解为要学习一种学习能力。但是什么是学习能力呢？如何描述？一般我们说某个学神学习能力强大概是说学神记忆力好，理解能力快之类的东西。但是什么是理解能力呢？依然非常难以描述。

实际上这里我们遇到了和高维空间一样的问题。我们很难去想象高维空间，我们也很难去想象 meta 的东西到底是什么。更何况，还有 meta-meta，meta-meta-meta... 的东西。也许这就是大概被称为智能的东西吧。

> 以前，我们只是不知道 Deep Learning 是怎么学的，但至少知道是在学习下棋还是学习图像识别，现在对于 Meta Learning，我们连学什么都不太清楚了。

那么 Meta 到底如何定义？我们只能回归到 Machine Learning 本身来考虑这个概念。这里我给出一个我对 Meta 的定义：

> 任何超出学习（Learning/Training）内部的东西都是 Meta 的！

什么是学习内部呢？

有两个视角，一个是深度学习技术上的学习，一般我们也称为训练，另一种是人类角度看的学习。

2.1 深度学习技术视角的 Meta


----------------------

现在大家都在自嘲自己是炼金术士，我们每天调的参数就是 Meta 的，也就是在学习外部！所谓的学习就是这个训练过程，也可以说是反向传播过程。超出这个 “学习的” 都是学习外部，都是 Meta 的。

这包含了以下这些类别：

1.  训练超参数 Hyper Parameters：包括 Learning rate，Batch Size，input size 等等目前要人为设定的参数
    
2.  神经网络的结构
    
3.  神经网络的初始化
    
4.  优化器 Optimizer 的选择。比如 SGD，Adam，RMSProp
    
5.  神经网络参数
    
6.  损失函数的定义。那么多 gan 的文章基本上就是改改损失函数。
    
7.  反向传播 Back-propagation。 对，这个也是 Meta 的，我们通过梯度下降的公式固定了参数的更新方式。神经网络傻傻只能按照这个公式来更新参数。
    

我发现要区别是不是 Meta 的就是把 “学习” 当做一个小孩，你叫他怎么做怎么做的东西都是 Meta 的。那么所谓的 “学习” 就是沿着你设定的方向实际移动的过程，比如梯度下降中参数实际变化过程叫学习。而学会学习就是使用神经网络自己去构造上面的任意设定。

也因此，我们可以把目前 meta learning 的一些研究工作排排坐了。比如说

[1611.01578] Neural Architecture Search with Reinforcement Learning

这就属于神经网络结构的，很明显。

[1703.03400] Model-Agnostic Meta-Learning for Fast Adaptation of Deep Networks

MAML 就属于参数的初始化。

[1606.04474] Learning to learn by gradient descent by gradient descent

这篇则是构造一个神经网络的优化器来替代 Adam，SGD。

Meta-Critic Networks for Sample Efficient Learning

这篇则是考虑学习一个更好的 loss。

还有

Optimization as a Model for Few-Shot Learning

这篇则既学习一个好的初始化，也学习网络的更新。

那么看到这里大家显然就可以看出来了，我们可以切入其中的某一个角度来做文章。

这里面可能最最困难的就是直接取代反向传播。如果能用一个神经网络来代替目前的反向传播，那就牛大了。

2.2 人类视角下的 Meta：Meta Knowledge


----------------------------------

说完深度学习技术角度的 Meta，我们再来看看人类视角下的 Meta。所谓人类视角下的 Meta，就是从真正的学习知识的角度来看什么是 Meta 的。

比如说目前 Reinforcement Learning 上一个大量被用来测试的实验是 3D Maze Navigation。让 agent 去导航到一个目标。实验中，目标要不断变化。这意味着要使 Agent 在下一个 episode 中快速的找到目标，需要 Agent 在每一次 episode 中不断的获取 Maze 的结构和目标的位置，从而在新的 episode 能够直接去寻找的这个目标。那么，从 Meta Learning 的角度看，Maze 的结构和目标的位置就是所谓的 Meta 知识。那么这种知识在 RL 下只使用 state,action 的数据是无法得知的，还需要 reward 才能在顶层进行判断。因此，meta RL 的基本思想非常简单，就是在输入增加上一次的 reward，或者用之前的（state,action,reward）来推断 Meta 知识。虽然说 Meta-RL 是通用的，按照原作者的意思，是学习一个 rl 算法，但是在具体的比如这里的 3d navigation 问题上，推断目标位置和记忆 Maze 形状是最主要的。

而在监督学习上，Meta 知识又完全不一样了。人类很神奇的天生具备 meta 知识不用学。比如人视觉具有的 one shot learning 能力。看一个新东西就能分辨。这个实际上是天然的具备比较不同视觉物体相似情况的能力。再有比如视觉跟踪，之前大家都只是把它当做一个计算机视觉的问题在看待，但是实际上视觉跟踪是人类的一种 Meta 能力，也是不用学的，就直接可以跟踪任何新的没见到的物体。如果有人看到一个全新的东西眼球就不会转了，那么这个世界的规律大概就不一样了。

从一定程度上说，Meta Learning 的本意就是要学习 Meta Knowledge。Meta knowledge 可以具体如视觉的跟踪比较，也可以抽象的就是某一个学习算法。但是，显而易见的，越具体的东西越容易去研究，越抽象的东西就越难去学习。

这里举最新的一篇 Robot Learning 的文章为例，来自 Fei-Fei Li：

https://arxiv.org/abs/1710.01813

这篇文章采用 Neural Program Induction 这个很酷的方法来做 few shot imitation learning，取得了很不错的效果。但是，我们要深究一下原理，实际上这篇文章是把一个机器人的 manipulation 通过人为的方式分解成多步动作，神经网络只需要学习如何组成这些动作，然后执行。这一定程度上大幅度简化了问题的难度。为了实现 few shot imitation learning，则让神经网络能够 meta 的根据已有的 demo 去构建动作的顺序。所以，这里的 meta 知识就是去组织动作。这个在人为的分解下变得简单了。如果没有人为分解，要求神经网络自己去发现步骤，并组织这些步骤，难度就大太多了。

Meta knowledge 又天然的和 Hierarchical Reinforcement Learning（HRL）联系起来。我们知道 HRL 最难的部分就是去自主的学习一个好的 Hierarchy，好的 Option。但是实际上仅靠单一任务是很难做的，因为没有对比，就是我们人也不知道哪些是顶层的，哪些是底层的。但是 Meta Learning 的设定则提供了这样的实现机会。我们可以通过多个类似的任务来学习一个 meta knowledge，这个 meta knowledge 就是 hierarchy，就是高层的知识。在 OpenAI 之前很火的高中生的那篇 paper 中对这个进行了验证并取得了不错的效果：

https://arxiv.org/abs/1710.09767

3 未来的 Meta Learning 将会如何发展？


-------------------------------

2018 年马上就要到来了，这里也大胆预测一下 Meta Learning 未来的发展。可以说，当前的 Meta Learning 和 2015 年的 Deep Reinforcement Learning 非常像，刚刚起步，开始有一些很酷的应用效果（特别是 Pieter Abbeel 团队的 One shot Visual Imitation Learning 工作），目前的工作也就是几十篇文章。因此，明年，Meta Learning 必将会有一波爆发，将体现在以下两个大的方面：

（1）理论算法研究。目前的 Meta Learning 还是百花齐放的情况，当然从我们前面对 Meta 的分析可以看到 Meta 不同角度可以看到完全不同的东西。因此，接下来还会有很多工作会基于不同的视角提出不同的算法。我们依然期待一个大一统的框架。然而目前的情况是越视角单一，越可能做的效果好，毕竟视角的选择等价于人类的知识赋予，约等于简化了神经网络的学习难度。完全通用的 Meta Learning 比如 MAML 这种要打败专门设计的算法会有难度，这其实也给了大家很多机会（现在 Chelsea Finn 在 MAML 上疯狂的灌）

（2）具体应用研究。目前监督学习主要是 image recognition。这显然是很局限的。Meta Learning，或者 few shot learning 可以应用到视觉语音等各种方面。因此，新的应用研究也是很值得做的。从今年 few shot visual recognition 研究中已经可以看到迹象，小实验像 omniglot 这种已经基本解决，开始向 large scale 的数据发起进攻。从这里也看出 few shot learning 开始从很不可能变成可能了。然后另一块很重要的当然是 Robot Learning 了。本来研究 Meta Learning 就不是为了区区的视觉图像，而是机器人的快速学习，这个有很强的现实意义，是大幅度推进机器人革命的关键一步。因此，明年如果没有超越 MAML 的算法出现，会是难以想象的。更强的 few shot imitation learning 感觉会彻底的改变 imitation learning 这个领域。然后 HRL 结合 Meta Learning 也是必然的趋势，高中生那篇文章还是稍显简单了，更通用更强大的 Meta HRL 算法恐怕也要问世，而这显然可以应用在星际 2 上，所以 DeepMind 不研究也不行呀。

