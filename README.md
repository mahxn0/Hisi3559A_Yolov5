# HISI3559A  YOLOV5训练部署全流程

代码地址

## yolov5网络简介  

https://zhuanlan.zhihu.com/p/172121380

## hisi3559a开发板简介

CPU:

双核 ARM Cortex A73@1.8GHz，32KB I-Cache，64KB D-Cache /512KB L2 cache

双核 ARM Cortex A53@1.2GHz，32KB I-Cache，32KB D-Cache /256KB L2 cache

单核 ARM Cortex A53@1.2GHz，32KB I-Cache，32KB D-Cache /128KB L2 cache

支持 Neon 加速，集成 FPU 处理单元

GPU:

双核 ARM Mali G71@900MHz，256KB cache

支持 OpenCL 1.1/1.2/2.0

支持 OpenGL ES 3.0/3.1/3.2

智能视频分析：

提供视觉计算处理能力

四核 DSP@700MHz，32K I-Cache /32K IRAM/512KB DRAM

双核 NNIE@840MHz 神经网络加速引擎 INT8 4T算力

内置双目深度检测单元

## yolov5网络模型训练

```
1）下载yolov5源码
	a、git clone https://github.com/ultralytics/yolov5.git
	b、git reset --hard 69be8e738
	
2）安装yolov5训练环境
	a、conda create --name yolov5 python=3.7.9 -y
	b、conda activate yolov5
	c、修改requirements.txt，删除coremltools、onnx、scikit-learn前的”#“，增加一行“onnx-simplifier”
	d、pip install -r requirements.txt
	
3）修改训练参数和模型结构
	a、修改data/coco.yaml文件中类别数目、类别名、train/test/val的路径，按照自己的项目规划修改
	b、修改models/yolov5s.yaml文件中类别数目
	c、修改models/yolov5s.yaml中的网络结构，将focus层修改为卷积层，并设置stride为2
	
  backbone:
  # [from, number, module, args]
  # [[-1, 1, Focus, [64, 3]],  # 0-P1/2
  [
   [-1, 1, Conv, [64, 3, 2]],
   [-1, 1, Conv, [128, 3, 2]],  # 1-P2/4
   [-1, 3, C3, [128]],
   [-1, 1, Conv, [256, 3, 2]],  # 3-P3/8
   [-1, 9, C3, [256]],
   [-1, 1, Conv, [512, 3, 2]],  # 5-P4/16
   [-1, 9, C3, [512]],
   [-1, 1, Conv, [1024, 3, 2]],  # 7-P5/32
   [-1, 1, SPP, [1024, [5, 9, 13]]],
   [-1, 3, C3, [1024, False]],  # 9
  ]
  或者用yolov5 release6.0版本已经去掉focus层，
  	backbone:
  # [from, number, module, args]
  [[-1, 1, Conv, [64, 6, 2, 2]],  # 0-P1/2 <--- update
   [-1, 1, Conv, [128, 3, 2]],  # 1-P2/4
   [-1, 3, C3, [128]],
   [-1, 1, Conv, [256, 3, 2]],  # 3-P3/8
   [-1, 9, C3, [256]],
   [-1, 1, Conv, [512, 3, 2]],  # 5-P4/16
   [-1, 9, C3, [512]],
   [-1, 1, Conv, [1024, 3, 2]],  # 7-P5/32
   [-1, 1, SPP, [1024, [5, 9, 13]]],
   [-1, 3, C3, [1024, False]],  # 9
  ]
  
4）启动模型训练
	python train.py --data data/coco.yaml  --cfg models/yolov5s.yaml --weights ''  --batch-size 64 --img-size 416 --noautoanchor
	
5）模型导出
	python models/export.py --weights weights/last.pt
	
6）模型简化
	python -m onnxsim weights/last.onnx  weights/simple.onnx (先安装onnx-simpler)
```

## torch 转caffe模型

确保已经搭建好caffe 

```
cd yolov5_onnx2caffe

修改 convertCaffe.py 中路径  

设置onnx_path（上面转换得到的简化后onnx模型），prototxt_path（caffe的prototxt保存路径），caffemodel_path（caffe的caffemodel保存路径）

python convertCaffe.py
```

 得到转换后的caffemodel.  

### caffe模型转hisi3559 wk

本地linux转换环境搭建参考：https://blog.csdn.net/racesu/article/details/107045858

配置文件注意预处理顺序即可

![image-20220110185812410](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220110185812410.png)

### 后处理代码

https://github.com/mahxn0/Hisi3559A_Yolov5.git