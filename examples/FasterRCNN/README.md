# Faster-RCNN / Mask-RCNN on COCO
This example provides a minimal (<2k lines) and faithful implementation of the following papers:

+ [Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks](https://arxiv.org/abs/1506.01497)
+ [Feature Pyramid Networks for Object Detection](https://arxiv.org/abs/1612.03144)
+ [Mask R-CNN](https://arxiv.org/abs/1703.06870)

with the support of:
+ Multi-GPU / distributed training
+ [Cross-GPU BatchNorm](https://arxiv.org/abs/1711.07240)

## Dependencies
+ Python 3; TensorFlow >= 1.6 (1.4 or 1.5 can run but may crash due to a TF bug);
+ [pycocotools](https://github.com/pdollar/coco/tree/master/PythonAPI/pycocotools), OpenCV.
+ Pre-trained [ImageNet ResNet model](http://models.tensorpack.com/ResNet/) from tensorpack model zoo.
+ COCO data. It needs to have the following directory structure:
```
COCO/DIR/
  annotations/
    instances_train2014.json
    instances_val2014.json
    instances_minival2014.json
    instances_valminusminival2014.json
  train2014/
    COCO_train2014_*.jpg
  val2014/
    COCO_val2014_*.jpg
```
`minival` and `valminusminival` are optional. You can download them
[here](https://github.com/rbgirshick/py-faster-rcnn/blob/master/data/README.md).


## Usage
To train:
```
./train.py --config \
    MODE_MASK=True MODE_FPN=True \
    DATA.BASEDIR=/path/to/COCO/DIR \
    BACKBONE.WEIGHTS=/path/to/ImageNet-ResNet50.npz \
```
Options can be changed by either the command line or the `config.py` file. 
Recommended configurations are listed in the table below.

The code is only valid for training with 1, 2, 4 or 8 GPUs.
Not training with 8 GPUs may result in different performance from the table below.

To predict on an image (and show output in a window):
```
./train.py --predict input.jpg --load /path/to/model --config SAME-AS-TRAINING
```

Evaluate the performance of a model on COCO, and save results to json.
(Trained COCO models can be downloaded in [model zoo](http://models.tensorpack.com/FasterRCNN):
```
./train.py --evaluate output.json --load /path/to/COCO-ResNet50-MaskRCNN.npz \
    --config MODE_MASK=True DATA.BASEDIR=/path/to/COCO/DIR
```
Evaluation or prediction will need the same config used during training.

## Results

These models are trained with different configurations on trainval35k and evaluated on minival using mAP@IoU=0.50:0.95.
MaskRCNN results contain both box and mask mAP.

 | Backbone | mAP<br/>(box/mask) | Detectron mAP <br/> (box/mask) | Time           | Configurations <br/> (click to expand)                                                                                                                                                           |
 | -        | -                  | -                              | -              | -                                                                                                                                                                                                |
 | R50-C4   | 33.1               |                                | 18h on 8 V100s | <details><summary>super quick</summary>`MODE_MASK=False FRCNN.BATCH_PER_IM=64`<br/>`PREPROC.SHORT_EDGE_SIZE=600 PREPROC.MAX_SIZE=1024`<br/>`TRAIN.LR_SCHEDULE=[150000,230000,280000]` </details> |
 | R50-C4   | 36.6               | 36.5                           | 49h on 8 V100s | <details><summary>standard</summary>`MODE_MASK=False` </details>                                                                                                                                 |
 | R50-FPN  | 37.5               | 37.9<sup>[1](#ft1)</sup>       | 28h on 8 V100s | <details><summary>standard</summary>`MODE_MASK=False MODE_FPN=True` </details>                                                                                                                   |
 | R50-C4   | 36.8/32.1          |                                | 39h on 8 P100s | <details><summary>quick</summary>`MODE_MASK=True FRCNN.BATCH_PER_IM=256`<br/>`TRAIN.LR_SCHEDULE=[150000,230000,280000]` </details>                                                               |
 | R50-C4   | 37.8/33.1          | 37.8/32.8                      | 51h on 8 V100s | <details><summary>standard</summary>`MODE_MASK=True` </details>                                                                                                                                  |
 | R50-FPN  | 38.1/34.9          | 38.6/34.5<sup>[1](#ft1)</sup>  | 32h on 8 V100s | <details><summary>standard</summary>`MODE_MASK=True MODE_FPN=True` </details>                                                                                                                    |
 | R50-FPN  | 38.5/34.8          | 38.6/34.2<sup>[2](#ft2)</sup>  | 34h on 8 V100s | <details><summary>standard+convhead</summary>`MODE_MASK=True MODE_FPN=True`<br/>`FPN.FRCNN_HEAD_FUNC=fastrcnn_4conv1fc_head` </details>                                                          |
 | R101-C4  | 40.8/35.1          |                                | 63h on 8 V100s | <details><summary>standard</summary>`MODE_MASK=True `<br/>`BACKBONE.RESNET_NUM_BLOCK=[3,4,23,3]` </details>                                                                                      |
 
 <a id="ft1">1</a>: Slightly different configurations.

 <a id="ft2">2</a>: Numbers taken from [Group Normalization](https://arxiv.org/abs/1803.08494)
 
 Performance in [Detectron](https://github.com/facebookresearch/Detectron/) can be reproduced.
 Note that most of these numbers are better than what's in the paper. 

## Notes

See [Notes on This Implementation](NOTES.md)
