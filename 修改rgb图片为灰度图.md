
# 总目的：将三通道（RGB）dcm文件转为单通道dcm文件
### 注意！
1.不可以将dcm文件内的tag信息修改或是打乱顺序
2.最好是可以在内存中解决RGB图->灰度图
3.用simpleITK
4.不要使用手动输入公式来对数据进行处理
5.读取dicom的tag组成新的dicom

 - 关于注意2
无法实现在内存中直接修改图形类型，例如：
```
import SimpleITK as sitk
import numpy as np

ds = sitk.ReadImage(input_dcm_path)
arr = sitk.GetArrayFromImage(ds)
# ds读取文件
# arr将ds内的image以numpy数组形式表示
```
此处无法直接修改arr而改变ds，因为导入的dcm的文件格式是按照rgb图像的格式构建的
- 关于注意1，由于注意2的不可实现，所以仍然需要重新建立一个dcm文件来存储numpy数组与tag信息并构成新的dcm文件

## 思路
1.先读取dcm文件
2.读取表示图像的像素矩阵，转化为numpy数组
3.将rgb图像的numpy数组转化为灰度图像的numpy数组
4.读取dcm中的tag信息，作为key-value内容
5.创建一个新的dcm类型的文件，将表示灰度图像的numpy数组与tag信息进行组合作为dcm文件输出


## 具体实现
-思路1：先读取dcm文件
使用simpleITK读取dcm文件
```
ds = sitk.ReadImage(input_dcm_path)
```
- 思路2：读取表示图像的像素矩阵，转化为numpy数组
使用getarrayfromimage仅获取dcm中包含的像素数据
```
arr = sitk.GetArrayFromImage(ds)
```
- 思路3：将rgb图像的numpy数组转化为灰度图像的numpy数组
	- 根据scikit-image的rgb2gray库进行灰度图的转化
	- 使用opencv进行灰度图的转化
```
# 需要遍历每个dcm的每帧
for i in range(input_arr.shape[0]):
	gray_image[i] = cv2.cvtColor(input_arr[i], cv2.COLOR_RGB2GRAY)
```
	- 根据scikit-image的公式自己编写公式（dot）进行灰度图的转化
```
gray_arr = np.dot(arr[...,:3], [0.2989, 0.5870, 0.1140]).astype(np.uint8)
```
- 思路4：读取dcm中的tag信息，作为key-value内容
使用字典存储tag信息，放置后续dicom中丢失信息，如果丢失可以从tag中提取然后塞入信息中
```
# 创建tags字典存储tag信息
tags = {}
for key in input_dcm.GetMetaDataKeys():
	tags[key] = input_dcm.GetMetaData(key).encode('utf-8', 'ignore').decode('utf-8')
```
- 思路5：











<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY2NDExMTMzLDE3OTA4Mzg5NDcsLTE4MD
A4NzEyMzUsOTIwMzEwMjY2LC00MTg2NzI1NDEsLTQ2NDg5NDI3
OSwxNDE3MzQ4OTUsMjA3MjUwMzQ5NywtNjc1NDU3OTg4LC0xNT
Q4Mzg3MjYsMjA0MDI5NzYyMl19
-->