# MXNet: A Scalable Deep Learning Framework
MXNet is an open-source deep learning framework that allows you to define, train, and deploy deep neural networks on a wide array of devices, from cloud infrastructure to mobile devices. 
It is highly scalable, allowing for fast model training, and supports a flexible programming model and multiple languages. MXNet allows you to mix symbolic and imperative programming flavors to maximize both efficiency and productivity. 
MXNet is built on a dynamic dependency scheduler that automatically parallelizes both symbolic and imperative operations on the fly. 
A graph optimization layer on top of that makes symbolic execution fast and memory efficient. The MXNet library is portable and lightweight, and it scales to multiple GPUs and multiple machines.


&nbsp;

**Flexible Programming Model**
Supports both imperative and symbolic programming, maximizing efficiency and productivity


&nbsp;

**Portable from the Cloud to the Client**
Runs on CPUs or GPUs, and on clusters, servers, desktops, or mobile phones


&nbsp;

**Multi-Lingual**
Supports over seven programming languages, including C++, Python, R, Scala, Julia, Matlab, and Javascript


&nbsp;

**Native Distributed Training**
Supports distributed training on multiple CPU/GPU machines to take advantage of cloud scale 


&nbsp;

**Performance Optimized** 
Parallelizes both I/O and computation with an optimized C++ backend engine, and performs optimally no matter which language you program in


&nbsp;

# MXNet Open Source Community 

**Broad Model Support** – Train and deploy the latest deep convolutional neural networks (CNNs) and long short-term memory (LSTMs) models 


&nbsp;

**Extensive Library of Reference Examples** – Build on sample tutorials (with code), such as image classification, language modeling, neural Artart, and Speech speech recognition, and more.  


&nbsp;

**Open and Collaborative Community** – Support and contributions from many top tier universities and industry partners


&nbsp;

