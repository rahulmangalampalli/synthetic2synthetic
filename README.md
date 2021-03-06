### Synthetic2Synthetic Challenge:

* This challenge has a dataset of synthetically generated objects of 330 classes and they are needed to be localized by a CNN object detection model.

* The dataset has 13200 images with an avaerage of 70 instances per image.

* So here we used a faster-rcnn model with inception V2 as the backbone. It was transfer learned for 2 lakh iterations.

* The configuration for the same is given below:

```bash
model {
  faster_rcnn {
    num_classes: 330
    image_resizer {
      keep_aspect_ratio_resizer {
        min_dimension: 600
        max_dimension: 1024
      }
    }
    feature_extractor {
      type: "faster_rcnn_inception_v2"
      first_stage_features_stride: 16
    }
    first_stage_anchor_generator {
      grid_anchor_generator {
        height_stride: 16
        width_stride: 16
        scales: 0.25
        scales: 0.5
        scales: 1.0
        scales: 2.0
        aspect_ratios: 0.5
        aspect_ratios: 1.0
        aspect_ratios: 2.0
      }
    }
    first_stage_box_predictor_conv_hyperparams {
      op: CONV
      regularizer {
        l2_regularizer {
          weight: 0.0
        }
      }
      initializer {
        truncated_normal_initializer {
          stddev: 0.00999999977648
        }
      }
    }
    first_stage_nms_score_threshold: 0.0
    first_stage_nms_iou_threshold: 0.699999988079
    first_stage_max_proposals: 100
    first_stage_localization_loss_weight: 2.0
    first_stage_objectness_loss_weight: 1.0
    initial_crop_size: 14
    maxpool_kernel_size: 2
    maxpool_stride: 2
    second_stage_box_predictor {
      mask_rcnn_box_predictor {
        fc_hyperparams {
          op: FC
          regularizer {
            l2_regularizer {
              weight: 0.0
            }
          }
          initializer {
            variance_scaling_initializer {
              factor: 1.0
              uniform: true
              mode: FAN_AVG
            }
          }
        }
        use_dropout: false
        dropout_keep_probability: 1.0
      }
    }
    second_stage_post_processing {
      batch_non_max_suppression {
        score_threshold: 0.300000011921
        iou_threshold: 0.600000023842
        max_detections_per_class: 100
        max_total_detections: 100
      }
      score_converter: SOFTMAX
    }
    second_stage_localization_loss_weight: 2.0
    second_stage_classification_loss_weight: 1.0
  }
}
train_config {
  batch_size: 1
  data_augmentation_options {
    random_horizontal_flip {
    }
  }
  optimizer {
    momentum_optimizer {
      learning_rate {
        manual_step_learning_rate {
          initial_learning_rate: 0.000199999994948
          schedule {
            step: 900000
            learning_rate: 1.99999994948e-05
          }
          schedule {
            step: 1200000
            learning_rate: 1.99999999495e-06
          }
        }
      }
      momentum_optimizer_value: 0.899999976158
    }
    use_moving_average: false
  }
  gradient_clipping_by_norm: 10.0
  fine_tune_checkpoint: "/content/models/research/object_detection/faster_rcnn_inception_v2_coco_2018_01_28/model.ckpt"
  from_detection_checkpoint: true
  num_steps: 200000
}
train_input_reader {
  label_map_path: "/content/models/research/object_detection/training/labelmap.pbtxt"
  tf_record_input_reader {
    input_path: "/content/models/research/object_detection/train.record"
  }
}
eval_config {
  num_examples: 8000
  max_evals: 10
  use_moving_averages: false
}
eval_input_reader {
  label_map_path: "/content/models/research/object_detection/training/labelmap.pbtxt"
  shuffle: false
  num_readers: 1
  tf_record_input_reader {
    input_path: "/content/models/research/object_detection/test.record"
  }
}
```

* The model started out with a log loss of 12 amd final loss was around 1.

* Dataset can be found [here](https://drive.google.com/file/d/1NucPkIvxsbggNw6ExjSV9wGzrS2W3S_t/view?usp=sharing), it has annotations.csv which has filename,xmin,ymin,xmax,ymax and class as columns.

* Details about training the model is given in this [jupyter_notebook](https://github.com/rahulmangalampalli/synthetic2synthetic/blob/main/Hackathon_syn.ipynb), it was trained on google with gpu mode.

* **Some detections**:

<p align="center">
  <img src="detect/detected_1.jpg" width = 480>
</p>


<p align="center">
  <img src="detect/detected_4.jpg" width = 480>
</p>

### How to use the model:

* Clone this repository and make sure you have python 3.6

* Install dependencies:

```python
%cd synthetic2synthetic
pip3 install -r requirements.txt
!gdown 1sqDieIPtY4qifsbjBPFcbLQb79Evy4l2 # Downloads model, keep it in the synthetic2synthetic directory.
```

* Run the program:

```python
python3 detect.py "/path/to/image"
```

### Synthetic2Real challenge:

* The same model used for synthetic2sythetic was used for this model also.

* 500 real and 500 synthetic images were used for training.

* The frozen graph can be found [here](https://drive.google.com/file/d/1nwFdiH0KMpUOdZ_CEv9GLOsN2bAZblpY/view?usp=sharing)

* Some detections:

<p align="center">
  <img src="detect/detected_2.jpg" width = 480>
</p>

<p align="center">
  <img src="detect/detected_3.jpg" width = 480>
</p>

* Training Info can be found [here](https://github.com/rahulmangalampalli/synthetic2synthetic/blob/main/Synthetic2Real.ipynb)
