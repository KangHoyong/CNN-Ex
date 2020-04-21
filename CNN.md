# CNN model 
## AlexNet 
구조 8개 layer 

특징: ReLU, Drop out 등의 효율적 학습 기법을 제안

적용된 검출방법 : R-CNN , Fast-R-CNN 

## VGG
구조 19개 layer 

특징 : 3x3 kernel을 사용하여 망이 깊은 구조를 만듬

적용된 검출방법 : Fast-R-CNN

## GoogLeNet 
구조 22개의 layer

특징 Inception 모듈을 적용하여 망의 깊이가 깊어져도 연산량이 증가하지 않는 구조 제안

적용된 검출방법 : R-CNN

## ResNet 
구조 152개 layer 

특징 Residual learning 을 통해 깊은 네트워크도 쉽게 최적화 기능을 제한 

적용된 검출방법 : Faster R-CNN

## densent 

## senet 

# Localization, Detection, Segmentation 
공통점은 : 모두 어떤 Ojbect에 대한 위치를 찾는것 

Localization : 1(사진) : 1(Object)
 - Dataset : C(label), x,y,w,h

Detection : 1(사진) : N(Object)
 - Dataset : CAT (x,y,w,h)
 
 (박스가 여러개이므로 Dataset이 여러개)

Segmentation : 픽셀 단위로 표시 
  - Dataset : 박스의 X,Y,W,H 대신 14*14px 안에 물체가 있느냐 
    없느냐를 판단 0 : 없음 1 : 있음 

  
Localization 구성 
모델 Cov Layer 의 Filter들 처럼 Sliding window 방식으로 돌아다니면서, 강아지가 있는지 없는지 predict 한다.

(확률기반) 

결과적 : 곰이 있는 위치에는 확률을 높혀 노란색 / 중간 보라색 / 없는 위치는 검은색

Detection 구성
1. One - Stage Method : 속도가 빠름 

2. Two - Stage Method : 정활도가 높음 



# Semantic Segmentation

- FCN 

- U-net 

# 객체 인식 
## (Two-stage)

 ## R-CNN
  기존알고리즘으로 먼저 찾고 + 딥러닝으로 다시 찾는것 (2stage)

  1. image classifiction 모델이라는 전제하에 시작한다. 그것을 Transfer learning 
  2. classification 모델 2개로 복사 
   
     1) 기존 classificaiton 모델은 1000개 카테고리를 맞추는 imageNet 챌린지용 너무많으므로 기존 모델의 output Layer만 변경 
     output(카테고리) 20개 + (empty dataction 못한경우 background)1 = 21개 
     2) Regrssion 모델 : box를 쳐주는모델 
       -> x y w h 맞춰야한다.
     3) 2개 모델을 따로 predict 한다
      -> 성능이 잘 안나와서 기존 Detection알고리즘 사용해서 box후보군을 잘러서 넣어줄 것이다.
  3. 기존 Obejct Detection 알고리즘 중 성능이 좋은 Selective Search 알고리즘을2개모델 앞에 적용 
  - 전체 이미지 중 box쳐질만한 이미지의 일부분을 잘라주는 기존 Detection 알고리즘
  - Selective search 는 하나의 이미지에서 2000개의 box후보군을 잘라준다.

  4. Selective search 로 쪼개진 이미지의 일부분을 CNN Classification 모델 / Regression 모델에 각각 넣어준다.

  참고 : mAP (mean Average Precision)는 예측한 box와 정답 box의 일치도

  단점 : 복잡 + 느리다 
  
  기본적으로 3개의 모델이 필요
  
  1. per-trained 된 classification 모델
  2. 그것의 output 개수를 고친 classification 모델 
  3. 그것으로 BoundBox의 x y w h 를 맞추는 Regression 모델 

  4. 이미지를 input할때 Selective search 로 후보 box 들을 잘라내기위해 2000번 자를때 속도가 느림 그많큼 용량도 많이 차지 

        (이미지넷 챌린지 이미지 압축해도 60GB -> one 이미지당 2000번 쪼개면 60 * 2000 GB...)

