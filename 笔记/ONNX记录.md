
# 什么是onnx
开放神经网络交换（Open Neural Network Exchange）简称 ONNX 是微软和 Facebook 提出用来表示深度学习模型的**开放**格式。所谓开放就是 ONNX 定义了一组和环境，平台均无关的标准格式，来增强各种 AI 模型的可交互性。
# ProtoBuf
ONNX 使用的是 Protobuf 这个序列化数据结构去存储神经网络的权重信息。
Protobuf 是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。目前提供了 C++、Java、Python 三种语言的 API（摘自官方介绍）。

## ProtoBuf与xml，json比较
[深入ProtoBuf](https://www.jianshu.com/p/a24c88c0526a)

- XML、JSON、ProtoBuf 都具有**数据结构化**和**数据序列化**的能力
- XML、JSON 更注重**数据结构化**，关注人类可读性和语义表达能力。ProtoBuf 更注重**数据序列化**，关注效率、空间、速度，人类可读性差，语义表达能力不足（为保证极致的效率，会舍弃一部分元信息）
- ProtoBuf 的应用场景更为明确，XML、JSON 的应用场景更为丰富。
# onnx结构

核心对象有以下
- `ModelProto`
- `GraphProto`
- `NodeProto`
- `ValueInfoProto`
- `TensorProto`
- `AttributeProto`
当我们加载了一个 ONNX 之后，我们获得的就是一个`ModelProto`，它包含了一些版本信息，生产者信息和一个`GraphProto`。在`GraphProto`里面又包含了四个`repeated`数组，它们分别是`node`(`NodeProto`类型)，`input`(`ValueInfoProto`类型)，`output`(`ValueInfoProto`类型) 和`initializer`(`TensorProto`类型)，其中`node`中存放了模型中所有的计算节点，`input`存放了模型的输入节点，`output`存放了模型中所有的输出节点，`initializer`存放了模型的所有权重参数。

我们知道要完整的表达一个神经网络，不仅仅要知道网络的各个节点信息，还要知道它们的拓扑关系。这个拓扑关系在 ONNX 中是如何表示的呢？ONNX 的每个计算节点都会有`input`和`output`两个数组，这两个数组是 string 类型，通过`input`和`output`的指向关系，我们就可以利用上述信息快速构建出一个深度学习模型的拓扑图。这里要注意一下，`GraphProto`中的`input`数组不仅包含我们一般理解中的图片输入的那个节点，还包含了模型中所有的权重。例如，`Conv`层里面的`W`权重实体是保存在`initializer`中的，那么相应的会有一个同名的输入在`input`中，其背后的逻辑应该是把权重也看成模型的输入，并通过`initializer`中的权重实体来对这个输入做初始化，即一个赋值的过程。

最后，每个计算节点中还包含了一个`AttributeProto`数组，用来描述该节点的属性，比如`Conv`节点或者说卷积层的属性包含`group`，`pad`，`strides`等等，每一个计算节点的属性，输入输出信息都详细记录在`https://github.com/onnx/onnx/blob/master/docs/Operators.md`。