Refer [Setup and Installation](http://mxnet.io/get_started/setup.html) guide to set up and start using MXNet.


&nbsp;


&nbsp;



# Train an MLP on MNIST
 
Before getting started with MXNet, let's take a look at the MXNet programming interface by training a basic digit-recognition model. We will train a [multi-layer perceptron](https://en.wikipedia.org/wiki/Multilayer_perceptron) (MLP) on the [MNIST handwritten digit dataset](http://yann.lecun.com/exdb/mnist/).
Then we'll look at the MXNet interface for tensor computation.


## Symbolic Computation
On MNIST, each example consists of a 28-pixel x 28-pixel gray image of a handwritten digit,
such as
<img src="https://raw.githubusercontent.com/dmlc/web-data/master/mxnet/image/mnist.png"
height="36" >
and its label, which is an integer between 0 and 9.  The
784-length vector of the image and the label is denoted by *x* and *y* pixels, respectively.

Without a hidden layer, the MLP predicts the label probabilities as follows:

 ```eval_rst
    .. math ::
		\textrm{softmax}(W x + b)
 ```

*W* is a 784-by-10 weight matrix and *b* is a 10-length bias vector. The
*softmax* normalizes a vector into a probability distribution as follows:

 ```eval_rst
    .. math ::
      \textrm{softmax}(x) = \left[ \ldots, \frac{\exp(x_i)}{\sum_j \exp(x_j)}\ldots \right]
 ```

To get an MLP with multiple hidden layers, we can stack the layers one by one. Make
*x<sub>0</sub> = x* the output of layer 0. Then layer *i*  outputs a
*n<sub>i</sub>*-length vector:

 ```eval_rst
    .. math ::
      x_i = \sigma_i (W_i x_{i-1} + b_i)
 ```

*σ<sub>i</sub>* is the activation function, such as *tanh*, and
*W<sub>i</sub>* has the size *n<sub>i</sub>*-by-*n<sub>i-1</sub>*. Next, apply
*softmax* to the last layer to obtain the prediction.

The goal of training is to obtain both weights and bias for each layer to
minimize the difference between the predicted label and the real label on the
training data.

The following sections show how to implement the training program
using different languages.

### Python

First, import MXNet:

 ```python
    import mxnet as mx
 ```

Declare the data iterators to the training and validation datasets:

 ```python
    train = mx.io.MNISTIter(
        image      = "mnist/train-images-idx3-ubyte",
        label      = "mnist/train-labels-idx1-ubyte",
        batch_size = 128,
        data_shape = (784, ))
    val   = mx.io.MNISTIter(...)
 ```

Declare a two-layer MLP:

 ```python
    data = mx.symbol.Variable('data')
    fc1  = mx.symbol.FullyConnected(data = data, num_hidden=128)
    act1 = mx.symbol.Activation(data = fc1, act_type="relu")
    fc2  = mx.symbol.FullyConnected(data = act1, num_hidden = 64)
    act2 = mx.symbol.Activation(data = fc2, act_type="relu")
    fc3  = mx.symbol.FullyConnected(data = act2, num_hidden=10)
    mlp  = mx.symbol.SoftmaxOutput(data = fc3, name = 'softmax')
 ```

Train a model on the data:

 ```python
    model = mx.model.FeedForward(
        symbol = mlp,
        num_epoch = 20,
        learning_rate = .1)
    model.fit(X = train, eval_data = val)
 ```

Now, generate the prediction: 
 ```python
    test = mx.io.MNISTIter(...)
    model.predict(X = test)
 ```

### R

Use the `require` command to get the `mxnet` package:

 ```r
    require(mxnet)
 ```

Declare the data iterators to the training and validation datasets:

 ```r
    train <- mx.io.MNISTIter(
      image       = "train-images-idx3-ubyte",
      label       = "train-labels-idx1-ubyte",
      input_shape = c(28, 28, 1),
      batch_size  = 100,
      shuffle     = TRUE,
      flat        = TRUE
    )

    val <- mx.io.MNISTIter(
      image       = "t10k-images-idx3-ubyte",
      label       = "t10k-labels-idx1-ubyte",
      input_shape = c(28, 28, 1),
      batch_size  = 100,
      flat        = TRUE)
 ```

Declare a two-layer MLP:

 ```r
    data <- mx.symbol.Variable('data')
    fc1  <- mx.symbol.FullyConnected(data = data, name = 'fc1', num_hidden = 128)
    act1 <- mx.symbol.Activation(data = fc1, name = 'relu1', act_type = "relu")
    fc2  <- mx.symbol.FullyConnected(data = act1, name = 'fc2', num_hidden = 64)
    act2 <- mx.symbol.Activation(data = fc2, name = 'relu2', act_type = "relu")
    fc3  <- mx.symbol.FullyConnected(data = act2, name = 'fc3', num_hidden = 10)
    mlp  <- mx.symbol.SoftmaxOutput(data = fc3, name = 'softmax')
 ```

Train the model:

 ```r
    model <- mx.model.FeedForward.create(
    X                  = train,
    eval.data          = val,
    ctx                = mx.cpu(),
    symbol             = mlp,
    eval.metric        = mx.metric.accuracy,
    num.round          = 5,
    learning.rate      = 0.1,
    momentum           = 0.9,
    wd                 = 0.0001,
    array.batch.size   = 100,
    epoch.end.callback = mx.callback.save.checkpoint("mnist"),
    batch.end.callback = mx.callback.log.train.metric(50)
    )
 ```

Read a new dataset and generate a prediction:

 ```r
    test <- mx.io.MNISTIter(...)
    preds <- predict(model, test)
 ```

### Scala

Import MXNet:

 ```scala
    import ml.dmlc.mxnet._
 ```

Declare the data iterators to the training and validation datasets:

 ```scala
    val trainDataIter = IO.MNISTIter(Map(
    "image" -> "data/train-images-idx3-ubyte",
    "label" -> "data/train-labels-idx1-ubyte",
    "data_shape" -> "(1, 28, 28)",
    "label_name" -> "sm_label",
    "batch_size" -> batchSize.toString,
    "shuffle" -> "1",
    "flat" -> "0",
    "silent" -> "0",
    "seed" -> "10"))

    val valDataIter = IO.MNISTIter(Map(
      "image" -> "data/t10k-images-idx3-ubyte",
      "label" -> "data/t10k-labels-idx1-ubyte",
      "data_shape" -> "(1, 28, 28)",
      "label_name" -> "sm_label",
      "batch_size" -> batchSize.toString,
      "shuffle" -> "1",
      "flat" -> "0", "silent" -> "0"))
 ```

Declare a two-layer MLP:

 ```scala
    val data = Symbol.Variable("data")
    val fc1 = Symbol.FullyConnected(name = "fc1")(Map("data" -> data, "num_hidden" -> 128))
    val act1 = Symbol.Activation(name = "relu1")(Map("data" -> fc1, "act_type" -> "relu"))
    val fc2 = Symbol.FullyConnected(name = "fc2")(Map("data" -> act1, "num_hidden" -> 64))
    val act2 = Symbol.Activation(name = "relu2")(Map("data" -> fc2, "act_type" -> "relu"))
    val fc3 = Symbol.FullyConnected(name = "fc3")(Map("data" -> act2, "num_hidden" -> 10))
    val mlp = Symbol.SoftmaxOutput(name = "sm")(Map("data" -> fc3))
 ```

Train a model on the data:

 ```scala
    import ml.dmlc.mxnet.optimizer.SGD
    // setup model and fit the training set
    val model = FeedForward.newBuilder(mlp)
      .setContext(Context.cpu())
      .setNumEpoch(10)
      .setOptimizer(new SGD(learningRate = 0.1f, momentum = 0.9f, wd = 0.0001f))
      .setTrainData(trainDataIter)
      .setEvalData(valDataIter)
      .build()
 ```

Generate a prediction:

 ```scala
    val probArrays = model.predict(valDataIter)
    // in this case, we do not have multiple outputs
    require(probArrays.length == 1)
    val prob = probArrays(0)
    // get predicted labels
    val py = NDArray.argmaxChannel(prob)
    // deal with predicted labels 'py'
 ```

### Julia

Import MXNet:

 ```julia
    using MXNet
 ```

Load the data:

 ```julia
    batch_size = 100
    include("mnist-data.jl")
    train_provider, eval_provider = get_mnist_providers(batch_size)
 ```

Define the MLP:

 ```julia
    mlp = @mx.chain mx.Variable(:data)  =>
      mx.FullyConnected(num_hidden=128) =>
      mx.Activation(act_type=:relu)     =>
      mx.FullyConnected(num_hidden=64)  =>
      mx.Activation(act_type=:relu)     =>
      mx.FullyConnected(num_hidden=10)  =>
      mx.SoftmaxOutput(name=:softmax)
 ```

Train the model:

 ```julia
    model = mx.FeedForward(mlp, context=mx.cpu())
    optimizer = mx.SGD(lr=0.1, momentum=0.9, weight_decay=0.00001)
    mx.fit(model, optimizer, train_provider, n_epoch=20, eval_data=eval_provider)
 ```

Generate a prediction:

 ```julia
    probs = mx.predict(model, test_provider)
 ```

## Tensor Computation

Now let's take a look at the tensor computation interface. The tensor computation interface is often more
flexible than the symbolic interface. It is often used to
implement the layers, define weight updating rules, and debug.


### Python

The Python interface is similar to `numpy.NDArray`:

 ```python
    >>> import mxnet as mx
    >>> a = mx.nd.ones((2, 3),
    ... mx.gpu())
    >>> print (a * 2).asnumpy()
    [[ 2.  2.  2.]
     [ 2.  2.  2.]]
 ```

### R

 ```r
    > require(mxnet)
    Loading required package: mxnet
    > a <- mx.nd.ones(c(2,3))
    > a
         [,1] [,2] [,3]
    [1,]    1    1    1
    [2,]    1    1    1
    > a + 1
         [,1] [,2] [,3]
    [1,]    2    2    2
    [2,]    2    2    2
 ```

### Scala

You can perform tensor or matrix computation in pure Scala:

 ```scala
    scala> import ml.dmlc.mxnet._
    import ml.dmlc.mxnet._

    scala> val arr = NDArray.ones(2, 3)
    arr: ml.dmlc.mxnet.NDArray = ml.dmlc.mxnet.NDArray@f5e74790

    scala> arr.shape
    res0: ml.dmlc.mxnet.Shape = (2,3)

    scala> (arr * 2).toArray
    res2: Array[Float] = Array(2.0, 2.0, 2.0, 2.0, 2.0, 2.0)

    scala> (arr * 2).shape
    res3: ml.dmlc.mxnet.Shape = (2,3)
 ```

# Next Steps
* [Setup and Installation](http://mxnet.io/get_started/setup.html)
* [Tutorials](http://mxnet.io/tutorials/index.html)
* [How To](http://mxnet.io/how_to/index.html)
* [Architecture](http://mxnet.io/architecture/index.html)