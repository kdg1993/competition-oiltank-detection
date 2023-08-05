# competition-oiltank-detection
Competition for oil tank detection in satellite images. 
* Host : Korea Aerospace Research Institute (KARI)
* Organizer : AI Factory
* Link : https://aifactory.space/task/2454/overview
* Team name : 3M1B

## Problem definition
### Data
* 2D Color Image (1024x1024)
* Satellite Image

### Task
* Object detection
  - Single object : only one class (oil tank)
  - Oriented object detection : bounding box is rotated

* Metric
  - Average Precision (AP)
  - Intersection over Union (IoU) > 0.5

## Challenge
* Bounding Box is rotated (different from COCO)
* Very small data size (Only 70 Images)
  - Pretrained weight is needed & Hard to validate
  - Fine tuning is difficult
* Oil tanks vary in size

## Approach
### Model
* Yolov5
  - Good
    - It is not SOTA but verified and stable &rarr; User friendly and nice docs
    - Easy to apply for custom data &rarr; Time saving
  - Bad
    - No simple option for using oriented object detection
    - Less models than MMdetection

* Bigger model
  - Usually the bigger the model, the higher the performance
  - Does it worked?
    - Yolov5x (the biggest) outperformed yolov5m (the middle among models)
    - Bigger model forces to use of smaller batch sizes due to the memory. \
    However, the advantage of a bigger model seems to exceed the disadvantage of smaller batch size

* Optimal balance between image size & batch size
  - Since the model has batch normalization, bigger batch might be helpful
  - Due to the GPU memory restriction, the bigger the batch, the smaller the image size
  - Originally, yolov5 was trained by 640x640 size image but competition gave us 1024 image
  - Does it worked?
    - Increase image size and sacrifices batch size was better 
    - Reducing image size to 640 showed low performance. \
    Reducing image size might be made it harder to detect small size oil tanks

* Augmentation and training technique
  - Multiscale learning : using += 50% image size while training
    - It forces to use smaller batch size due to the increased size while training (16 &rarr; 8)
  - Cosine annealing
    - Not sure it worked because we couldn't see 
  - Does it worked?
    - No, performance was decreased
    - There was no clear difference of validation score bewteen using this technique or not. \
    We suspect generalization performance was worsened
    - Reduced batch size due to the memory is presumed to be a reason of bad generalization performance

### Data
* Ignore bounding box rotation
  - Why? 
    - Yolo pretrained weight is based on COCO (No rotation)
    - Oil tank bounding box closes to the square shape due to its nature
  - How ?
    - Center of mass of polygon + Square shape bounding box assumption
    - Width and height are equal and calcuated by square root of bounding box area
  - Does it worked?
    - I worked but performance is not high
    - It is well matched to the square-shaped nature of oil tank (In terms of IoU)

## What we learned
* How to use Yolo(v5) for custom data
  - A non-COCO format data to COCO format
  - Still, for DOTA dataset, MMdetection could be a better choice
* How to improve performance
  - Using bigger model
  - Consider carefully when using increasing image size technique due to the trade off
