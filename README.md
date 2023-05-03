# DACON CAR CRASH CLASSIFICATION 

--- 


[링크](https://dacon.io/competitions/official/236064/overview/description)

<!-- ![v2](https://user-images.githubusercontent.com/84311270/174434235-a5149804-4091-4d9e-808f-49c86fa7bbac.png) -->

**[목적]** 차량의 충돌여부와 기상, 날씨를 분류하는 모델 개발
**[요약]** 영상데이터를 입력으로 주어지면 차량의 충돌여부와 기상(일반, 비, 눈), 날씨(낮, 밤)을 판단분석

## Dataset  

**Dataset Info.**

- train [폴더]
  - 학습용 차량 블랙박스 영상
  - TRAIN_0000.mp4 ~ TRAIN_2697.mp4

<br>

- test [폴더]
  - 평가용 차량 블랙박스 영상
  - TEST_0000.mp4 ~ TEST_1799.mp4

<br>


- train.csv [파일]
  - sample_id : 영상 샘플 고유 id
  - video_path : 학습용 차량 블랙박스 영상 경로
  - label : 13가지의 차량 충돌 상황

<br>


- test.csv [파일]
  - sample_id : 영상 샘플 고유 id
  - video_path : 학습용 차량 블랙박스 영상 경로

<br>


- sample_submission.csv [제출양식]
  - sample_id : 영상 샘플 고유 id
  - label : 예측한 차량 충돌 상황 (13가지 Class)

<br>

**Label Info.**

- 13가지의 차량 충돌 상황 Class의 세부 정보
- crash : 차량 충돌 여부 (No/Yes)
- ego-Involve : 본인 차량의 충돌 사고 연류 여부 (No/Yes)
- weather : 날씨 상황 (Normal/Snowy/Rainy)
- timing : 낮과 밤 (Day/Night)

<br>

- ※ ego-Involve, weather, timing의 정보는 '차량 충돌 사고'가 일어난 경우에만 분석

<br>

![이미지](https://dacon.s3.ap-northeast-2.amazonaws.com/competition/236064/editor-image/1675581601829146.jpeg)

<br>

------------------


<br>
  
# Training Plan

## 문제인식

<br>

1. 주어진 라벨링 기준으로는 crash 0 에 대한 다른 클래스 데이터를 활용할 수 없음

<br>

2. 데이터가 잘못 분류되어 있거나(낮인데 밤), 아예 판단할 수 없는 데이터가 포함되어 있음

<br>

## 해결방안

1. **데이터를 최대한 활용하는 방법**

   - stage 2로 나누어 처음 stage1에서는 crash 1인 데이터만을 우선적으로 학습 후 0에 에 ego-involve와 weather, timing에 대한 예측을 실시

   - stage 2에서는 예측된 데이터들을 다시 라벨 수정을 통해 전체 데이터를 활용하고자 함

   - stage 2에서 수정된 데이터를 가지고 다시 학습을 진행, 이때 crash와 ego-involve를 합쳐서 0, 1, 2로 나눔, 이 때 0이 나오면 label을 0으로 지정, 1,2가 나오면 crash는 1, ego-involve도 0 또는 1로 구분하여 label을 지정

<br>

2. **데이터의 오분류된 부분을 수정한 후 학습**
   - 1차 라벨 수정 후 학습(stage1) > crash 0 에 대한 다른 클래스 예측 > 다시 수정 후 학습(stage2) > 최종 테스트 데이터 예측 > 제출

<br>

# Data Clean, Pseudo Label

<br>

- 데이터를 살펴본 결과

    - **crash**는 "차량(나, 제 3의 차량)이 다른차량과의 충돌에서 조금의 회피기동, 또는 다른 차량과 충돌 직전 또는 충돌했을 때"  충돌했다고 판단
    - **ego-involve**는 충돌에 나의 차량이 포함되는지 판단하는 클래스이므로 crash의 판단기준에서 나의 상관여부만을 판단
    - **weather**은 비와 눈이 내리면 명확하게 비와 눈으로 판단되지만, 흐린 날에 비인지 눈인지 판단하기 어려울 경우, 바닥에 물이 많은지 또는 눈이 많은지에 따라 기준설정
    - **timing**은 낮과 밤을 구분, 주차장이나 낮과 밤을 알지 못하는 영상은 삭제, 또한 영상 속에서 시간이 적혀있는 경우 밤이라고 판단되는데 아침인 경우도 있고 아침인데 밤이라고 체크, 이 경우 영상을 보고 낮이면 낮, 밤이면 밤이라고 판단

<br>

# Data Augmentation

- 각 클래스를 구분하는데 필요한 필수 요건(밤, 낮이면 "밝음")에 많이 영향을 주지 않으면서 노이즈를 추가해줄 수 있는 방법으로 augment를 진행

<br>

# Modeling

- 영상 분류에서 많이 사용되는 모델(slowfast, mvit-400 등)로 학습을 진행
    - MVIT 는 16개의 frame만을 가지고 학습을 진행해야하는 제한이 있었음
        - 이는 영상을 보고 crash는 영상 끝쪽에 특징이 몰려있었고, 다른 클래스들은 영상 전반에 특징이 있었기 때문에 50frame중 끝의 32프레임을 2frame씩 건너뛰면서 진행, 또는 32frame을 임의로 고정시켜 진행
    - slowfast는 64frame을 사용했기 때문에 50frame 전체와 마지막부터 14프레임을 반대로 지정 (49,50,49,48....)


<br>

# Result

- 라벨을 수정하고 학습을 진행한 결과 public과 private 점수가 동시에 상승
- 그 중 private 점수가 public 점수보다 높아진 것을 보아 기준 설정이 어느 정도 맞았다고 판단

- **public 0.55262 - 52 place / private 0.65142 - 11 place**

<br>

# Review

- 모델의 학습속도와 컴퓨팅리소스 제한으로 인해 실험을 많이 못해본 것
- 또한 한 모델로 여러 클래스를 판단하는 것보다 하나의 모델로 하나의 클래스를 판단하는 것이 데이터를 전체를 더 적극적으로 사용할 수 있었다고 생각