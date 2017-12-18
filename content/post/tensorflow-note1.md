+++
draft = false
date = "2017-11-22T17:03:51+08:00"
title = "tensorflow学习笔记-模型包装"
wordCount = 234
readtime = 5
author = "fwyan"
categories = ["ai", "tensorflow"]
tags = ["ai", "tensorflow"]
+++


> tf的模型可以自己完全手写，用tensorflow提供的底层API自己手写，对于tf底层API比较熟悉可以这样，也可以采用tf的高阶API来编写自己的模型，比如用内置模块**tf.estimator**，本文主要介绍后者。


<!--more-->



#### 采用底层API自定义模型逻辑

这里需要理解tf的基本的运行机制，基于tensor，节点，流向，图等概念的模型结构
通常的步骤是：

1. 定义输入输出节点，以及中间节点逻辑
2. 定义评估函数，损失函数
3. 喂输入数据，迭代训练
4. 训练收敛后，用测试集进行评估
5. 保存模型，预测未知

以logistic回归模型为例，典型的代码组织如下：

定义模型

```
# 1. define themodel
# tf Graph Input
x = tf.placeholder(tf.float32, [None, 784]) # mnist data image of shape 28*28=784
y = tf.placeholder(tf.float32, [None, 10]) # 0-9 digits recognition => 10 classes

# Set model weights
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))

# Construct model
pred = tf.nn.softmax(tf.matmul(x, W) + b) # Softmax

# Minimize error using cross entropy
cost = tf.reduce_mean(-tf.reduce_sum(y*tf.log(pred), reduction_indices=1))
# Gradient Descent
optimizer = tf.train.GradientDescentOptimizer(learning_rate).minimize(cost)

# Initialize the variables (i.e. assign their default value)
init = tf.global_variables_initializer()

saver = tf.train.Saver(max_to_keep=1)

# Start training
with tf.Session() as sess:

    # Run the initializer
    sess.run(init)
```
训练模型

```
    # 2. feed train_set, train the model
    for epoch in range(training_epochs):
        avg_cost = 0.
        total_batch = int(mnist.train.num_examples/batch_size)
        # Loop over all batches
        for i in range(total_batch):
            batch_xs, batch_ys = mnist.train.next_batch(batch_size)
            # Run optimization op (backprop) and cost op (to get loss value)
            _, c = sess.run([optimizer, cost], feed_dict={x: batch_xs,
                                                          y: batch_ys})
            # Compute average loss
            avg_cost += c / total_batch
        # Display logs per epoch step
        if (epoch+1) % display_step == 0:
            print("Epoch:", '%04d' % (epoch+1), "cost=", "{:.9f}".format(avg_cost))

    print("train Finished!")
```

评估模型

```
    # 3. evaluate the model
    correct_prediction = tf.equal(tf.argmax(pred, 1), tf.argmax(y, 1))
    # Calculate accuracy
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
    print("Accuracy:", accuracy.eval({x: mnist.test.images, y: mnist.test.labels}))

```
保存模型
```
    
    # 4. save trained model to filedisk
    save_path = saver.save(
                sess, '{dir}/{model_name}_{ts}/{model_name}.ckpt'.format(
                dir=self.model_path, ts=int(time.time()), model_name=self.__class__.__name__))
    # 5. predict new data
```


#### 高阶API编写更优雅的模型

通过观察模型的一个整个生命周期流程，基本都有几个必须要的流程，比如构建模型，训练模型，评估模型，预测未知，后面3个还需要分别喂给训练数据，验证数据，未知数据。

tf的内置对象**tf.estimator**模块对这些流程进行了封装，统一用**estimator**来标识：

##### 模型包装：

- 可用模型函数自定义的模型：tf.estimator.Estimator, tf.estimator.EstimatorSpec

