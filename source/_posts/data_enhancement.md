---
title: 图像分类任务数据增广
date: 2021-12-30 13:23:28
categories: 
- 深度学习
tags:
- 深度学习
---
# 图像分类任务中数据增广的两种方式

在图像分类任务中，为了提高模型的准确度及提升其泛化能力，我们通常会对数据进行增广处理，常用的操作有裁剪、缩放、镜像，颜色空间转换等，下面介绍图像增广的两种方式，PIL.ImageEnhance和keras...ImageGenerator。

## 一、使用PIL.ImageEnhance多线程进行图像强化增广

```python
# -*- coding:utf-8 -*-
"""数据增强
   1. 翻转变换 flip
   2. 随机修剪 random crop
   3. 色彩抖动 color jittering
   4. 平移变换 shift
   5. 尺度变换 scale
   6. 对比度变换 contrast
   7. 噪声扰动 noise
   8. 旋转变换/反射变换 Rotation/reflection
"""

from PIL import Image, ImageEnhance, ImageOps, ImageFile, ImageFilter
import numpy as np
import random
import threading, os, time
import logging

logger = logging.getLogger(__name__)
ImageFile.LOAD_TRUNCATED_IMAGES = True


class DataAugmentation:
    """
    包含数据增强的八种方式
    """

    def __init__(self):
        pass

    @staticmethod
    def openImage(image):
        return Image.open(image, mode="r")

    @staticmethod
    def randomRotation(image, mode=Image.BICUBIC):
        """
         对图像进行随机任意角度(0~360度)旋转
        :param mode 邻近插值,双线性插值,双三次B样条插值(default)
        :param image PIL的图像image
        :return: 旋转转之后的图像
        """
        random_angle = np.random.randint(1, 360)
        return image.rotate(random_angle, mode)

    @staticmethod
    def randomCrop(image):
        w, h = image.size
        # assert w >= 256 and h >= 256  # 确保height 和width大于256，因为256时我们要裁剪的大小
        assert w >= 128 and h >= 128  # 确保height 和width大于128，因为128时我们要裁剪的大小
        if w == 128 and h == 128:
            return 0, 0, h, w

        i = random.randint(0, h - 128)
        j = random.randint(0, w - 128)
        imgCrop = image.crop((j, i, j + 128, i + 128))

        return imgCrop

    @staticmethod
    def randomColor(image):
        # hue = random.uniform(0.5, 1.5)  # 控制饱和度
        # img = ImageEnhance.Color(image).enhance(hue)
        #
        # bri = random.uniform(0.8, 1.5)  # 控制亮度
        # img = ImageEnhance.Brightness(img).enhance(bri)
        #
        # con = random.uniform(0.8, 1.5)  # 控制对比度
        # img = ImageEnhance.Contrast(img).enhance(con)
        #
        # shap = random.uniform(0, 2)  # 控制对比度
        # img = ImageEnhance.Sharpness(img).enhance(shap)
        #
        # return image

        """
        对图像进行颜色抖动
        :param image: PIL的图像image
        :return: 有颜色色差的图像image
        """
        random_factor = np.random.randint(0, 21) / 10.  # 随机因子
        color_image = ImageEnhance.Color(image).enhance(random_factor)  # 调整图像的饱和度
        random_factor = np.random.randint(10, 11) / 10.  # 随机因子
        brightness_image = ImageEnhance.Brightness(color_image).enhance(random_factor)  # 调整图像的亮度
        random_factor = np.random.randint(10, 21) / 10.  # 随机因1子
        contrast_image = ImageEnhance.Contrast(brightness_image).enhance(random_factor)  # 调整图像对比度
        random_factor = np.random.randint(0, 21) / 10.  # 随机因子
        return ImageEnhance.Sharpness(contrast_image).enhance(random_factor)  # 调整图像锐度

    @staticmethod
    def randomGaussian(img):
        radius = random.uniform(0.1, 3.0)
        img = img.filter(ImageFilter.GaussianBlur(radius))
        return img

    # def randomGaussian(image, mean=0.2, sigma=0.3):
    #     """
    #      对图像进行高斯噪声处理
    #     :param image:
    #     :return:
    #     """
    #
    #     def gaussianNoisy(im, mean=5, sigma=5):
    #         """
    #         对图像做高斯噪音处理
    #         :param im: 单通道图像
    #         :param mean: 偏移量
    #         :param sigma: 标准差
    #         :return:
    #         """
    #         for _i in range(len(im)):
    #             im[_i] += random.gauss(mean, sigma)
    #         return im
    #
    #     # 将图像转化成数组
    #     img = np.array(image)
    #     img.flags.writeable = True  # 将数组改为读写模式
    #     width, height = img.shape[:2]
    #     img_r = gaussianNoisy(img[:, :, 0].flatten(), mean, sigma)
    #     img_g = gaussianNoisy(img[:, :, 1].flatten(), mean, sigma)
    #     img_b = gaussianNoisy(img[:, :, 2].flatten(), mean, sigma)
    #     img[:, :, 0] = img_r.reshape([width, height])
    #     img[:, :, 1] = img_g.reshape([width, height])
    #     img[:, :, 2] = img_b.reshape([width, height])
    #     return Image.fromarray(np.uint8(img))

    @staticmethod
    def saveImage(image, path):
        image.save(path)


def makeDir(path):
    try:
        if not os.path.exists(path):
            if not os.path.isfile(path):
                # os.mkdir(path)
                os.makedirs(path)
            return 0
        else:
            return 1
    except Exception as e:
        print(str(e))
        return -2


def imageOps(func_name, image, des_path, file_name, times=5):
    funcMap = {
        "randomCrop": DataAugmentation.randomCrop,
        "randomRotation": DataAugmentation.randomRotation,

        "randomColor": DataAugmentation.randomColor,
        "randomGaussian": DataAugmentation.randomGaussian
    }
    if funcMap.get(func_name) is None:
        logger.error("%s is not exist", func_name)
        return -1

    for _i in range(0, times, 1):
        new_image = funcMap[func_name](image)
        DataAugmentation.saveImage(new_image, os.path.join(des_path, func_name + str(_i) + file_name))


opsList = {
    "randomCrop",
    "randomRotation",
    "randomColor",
    "randomGaussian"
}


def threadOPS(path, new_path):
    """
    多线程处理事务
    :param src_path: 资源文件
    :param des_path: 目的地文件
    :return:
    """
    if not os.path.exists(new_path):
        os.mkdir(new_path)
    if os.path.isdir(path):
        img_names = os.listdir(path)
    else:
        img_names = [path]
    for img_name in img_names:
        # print(img_name)
        tmp_img_name = os.path.join(path, img_name)
        print(tmp_img_name)
        if os.path.isdir(tmp_img_name):
            if makeDir(os.path.join(new_path, img_name)) != -1:
                threadOPS(tmp_img_name, os.path.join(new_path, img_name))
            else:
                print('create new dir failure')
                return -1
                # os.removedirs(tmp_img_name)
        elif tmp_img_name.split('.')[1] != "DS_Store":
            # 读取文件并进行操作
            image = DataAugmentation.openImage(tmp_img_name)
            threadImage = [0] * 5
            _index = 0
            for ops_name in opsList:
                threadImage[_index] = threading.Thread(target=imageOps,
                                                       args=(ops_name, image, new_path, img_name,))
                threadImage[_index].start()
                _index += 1
                time.sleep(0.2)


if __name__ == '__main__':
    threadOPS("test",
              "result")


```

