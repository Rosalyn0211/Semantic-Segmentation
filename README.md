# Semantic-Segmentation

### Paper Reading
 - FCN Fully Convolutional Networks for Semantic Segmentation
 - U-Net: Convolutional Networks for BiomedicalImage Segmentation  
 - MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications
 - MobileNetV2: Inverted Residuals and Linear Bottlenecks
   
   

## FCN
### 关键问题
 - 解决了什么问题？
    - 建立全卷积网络，可接受任意大小的输入，解决图像语义分割问题。
 - 使用了什么方法？
    - 将目前的分类网络（AlexNet，VGG net和GoogLeNet）转变为完全卷积网络，并通过微调将其转移至分割任务。
    - 跳级("skip")结构，综合了不同深度的语义信息，提高分割的精度
    - 使用了转置卷积，对低分辨率的图像进行上采样，输出同分辨率的图像。
 - 效果如何？
    - 在PASCAL VOC数据集上，mean IU达到62.2%，较之前的方法提升了20%。
    - 时间减少了五分之一，每张图像处理时间约为0.2s。
 - 存在问题？
    - 因为模型是基于CNN改进而来，依然是独立像素进行分类，没有考虑到像素与像素之间的关系。
    - 分割结果不够精细，图像过于模糊或平滑，没有分割出目标图像的细节。

### CNN到FCN
以AlexNet为例，FCN将CNN的全连接层替换为卷积层。  

![image](images/alexnet.png)  

 - 为什么要将全连接层换为卷积层呢？
   - 如果卷积核的 kernel_size 和输入 feature maps 的 size 一样，那么相当于该卷积核计算了全部 feature maps 的信息，则相当于是一个 kernel_size∗1 的全连接。
   - 全连接的结构是固定的，当我们训练完时每个连接都是有权重的。而卷积过程我们其实为训练连接结构，学习了目标和那些像素之间有关系，权重较弱的像素我们可以忽略。
   - 全连接不会学习过滤，会给每个连接分权重并不会修改连接关系。卷积则是会学习有用的关系，没用得到关系它会弱化或者直接 dropout。这样卷积块可以共用一套权重，减少重复计算，还可以降低模型复杂度。
   
### 转置卷积
![image](images/转置卷积效果.png)  

a 是输入图像，b 是经过卷积得到的特征图，分辨率明显下降。经过上采样（转置卷积）提升分辨率得到同时，还保证了特征所在区域的权重，最后将图片的分辨率提升原图一致后，权重高的区域则为目标所在区域。

FCN 模型处理过程也是这样，通过卷积和转置卷积我们基本能定位到目标区域，但是，我们会发现模型前期是通过卷积、池化、非线性激活函数等作用输出了特征权重图像，我们经过反卷积等操作输出的图像实际是很粗糙的，毕竟丢了很多细节。因此我们需要找到一种方式填补丢失的细节数据，所以就有了跳跃结构。

### Skip结构
![image](images/跳级结构.png)  

 - FCN-32s
    - 从特征小图（16*16*4096）预测分割小图（16*16*21），之后直接升采样为大图。转置卷积的步长为32，这个网络称为FCN-32s。
 - FCN-16s
    - 升采样分为两次完成。在第二次升采样前，把第4个pooling层的预测结果融合进来。使用跳级结构提升精确性。第二次转置卷积步长为16，这个网络称为FCN-16s。
 - FCN-8s
    - 升采样分为三次完成。进一步融合了第3个pooling层的预测结果。第三次反卷积步长为8，记为FCN-8s。 

![image](images/跳级结构效果.png)  
较浅层的预测结果包含了更多细节信息。比较2,3,4阶段可以看出，跳级结构利用浅层信息辅助逐步升采样，有更精细的结果。 


## U-Net

### 关键问题
 - 解决了什么问题？
    - 提出一种网络结构和训练策略，能够适应很小的训练集。并且适合超大图像分割，适合医学图像分割。
 - 使用了什么方法？
    - 编码器-解码器结构。编码器逐渐减少池化层的空间维度，解码器逐步修复物体的细节和空间维度。编码器和解码器之间通常存在快捷连接，因此能帮助解码器更好地修复目标的细节。


