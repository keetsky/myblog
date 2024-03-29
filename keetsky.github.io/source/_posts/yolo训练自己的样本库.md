---
title: yolo训练自己的样本库
date: 2018-02-10 20:04:19 
comments: true
english_title: yolo train   #对于中文标题需要个英文字体代替中文显示在URL上
categories:  #分类 
- DeepLearning 
tags:	     #两个标签
- yolo
---


### YOLO 训练笔记
------------------------------------------------------------
#### labelImg 安装
[参考网址下载]( http://tzutalin.github.io/labelImg/)
  `labelImg`图片标注工具用于生成xml,用于yolo训练label数据生成可执行文件存放路径不要有中文。使用前请将data目录下的txt文件修改成自己所需的类别，请用notepad++等打开。

1. **ubuntu**
    [参考github安装教程](https://github.com/tzutalin/labelImg)

2. **windows**

在anaconda4.2.0 中安装pyqt4
```
conda install pyqt=4
```
或直接使用.exe文件(推荐)
	    [参考教程](http://blog.csdn.net/samylee/article/details/77966660?locationNum=10&fps=1)
	
对于python3.5
	    [参考教程](http://blog.csdn.net/u010807846/article/details/73480628)
```
pip install PyQt5 
pip install pyqt5-tools
```
运行  
```
pyrcc4 -o resources.py resources.qrc
python labelImg.py
```
#### 编译测试darknet工程
具体不在描述
#### 准备数据库
(1) 图片文件库
(2) xml文件库
(3) train.txt 文件,内容为所有图片小数点前名,如10004
>xml的生成可以使用labelImage 软件标注生成，xml文件名要求与对应图片文件名相同
xml信息主要有图片的长宽、类别名、类别的矩形框的左上角和右下角坐标
train.txt自己写个程序生成,在这里我的为imgprocess.py
##### 处理数据库
为了偷懒，准备使用voc结构训练，需要将准备的数据库做成voc数据库结构
```
--voc
   --VOCdevkit
        --VOCsaobin
           --Annotations
	       --10004.xml
               --10005.xml
                   ... 
           --ImageSets
               --Main 
                   --train.txt
	           --test.txt
                   --val.txt
            --JPEGImages
                   --10004.jpg
                   --10005.jpg
    --voc_label.py
```

>将图片放入JPEGImages中
    xml放入Annotations文件夹中
    train.txt中将20% 20%数据剪切到test.txt val.txt,再将train.txt test.txt val.txt放入Main文件夹中
```
voc_label.py中修改:
	sets=[('saobin', 'train'), ('saobin', 'test'),('saobin', 'val')]
    classes = ["nan","nv"]   #类别名字 
```


将voc文件夹放入data/　下  　　
运行voc_label.py:
```
$cd data/voc
$python voc_label.py
```

 查看运行后生成文件:
         voc/ 目录下会新生成  `saobin_train.txt  saobin_test.txt	saobin_val.txt`
         Vocsaobin/ 目录下新生成 `labels`
```
--labels
   --10004.txt
   --10005.txt
   ...
```
因为我们并没有用到val验证,所以将val数据集加入训练集:
```
$cd voc
$cat saobin_train.txt saobin_val.txt >train.txt
```

#### 准备训练
  修改cfg/voc.data:
     在这里我们只训练2类,`<path-to-voc>`数集的绝对路径代替为/home/keetsky/darknet/data/voc
     `names`为类别名,`backup`为训练权重数据保存路径 
```
classes= 2      
train  = <path-to-voc>/train.txt
valid  = <path-to-voc>saobin_test.txt
names = data/voc.names
backup = backup
```
修改data/voc.names:在这里训练2类
```
nan
nv
```


修改cfg/yolo-voc.cfg:
修改batch使计算机能够运行的数(参数可重大到小运行程序试到程序能运行),max_batches训练轮数```
```
[net]
batch=25 
max_batches = 45000
...
[region]
classes=2   #改为两类
```


修改[region]前一[convolutional],因为在这里只识别两类，将filters改为35
`（filter的公式filters=(classes+ coords+ confidence)* (NUM)    (2+4+1)*5=35`
  解释下最后层filers:yolo2采用Anchor Boxes,最后层13x13xfilter
  >  classes:类别数
      coords:坐标
      confidence:概率（置信度）
      NUM:anchor boxes数,这里采用5个即预测建议框,也可修改其他 
　　　　一个boxes：[坐标＋框confidence＋每个类别的概率]=[x,y,w,h,0.75,0.45,0.78]

#### [下载前训练完成模型](https://pjreddie.com/darknet/imagenet/#extraction)
下载
```
$wget https://pjreddie.com/media/files/darknet19_448.conv.23

或自己训练（https://pjreddie.com/darknet/imagenet/#darknet19_448）
./darknet partial cfg/darknet19_448.cfg darknet19_448.weights darknet19_448.conv.23 23
```


#### 训练    
开始训练

```
$./darknet detector train cfg/voc.data cfg/yolo-voc.cfg darknet19_448.conv.23

```


如果需要限定gpu数量进行训练
```
 $./darknet detector train cfg/voc.data cfg/yolo-voc.cfg darknet19_448.conv.23 -gpus 0,1,2,3
```
  从backup恢复中断的训练,比如从第700轮恢复训练
```
 $./darknet detector train cfg/voc.data cfg/yolo-voc.cfg backup/yolo-voc_700.weights 
```

#### 训练后测试

```
$./darknet detector test cfg/voc.data cfg/yolo-voc.cfg backup/yolo-voc_final.weights testpicture/001.jpg      #测试图片
$./darknet detector demo cfg/voc.data cfg/yolo-voc.cfg backup/yolo-voc_final.weights  test.avi  #测试视频
```

  也可加载中间的模型参数如 yolo-voc_1000.weights

#### 定制模型
  自己可设计网络结构进行训练,需要修改cfg/yolo-voc.cfg成自己的结构
