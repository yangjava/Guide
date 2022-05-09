TensorFlow是 [谷歌](https://baike.baidu.com/item/谷歌)基于DistBelief进行研发的第二代 [人工智能](https://baike.baidu.com/item/人工智能/9180) [学习系统](https://baike.baidu.com/item/学习系统)，其命名来源于本身的运行原理。Tensor（张量）意味着N维数组，Flow（流）意味着基于数据流图的计算，TensorFlow为张量从流图的一端流动到另一端计算过程。TensorFlow是将复杂的数据结构传输至人工智能神经网中进行分析和处理过程的系统。

TensorFlow可被用于 [语音识别](https://baike.baidu.com/item/语音识别)或 [图像识别](https://baike.baidu.com/item/图像识别)等多项机器学习和深度学习领域。

CNN，卷及神经网络。

其实TensorFlow大部分内核并不是用Python编写的 ：它是高度优化了C++和CUDA（Nvidia用于编程GPU的语言）的组合。 相反，通常它是使用了Eigen （高性能C ++和CUDA库）和NVidia的cuDNN （用于NVidia GPU的非常优化的DNN库，用于卷积等功能）。

TensorFlow的模型是程序员用“一种语言”（很可能是Python！）来表达。

所以说，我们再说一下这个问题：为什么TensorFlow选择Python作为表达和控制模型训练而且支持的非常好的语言？
答案很简单：Python可能是大量数据科学家和机器学习专家用的最舒适的语言，也是易于集成和控制C ++后端的语言，同时也是广泛使用与谷歌的公司内外和他们的开源产品。 鉴于使用TensorFlow的基本模型，Python的性能并不重要，这是一个很自然的契合。 NumPy也是一个巨大的加分，它可以很容易地在Python中进行预处理（也是高性能），然后将它们提供给TensorFlow，以获得真正CPU-heavy的东西。