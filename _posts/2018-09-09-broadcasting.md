---
layout: post
title: Numpy和Tensorflow中矩阵运算的broadcast机制
categories: TensorFlow python
description: Numpy和Tensorflow中的广播机制
keywords: python TensorFlow
mathjax: true

---


在深度学习的模型中，矩阵运算在模型训练过程中占有非常大的比重，优化矩阵计算的时间相当重要。在在Python语言中，我们会用到Numpy、Scipy和Tensorflow等等做科学计算的包，这些库都封装好了非常高效的矩阵计算方法。以下笔记着重讲解在这些包中矩阵运算的广播broadcast机制。


## 1 问题引出
在写卷积神经网络时，离不开这两行关键的代码：
```python
h_conv_z=tf.nn.conv2d(input, filter, strides, padding, use_cudnn_on_gpu=None, name=None)
h_conv_a = tf.nn.sigmoid(h_conv_z + b_conv)
```
`h_conv_z`为对input做卷积的结果，其返回值为一个tensor，也可是说是feature map，其维度一般为：
[batch_size × height × width × channels]，其中：

+ **batch_size**为批量大小
+ **height**卷积之后图片的高度，即有多少行
+ **width**卷积之后图片的宽度，即有多少列
+ **channels**卷积之后图片的数量，即通道数量/卷积核个数

`b_conv`为对卷积后的结果`h_conv_z`加上的bias，值得注意的是`b_conv`的初始化方法为
```python
tf.Variable(tf.truncated_normal(channels, stddev=0.1))
```
可见`b_conv`是一个一维tensor，其维度为[channels,]。
那么问题来了，在`h_conv_z + b_conv`中，一个4维tensor`h_conv_z`为什么能和一个1维tensor`b_conv`直接相加？

