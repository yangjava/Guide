亚马逊将MXNet指定为官方深度学习平台



目前支持以下的语言：

- python
- R
- C++
- Julia
- Scala



| **比较项**   | **Caffe**     | **Torch**    | **Theano**      | **TensorFlow** | **MXNet**         |
| ------------ | ------------- | ------------ | --------------- | -------------- | ----------------- |
| **主语言**   | C++/cuda      | C++/Lua/cuda | Python/c++/cuda | C++/cuda       | C++/cuda          |
| **从语言**   | Python/Matlab | -            | -               | Python         | Python/R/Julia/Go |
| **硬件**     | CPU/GPU       | CPU/GPU/FPGA | CPU/GPU         | CPU/GPU/Mobile | CPU/GPU/Mobile    |
| **分布式**   | N             | N            | N               | Y(未开源)      | Y                 |
| **速度**     | 快            | 快           | 中等            | 中等           | 快                |
| **灵活性**   | 一般          | 好           | 好              | 好             | 好                |
| **文档**     | 全面          | 全面         | 中等            | 中等           | 全面              |
| **适合模型** | CNN           | CNN/RNN      | CNN/RNN         | CNN/RNN        | CNN/RNN?          |
| **操作系统** | 所有系统      | Linux, OSX   | 所有系统        | Linux, OSX     | 所有系统          |
| **命令式**   | N             | Y            | N               | N              | Y                 |
| **声明式**   | Y             | N            | Y               | Y              | Y                 |
| **接口**     | protobuf      | Lua          | Python          | C++/Python     | Python/R/Julia/Go |
| **网络结构** | 分层方法      | 分层方法     | 符号张量图      | 符号张量图     | ?                 |