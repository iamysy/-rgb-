
# 总目的：将三通道（RGB）dcm文件转为单通道dcm文件
## 注意！
1.不可以将dcm文件内的tag信息修改或是打乱顺序
2.最好是可以在内存中解决RGB图->灰度图
3.用simpleITK
4.不要使用手动输入公式来对数据进行处理
5.读取dicom的tag组成新的dicom

### 具体实现

 - 关于问题2
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
- 关于问题1，由于问题2的不可实现，所以仍然需要重新建立一个dcm文件来存储numpy数组与tag信息并构成新的dcm文件

###
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQxNzM0ODk1LDIwNzI1MDM0OTcsLTY3NT
Q1Nzk4OCwtMTU0ODM4NzI2LDIwNDAyOTc2MjJdfQ==
-->