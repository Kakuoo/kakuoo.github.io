---
layout: post
title: Pytorch 载入图像与ToTensor操作解读
subtitle: false
gh-repo: Kakuoo/kakuoo.github.io
gh-badge: [star, fork, follow]
tags: [PyTorch]
comments: true
---

PyTorch在做一般的深度学习图像处理任务时，先使用dataset类和dataloader类读入图片，在读入的时候需要做transform变换，其中transform一般都需要ToTensor()操作，将dataset类中__getitem__()方法内读入的PIL或CV的图像数据转换为torch.FloatTensor。详细过程如下

## PIL与CV数据格式

### 1.PIL（RGB）

PIL(Python Imaging Library)是Python中最基础的图像处理库

``` python
from PIL import Image
import numpy as np
image = Image.open('test.jpg') # 图片是400x300 宽x高
print type(image) # out: PIL.JpegImagePlugin.JpegImageFile
print image.size  # out: (400,300) （h,w,c)
print image.mode # out: 'RGB'
print image.getpixel((0,0)) # out: (143, 198, 201)
# resize w*h
image = image.resize((200,100), Image.NEAREST)
print image.size # out: (200,100)

"""
代码解释：
注意image是 class:`~PIL.Image.Image` object，有很多属性
size是(w,h),通道是RGB，，他也有很多方法，比如获取getpixel((x,y))某个位置的像素，得到三个通道的值，x最大可取w-1，y最大可取h-1

注意这几种插值方法，默认NEAREST最近邻（分割常用），分类常用BILINEAR双线性，BICUBIC立方
:returns: An :py:class:`~PIL.Image.Image` object
"""

image = np.array(image,dtype=np.float32) # image = np.array(image)默认是uint8
print image.shape # out: (100, 200, 3)

"""
此时w和h换了，变成(h,w,c)了，是因为ndarray中是 行-->row h，列-->col w，维度dim
"""
```

### 2.OpenCV（python版）（BGR）

```python
import cv2
import numpy as np
image = cv2.imread('test.jpg')
print type(image) # out: numpy.ndarray
print image.dtype # out: dtype('uint8')
print image.shape # out: (300, 400, 3) (h,w,c) 和skimage类似
print image # BGR

# w*h
image = cv2.resize(image,(100,200),interpolation=cv2.INTER_LINEAR)
print image.dtype # out: dtype('uint8')
print image.shape # out: (200, 100, 3)

'''
注意注意注意 和skimage不同 ！！！
resize(src, dsize[, dst[, fx[, fy[, interpolation]]]])
关键字参数为dst,fx,fy,interpolation
dst为缩放后的图像
dsize为(w,h),但是image是(h,w,c)
fx,fy为图像x,y方向的缩放比例，
interplolation为缩放时的插值方式，有三种插值方式：
cv2.INTER_AREA:使用象素关系重采样。当图像缩小时候，该方法可以避免波纹出现。当图像放大时，类似于 CV_INTER_NN方法　　　　
cv2.INTER_CUBIC: 立方插值
cv2.INTER_LINEAR: 双线形插值　
cv2.INTER_NN: 最近邻插值
[详细可查看该博客](http://www.tuicool.com/articles/rq6fIn)
'''

'''
cv2.imread(filename, flags=None):
flag:
cv2.IMREAD_COLOR 1: Loads a color image. Any transparency of image will be neglected. It is the default flag. 正常的3通道图
cv2.IMREAD_GRAYSCALE 0: Loads image in grayscale mode 单通道灰度图
cv2.IMREAD_UNCHANGED -1: Loads image as such including alpha channel 4通道图
注意: 默认应该是cv2.IMREAD_COLOR，如果你cv2.imread('gray.png')，虽然图片是灰度图，但是读入后会是3个通道值一样的3通道图片
'''
```

另外，PIL图像(h,w,c)在转换为numpy.ndarray后，格式为(h,w,c)，像素顺序为RGB；
OpenCV在cv2.imread()后数据类型为numpy.ndarray，格式为(h,w,c)，像素顺序为BGR。



## torchvision.transforms.ToTensor()

参照torchvision.transforms.transform.py，如下：

```python
class ToTensor(object):
    """Convert a ``PIL Image`` or ``numpy.ndarray`` to tensor.

    Converts a PIL Image or numpy.ndarray (H x W x C) in the range
    [0, 255] to a torch.FloatTensor of shape (C x H x W) in the range [0.0, 1.0]
    if the PIL Image belongs to one of the modes (L, LA, P, I, F, RGB, YCbCr, RGBA, CMYK, 1)
    or if the numpy.ndarray has dtype = np.uint8

    In the other cases, tensors are returned without scaling.
    """

    def __call__(self, pic):
        """
        Args:
            pic (PIL Image or numpy.ndarray): Image to be converted to tensor.

        Returns:
            Tensor: Converted image.
        """
        return F.to_tensor(pic)

    def __repr__(self):
        return self.__class__.__name__ + '()'
```

参照torchvision.transforms.functional.py，如下：

```python
def to_tensor(pic):
    """Convert a ``PIL Image`` or ``numpy.ndarray`` to tensor.

    See ``ToTensor`` for more details.

    Args:
        pic (PIL Image or numpy.ndarray): Image to be converted to tensor.

    Returns:
        Tensor: Converted image.
    """
    if not(_is_pil_image(pic) or _is_numpy_image(pic)):
        raise TypeError('pic should be PIL Image or ndarray. Got {}'.format(type(pic)))

    if isinstance(pic, np.ndarray):
        # handle numpy array
        if pic.ndim == 2:
            pic = pic[:, :, None]

        img = torch.from_numpy(pic.transpose((2, 0, 1)))
        # backward compatibility
        if isinstance(img, torch.ByteTensor):
            return img.float().div(255)
        else:
            return img

    if accimage is not None and isinstance(pic, accimage.Image):
        nppic = np.zeros([pic.channels, pic.height, pic.width], dtype=np.float32)
        pic.copyto(nppic)
        return torch.from_numpy(nppic)

    # handle PIL Image
    if pic.mode == 'I':
        img = torch.from_numpy(np.array(pic, np.int32, copy=False))
    elif pic.mode == 'I;16':
        img = torch.from_numpy(np.array(pic, np.int16, copy=False))
    elif pic.mode == 'F':
        img = torch.from_numpy(np.array(pic, np.float32, copy=False))
    elif pic.mode == '1':
        img = 255 * torch.from_numpy(np.array(pic, np.uint8, copy=False))
    else:
        img = torch.ByteTensor(torch.ByteStorage.from_buffer(pic.tobytes()))
    # PIL image mode: L, LA, P, I, F, RGB, YCbCr, RGBA, CMYK
    if pic.mode == 'YCbCr':
        nchannel = 3
    elif pic.mode == 'I;16':
        nchannel = 1
    else:
        nchannel = len(pic.mode)
    img = img.view(pic.size[1], pic.size[0], nchannel)
    # put it from HWC to CHW format
    # yikes, this transpose takes 80% of the loading time/CPU
    img = img.transpose(0, 1).transpose(0, 2).contiguous()
    if isinstance(img, torch.ByteTensor):
        return img.float().div(255)
    else:
        return img
```

可以从`F.to_tensor()`函数看出，函数接受PIL Image或numpy.ndarray，将其先由`[h，w，c]`转换成`[c，h，w]`格式，数据从`int8`转换成`float32`后对每个像素除以255，范围调整至`[0, 1]`，像素顺序仍为`RGB`。



**经常检查tensor_size是个好习惯: assert tensor.size() == (BS, C, H, W)**
