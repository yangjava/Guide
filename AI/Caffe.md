Caffe

（卷积神经网络框架，Convolutional Architecture for Fast Feature Embedding）

caffe是一个清晰，可读性高，快速的深度学习框架。作者是贾扬清，加州大学伯克利的ph.D，现就职于Facebook。



**Caffe**的全称应该是Convolutional Architecture for Fast Feature Embedding，它是一个清晰、高效的深度学习[框架](https://so.csdn.net/so/search?q=框架&spm=1001.2101.3001.7020)，它是开源的，核心语言是C++，它支持命令行、Python和Matlab接口，它既可以在CPU上运行也可以在GPU上运行。它的license是BSD 2-Clause。

 **Deep Learning比较流行的一个原因，主要是因为它能够自主地从数据上学到有用的feature**。特别是对于一些不知道如何设计feature的场合，比如说图像和speech。

 **Caffe的设计**：基本上，Caffe follow了神经网络的一个简单假设----**所有的计算都是以layer的形式表示的**，**layer做的事情就是take一些数据，然后输出一些计算以后的结果**，比如说卷积，就是输入一个图像，然后和这一层的参数（filter）做卷积，然后输出卷积的结果。