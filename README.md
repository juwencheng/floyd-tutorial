> 工欲善其事，必先利其器

随着 『deel learning』 学习深入，愈发觉得电脑力不从心，训练一层 『hidden layer』 的时间还能忍受，多层或者更复杂的模型所花费的时间动辄几十分钟，甚至几小时。等待虽然美好，可是太煎熬，何况谁也不能保证一次就能得到最好的结果，反复折磨下来肯定身心疲惫。

因此，自己买不了好的显卡，只能用别人的资源，当然是有偿使用哈。加入 [Udacity](https://www.udacity.com) 课程后，收到 [AWS](https://aws.amazon.com) 云服务器的 $100 优惠券，可以直接使用的。按照课程介绍创建了 G2 型服务器，操作起来有点复杂，价格也不便宜。后来在论坛里看到 [Floyd](https://www.floydhub.com/) 的介绍，又去申请注册了账户，能领取 100 小时的免费 GPU 使用。目前为止使用体验不错，特记录下使用过程中遇到的问题。（申请账户的过程就省略了）

## 配置环境

申请账户之后，需要在本地配置 floyd-cli 。[官方文档](https://www.floydhub.com/welcome)。

```bash
# 安装 cli
pip install -U floyd-cli

# 登录账户
floyd login

# 拷贝授权码到终端
# 输入登录命令后会提示你授权码页面将打开您的浏览器，是否打开，输入y 即可
```

## [项目] Hello World!

官方文档是训练一个 RNN 模型，为了简化过程我们输出一个 `Hello World!`吧。在本地创建文件夹 `floyd-tutorial` ，
然后创建 `hello.py` ，输入如下内容：

{% gist owenju/143b22536997cd0757491e044206cf4d %}

现在目录结构如下：

```
.
└── hello.py

0 directories, 1 file
```

### 使用 floyd 初始化工程

确保当前位置是在刚创建文件夹下面。

```bash
floyd init hello
```

现在目录结构如下：

```bash
.
├── .floydexpt
├── .floydignore
└── hello.py

0 directories, 3 files
```

可以看到多了两个 floyd 的配置文件。第二个文件应该很熟悉，和 `.gitignore` 作用类似，即排除不需要传到 floyd 服务器的文件。

### 在 floyd 执行代码

floyd 会将本地代码传到服务器，并在执行你的代码（从日志中可以看到，服务端使用的是 `docker` 技术）。

```bash
floyd run
```

如果提示没有登录，先执行 `floyd login` ，成功之后再执行 `floyd run` 。返回结果如下：

```bash
Creating project run. Total upload size: 328.0B
Syncing code ...
Done=============================] 1177/1177 - 00:00:00
RUN ID                  NAME              VERSION
----------------------  --------------  ---------
Y65Vm9BqhB7CvvNZNs3yHN  owenju/hello:1          1


To view logs enter:
    floyd logs Y65Vm9BqhB7CvvNZNs3yHN
```

### 查看运行结果

运行查看日志命令

```bash
floyd logs Y65Vm9BqhB7CvvNZNs3yHN
```

可以看到如下输入（截取部分）

```bash
################################################################################

2017-05-11 17:36:41,858 INFO - Run Output:
2017-05-11 17:36:41,891 INFO - Hello World!
2017-05-11 17:36:41,892 INFO - 你好!
2017-05-11 17:36:41,931 INFO -
################################################################################
```

## [项目] jupyter notebook

`jupyter notebook` 有很多优点，在上面学习，实验都很方便。唯一的缺点就是没及时关闭会一直计费。`jupyter notebook` 启动后服务器已为你分配了资源，即使没有在上面进行任何操作也会收费。

因此我的使用习惯是，在 floyd 上运行 python 代码。执行完就会自动结束。这样也不会花费太多的 money 。使用方法如下：

```bash
floyd run --mode jupyter
```

## floyd 管理

介绍 experiments modules data 概念及介绍如何管理

### Data 管理

#### 查看状态

查看有当前实验产生了多少数据，用 `floyd data status`

#### 删除

删除某个实验产生的数据

```bash
floyd data delete [dataid]
```

#### 上传

进入到需要上传的文件目录

```bash
cd mnist_data/
floyd data init mnist-data
floyd data upload
```

在实验中要使用这个数据，先要挂载到某个目录，然后将代码里数据读取路径改为该目录。

```bash
floyd run --data GY3QRFFUA8KpbnqvroTPPW "ls -la /input/mnist-data"
```

使用多个数据源

```bash
floyd run --data GY3QRFFUA8KpbnqvroTPPW:training --data 4T9bF6vtyvrHLdUsFbUumA:testing "python script.py"
```

会自动挂载到 `/training` 和 `/testing` 目录，如果不指定名字，则挂载到 `/GY3QRFFUA8KpbnqvroTPPW` 和 `/4T9bF6vtyvrHLdUsFbUumA` 目录。

#### 把 output 作为下一个实验的数据源

可以看到使用数据源是需要得到代表数据的数据 id，如果能够获得上次实验的输出数据 id 就可以再次利用。使用方法如下：

{% gist owenju/143b22536997cd0757491e044206cf4d %}


### Experiments 管理

一个项目有很多实验，多次实验后会产生很多多余数据，因此需要管理。

#### 查看状态

```bash
floyd status
```

#### 删除

```bash
floyd delete [experimentid]
```

## 常见问题

### 1. 指定以 GPU 方式运行

Hello World! 例子中实现在服务器运行代码功能，默认使用 CPU 运行。如果要使用 GPU 来训练模型，只需要加上 `--gpu` 可选项即可

```bash
floyd run --gpu 'python hello.py'
```

### 2. 指定 tensorflow 版本

开始在 floyd 上运行 `jupyter notebook` 时，提示版本太低，为了解决这个问题特意咨询了他们工程师，找到的方法是：

```bash
floyd run --mode jupyter --env tensorflow-1.0
```

### 3. 保存输出文件

训练模型之后需要将模型文件保存到服务器供我们下载。如果不注意保存路径，训练完后无法找到任何输出文件，白白浪费了 GPU 资源。下面是训练 GPU 模型保存代码片段，需要将文件保存到 `/output/` 路径下才行。

```python
saver = tf.train.Saver()
save_path = saver.save(sess, "/output/conv_model")
```