两个tensor相加，如果维度不匹配的话，就运用到tensorflow中的[广播broadcast](https://docs.scipy.org/doc/numpy-1.13.0/user/basics.broadcasting.html)机制。当然并不是任何两个维度不匹配的tensor进行加减乘除运算都会调用广播机制，它们的维度还是需要满足一定的条件。


## 2 broadcast
### 2.1 定义
引用[官方文档](https://docs.scipy.org/doc/numpy-1.13.0/user/basics.broadcasting.html)的解释：
> The term broadcasting describes how numpy treats arrays with different shapes during arithmetic operations. Subject to certain constraints, the smaller array is “broadcast” across the larger array so that they have compatible shapes. Broadcasting provides a means of vectorizing array operations so that looping occurs in C instead of Python. It does this without making needless copies of data and usually leads to efficient algorithm implementations. There are, however, cases where broadcasting is a bad idea because it leads to inefficient use of memory that slows computation.
广播用以描述numpy中对两个形状不同的阵列进行数学计算的处理机制。较小的阵列“广播”到较大阵列相同的形状尺度上，使它们对等以可以进行数学计算。广播提供了一种向量化阵列的操作方式，因此Python不需要像C一样循环。广播操作不需要数据复制，通常执行效率非常高。然而，有时广播是个坏主意，可能会导致内存浪费以致计算减慢。

首先看以下代码：
```python
>>>a = np.array([1,2,3])
>>>b = np.array([2,2,2])
>>>a * b
>>>array([ 2.,  4.,  6.])
```
Numpy中的`*`运算符表示元素积（element-wise product，对应位置相乘），如果利用numpy的传播机制，则上述代码等价于：
```python
>>>a*np.array([2])
>>>array([ 2.,  4.,  6.])
```
上面两种结果是一样的，我们可以认为尺度值b在计算时被延展得和a一样的形状。延展后的b的每一个元素都是原来尺度值的复制，可以说b被延展成array([2,2,2])，再与a进行元素积。延展的类比只是一种概念性的，实际上，numpy并不需要真的复制这些尺度值，所以广播运算在内存和计算效率上尽量高效，因为广播在乘法计算时动用更少的内存。
### 2.2 条件
两个矩阵a、b，将它们的维度从后往进行比较，如果所有维度满足以下条件之一才能进行广播。
>1. 相等
>2. 其中有一个为1

有点拗口，举几个栗子就能豁然开朗了。

**例1**
```python
>>>a=np.array([ [[ 0,  1,  2],[ 3,  4,  5]],
                [[ 6,  7,  8],[ 9, 10, 11]]     ])
>>>b=np.array([1,2,3]).reshape(1,3)
>>>a.shape
>>>(2, 2, 3)
>>>b.shape
>>>(1,3)
>>>a+b
>>>array([  [[ 1,  3,  5],[ 4,  6,  8]],
            [[ 7,  9, 11],[10, 12, 14]]     ])
>>>a*b
>>>array([  [[ 0,  2,  6],[ 3,  8, 15]],
            [[ 6, 14, 24],[ 9, 20, 33]]     ])
>>>(a*b).shape
>>>(2,2,3)
>>>(a+b).shape
>>>(2,2,3)
```
可以发现a的shape为(2,2,3)，b的shape为(1,3)，从后往前比较，最后一维都为3满足条件1，倒数第二维分别为2和1同样满足条件2，所以a和b能够在维度不符合数学规定的情况下实现a+b和a*b的 (element-wise运算)。还有一点是a.shape=(a**b).shape=(a+b).shape。

我们重新reshape一下a和b的shape，观察是否能够满足传播机制。

**例2**
```python
>>>a=a.reshape(4,1,1,3)
>>>b.reshape(3,1)
>>>a
>>> array([ [[[ 0,  1,  2]]],
            [[[ 3,  4,  5]]],
            [[[ 6,  7,  8]]],
            [[[ 9, 10, 11]]]    ])
>>>b
>>>array([  [1],
            [2],
            [3] ])

>>>a+b
>>>array([  [[[ 1,  2,  3],[ 2,  3,  4],[ 3,  4,  5]]],
            [[[ 4,  5,  6],[ 5,  6,  7],[ 6,  7,  8]]],
            [[[ 7,  8,  9],[ 8,  9, 10],[ 9, 10, 11]]],
            [[[10, 11, 12],[11, 12, 13],[12, 13, 14]]]  ])
>>>(a+b).shape
>>>(4, 1, 3, 3)
```
可以发现a的shape(4,1,1,3)和b的shape(3,1)同样是满足以上的两个条件的，所以通过传播机制同样能够字节计算a+b和a*b，只不过(a+b)的shape变成了(4, 1, 3, 3)。

不同于例1的是发现(4, 1, 3, 3)比原来a的shape(4,1,1,3)和b的shape(3,1)都要多出不少，原因是因为a和b的最后两维的维度都是一个为1另一个不为1，那么不为1的维度将被用作最终结果的维度。也就是说，尺寸为1的维度将延展或“复制”到与另一个维度匹配，说的有点拗口，再举一个简单的例子说明如下：

**例3**
```python
>>>a=b=np.array([1,2,3])
>>>a=a.reshape(1,3)
>>>b=b.reshape(3,1)
>>>a
>>>array([1, 2, 3])
>>>b
>>>array([  [1],
            [2],
            [3] ])
>>>a+b
>>>array([  [2, 3, 4],
            [3, 4, 5],
            [4, 5, 6]   ])
```
可以这样理解，b的shape为(3,1)，先将b矩阵“复制”，使维度为(3,3),如下：
```python
>>>b_copy=np.array([    [1, 1, 1],
                        [2, 2, 2],
                        [3, 3, 3]   ])
```
再将a矩阵“复制”，使维度为(3,3),如下：
```python
>>>a_copy=np.array([    [1, 2, 3],
                        [1, 2, 3],
                        [1, 2, 3]   ])
```

然后再执行b_copy+a_copy，即可得到最终a+b的结果。可以看到计算过程其实也非常简单，先“复制”，再相加而已。

再回到问题的引出中例子：
```Python
h_conv_a = tf.nn.sigmoid(h_conv_z + b_conv)
```
`h_conv_z`的维度为[batch_size , height , width , channels]，`b_conv`的维度为[channels,]，他们最后一维的维度相等，所以可以利用broadcast机制直接相加。

## 3 补充
上文提到的矩阵运算都指element-wise运算，对应位置相加减乘除，不涉及矩阵的乘法。对于矩阵乘法，还需要补充一点的是：

+ 一维数组置于矩阵乘法的左部，被视为一个行向量；
+ 一维数组置于矩阵乘法的右部，被视为一个列向量；
+ 一维数组与矩阵乘法运算的结果仍是一维数组。

先看如下代码：
```python
>>>a=np.array([1,2,3])
>>>a.shape
>>>(3,)
>>>a1=a.reshape(1,3)
>>>a1
>>>array([[1, 2, 3]])
>>>a2=a.reshape(3,1)
>>>a2
>>>array([  [1],
            [2],
            [3] ])
```
a首先由np.array([1,2,3])进行初始化，此时a的shape为(3,)，a1的shape为(1,3)，a2的shape为(3,1)。那么a的维度到底是和a1一样还是和a2一样，还是都不一样？来看以下实验：
```python
>>>b=np.array([ [0, 1, 2],
                [3, 4, 5],
                [6, 7, 8]   ])

>>>np.matmul(a,b)
>>>array([24, 30, 36])
>>>np.matmul(a1,b)
>>>array([[24, 30, 36]])

>>>np.matmul(b,a)
>>>array([ 8, 26, 44])
>>>np.matmul(b,a2)
>>>array([  [ 8],
            [26],
            [44]    ])
```
实验发现：当a位于b的左边时，np.matmul(a,b)等于np.matmul(a1,b)，a被当做行向量计算；
          当a位于b的左边时，np.matmul(b,a)等于np.matmul(b,a2)，a被当做列向量计算。
所以a是一个非常灵活的数据结构。

## 4 附录

When $$(a \ne 0)$$, there are two solutions to $$(ax^2 + bx + c = 0)$$ and they are

$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$
Numpy或tensorflow中的常用的矩阵乘法函数，对于array对象(注意不是matrix对象)：
+ tf.matmul(A,B)和np.matmul(A,B)和`@`云算法表示一般的矩阵乘法，即A<sub>p×q </sub>* B<sub>q×r</sub> = C<sub>p×r</sub>
+ tf.multiply(A,B)、np.multiply(A,B)，按位(element-wise)运算，即对应位置相乘，支持broadcast，如 A<sub>p×q </sub>* B<sub>p×q</sub> = C<sub>p×q</sub> ，此外四则运算符`+`/`-`/`×`/`÷`都是按位进行运算。
+ np.dot(A,B)，在A,B都是2-D的时候，等价于tf.matmul(A,B)