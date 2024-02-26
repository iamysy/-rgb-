
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
6.单个案例成功后，对文件加进行遍历处理


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
- 思路5：创建一个新的dcm类型的文件，将表示灰度图像的numpy数组与tag信息进行组合作为dcm文件输出
```
# 先将numpy数组转为像素信息
gray_img = sitk.GetImageFromArray(gray_arr)
# 再将tags内的信息存入至dcm文件内
for tag,value in tags.items():
	gray_image.SetMetaData(str(tag), str(value))
# 存入tag会出现，漏传（group lenth），错传(series uid)，无法识别(patien's id)需要手动添加
# 手动添加的方式与上述方法相同，不过是用 组号|元素 来替代tag，用指定的str替代value
patients_name = "John"
gray_image.SetMetaData("0010|0010", patients_name)
# 再将img保存
sitk.WriteImage(gray_image, output_dcm_path)
```
- 注意步骤5中，部分tag信息含有中文，在转移中，会出现编码错误导致不能正确转移name的状况
	- 解决该问题需要明确中文内容编码错误在哪儿，需要怎么改
		- dcm文件的编码形式由’0008|0005表示‘
	- 尝试的解决方法：
		- 1.使用编码形式转化
			- 可以使用simpleITK读出的tag信息数据进行转换
```
# 使用strip()移除str以外的字符
# 使用surrogateescape，允许在字节到字符串的转换中保留无法直接转换的数据，尤其是在本case中可以使用
patient_name = input_dcm.GetMetaData('0010|0010').strip().encode("utf-8", "surrogateescape").decode('gbk', 'replace')
```
		可以使用pydicom读取出tag信息，再将tag信息进行编码转化
```
dicom_file = pydicom.dcmread(input_dcm_path)
patient_name = dicom_file.PatientName
patient_name = patient_name.encode('utf-8').decode('utf-8')
```
- 思路6：单个案例成功后，对文件加进行遍历处理
	- 首先确定文件夹的格式，各个目录的内容
	- 根据目录内容进行遍历
	- 将转化后的文件以同样的名字命名，并保存在另一个文件夹下
```
- data
	- folder1
		- file1
		- file2
		- ...
	- folder2
	- ...
```


```
# 举例说明
file = '/data'
# 新建变量subfolder来保存file下的所有文件夹 
subfolder = [os.path.basename(f.path) for f in os.scandir(file) if f.is_dir()]
```






<!--stackedit_data:
eyJoaXN0b3J5IjpbMTM1MTkyOTI0MCwtMTI0MDM4NDA2MiwtMT
czNjMxMzIxLDE3MDUyMTA0NDIsMTUxOTc4NzMzNSwxMjczNjcw
ODEwLC0xODg1NTQ2NTU3LC0xMTE1NzY2Njg4LDE3OTA4Mzg5ND
csLTE4MDA4NzEyMzUsOTIwMzEwMjY2LC00MTg2NzI1NDEsLTQ2
NDg5NDI3OSwxNDE3MzQ4OTUsMjA3MjUwMzQ5NywtNjc1NDU3OT
g4LC0xNTQ4Mzg3MjYsMjA0MDI5NzYyMl19
-->