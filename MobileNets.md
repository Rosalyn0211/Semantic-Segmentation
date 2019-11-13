# MobileNets: Efficient Convolutional Neural Networks for Mobile Vision Applications

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

### 全局超参数

### 网络结构

### 训练策略
