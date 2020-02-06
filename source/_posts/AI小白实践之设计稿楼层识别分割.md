---
title: AI小白实践之设计稿楼层识别分割
date: 2020-02-06 18:00:00
tags: [AI]
category: intensive study
toc: true
comments: true
description: 从长设计稿上识别多个楼层进行分割，得出分割区域坐标。本文使用的深度学习工具为TensorFlow。主要通过改造官方例子，来训练自己的模型并测试效果，其中模型训练部分通过python实现，效果测试由前端页面展示。
---
> 本文不包含底层数学基础，目标是利用已有算法和模型实现需求。

## 业务背景
从长设计稿上识别多个楼层进行分割，得出分割区域坐标。     
本文使用的深度学习工具为`TensorFlow`。主要通过改造官方例子，来训练自己的模型并测试效果，其中模型训练部分通过**python**实现，效果测试由前端页面展示。    
项目地址：https://github.com/winniecjy/floorDetection   
demo地址：https://winniecjy.github.io/floorDetection/   
最终效果如下：          
![效果图](https://img11.360buyimg.com/imagetools/jfs/t1/109913/23/5556/116384/5e3b9264E1d3573b2/daa93409cdfe3a38.jpg)   
## 制作数据集
这里使用labelImg来标注检测目标，MacOS下的安装步骤如下，其他系统可参考https://github.com/tzutalin/labelImg：    
```shell
pip3 install pyqt5 lxml # Install qt and lxml by pip

git clone https://github.com/tzutalin/labelImg.git
cd labelImg
make qt5py3
python labelImg.py
python labelImg.py [IMAGE_PATH] [PRE-DEFINED CLASS FILE]
```
![labelimg](https://img10.360buyimg.com/imagetools/jfs/t1/91139/3/11302/676536/5e31741eE18cc7f5f/9f034107bfe78735.jpg)
这里在花瓣网上找了100多张会场设计图来进行处理。将图片一分为二，一部分作为训练集，一部分作为测试集，训练集和测试集比例为2:1，分别放在两个文件夹train和test中。标记完成后，每个图片都得到一个对应的xml文件描述了对应的区域标注。            
## 数据集处理   
### xml合并为csv
把所有的xml合并到csv文件，把以下代码复制到一个python文件中，需要修改两处位置：   
```python
'''
需要修改两个位置
fileDir: 为xml文件所在的目录
outputName：为csv文件的输出名称
'''
import os
import glob
import pandas as pd
import xml.etree.ElementTree as ET

fileDir = 'output/train' # xml文件夹地址
outputName = 'train.csv' # 生成csv文件地址
os.chdir(fileDir)
path = fileDir

def xml_to_csv(path):
    xml_list = []
    for xml_file in glob.glob('*.xml'):
        tree = ET.parse(xml_file)
        root = tree.getroot()
        for member in root.findall('object'):
            value = (root.find('filename').text,
                     int(root.find('size')[0].text),
                     int(root.find('size')[1].text),
                     member[0].text,
                     int(member[4][0].text),
                     int(member[4][1].text),
                     int(member[4][2].text),
                     int(member[4][3].text)
                     )
            print('value: ', value)
            xml_list.append(value)
    column_name = ['filename', 'width', 'height', 'class', 'xmin', 'ymin', 'xmax', 'ymax']
    xml_df = pd.DataFrame(xml_list, columns=column_name)
    return xml_df

def main():
    image_path = path
    xml_df = xml_to_csv(image_path)
    xml_df.to_csv(outputName, index=None)
    print('Successfully converted xml to csv.')

main()
```
![csv](https://img12.360buyimg.com/imagetools/jfs/t1/85204/40/11269/350026/5e31741aE11cb3d2a/c20da86272de1a2c.jpg)    
### csv转换为TFRcords Format
由于使用到的`TensorFlow Object Detection API`数据输入格式为`TFRcords Format`，所以还需要进行一次csv格式转换。    
```python
'''
  使用:
  # csv_input为输入的csv文件目录，output_path为输出的文件目录
  python csv2record.py --csv_input=data/train.csv --output_path=data/train.record

  需要修改两个位置（标记为修改处）：
  #1. 'images/train'为图片所在目录
  path = os.path.join(os.getcwd(), 'images/train')
  #2. 对应的标签返回一个整数，后面需要使用
  def class_text_to_int(row_label):
    if row_label == 'floors':
        return 1
    elif row_label == 'toutu':
        return 2
    else:
        None
'''
import os
import io
import pandas as pd
import tensorflow as tf

from PIL import Image
from object_detection.utils import dataset_util
from collections import namedtuple, OrderedDict

# 切换到脚本所在目录
# py_dir = '/'

# print(os.getcwd())
# os.chdir(py_dir)
# print(os.getcwd())

flags = tf.app.flags
flags.DEFINE_string('csv_input', '', 'Path to the CSV input')
flags.DEFINE_string('output_path', '', 'Path to output TFRecord')
FLAGS = flags.FLAGS

# 修改处：标签数字对应
def class_text_to_int(row_label):
    if row_label == 'floors':
        return 1
    elif row_label == 'toutu':
        return 2
    else:
        None

'''
csv按照图片名分组；
将同一图片名中多个标记区域分为一组；
'''
def split(df, group):
    data = namedtuple('data', ['filename', 'object']) # data有两个属性，filename和object
    gb = df.groupby(group) # 按照'filename'对data中的数据进行分组
    # data(filename, gb.get_group(x))存放每个图片名、该图片的相关信息
    return [data(filename, gb.get_group(x)) for filename, x in zip(gb.groups.keys(), gb.groups)]

def create_tf_example(group, path):
    with tf.gfile.GFile(os.path.join(path, '{}'.format(group.filename)), 'rb') as fid: # rb指定二进制形式读取图片
        encoded_jpg = fid.read()
    encoded_jpg_io = io.BytesIO(encoded_jpg)
    image = Image.open(encoded_jpg_io)
    width, height = image.size

    filename = group.filename.encode('utf8')
    image_format = b'jpg'
    xmins = []
    xmaxs = []
    ymins = []
    ymaxs = []
    classes_text = []
    classes = []

    for index, row in group.object.iterrows():
        xmins.append(row['xmin'] / width)
        xmaxs.append(row['xmax'] / width)
        ymins.append(row['ymin'] / height)
        ymaxs.append(row['ymax'] / height)
        classes_text.append(row['class'].encode('utf8'))
        classes.append(class_text_to_int(row['class']))

    tf_example = tf.train.Example(features=tf.train.Features(feature={
        'image/height': dataset_util.int64_feature(height),
        'image/width': dataset_util.int64_feature(width),
        'image/filename': dataset_util.bytes_feature(filename),
        'image/source_id': dataset_util.bytes_feature(filename),
        'image/encoded': dataset_util.bytes_feature(encoded_jpg),
        'image/format': dataset_util.bytes_feature(image_format),
        'image/object/bbox/xmin': dataset_util.float_list_feature(xmins),
        'image/object/bbox/xmax': dataset_util.float_list_feature(xmaxs),
        'image/object/bbox/ymin': dataset_util.float_list_feature(ymins),
        'image/object/bbox/ymax': dataset_util.float_list_feature(ymaxs),
        'image/object/class/text': dataset_util.bytes_list_feature(classes_text),
        'image/object/class/label': dataset_util.int64_list_feature(classes),
    }))
    return tf_example

def main(_):
    writer = tf.io.TFRecordWriter(FLAGS.output_path)
    path = os.path.join(os.getcwd(), 'images-test') # 修改处
    examples = pd.read_csv(FLAGS.csv_input)
    grouped = split(examples, 'filename')
    for group in grouped:
        tf_example = create_tf_example(group, path)
        writer.write(tf_example.SerializeToString())

    writer.close()
    output_path = os.path.join(os.getcwd(), FLAGS.output_path)
    print('Successfully created the TFRecords: {}'.format(output_path))

if __name__ == '__main__':
    tf.compat.v1.app.run()
```
## 模型选取 
tensorflow/models仓库提供了丰富的TensorFlow模型，目标检测API位于目录tensorflow/models/research/object_detection/models，本文选取的是`ssd_mobilenet_v1_coco`模型，更多的模型描述可以参照[model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md)，需要对其中的`*.config`文件进行修改：   
```python
# SSD with Mobilenet v1 configuration for MSCOCO Dataset.
# Users should configure the fine_tune_checkpoint field in the train config as
# well as the label_map_path and input_path fields in the train_input_reader and
# eval_input_reader. Search for "PATH_TO_BE_CONFIGURED" to find the fields that
# should be configured.
'''
修改位置已标记为：修改处
1. ssd {
  num_classes: 2
  ...
}
num_classes为标签类别数，此次训练有头图和楼层两个类型，所以为2
2. train_config: {
  batch_size: 1
  ...
}
batch_size是每次迭代的数据数，这里设为1
3. train_input_reader: {
  tf_record_input_reader {
    input_path: "output/train.record"
  }
  label_map_path: "output/floors.pbtxt"
}
input_path是训练数据的路径，label_map_path是label路径都需要改为对应的路径
4. eval_input_reader: {
  tf_record_input_reader {
    input_path: "output/test.record"
  }
  label_map_path: "output/floors.pbtxt"
  shuffle: false
  num_readers: 1
}
input_path是测试数据的路径，label_map_path是label路径都需要改为对应的路径
5.fine_tune_checkpoint: "training/model.ckpt"
  from_detection_checkpoint: true
由于是从头开始训练，不使用预训练的模型数据，所以这两行注释，如果是对预训练模型参数来进行微调，则该参数为模型位置
'''

model {
  ssd {
    num_classes: 2 # 修改处：标签类别数
    box_coder {
      faster_rcnn_box_coder {
        y_scale: 10.0
        x_scale: 10.0
        height_scale: 5.0
        width_scale: 5.0
      }
    }
    matcher {
      argmax_matcher {
        matched_threshold: 0.5
        unmatched_threshold: 0.5
        ignore_thresholds: false
        negatives_lower_than_unmatched: true
        force_match_for_each_row: true
      }
    }
    similarity_calculator {
      iou_similarity {
      }
    }
    anchor_generator {
      ssd_anchor_generator {
        num_layers: 6
        min_scale: 0.2
        max_scale: 0.95
        aspect_ratios: 1.0
        aspect_ratios: 2.0
        aspect_ratios: 0.5
        aspect_ratios: 3.0
        aspect_ratios: 0.3333
      }
    }
    image_resizer {
      fixed_shape_resizer {
        height: 300
        width: 300
      }
    }
    box_predictor {
      convolutional_box_predictor {
        min_depth: 0
        max_depth: 0
        num_layers_before_predictor: 0
        use_dropout: false
        dropout_keep_probability: 0.8
        kernel_size: 1
        box_code_size: 4
        apply_sigmoid_to_scores: false
        conv_hyperparams {
          activation: RELU_6,
          regularizer {
            l2_regularizer {
              weight: 0.00004
            }
          }
          initializer {
            truncated_normal_initializer {
              stddev: 0.03
              mean: 0.0
            }
          }
          batch_norm {
            train: true,
            scale: true,
            center: true,
            decay: 0.9997,
            epsilon: 0.001,
          }
        }
      }
    }
    feature_extractor {
      type: 'ssd_mobilenet_v1'
      min_depth: 16
      depth_multiplier: 1.0
      conv_hyperparams {
        activation: RELU_6,
        regularizer {
          l2_regularizer {
            weight: 0.00004
          }
        }
        initializer {
          truncated_normal_initializer {
            stddev: 0.03
            mean: 0.0
          }
        }
        batch_norm {
          train: true,
          scale: true,
          center: true,
          decay: 0.9997,
          epsilon: 0.001,
        }
      }
    }
    loss {
      classification_loss {
        weighted_sigmoid {
          anchorwise_output: true
        }
      }
      localization_loss {
        weighted_smooth_l1 {
          anchorwise_output: true
        }
      }
      hard_example_miner {
        num_hard_examples: 3000
        iou_threshold: 0.99
        loss_type: CLASSIFICATION
        max_negatives_per_positive: 3
        min_negatives_per_image: 0
      }
      classification_weight: 1.0
      localization_weight: 1.0
    }
    normalize_loss_by_num_matches: true
    post_processing {
      batch_non_max_suppression {
        score_threshold: 1e-8
        iou_threshold: 0.6
        max_detections_per_class: 100
        max_total_detections: 100
      }
      score_converter: SIGMOID
    }
  }
}

train_config: {
  batch_size: 1 # 修改处：每次迭代数据量
  optimizer {
    rms_prop_optimizer: {
      learning_rate: {
        exponential_decay_learning_rate {
          initial_learning_rate: 0.004
          decay_steps: 800720
          decay_factor: 0.95
        }
      }
      momentum_optimizer_value: 0.9
      decay: 0.9
      epsilon: 1.0
    }
  }
  # fine_tune_checkpoint: "training/model.ckpt" # 修改处
  # from_detection_checkpoint: true # 修改处
  # Note: The below line limits the training process to 200K steps, which we
  # empirically found to be sufficient enough to train the pets dataset. This
  # effectively bypasses the learning rate schedule (the learning rate will
  # never decay). Remove the below line to train indefinitely.
  num_steps: 200000
  data_augmentation_options {
    random_horizontal_flip {
    }
  }
  data_augmentation_options {
    ssd_random_crop {
    }
  }
}

train_input_reader: {
  tf_record_input_reader {
    input_path: "output/train.record" # 修改处：训练数据路径
  }
  label_map_path: "output/floors.pbtxt" # 修改处：label路径
}

eval_config: {
  num_examples: 8000
  # Note: The below line limits the evaluation process to 10 evaluations.
  # Remove the below line to evaluate indefinitely.
  max_evals: 10
}

eval_input_reader: {
  tf_record_input_reader {
    input_path: "output/test.record" # 修改处：测试数据路径
  }
  label_map_path: "output/floors.pbtxt" # 修改处：label路径
  shuffle: false
  num_readers: 1
}
```
上述`*.config`文件中涉及到的label映射文件`floors.pbtxt`，可以在`TensorFlow/models/research/object_detection`中找一个文件复制出来修改即可：      
```
item {
  name: "floors"
  id: 1
  display_name: "floor"
}
item {
  name: "toutu"
  id: 2
  display_name: "toutu"
}
```
## 模型训练
将https://github.com/tensorflow/models仓库克隆到本地，以其为模型训练的环境。以下步骤为一些环境配置，如清楚可直接跳过，参照https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md。              
1. 将`models/research/object_detection/legacy/train.py`复制到工作目录下。     
2. 安装`slim`模块，借用models的环境，在`models/research/slim`目录下运行`python setup.py install`。      
3. 将slim库暴露到运行环境中`export PYTHONPATH=$PYTHONPATH:pwd:pwd/slim`。     
4. Protobuf编译，在`models/research`目录下执行`protoc object_detection/protos/*.proto --python_out=.`。    

执行命令行如下：       
```
python train.py --logtostderr --train_dir=training/ --pipeline_config_path=training/ssd_mobilenet_v1_coco.config
```
![训练ing](https://img11.360buyimg.com/imagetools/jfs/t1/89793/3/11342/343407/5e33bcc7Ecbebbd38/a67ee67634ca3ed2.jpg)
如果没有报错，可以看到以上输出，慢慢等待模型训练结果即可。   
可以通过以下命令看到优化的情况：   
```shell
tensorboard --logdir=training
```
## 模型效果展示
### python模型导出
通过python测试模型效果不错，将模型导出为tensorflowjs可运行模式，将训练完成的模型导出，TensorFlow Object Detection API提供了一个export_inference_ graph.py脚本用于导出训练好的模型（位于models/research/object_detection目录下）。执行：    
```shell
python export_inference_graph.py --input_type image_tensor --pipeline_config_path training/ssd_mobilenet_v1_coco.config --trained_checkpoint_prefix training/model.ckpt-200000 --output_directory floors_inference_graph
```
![](https://img13.360buyimg.com/imagetools/s241x170_jfs/t1/89499/2/11712/41421/5e3ab6e8E9c448f04/f7cf26d08ea4e292.png)
### 将导出模型转换为TensorFlow.js可运行模型
将模型转换为TensorFlow.js可用的web格式：    
```python
# 模型转换器安装
pip install tensorflowjs

# 运行转换器提供的转换脚本，以下命令任选一
# 参考：https://github.com/tensorflow/tfjs-converter/tree/master/tfjs-converter

# covert from saved_model
tensorflowjs_converter --input_format=tf_saved_model --output_format=tfjs_graph_model --signature_name=serving_default --saved_model_tags=serve ./saved_model ./web_model

# convert from frozen_model
tensorflowjs_converter --input_format=tf_frozen_model --output_node_names='num_detections,detection_boxes,detection_scores,detection_classes' ./frozen_inference_graph.pb  ./web_model
```
得到如下的TensorFlow.js可运行模型数据。   
![web model](https://img13.360buyimg.com/imagetools/s210x170_jfs/t1/95639/32/11688/89905/5e3ab5e5E31a1f5ec/b8a8a16eeaf76b8e.jpg)   
### 前端运行模型
此处只给出关键代码，需要注意的点是，输出数据维度名称与`export_inference_graph.py`中相对应。   
```javascript
// 导入本地模型
model = await tf.loadGraphModel('./web_model/model.json');
// 模型输入处理
let image = tf.browser.fromPixels(canvas);
const t4d = image.expandDims(0);
/**
   *  获取所需的输出维度
   * num_detections: 检测总数
   * detection_boxes: 检测框张量，[ymin , xmin , ymax , xmax]为归一化数据，对应图片检测框位置只需乘以对应宽高
   * detection_scores: 检测框分数，即概率
   * detection_classes: 类别ID，与label_map中相对应
   */
let tensor = await model.executeAsync({'image_tensor': t4d}, `${dim}:0`);
modelOut[dim] = await tensor.data();
```
## 发散思考与深入
- 训练结果有一定**随机性**，增加少量数据不一定会越来越精准，所以模型的训练需要一定时间的数据集积累，才能得到较好结果；由于数据集限制，对于规整的设计稿识别效果显然更好。    
- 需要测试图片颜色对识别效果的影响，后续可能需要加入**交互稿的识别**；对于**多种类型的设计稿**进行匹配，目前仅仅测试了会场类型；对于某些特殊、有显著特征的楼层类型可以进行**更细致分类的识别**，比如头图、优惠券等。   
- 使用python训练的模型不一定能转换为tensorflowjs可运行模型，建议**在哪个环境使用，就在哪个环境训练模型**。不同的TensorFlow版本训练出的模型效果可能有很大差别，不能盲目使用新版本，容易出现未知bug无法解决。            
- 楼层分割这个需求应用比较局限且比较困难。    
楼层是一个复合结构，一个楼层可能有多个组件结合嵌套组成，情况较为复杂。组件维度的分割是更加有价值且合理的分割行为，且组件分割的应用也会更加广泛，例如在设计稿中识别某个组件及其类型，并与沉淀代码库进行匹配。近几年的一些设计图直接生成代码也采用类似的思路，比如微软的sketch2code，淘宝的imgcook。      



## 问题记录
1. `ModuleNotFoundError: No module named 'nets'`    
需要安装slim模块，如果借用`tensorflow/models`环境，则在`models/research/slim`目录下运行`python setup.py install`。另外需要将该模块暴露到环境中`export PYTHONPATH=$PYTHONPATH:pwd:pwd/slim`。其中pwd表示的是当前克隆目录，即research文件所在。            
2. `ImportError: cannot import name 'preprocessor_pb2' from 'object_detection.protos'`
需要` protoc object_detection/protos/*.proto --python_out=. `，参考https://github.com/tensorflow/models/issues/2930
3. `ValueError: Unsupported Ops in the model before optimization LogSoftmax NonMaxSuppressionV5`
python训练得到的模型不能百分百转换到tensorflowjs下运行，google到建议加入参数`--skip_op_check`，参考https://github.com/tensorflow/tfjs/issues/684，可以转化成功但是生成的模型有问题。   
tensorflowjs有许多操作不支持，幸好刚刚更新的版本加入了NonMaxSuppressionV5支持，升级到1.5.2版本搞定。    
4. python安装tensorflowjs最新版本失败
python版本过高的问题，官方建议版本为3.6.8。    
5. 导出saved model的variables为空
参考issue：https://github.com/tensorflow/models/issues/1988，修改`export.py`中`write_saved_model`函数。   
          
## 参考
[[1] Training Custom Object Detector](https://tensorflow-object-detection-api-tutorial.readthedocs.io/en/latest/training.html)   
[[2] 目标检测Tensorflow object detection API之构建自己的模型](https://zhuanlan.zhihu.com/p/35854575)   
[[3] Tensorflow Guide](https://tensorflow.google.cn/guide)   