通过tf.estimator.EstimatorSpec对象定义模型模板，然后用tf.estimator.Estimator创建模型对象
```
# Define the model function (following TF Estimator Template)
def model_fn(features, labels, mode):
    ......
    estim_specs = tf.estimator.EstimatorSpec(
        mode=mode,
        predictions=pred_classes,
        loss=loss_op,
        train_op=train_op,
        eval_metric_ops={'accuracy': acc_op})

    return estim_specs

# Build the Estimator
classifier = tf.estimator.Estimator(model_fn)
```

- tf内置的经典模型，比如DNN，这些直接通过tf.estimator模块即可引用到，可以查询tf的API文档，例如：

```
# Build 3 layer DNN with 10, 20, 10 units respectively.
classifier = tf.estimator.DNNClassifier(feature_columns=feature_columns,
                                      hidden_units=[10, 20, 10],
                                      n_classes=3,
                                      model_dir="/tmp/iris_model")
......
```

这里的classifier即是象estimator的实例，除了定义了模型本身的一些基本属性，比如输入输出数据格式，还自带了一些实用的方法，比如抽象出来的训练，评估，预测方法：

```
# train train data
classifier.train(input_fn=train_input_fn, steps=2000)
# evaluate test data
classifier.evaluate(input_fn=test_input_fn)
# predict new unknown data
classifier.predict(input_fn=predict_input_fn)
```

这三个方法都有一个共同的参数：input_fn输入数据，这是接下来要介绍的输入数据对象。

##### 数据输入规则

模型定义的模型的学习逻辑，数据输入规则是一个函数，这里可以对原始数据集进行一些预处理操作，使之符合模型需要的数据格式要求。

这个函数需要按下面的格式定义，并返回数据：
```
def my_input_fn():

    # Preprocess your data here...

    # ...then return 1) a mapping of feature columns to Tensors with
    # the corresponding feature data, and 2) a Tensor containing labels
    return feature_cols, labels
```
实际情况中，通常我们的数据用numpy的数组格式，可以用tf.estimator.inputs.numpy_input_fn函数来构建我们的输入，函数参数x,y分别对应输入和输出，num_epochs参数和shuffle参数是可选的。

> Two additional arguments are provided: num_epochs: controls the number of epochs to iterate over data. For training, set this to None, so the 
input_fn keeps returning data until the required number of train steps is reached. For evaluate and predict, set this to 1, so the input_fn will iterate over the data once and then raise OutOfRangeError. That error will signal the Estimator to stop evaluate or predict. shuffle: Whether to shuffle the data. For evaluate and predict, set this to False, so the input_fn iterates over the data sequentially. For train, set this to True.


```
# Define the training inputs
  train_input_fn = tf.estimator.inputs.numpy_input_fn(
      x={"x": np.array(training_set.data)},
      y=np.array(training_set.target),
      num_epochs=None,
      shuffle=True)

# Define the test inputs
  test_input_fn = tf.estimator.inputs.numpy_input_fn(
      x={"x": np.array(test_set.data)},
      y=np.array(test_set.target),
      num_epochs=1,
      shuffle=False)

# Define the prediction inputs
new_samples = np.array(
      [[6.4, 3.2, 4.5, 1.5],
       [5.8, 3.1, 5.0, 1.7]], dtype=np.float32)
  predict_input_fn = tf.estimator.inputs.numpy_input_fn(
      x={"x": new_samples},
      num_epochs=1,
      shuffle=False)
```

输入数据定义好后，即可传递给模型的相应的方法，比如train, evaluate, predict
```
# train train data
classifier.train(input_fn=train_input_fn, steps=2000)
# evaluate test data
classifier.evaluate(input_fn=test_input_fn)
# predict new unknown data
classifier.predict(input_fn=predict_input_fn)
```

> 官方推荐的基于tensorflow的工作流：
Recommended workflow
We recommend the following workflow:

1. Assuming a suitable pre-made Estimator exists, use it to build your first model and use its results to establish a baseline.

2. Build and test your overall pipeline, including the integrity and reliability of your data with this pre-made Estimator.

3. If suitable alternative pre-made Estimators are available, run experiments to determine which pre-made Estimator produces the best results.

4. Possibly, further improve your model by building your own custom Estimator.