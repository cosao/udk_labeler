# udk_labeler

[![UBUNTU18.04 LTS](https://img.shields.io/badge/UBUNTU-18.04LTS-orange)](https://www.python.org/downloads/release/python-360/)
[![WINDOWS10](https://img.shields.io/badge/WINDOWS-10-blue)](https://www.python.org/downloads/release/python-360/)
[![Python 3.7](https://img.shields.io/badge/Python-3.7-3776AB)](https://www.python.org/downloads/release/python-360/)

Object Detection Task는 객체마다 Bounding Box 처리해주는 이른바 Label작업이 필요합니다. 
작업은 주로 사용자가 수동으로 진행해야했으며, 이는 상당한 시간과 정신을 소모하게 됩니다.

이러한점에 착안하여 Label작업을 자동으로 해주는 **udk_labeler**를 개발하였습니다.

**udk_labeler**는 오픈소스 labelImg를 수정하여 개발되었습니다. 대부분의 UI는 변경된것이 없으며 한가지 기능이 추가되었습니다.

- **active learning** 기반의 AI 엔진이 탑재되어있습니다. AI 엔진은 Pytorch 기반의 RetinaNet 모델입니다.
   
추후에 다양한 AI모델을 추가할 예정에 있으며, UI/UX를 고려한 디자인도 보완해나갈 예정입니다.

## Install

conda, venv 등 가상환경위에서 설치하는것을 추천합니다.

아래의 패키지들을 설치해줍니다. 만약 설치 도중 에러가 나는 모듈이 있다면 ISSUE에 등록 부탁드립니다. 

```Shell
pip install torch
pip install torchvision
pip install PyQt5
pip install pycocotools
pip install scikit-image
pip install opencv-python
pip install tqdm
pip install tensorboardX
pip install lxml
```
**(중요) Active Learning 사용은 GPU 장치가 있어야 사용가능합니다.**

### Windows

- [pycocotools] Visual C++ error: [https://visualstudio.microsoft.com/ko/vs/older-downloads/](https://visualstudio.microsoft.com/ko/vs/older-downloads/) 
에서 *재배포 가능 패키지 및 빌드 도구* 하위 *Microsoft Build Tools 2015 업데이트3* 을 설치해주시기 바랍니다. 

### Ubuntu

- [opencv] lib error: 아래의 패키지를 설치해주세요.

```Shell
sudo apt-get install -y libglib2.0-0 libsm6 libxext6 libxrender1 libfontconfig1
```

발생했던 에러에 대해서 제보해주시면 case별로 다루도록 하겠습니다.

## How to Use

UI와 기능은 모두 labelImg와 동일한 형태입니다. Auto Labeler를 사용하지 않을 경우에는 기존 방법대로 사용합니다.

.. image:: /demo/udk_label.png
     :alt: Demo Image

### Step

1. 이미지 폴더를 불러옵니다.
2. annotation이 저장될 폴더를 지정합니다. annotation은 기본 labelImg와 같이 xml형식으로 저장되며,
   해당 폴더에는 반드시 xml파일들만 존재해야 합니다.
   
3. 전체이미지 대비 10% 또는 100장 이상의 이미지에 대하여 사용자가 직접 Label작업을 진행합니다.
   이를 기반으로 학습이 진행되기 때문에 최대한 많은 이미지에 대해서 Label을 진행해주는것을 추천합니다.
   
4. 활성화된 Active Learning 버튼을 누릅니다. 우측 하단에 상태바에 진행상황이 표시됩니다.

5. 완료가 되면 Unconfirmed File List에 100장의 이미지 목록이 나타납니다.
   사용자는 해당 이미지에 대해서 다시 Label 작업을 진행합니다. 작업이 완료되면 Continue 버튼을 누릅니다.

6. 5번 과정을 반복해서 진행합니다.
   
7. 반복작업이 어느정도 진행되었다고 판단이 되면 Stop 버튼을 누릅니다. 
   그러면, Label작업을 하지 않은 나머지 이미지에 대해서 모두 Auto Label을 진행합니다.
   
## Concept



### 1. Set folder structure

LabelImg를 통해 생성된 xml파일을 아래와 같은 폴더구조가 되도록 정리해줍니다.
 
```
root_dir/
│
├── img/ - Img
│      │
│      ├ 1.jpg
│      ├ 2.jpg
│      ├  ...
│
├── anno/ - Annotation
│      │
│      ├ 1.xml
│      ├ 2.xml
│      ├  ...
```

### 2. Create Label_map(label_map.txt)

적당한 경로에 `label_map.txt` 파일을 생성하고 사용될 class명을 적어줍니다. 추천하는 경로는 /dataloader/dataset/label_map/label_map.txt 이지만 반드시 설정할 필요는 없습니다. __(주의) 맨 마지막 Line 이후 줄 개행이 되지 않도록 합니다.__

```text
SOOT
SPOTTED
WATER_DROP
SHADOW
PRINTING
DATE_STAMP
unknown
```

### 3. (Optional) Crop image

제공받은 면 이물 이미지는 4096*4096 크기로 매우커서 Out Of Memory(OOM)이 발생합니다. 해당 이미지를 학습할 수 있도록 적당한 크기로 잘라주는 과정입니다.

```
python ./utils/crop_img.py --img_dir {img_dir} \
                           --anno_dir {anno_dir} \
                           --label_map {label_map_path} \
                           --crop_H {heigth of cropped image}
                           --crop_W {weight of cropped image}
                           --save_root_dir {Path where the cropped image will be saved}
```

실행 후 결과는 아래와 같이 구성됩니다.

```
save_root_dir/
│
├── img/ - Croped img
│      │
│      ├ 1.jpg
│      ├ 2.jpg
│      ├  ...
│
├── anno/ - Croped annotation
│      │
│      ├ 1.xml
│      ├ 2.xml
│      ├  ...
│
└── (SOOT_list.txt)
```


현재는 검댕이(SOOT), 점박이(SPOTTED) class가 매우적기 때문에 Augmentation을 적용하였습니다. Target Class에 대하여 Crop을 진행할때 1개의 ROI에서 4개의 이미지가 생성됩니다. (생성된 이미지는 512x512 size)

Test를 해본 결과 검댕이와 음영(SHADOW)의 Feature가 유사하여 구분을 잘 못하는것 같습니다. 배경에 보이는 특정패턴(unknown)에 대해서도 동일합니다. 때문에, SHADOW와 unknown 대해서도 ROI당 4개의 이미지를 생성하도록 하였습니다.

Target class인 검댕이에 대해서만 Train/Val, Test로 나누어지게 되며 Train/Val : Test = 8 : 2 입니다. Test에 사용될 데이터의 경로에 대한 정보는 {root_dir/SOOT_list.txt}에 저장됩니다. 만약 Test 데이터를 별도로 구성하고 싶지 않다면 `./utils/crop_img.py` 파일의 맨 아래쪽 **_#임시코드시작 ~ #임시코드끝_** 부분을 모두 주석처리하면 됩니다.

### 4. Run voc2coco.py
xml파일들을 COCO 스타일의 json형태로 바꿔줍니다. 데이터의 비율은 Train : Validation = 8 : 2 입니다.
```Shell
python ./utils/voc2coco.py --ann_dir {input_anno_dir} --labels {label_map_path} --output {output_dir}
```

### 5. Run train.py

4번째 단계까지 모두 진행이 되었다면 데이터 폴더구조는 아래와 같습니다. __반드시 이 구조를 따라야만 합니다__
```
root_dir/
│
├── img/ - img to be used for training
│      │
│      ├ 1.jpg
│      ├ 2.jpg
│      ├  ...
│
├── anno/ - xml to be generated from labelImg
│      │
│      ├ 1.xml
│      ├ 2.xml
│      ├  ...
│
└── annotations/ - json to be used for training
        │
        ├ instances_train.json
        └ instances_val.json

```

학습은 train.py에서 시작하게 됩니다. 각종 환경설정은 `config.yaml`에서 설정하실 수 있습니다.

```yaml
# global Config
projectname: 'CJ_DEFECT_Not_Print_Img'          # 작업 단위의 이름을 설정합니다.
data_style: 'coco'                              # 현재 coco 스타일만 가능합니다.
cuda: True                                      # cuda를 사용하지 않으면 작동되지 않습니다. 반드시 True
tensorboard: True                               # Tensorboard 사용여부를 정합니다.

# Config DataLoader
root_dir: '/media/jsk/data/namwon/defect/Not_Print_img/crop/'  # 이미지 데이터의 최상위 경로입니다.
resize: [608, 1024]          # 기본값은 height, width=[608, 1024] 입니다. 개발중입니다. 사용x
batch_size: 4
start_epoch: 0
epoch: 100
num_worker: 3
pin_memory: True

# Config model
model: 'retinanet'           # 사용할 모델을 적어줍니다. retinanet만 가능
depth: 50                    # backbone으로 사용할 resnet의 depth를 정합니다. 18, 34, 50, 101, 152
resume:                      # 학습할때 checkpoint 사용여부를 정합니다. 아직 개발중
lr: 0.00001                  # learning rate를 설정합니다.
```

`config.yaml` 설정을 완료한 후 아래와 같이 실행합니다.

```
python train.py --load_config {config_file_path}
```

학습이 진행되면 모델파일 및 Tensorboard와 같은 산출물은 `/run/{projectname}/{오늘날짜_실행횟수}/` 경로에 저장됩니다. projectname은 __작업단위__ 를 뜻하기 때문에 Best model파일인 `model_best.pth.tar`는 `/run/{projectname}/` 아래에 생성됩니다.

### 6. Tensorboard

Tensorboard를 True로 설정했다면 활용하실 수 있습니다.

```Shell
$ tensorboard --logdir=/run/{projectname/{오늘날짜_실행횟수}} --port 6006
```

Tensorboard에는 아래의 사항들을 확인할 수 있습니다.
- Train_Classification_Loss
- Train_Regression_Loss
- Train_Total_Loss
- mAP with regard to IOU, Image Size(Small, Medium, Large)
- AR with regard to IOU, Image Size(Small, Medium, Large)


## Inference

추론부의 메인은 `visualize_single_image.py`에서 시작하게 됩니다. 이미지를 1장씩 입력받아 추론을 하고, 결과에 대한 정보는 xml파일로 저장이 됩니다. 또한 bbox의 시각적 정보도 포함되어 img로 저장이 됩니다.

```
python visualize_single_image.py --image_dir {img_dir to be inferred} \ 
                                 --model_path {modelfile_path} \ 
                                 --class_list {label_map_path} \ 
                                 --save_dir {img_dir to be saved}
```



## Appendix

### 1. 평가지표
Validation이 진행할 때 Epoch마다 AP, AR 정보를 출력해줍니다.

```Shell
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.335
 Average Precision  (AP) @[ IoU=0.50      | area=   all | maxDets=100 ] = 0.499
 Average Precision  (AP) @[ IoU=0.75      | area=   all | maxDets=100 ] = 0.357
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.167
 Average Precision  (AP) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.369
 Average Precision  (AP) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.466
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=  1 ] = 0.282
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets= 10 ] = 0.429
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=   all | maxDets=100 ] = 0.458
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= small | maxDets=100 ] = 0.255
 Average Recall     (AR) @[ IoU=0.50:0.95 | area=medium | maxDets=100 ] = 0.508
 Average Recall     (AR) @[ IoU=0.50:0.95 | area= large | maxDets=100 ] = 0.597
```

하지만 _면 이물 검출_ 과제의 경우에는 검댕이를 정확히 구분해내기 위해 다양한 class를 추가했습니다. 추가된 class는 사실 검출하면 안되는것이기 때문에 mAP, AR의 평가지표를 사용하는것은 큰 의미가 없습니다.

따라서 Ground Truth 이미지 內 검댕이와, Prediction 이미지 內 검댕이의 IOU를 계산하여 0.5이상이 되는 경우에 해당 이미지를 **_NG_** 처리하고 그렇지 않는경우에는 **_OK_** 처리 하는 방법을 사용했습니다. 즉,  classification에서 자주 사용하는 형태의 Confusion Metric을 이용한 Binary 분류지표를 사용하였습니다.

```
python ./utils/confusion_metric.py --true_xml_dir {Ground Truth_xml_dir} \ 
                                   --pred_xml_dir {prediction_xml_dir}
```

```
# Result from Binary CunfusionMetric

    NG   OK
NG  1    1
OK  1    1
```

### 2. Docker

Dockerfile을 실행하기 위한 Docker install을 해줍니다.
```Shell
# Docker install
curl -fsSL https://get.docker.com/ | sudo sh 

# 현재 접속중인 사용자에게 권한주기
sudo usermod -aG docker $USER
```

Terminal을 내렸다가 다시 실행한 후 잘 설치되었는지 확인합니다

```Shell
docker version

Client: Docker Engine - Community
 Version:           19.03.12
 API version:       1.40
 Go version:        go1.13.10
 Git commit:        48a66213fe
 Built:             Mon Jun 22 15:45:36 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.12
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.10
  Git commit:       48a66213fe
  Built:            Mon Jun 22 15:44:07 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.2.13
  GitCommit:        7ad184331fa3e55e52b890ea95e65ba581ae3429
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

GPU를 사용할 수 있도록 [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)를 설치합니다.

```Shell
# Add the package repositories
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo systemctl restart docker
```

Docker를 사용할 수 있는 모든 준비를 마쳤습니다. 아래의 경로로 이동한 후 Build를 해줍니다.

```Shell
:~/{source_root}/Dockerfile$ docker build -t '{docker image name}' .

# example
# docker build -t 'test' .
```

{source_root}로 이동하여 `run_container.sh`를 열어준 후 `SOURCE_FOLDER`, `DATA_FOLDER`를 Local에 존재하는 소스코드와 데이터 경로로 수정해줍니다. *__(중요) `-v` 옵션으로 마운트된 형태이기 때문에 반드시 Local에 소스코드와 데이터가 존재해야합니다.__*

 해당 파일을 실행하면 Build된 Docker Image의 Container를 생성하고 가상환경으로 들어가게 됩니다. Container 내부에서는 학습, 추론 등 모든 기능을 동일하게 사용할 수 있습니다.


## References

- [tzutalin/labelImg](https://github.com/tzutalin/labelImg)
- [yhenon/pytorch-retinanet](https://github.com/yhenon/pytorch-retinanet)