### 网络结构
![image](images/UNET.png)  
 - 在上图中，每一个蓝色块表示一个多通道特征图，特征图的通道数标记在顶部，X-Y尺寸设置在块的左下边缘。不同颜色的箭头代表不同的操作。图的左半部分是收缩路径，右半部分是扩展路径。
 - 网络对于输入的大小也是有要求的。为了使得输出的分割图无缝拼接，重要的是选择输入块的大小，以便所有的2 X 2的池化层都可以应用于偶数的 x 层和 y 层。一个比较好的方法是从最下的分辨率从反向推到，比如说在网络结构中，最小的是32 X 32，沿着收缩路径的反向进行推导可知，输入图像的尺寸应该为572×572。
 - 其中需要注意的是，每经过一次上采样都会将通道数减半，再与收缩路径对应的特征图进行拼接。在拼接之前进行 crop 是必要的(例如在上图中，6464大与5656，为了使这两个特征图能够顺利拼接，取6464中间部分5454的大小，然后拼接)，因为两者的尺寸并不相同（主要是因为 valid conv 造成的）。最后一层使用1 X 1大小的卷积核，将通道数降低至特定的数量（如像素点的类别数量）。
 
### U-Net与FCN的区别
U-net与FCN的不同在于，U-net的上采样依然有大量的通道，这使得网络将上下文信息向更高层分辨率传播，作为结果，扩展路径与收缩路径对称，形成一个U型的形状（如上图所示）。 网络没有全连接层并且只是用每一个卷积层的有效部分。

### Overlap-tile
![image](images/overlap_tile.png)  
 - 上图是针对任意大小的输入图像的无缝分割的 Overlap-tile 策略。如果我们要预测黄色框内区域（即对黄色的内的细胞进行分割，获取它们的边缘），需要将蓝色框内部分作为输入，如果黄色区域在输入图像的边缘的话，那么缺失的数据使用镜像进行补充。如上图左边图像所示，输入图像周围一圈都进行了镜像补充。
 - 因为进行的是valid卷积，即上下文只取有效部分，可以理解为padding为0，卷积之后的图像尺寸会改变，所以需要取比黄色框大的图像来保证上下文的信息是有意义的，缺失的部分用镜像的方法补充是填充上下文信息最好的方法了。这种方法通常需要将图像进行分块的时候才使用。
 - 那么为什么要对图像分块，不输入整张图像呢？因为内存限制，有的机器内存比较小，需要分块输入。但比之前的滑窗取块要好很多，一方面不用取那么多块，另一方面块之间也没有那么大的区域重叠。通过Overlap-tile 策略可以将图像分块输入，否则的话就只能对图像进行 resize 了，但是这样会降低输入图像的分辨率。
 
### 卷积的三种模式:full, same, valid
三种不同模式是对卷积核移动范围的不同限制。  
 - full conv

<img width="300" height="300" src="images/fullconv.png"/>

橙色部分为image, 蓝色部分为filter。full模式的意思是，从filter和image刚相交开始做卷积，白色部分为填0。filter的运动范围如图所示。  

 - same conv  
   
<img width="300" height="300" src="images/sameconv.png"/>

当filter的中心(K)与image的边角重合时，开始做卷积运算，可见filter的运动范围比full模式小了一圈。注意：这里的same还有一个意思，卷积之后输出的feature map尺寸保持不变(相对于输入图片)。当然，same模式不代表完全输入输出尺寸一样，也跟卷积核的步长有关系。same模式也是最常见的模式，因为这种模式可以在前向传播的过程中让特征图的大小保持不变，调参师不需要精准计算其尺寸变化(因为尺寸根本就没变化)。  

 - valid conv  
 
 <img width="300" height="300" src="images/validconv.png"/>

 
 当filter全部在image里面的时候，进行卷积运算，可见filter的移动范围较same更小了。

## MobileNets

### 关键问题
 - 解决了什么问题？
    - 为了提高精度，现有的卷积神经网络变得越来越复杂，从网络的尺寸和速度来讲，这些设计并不是必要的。在实际应用中，比如自动驾驶、人机交互、增强现实等计算量有限的平台需要更轻量的卷积神经网络。
 - 使用了什么方法？
    - 深度可分离卷积(depthwise separable convolution)
    - 全局超参数(width multiplier&resolution multiplier)
 - 效果如何？
    - 在ImageNet上达到了70.6%的准确率，相比于Conv MobileNet值降低了10%左右。
 - 存在问题？
    - 在轻量级与速度的基础上，准确率有待提高。
    
### Depthwise separable convolution
 - Depthwise separable convolution = depthwise conv + pointwise conv  
 
 ![conv](images/conv.png)  
 
 ![conv](images/dw.png)  
  
 ![conv](images/vs.png)  
 
 - 减少了计算次数及参数
 
