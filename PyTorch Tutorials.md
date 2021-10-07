## PyTorch Tutorials——LEARN THE BASICS

#### 前言

苦于不熟悉网络的构成以及没有合适的练手项目，所以选择了pytorch官方的教程入手进行学习，希望可以对相关概念和技术比较熟悉，运行此网络时是在服务器的终端上手写，由于没有拼写检查功能，导致漏洞百出，debug了30多遍，不过最终也运行成功。希望这个小demo可以见证我的进步与成长，故开此系列贴以做纪念。

### overview

- Most machine learning workflows
    - working with data
    
    - creating models
    
    - optimizing model parameters
    
    - saving the trained models
    

We’ll use the FashionMNIST dataset to train a neural network that predicts if an input image belongs to one of the following classes: 

- T-shirt/top
- Trouser
- Pullover
- Dress
- Coat
- Sandal
- Shirt
- Sneaker
- Bag
- Ankle boot

### 一些基础认知

#### 什么是batch？

1. 对于一个有 2000 个训练样本的数据集。将 2000 个样本分成大小为 500 的 batch，那么完成一个 epoch 需要 4 个 iteration。 
2. 如果把准备训练数据比喻成一块准备下火锅的牛肉，那么epoch就是整块牛肉，batch就是切片后的牛肉片，iteration就是涮一块牛肉片。