사실 Selective Search 로 자른 2000장의 새로운 후보이미지들은 너무 용량이 크므로, 하드에 저장했다가 다시 꺼내서 쓴다고 한다. 용량 뿐만 아니라 속도도 어머어마하게 느리다.

즉 결과적으로 모델을 줄이고 (3->1) + 용량을 줄여서 속도를 빠르게(Selective search 순서바꿈) 하는 새로운 Detection알고리즘이 필요 

## Faster R-CNN
 - R-CNN 에서는 앞에서 2000개를쪼개는 반면 
 Faster R-CNN 뒤에서 2000개를 쪼개는 방법을 이용

 즉 이미지를 먼저 넣고 Selective search 2000개를 쪼갠것을 -> CNN 넣는 방식 (R-CNN)

 Faster R-CNN : 이미지 1개 CNN넣고 나온 결과물을 Selective search 로 2000번 쪼개는방식

 여기서 CNN에서 나온 결과물(output feature)은 RGB 픽셀로 구성된 이미지가 아니라서 바로 Selective Search에 들어가 쪼개질 수 없다. 

 그래서 input image를

 1. CNN넣기 전에 이미지에 selective search 먼저 돌려서 x y w h 좌표만 구한다
 2. input image를 CNN에 넣으면 output feature(feature map)가 나온다. 

 3. output feature 는 이미지가 아니라서 selective search 에 못넣음 그러나 CNN을 통해 나온 output feature 가 input image에 비해 얼마나 작어 졌는지는 계산 가능 CNN 구조를 고려하여 input image의 사이즈가 얼마나 줄었는지 알 수 있음
 4. 유추한 계산을 바탕으로 + 1번에서 구해놓은 잘라야할 X Y W H 좌표를 이용해 Output Feature 를 2000번 자르면 되는 것이다.

 기존 R-CNN 에서 3개나 되었던 모델을 어떻게 줄였는가 ? 
 R-CNN에서 Transfer Learning(CNN)모델을 -> Reg 모델 1개 / Classification 모델 2개 나누어서 따로 튜닝

 Fast R-CNN에서는 Transfer Learning (CNN) 모델을 
 -> 다시 1개의 모델로 만든 다음, output Layer 2개로 나누어서 따로 loss funcation and Backpropagtion 하여구한다. 

 -> 1개의 모델을 output만 2개로 나눈형태 

 2013년도 이전 논문이다보니 마지막 단계에서 FC 층을 이용하고 있다 현재는 보통 1x1 Conv 를 사용하여 (객체의 정보를 보존한다.) 

 FC 단점 : input 되는 image의 사이즈가 서로 동일해야한다 
 FC 층 이전에 Selective search로 제각각 2000조각 난 output feature의 사이즈를 resizing 해야한다. 
 하지만 output feature (feature map)은 이미지가 아니라서 resizing 불가능 

 일반적으로 R-CNN 경우는 처음부터 Selective search 되어서 이미지를 크롭해준것이기 때문에 resizing이 바로가능
 Fast R-CNN은 resize 해줘야할 것이 이미지가 아니라, output feature 인 문제가 발생 ..

 Roi pooling 이라는 것을 개발하여 해결 Max Pooling 으로 사이즈를 절반으로 줄이는 것과 비슷한 개념 
 재각각 2000천개 쪼개진 것을 output feature (feature map)을 같은 사이즈로 변환 하여 Full-connected 하는 방식
 
 단점 : Fast R-CNN : 가장 성능이 느린 기존 알고리즘 Selective search(Region proposal)하는 부분도 딥러닝 만듬
 
## Mask R-CNN


(one-stage)
- YOLO v1

- YOLO v3

- SSD 

- RatinaNet 

- FCOS 





