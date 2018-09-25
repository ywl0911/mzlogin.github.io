---
layout: post
title: Tensorflow中变量的创建和共享方式
categories: TensorFlow python
description: Tensorflow中变量的创建和共享方式
keywords: python TensorFlow
mathjax: true

---

计算图像/文本相似度常常会用到Siamese网络，在实现Siamese网络时就会使用到变量的共享机制，Tensorflow提供了灵活的变量的创建和共享方式，做一些笔记加深理解。

## 1 变量创建的方式
tensorflow一般可以由以下3中方式创建变量。
**1、tf.Variable()**
```python
tf.Variable(initial_value=None,
            trainable=True,
            collections=None,
            validate_shape=True,
            caching_device=None,
            name=None,
            variable_def=None,
            dtype=None,
            expected_shape=None,
            import_scope=None
            )
```
函数功能：创建一个新的变量，变量的值是initial_value，创建的变量会被添加到[GraphKeys.GLOBAL_VARIABLES]默认的计算图列表中，如果trainable被设置为True,这个变量还会被添加到GraphKeys.TRAINABLE_VARIABLES计算图的集合中。
参数：
`initial_value`：初始的变量值，默认值是None，可以为tensor、可以转成tensor的python object。它必须有一个特殊的shape，除非validate_shape设置为False。
`trainable`：bool，是否可训练，默认的是True，变量还会被添加到GraphKeys.TRAINABLE_VARIABLES计算图集合中。
`collections`：变量会被添加的集合，默认的集合是[GraphKeys.GLOBAL_VARIABLES]。
`validate_shape`：bool，如果是False，则允许initial_value不确定shape，默认的是True。
`caching_device`：变量被存储或读取的设备。
`name`：str，变量的名字。
`dypte`：变量的类型，小数的默认是float32,整数默认是int32。

**2、tf.getVariable()**
```python
tf.get_variable(name,
                shape=None,
                dtype=None,
                initializer=None,
                regularizer=None,
                trainable=True,
                collections=None,
                caching_device=None,
                partitioner=None,
                validate_shape=True,
                use_resource=None,
                custom_getter=None)
```
函数功能：根据变量的名称来获取变量或者创建变量。
参数：
name：变量的名称（必选）。
shape：变量的shape。
dtype：变量的数据类型。
initializer：变量的初始化值。
在使用tf.get_variable()根据变量的名称来获取已经生成变量的时候，需要通过tf.variable_scope()函数来生成一个上下文管理器，并明确指定在这个上下文管理器中。获取变量值的时候，需要将上下文管理器中的reuse设置为True，才能直接获取已经声明的变量，如果不设置reuse会报错。需要注意的是，如果变量名在上下文管理器中已经存在，在获取的时候，如果不将reuse设置为True则会报错。同理，如果上下文管理器中不存在变量名，在使用reuse=True获取变量值的时候，也会报错。

**3、tf.placeholder()**
```python
tf.placeholder(
    dtype,
    shape=None,
    name=None
)
```
占位符，在图运行时赋值，实例如下：
```
x = tf.placeholder(tf.float32, shape=(1024, 1024))
y = tf.matmul(x, x)
with tf.Session() as sess:
    print(sess.run(y))  # ERROR: will fail because x was not fed.
    rand_array = np.random.rand(1024, 1024)
    print(sess.run(y, feed_dict={x: rand_array}))  # Will succeed.
```
## 2 tf.variable_scope()与tf.name_scope()的区别
tf.variable_scope()/tf.name_scope()两个函数类似，都是用来定义变量的上下文管理器。一般tf.variable_scope()作用域广，能管理所有方法创建的变量名；而tf.name_scope()只能管理由tf.Variable()创建的变量，看如下实例代码：
```
with tf.variable_scope('v') as scope:
    with tf.name_scope('n') as d:
        a = tf.Variable(initial_value=[1], name='a')
        b=tf.get_variable(initializer=[1], name='b')
        print(a.name)
        print(b.name)
>>>v/n/a:0
>>>v/b:0
```
可以发现变量`a`受tf.variable_scope()和tf.name_scope()共同作用，而`b`只受tf.variable_scope()的作用。

## 3 变量复用
在训练某些神经网络(例如Siamese网络)时，需要用到变量共享操作，这时则需要用到tf.getVariable()和tf.variable_scope()函数了。一般共享变量的方法有以下2种。
### 1、使用reuse = True/tf.AUTO_REUSE共享变量
```
with tf.variable_scope("foo", reuse=True):
    a = tf.get_variable("v", [1])
    b= tf.get_variable("v", [1])
    print(a.name)
    print(b.name)
>>>'foo/v:0'
>>>'foo/v:0'
```
### 2、通过捕获范围并设置重用来共享变量
```
with tf.variable_scope("foo") as scope:
    v = tf.get_variable("v", [1])
    scope.reuse_variables()
    v1 = tf.get_variable("v", [1])
assert v1 == v
```
另外值得的是，reuse（重用）标志是有继承性的：如果我们打开一个重用范围，那么它的所有子范围也会重用。看如下示例代码：

```
import tensorflow as tf
sess = tf.InteractiveSession()

def plus_1(a):
    with tf.variable_scope('f') as scope:
        b = tf.get_variable(initializer=tf.truncated_normal([1]), name='b')
        # b=tf.Variable(initial_value=tf.truncated_normal([1]), name='b')
    return b

with tf.variable_scope("plus") as scope:
    # Variables created here will be named "conv1/weights", "conv1/biases".
    o1 = plus_1(1)
    scope.reuse_variables()
    o2 = plus_1(1)

    sess.run(tf.global_variables_initializer())

    print(o1.eval())
    print(o2.eval())
    print(o1.name)
    print(o2.name)

>>>[1.2615478]
>>>[1.2615478]
>>>plus/f/b:0
>>>plus/f/b:0
```
可以发现plus作用域下，同名变量b的值相同。只要位于plus的variable_scope下，相同的变量名具有相同的值。