![img](https://upload-images.jianshu.io/upload_images/4563513-54bf713d97ff1d53.png?imageMogr2/auto-orient/strip|imageView2/2/w/557)

#### batch用来干什么？

- 在使用“模型-策略-算法”三大步骤构建网络模型时，需要决定使用什么方式将牛肉下火锅，即将数据喂给模型，训练出最佳参数
  - 理想状态：将所有数据全部喂给框架，求出最小化损失，再更新参数重复这个过程，但是就像是煮一整块牛肉一样，你不知道什么时候才能吃
  - 另一个极端状态：每次就下一个牛肉卷，每次只给模型喂一条数据，肉卷立马就熟了，虽然很快，但是一个不小心就会碎掉导致没法吃了，即可能无法求出局部最优
  - 折中方案：综合考虑又快又能得到较优的结果，就选择每次给锅里下一个肉片，讲数据切割成batch大小的一块，每次iteration只吃一块，即每次只计算这一个batch的损失函数并更改参数

#### 如何实现batch？

yield->generator

```python
# --------------函数说明-----------------
# sourceData_feature ：训练集的feature部分
# sourceData_label   ：训练集的label部分
# batch_size  ： 牛肉片的厚度
# num_epochs  ： 牛肉翻煮多少次
# shuffle ： 是否打乱数据

def batch_iter(sourceData_feature,sourceData_label, batch_size, num_epochs, shuffle=True):
    
    data_size = len(sourceData_feature)
    
    num_batches_per_epoch = int(data_size / batch_size)  # 样本数/batch块大小,多出来的“尾数”，不要了
    
    for epoch in range(num_epochs):
        # Shuffle the data at each epoch
        if shuffle:
            shuffle_indices = np.random.permutation(np.arange(data_size))
            
            shuffled_data_feature = sourceData_feature[shuffle_indices]
            shuffled_data_label   = sourceData_label[shuffle_indices]
        else:
            shuffled_data_feature = sourceData_feature
            shuffled_data_label = sourceData_label

        for batch_num in range(num_batches_per_epoch):   # batch_num取值0到num_batches_per_epoch-1
            start_index = batch_num * batch_size
            end_index = min((batch_num + 1) * batch_size, data_size)

            yield (shuffled_data_feature[start_index:end_index] , shuffled_data_label[start_index:end_index])

```




```python
batchSize = 100 # 定义具体的牛肉厚度
Iterations = 0  #  记录迭代的次数

# sess
sess = tf.Session()
sess.run(tf.global_variables_initializer())

# 迭代 必须注意batch_iter是yield→generator，所以for语句有特别
for (batchInput, batchLabels) in batch_iter(mnist.train.images, mnist.train.labels, batchSize, 30, shuffle=True):
    trainingLoss = sess.run([opt,loss], feed_dict = {X: batchInput, y:batchLabels})
    if Iterations%1000 == 0:  # 每迭代一千次，输出一次效果
        train_accuracy = sess.run(accuracy, feed_dict={X:batchInput, y:batchLabels})
        print("step %d, training accuracy %g"%(Iterations,train_accuracy))
    Iterations=Iterations+1    

```

### 另一种解释

#### batch_size

深度学习的优化算法，其实就是梯度下降，主要分为三种：

- 批量梯度下降算法（BGD，Batch gradient descent algorithm）
- 随机梯度下降算法（SGD，Stochastic gradient descent algorithm）
- 小批量梯度下降算法（MBGD，Mini-batch gradient descent algorithm）

批量梯度下降算法，每一次计算都需要遍历全部数据集，更新梯度，计算开销大，花费时间长，不支持在线学习。

随机梯度下降算法，每次随机选取一条数据，求梯度更新参数，这种方法计算速度快，但是收敛性能不太好，可能在最优点附近晃来晃去，hit不到最优点。两次参数的更新也有可能互相抵消掉，造成目标函数震荡的比较剧烈。

为了克服两种方法的缺点，现在一般采用的是一种折中手段，mini-batch gradient decent，小批的梯度下降，这种方法把数据分为若干个批，按批来更新参数，这样，一个批中的一组数据共同决定了本次梯度的方向，下降起来就不容易跑偏，减少了随机性。另一方面因为批的样本数与整个数据集相比小了很多，计算量也不是很大。

tf框架中的batch_size指的就是更新梯度中使用的样本数。当然这里如果把batch_size设置为数据集的长度，就成了批量梯度下降算法，batch_size设置为1就是随机梯度下降算法。

#### iterations

迭代次数，每次迭代更新一次网络结构的参数。

迭代是重复反馈的动作，神经网络中我们希望通过迭代进行多次的训练以到达所需的目标或结果。

每一次迭代得到的结果都会被作为下一次迭代的初始值。

一个迭代 = 一个（batch_size）数据正向通过（forward）+ 一个（batch_size）数据反向（backward）

前向传播：构建由（x1,x2,x3）得到Y=h_W,b_(x)的表达式

反向传播：基于给定的损失函数，求解参数的过程

![神经网络](https://img-blog.csdnimg.cn/2019022523170956.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aGlua2dhbWVyLmJsb2cuY3Nkbi5uZXQ=,size_16,color_FFFFFF,t_70)

#### epochs

epochs被定义为前向和反向传播中所有批次的单次训练迭代。这意味着1个周期是整个输入数据的单次前向和反向传递。

简单说，epochs指的就是训练过程中数据将被“轮”多少次

例如在某次模型训练过程中，总的样本数是10000，batch_size=100，epochs=10，其对应的伪代码如下：

```python
data = 
batch_size = 100
for i in range(epochs):
    for j in range(int(data_length / batch_size - 1)):
        x_data = data[begin:end, ]
        y_data = data[begin:end, ]
        mode.train(x_data, y_data)
        begin += batch_size
        end += batch_size
```

其中iterations = data_length / batch_size

有了以上基本认知后，对网络的理解就更通透了，下面看代码

###　CODE

```python
import torch
from torch import nn
from torch.utils.data import DataLoader
'''
PyTorch has two primitives to work with data: 
		- torch.utils.data.DataLoader
		- torch.utils.data.Dataset
Dataset stores the samples and their corresponding labels, and DataLoader wraps an iterable around the Dataset.
'''
from torchvision import datasets # 从torchvision中导入数据集
'''
PyTorch offers domain-specific libraries such as TorchText, TorchVision, and TorchAudio, all of which include datasets. For this tutorial, we will be using a TorchVision dataset.
Every TorchVision Dataset includes two arguments: 
        - transform
        - target_transform 
to modify the samples and labels respectively.
'''
from torchvision.transforms import ToTensor,Lambda,Compose
'''
Transforms are common image transformations. They can be chained together using Compose. Most transform classes have a function equivalent: functional transforms give fine-grained control over the transformations. 

The transformations that accept tensor images also accept batches of tensor images. 
A Tensor Image is a tensor with (C, H, W) shape, where C is a number of channels, H and W are image height and width. 
A batch of Tensor Images is a tensor of (B, C, H, W) shape, where B is a number of images in the batch.

The expected range of the values of a tensor image is implicitely defined by the tensor dtype. Tensor images with a float dtype are expected to have values in [0, 1). Tensor images with an integer dtype are expected to have values in [0, MAX_DTYPE] where MAX_DTYPE is the largest value that can be represented in that dtype.
'''
import matplotlib.pyplot as plt

# Download training data from open datasets.
training_data = datasets.FashionMNIST(
        root="data",
        train=False,
        download=True,
        transform=ToTensor(),
)

'''
Fashion-MNIST is a dataset of Zalando’s article images consisting of 60,000 training examples and 10,000 test examples. Each example comprises a 28×28 grayscale image and an associated label from one of 10 classes.

We load the FashionMNIST Dataset with the following parameters:
        "root" is the path where the train/test data is stored,
        "train" specifies training or test dataset,
        "download=True" downloads the data from the internet if it’s not available at root.
        "transform" and "target_transform" specify the feature and label transformations 
'''
# Download test data from open datasets.
test_data = datasets.FashionMNIST(
        root="data",
        train=False,
        download=True,
        transform=ToTensor(),


)
# 设置batch的尺寸
batch_size = 64

# create data loaders
# 此处读取数据集时将数据集按照batch_size进行分割，即64张图片为一组
'''
We pass the Dataset as an argument to DataLoader. This wraps an iterable over our dataset, and supports automatic batching, sampling, shuffling and multiprocess data loading. Here we define a batch size of 64, i.e. each element in the dataloader iterable will return a batch of 64 features and labels.
'''
train_dataloader = DataLoader(training_data,batch_size=batch_size)
test_dataloader = DataLoader(test_data,batch_size=batch_size)

for X,y in test_dataloader:
    # A  batch of Tensor Images is a tensor of (N, C, H, W) shape, where N is a number of images in the batch.
        print("shape of X [N,C,H,W]:",X.shape)
        print("shape of y :",y.shape,y.dtype)
        break

#create models
#get cpu or gpu device for training
'''
To define a neural network in PyTorch, we create a class that inherits from nn.Module. We define the layers of the network in the __init__ function. Every nn.Module subclass implements the operations on input data in the forward method. To accelerate operations in the neural network, we move it to the GPU if available.

'''
device = "cuda" if torch.cuda.is_available() else "cpu"
print("Using {} device".format(device))
#define model
class NeuralNetwork(nn.Module):
        def __init__(self):
                super(NeuralNetwork,self).__init__()
                self.flatten = nn.Flatten()
                #We initialize the nn.Flatten layer to convert each 2D 28x28 image into a contiguous array of 784 pixel values ( the minibatch dimension (at dim=0) is maintained).
				#nn.Sequential is an ordered container of modules. The data is passed through all the modules in the same order as defined. You can use sequential containers to put together a quick network like seq_modules.
                self.linear_relu_stack = nn.Sequential(
                        nn.Linear(28*28,512),
                    	#The linear layer is a module that applies a linear transformation on the input using its stored weights and biases.
                        nn.ReLU(),
                    	#Non-linear activations are what create the complex mappings between the model’s inputs and outputs. They are applied after linear transformations to introduce nonlinearity, helping neural networks learn a wide variety of phenomena.
                        nn.Linear(512,512),
                        nn.ReLU(),
                        nn.Linear(512,10)
        )


        def forward(self,x):
                x = self.flatten(x)
                logits = self.linear_relu_stack(x)
                return logits
#We create an instance of NeuralNetwork, and move it to the device, and print its structure.
model = NeuralNetwork().to(device)
print(model)
'''
打印出网络结构
NeuralNetwork(
  (flatten): Flatten(start_dim=1, end_dim=-1)
  (linear_relu_stack): Sequential(
    (0): Linear(in_features=784, out_features=512, bias=True)
    (1): ReLU()
    (2): Linear(in_features=512, out_features=512, bias=True)
    (3): ReLU()
    (4): Linear(in_features=512, out_features=10, bias=True)
  )
'''
#Optimizing the Model Parameters
#To train a model, we need a loss function and an optimizer.
'''
Loss Function

When presented with some training data, our untrained network is likely not to give the correct answer. Loss function measures the degree of dissimilarity of obtained result to the target value, and it is the loss function that we want to minimize during training. To calculate the loss we make a prediction using the inputs of our given data sample and compare it against the true data label value.

Common loss functions include nn.MSELoss (Mean Square Error) for regression tasks, and nn.NLLLoss (Negative Log Likelihood) for classification. nn.CrossEntropyLoss combines nn.LogSoftmax and nn.NLLLoss.

We pass our model’s output logits to nn.CrossEntropyLoss, which will normalize the logits and compute the prediction error.
'''
loss_fn = nn.CrossEntropyLoss()
'''
Optimization is the process of adjusting model parameters to reduce model error in each training step. Optimization algorithms define how this process is performed (in this example we use Stochastic Gradient Descent). All optimization logic is encapsulated in the optimizer object. Here, we use the SGD optimizer; additionally, there are many different optimizers available in PyTorch such as ADAM and RMSProp, that work better for different kinds of models and data.

We initialize the optimizer by registering the model’s parameters that need to be trained, and passing in the learning rate hyperparameter.
'''
optimizer = torch.optim.SGD(model.parameters(),lr=1e-3)

#此处lr即为Learning Rate
#how much to update models parameters at each batch/epoch. Smaller values yield slow learning speed, while large values may result in unpredictable behavior during training.
'''
Inside the training loop, optimization happens in three steps:

        Call optimizer.zero_grad() to reset the gradients of model parameters. Gradients by default add up; to prevent double-counting, we explicitly zero them at each iteration.
        Backpropagate the prediction loss with a call to loss.backwards(). PyTorch deposits the gradients of the loss w.r.t. each parameter.
        Once we have our gradients, we call optimizer.step() to adjust the parameters by the gradients collected in the backward pass.
'''


#In a single training loop, the model makes predictions on the training dataset (fed to it in batches), and backpropagates the prediction error to adjust the model’s parameters.

def train(dataloader, model, loss_fn, optimizer):
    size = len(dataloader.dataset)
    model.train()
    for batch, (X, y) in enumerate(dataloader):
        X, y = X.to(device), y.to(device)

        # Compute prediction error
        pred = model(X)
        loss = loss_fn(pred, y)
    
        # Backpropagation
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
    
        if batch % 100 == 0:
            loss, current = loss.item(), batch * len(X)
            print(f"loss: {loss:>7f}  [{current:>5d}/{size:>5d}]")

#We also check the model’s performance against the test dataset to ensure it is learning.

def test(dataloader,model,loss_fn):
        size = len(dataloader.dataset)
        num_batches = len(dataloader)
        model.eval()
        test_loss,correct = 0,0
        with torch.no_grad():
                for X,y in dataloader:
                        X,y = X.to(device),y.to(device)
                        pred = model(X)
                        test_loss += loss_fn(pred,y).item()
                        correct += (pred.argmax(1) == y).type(torch.float).sum().item()
        test_loss /= num_batches
        correct /= size
        print(f"Test Error:\n Accuracy:{(100*correct):>0.1f}%,avg loss: {test_loss:>8f}\n")
        
#The training process is conducted over several iterations (epochs). During each epoch, the model learns parameters to make better predictions. We print the model’s accuracy and loss at each epoch; we’d like to see the accuracy increase and the loss decrease with every epoch.

epochs = 5
for t in range(epochs):
      	print(f"Epoch {t+1}\n-------------------------------")
    	train(train_dataloader, model, loss_fn, optimizer)
    	test(test_dataloader, model, loss_fn)
print("Done!")

#A common way to save a model is to serialize the internal state dictionary (containing the model parameters).
torch.save(model.state_dict(), "model.pth")
print("Saved PyTorch Model State to model.pth")

#The process for loading a model includes re-creating the model structure and loading the state dictionary into it.
model = NeuralNetwork()
model.load_state_dict(torch.load("model.pth"))

classes = ["T-shirt/top",
    "Trouser",
    "Pullover",
    "Dress",
    "Coat",
    "Sandal",
    "Shirt",
    "Sneaker",
    "Bag",
    "Ankle boot",]

model.eval()
x,y = test_data[0][0],test_data[0][1]
with torch.no_grad():
        pred = model(x)
        predicted,actual = classes[pred[0].argmax(0)],classes[y]
        print(f'predicted:"{predicted}".actual:"{actual}"')
        
'''
超参数
Hyperparameters are adjustable parameters that let you control the model optimization process. 
Different hyperparameter values can impact model training and convergence rates 

We define the following hyperparameters for training:
- Number of Epochs - the number times to iterate over the dataset
- Batch Size - the number of data samples propagated through the network before the parameters are updated
- Learning Rate - how much to update models parameters at each batch/epoch. Smaller values yield slow learning speed, while large values may result in unpredictable behavior during training.
'''
