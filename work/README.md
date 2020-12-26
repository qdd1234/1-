[English](README_en.md) | 简体中文

# Paddle-YOLOv4

## 概述
Paddle-YOLOv4,参考自https://github.com/miemie2013/Keras-YOLOv4
和https://github.com/Tianxiaomo/pytorch-YOLOv4

## 推荐
本项目已经开源到AIStudio中，可直接跑：
https://aistudio.baidu.com/aistudio/projectdetail/570310

## 咩酱刷屏时刻

Keras版YOLOv3: https://github.com/miemie2013/Keras-DIOU-YOLOv3

Pytorch版YOLOv3：https://github.com/miemie2013/Pytorch-DIOU-YOLOv3

PaddlePaddle版YOLOv3：https://github.com/miemie2013/Paddle-DIOU-YOLOv3

PaddlePaddle完美复刻版版yolact: https://github.com/miemie2013/PaddlePaddle_yolact

yolov3魔改成yolact: https://github.com/miemie2013/yolact

Keras版YOLOv4: https://github.com/miemie2013/Keras-YOLOv4

Pytorch版YOLOv4: 制作中

Paddle版YOLOv4：https://github.com/miemie2013/Paddle-YOLOv4

Keras版SOLO: https://github.com/miemie2013/Keras-SOLO

Paddle版SOLO: https://github.com/miemie2013/Paddle-SOLO

## 更新日记

2020/06/25:支持yolact中的fastnms。运行demo_fast.py即可体验。经过试验发现并没有官方的yolo_box()、multiclass_nms()快。可能需要用C++ op重写。

2020/07/16:加入YOLOv3增强版。见https://github.com/PaddlePaddle/PaddleDetection/blob/release/0.3/docs/featured_model/YOLOv3_ENHANCEMENT.md
。项目根目录下
```
wget https://paddlemodels.bj.bcebos.com/object_detection/yolov3_r50vd_dcn_obj365_dropblock_iouloss.tar
```
下载模型。（PS:训练速度是比不上PaddleDetection的，仅研究用）

2020/08/19:经过仔细核对yolov4原始配置文件，发现网络部分写错了，现已修正。运行1_pytorch2paddle.py后再运行eval.py，获得mAP=0.478。

## 需要补充

加入YOLOv4中的数据增强和其余的tricks；更多调优。

## 环境搭建

AIStudio已经为我们搭建好大部分依赖。

## 训练
下载我从Tianxiaomo的仓库保存下来的pytorch模型yolov4.pt
链接：https://pan.baidu.com/s/152poRrQW9Na_C8rkhNEh3g
提取码：09ou

(在本地windows中操作)
将它放在项目根目录下。然后运行1_pytorch2paddle.py得到一个yolov4文件夹，它也位于根目录下。

(在AIStudio中操作)
在AIStudio中创建一个自己的项目，克隆这个仓库的代码到项目里。要求AIStudio的~/work/下直接有本仓库的annotation/、data/文件夹，
即AIStudio的~/work/就是项目的根目录。
把windows中的yolov4文件夹打包成zip，通过AIStudio的“创建数据集”将zip包上传。
创建的项目使用这个数据集和COCO2017数据集，就可以完成预训练模型上传了。
(为了方便大家使用，我已经上传了预训练模型，本仓库自带的数据集“yolov4_pretrained”就是预训练模型了，在~/data/data40855/目录下)
进入AIStudio，把上传的预训练模型解压：
```
cd ~/w*
cp ../data/data40855/yolov4.zip ./yolov4.zip
unzip yolov4.zip
```
此外，你还要安装pycocotools依赖、解压COCO2017数据集：
```
cd ~
pip install pycocotools
cd data
cd data7122
unzip ann*.zip
unzip val*.zip
unzip tes*.zip
unzip image_info*.zip
unzip train*.zip
cd ~/w*
```

运行train.py进行训练:
```
rm -f train.txt
nohup python train.py>> train.txt 2>&1 &
```
通过修改config.py代码来进行更换数据集、更改超参数以及训练参数。
训练时默认每5000步计算一次验证集的mAP。或者运行eval.py评估指定模型的mAP。该mAP是val集的结果。

训练时如果发现mAP很稳定了，就停掉，修改学习率为原来的十分之一，接着继续训练，mAP还会再上升。暂时是这样手动操作。

## 训练自定义数据集
自带的voc2012数据集是一个很好的例子。

将自己数据集的txt注解文件放到annotation目录下，txt注解文件的格式如下：
```
xxx.jpg 18.19,6.32,424.13,421.83,20 323.86,2.65,640.0,421.94,20
xxx.jpg 48,240,195,371,11 8,12,352,498,14
# 图片名 物体1左上角x坐标,物体1左上角y坐标,物体1右下角x坐标,物体1右下角y坐标,物体1类别id 物体2左上角x坐标,物体2左上角y坐标,物体2右下角x坐标,物体2右下角y坐标,物体2类别id ...
```
运行1_txt2json.py会在annotation_json目录下生成两个coco注解风格的json注解文件，这是train.py支持的注解文件格式。
在config.py里修改train_path、val_path、classes_path、train_pre_path、val_pre_path这5个变量（自带的voc2012数据集直接解除注释就ok了）就可以开始训练自己的数据集了。
如果需要跑demo.py、eval.py，与数据集有关的变量也需要修改一下，应该很容易看懂。

## 评估
训练时默认每5000步计算一次验证集的mAP。或者运行eval.py评估指定模型的mAP。该mAP是val集的结果。

## test-dev
运行test_dev.py。
运行完之后，进入results目录，把bbox_detections.json压缩成bbox_detections.zip，提交到
https://competitions.codalab.org/competitions/20794#participate
获得bbox mAP.

上述yolov4在test集的mAP是（input_shape = (608, 608)，分数阈值=0.001，nms阈值=0.45的情况下）
```
Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.411
Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.640
Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.444
Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.235
Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.448
Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.516
Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.322
Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.506
Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.533
Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.340
Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.579
Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.668
```

该mAP是test集的结果，也就是大部分检测算法论文的标准指标。有点谜，根据我之前的经验test集的mAP和val集的mAP应该是差不多的。原因已经找到，由于原版YOLO v4使用coco trainval2014进行训练，训练样本中包含部分评估样本，若使用val集会导致精度虚高。

## 预测
运行demo.py。

## 导出
```
python export_model.py
```
关于导出的参数请看export_model.py中的注释。导出后的模型默认存放在inference_model目录下，带有一个配置文件infer_cfg.yml。

用导出后的模型预测图片：
```
python deploy_infer.py --model_dir inference_model --image_dir images/test/
```

用导出后的模型预测视频：
```
python deploy_infer.py --model_dir inference_model --video_file D://PycharmProjects/moviepy/dddd.mp4
```

用导出后的模型播放视频：（按esc键停止播放）
```
python deploy_infer.py --model_dir inference_model --play_video D://PycharmProjects/moviepy/dddd.mp4
```



## 传送门
cv算法交流q群：645796480
但是关于仓库的疑问尽量在Issues上提，避免重复解答。

本人微信公众号：miemie_2013

技术博客：https://blog.csdn.net/qq_27311165

## 广告位招租
有偿接私活，可联系微信wer186259，金主快点来吧！