### 全局超参数
 - 宽度乘数 α：为了构建更小和更少计算量的网络，作者引入了宽度乘数α ，作用是改变输入输出通道数，减少特征图数量，让网络变瘦。其中，α 取值是0~1，应用宽度乘数可以进一步减少计算量，大约有 α^2 的优化空间。
 - 分辨率乘数ρ:分辨率乘数用来改变输入数据层的分辨率，同样也能减少参数。
### 网络结构
 ![conv](images/mobilenet.png)    
 
MobileNets结构建立在上述深度可分解卷积中（只有第一层是标准卷积）。该网络允许算法探索网络拓扑，找到一个适合的良好网络。除了最后的全连接层，所有层后面跟了batchnorm和ReLU，最终输入到softmax进行分类。
 
### 训练策略
 - RMSprop
 - less regularization and data augmentation(小网络难以过拟合)
 
## MobileNetV2: Inverted Residuals and Linear Bottlenecks

### 关键问题
 - 解决了什么问题？
    - MobileNetV1网络是一条路的单通道结构，没有feature map的复用。ResNet和DenseNet等网络的提出，也验证了feature map复用对提升网络性能的有效性，MobileNetV2便应运而生。
 - 使用了什么方法？
    - 具有线性瓶颈的反向残差结构(theinverted residual with linear bottleneck.)
 - 效果如何？
    - mIOU 75.32% Params 2.11M MAdds 2.75B
    
### Linear Bottlenecks
![image](images/Mv2_1.png)  

 - 流形相关
    - 高维空间有冗余，低维空间没冗余。也就是说，流形可以作为一种数据降维的方式。
    - 能够刻画数据的本质。也就是说，既然学习到了”将数据从高维空间降维到低维空间，还能不损失信息”的映射，那这个映射能够输入原始数据，输出数据更本质的特征(就像压缩一样，用更少的数据尽可能地表示原始数据)。
 - 假设某层的输出的feature map大小为HxWxD，经过激活层后称之为manifold of interest，可以理解为感兴趣流形或有用的信息，大小仍为HxWxD，经验证明manifold of interest完全可以压缩到低维子空间，在V1版本中便可以通过width multiplier parameter来降低激活空间的维数使得manifold of interest充满整个空间。
 - 在使用ReLU函数进行激活时，负数直接变为0，这样就会导致失去较多有用的信息  
 - 总结
    - 如果manifold of interest经过ReLU后均为非零，意味着只经过了一个线性变换
    - 除非input manifold位于输入空间的低维子空间，经过ReLU后才能保持完整的信息
    - 因此使用linear bottleneck来解决由于非线性激活函数造成的信息损失问题。linear bottleneck本质上是不带ReLU的1x1的卷积层。
    
### Inverted Residuals
![image](images/Mv2_2.png)  

<img width="400" height="400" src="images/Mv2_3.png"/>

在Inverted residual 结构中，bottleneck放在了首尾，中间则通过expand来扩展了channel。
具体的Inverted residual的结构参见下图，input首先经过expand layer，channel扩展为原来的6倍，然后再经过depthwise convolution layer和linear layer，恢复为原来的大小。在depthwise convolution layer首先将channel扩展，增加了冗余，以便后边的Depthwise Separable Convolutions能够选择到包含manifold of interest的channel。

ReLU6在低精度计算时具有鲁棒性。

### 网络结构
<img width="400" height="400" src="images/Mv2_4.png"/>

### 语义分割实验
 - 三种设计变体
    - 不同的特征提取
    - 简化DeepLabv3
    - 不同的推理策略
 - 结论
    - 推理策略（包括多尺度输入和添加左右翻转图像）显着增加了MAdds，因此不适合在设备上应用。
    - outputstride = 16比outputstride = 8效率更高
    - MobileNetV1已经是功能强大的功能提取器，与ResNet-101 [8]相比，只需要MAdds减少大约4.9-5.7倍（例如，mIOU：78.56vs82.70，而MAdds：941.9Bvs4870.6B
    - DeepLabv3 head在计算上很昂贵，删除ASPP模块会显着降低MAdds，而性能只会略有下降
    
### 如何获得轻量级网络
 - 改变内部卷积块的结构
 - 超参数优化
 - 网络剪枝
 - 连接性学习
 - 遗传算法
 - 强化学习
    
    