## 二、使用keras.preprocessing.image.ImageGenerator

讲完上面一个相对复杂的本地图像增广方法之后，下面介绍一下使用keras官方自带的功能进行数据增广操作。

##### 参数设置

每个参数的意思如下， 注意调整的范围及检查生成的图像是否失真，如果生成一堆无效的干扰的数据，将会使模型的精确度更低，适得其反。

```python
from keras.preprocessing.image import ImageDataGenerator

data_gen = ImageDataGenerator(
    # 数据集去中心化
    featurewise_center=False,
    # 使输入数据的每个样本均值为0
    samplewise_center=False,
    # 每个样本除以其自身的标注差，和上一个参数用于对图像进行标准化处理
    featurewise_std_normalization=False,
    # 每个样本除以其自身的标注差，只针对自身的图片
    samplewise_std_normalization=False,
    # 针对图片进行PCA降维操作，减少图片的冗余信息，保留最重要的特征
    zca_whitening=False,
    # zca的单位
    zca_epsilon=1e-06,
    # 旋转图片，指定旋转的角度, 单位为1
    rotation_range=0,
    # 水平位置方向的平移
    width_shift_range=0.1,
    # 垂直方向位置的平移
    height_shift_range=0.1,
    # 错切变换，效果为x or y 坐标保持不变，对应的y or x坐标按比例发生平移, 平移的大小和该点到x or y 轴的垂直距离成正比
    shear_range=0.,
    # 水平或者垂直方向的缩放
    zoom_range=0.,
    # 通过颜色通道的数值偏移，改变图片整体的颜色，”整体“
    channel_shift_range=0.,
    # 填充模式，constant, nearest, reflect, wrap, 对平移，缩放，错切后缺失的地方进行填充
    fill_mode='nearest',
    # 填充的参数
    cval=0.,
    # 水平翻转
    horizontal_flip=True,
    # 垂直翻转
    vertical_flip=False,
    # 缩放因子
    rescale=None,
    # set function that will be applied on each input
    preprocessing_function=None,
    # fraction of images reserved for validation (strictly between 0 and 1)
    data_format=None,
    # image data format, either "channels_first" or "channels_last"
    validation_split=0.1)
```

##### 检查生成的图像

在正式训练之前，keras可以设置图像增强后保存的文件夹(通过人工检查，防止图片失真)。

```python
# 保存生存图像的目录
# 保存生存图像的目录
save_dir = 'test'
if not os.path.exists(save_dir):
    os.mkdir(save_dir)
train_gen = data_gen.flow(x_train,  # 训练的图像
                          y_train,  # 训练的标签
                          batch_size=32,  # 批量大小
                          save_to_dir=save_dir  # 保存的路径
                          )
```

#### 开始训练

正式开始训练前记得把save_to_dir去掉，否则影响程序运行的效率，根据模型和数据集的情况，调整各项参数，粗略解释如下：

```python
model.fit_generator(train_gen,
                    # 整数，两个epochs之间generator生成的批次样本（生成的样本量), 一般为训练的样本量 / batch_size
                    steps_per_epoch=x_train.shape[0] // 32,
                    # 整数，训练迭代轮数
                    epochs=200,
                    # 日志显示模式，0：安静模式（无显示直到完成），1：进度条（显示每一次），2：显示每一轮
                    verbose=2,
                    # 验证集
                    validation_data=(x_test, y_test),
                    # 在多少条进程上执行生成器，默认为1， 设置为0则在主线程上执行
                    workers=1,
                    # 是否在每轮迭代前打乱batch的顺序
                    shuffle=True)
```

注意workers不要调太大，视硬件性能而定，否则可能报以下错误，虽然不影响程序继续运行，但是训练的效率：

```
2020-11-28 18:57:04.420183: W tensorflow/core/kernels/data/generator_dataset_op.cc:103] Error occurred when finalizing GeneratorDataset iterator: Cancelled: Operation was cancelled
```
