# Document_Type_Classification | `2024.04.11 - 2024.04.23`
[CV / Classification] 문서 이미지를 분류하는 모델을 개발

<br>

## Summary

### 🛠️ 목표

문서 데이터는 금융, 의료, 보험, 물류 등 산업 전반에 가장 많은 데이터이며, 많은 대기업에서 디지털 혁신을 위해 문서 유형을 분류하고자 한다. 이러한 문서 타입 분류는 의료, 금융 등 여러 비지니스 분야에서 대량의 문서 이미지를 식별하고 자동화 처리를 가능케 할 수 있다. <br>
17개의 카테고리로 이루어진 문서 이미지를 분류하는 모델을 구축하는 것이 목표이다. <br>

### ⚙️ 수행 역할

 -  **라벨링 에러 수정**: 수집된 문서 이미지 데이터셋에서 잘못된 라벨이나 누락된 라벨을 식별하고 수정하여 데이터 품질을 향상
 -  **데이터 증강**: 문서 회전, 왜곡, 노이즈 추가 등 문서 이미지를 현실적인 상황과 유사하게 변형하여 데이터 다양성을 확보
   
 -  **1차 분류기 및 2차 분류기 모델링**

    - 문서 유형을 대략적으로 분류하기 위한 1차 분류기를 설계
    - 세분화된 카테고리 분류를 위한 2차 분류기를 설계

### 📈 결과 및 직무에 적용할 점

 1. **데이터 관리 및 품질 제어**
    - 라벨링 에러 수정 및 데이터 품질 개선 경험을 통해 데이터 분석 직무에서 데이터 신뢰도 상승 기대
 2. **이미지 증강 기술 적용**
    - 다양한 라이브러리(`Albumentations, Augraphy`)를 활용하여 데이터셋의 다양성을 증가시키고 평가 데이터와 유사한 품질의 학습 데이터를 구축
 3. **모델 앙상블**
    - 단순 모델 앙상블이 아닌, 단계적 분류를 통한 앙상블 기법을 적용하여 성능을 극대화

<br>

## 프로젝트 상세

### 📝 데이터 설명

 - **데이터 구조**
   
   - train.csv 파일은 각 이미지 파일명(ID)에 해당하는 정답 클래스(target)를 제공한다. <br>
   - train 폴더에 존재하는 이미지들은 train.csv 파일에 의해 정답 클래스로 맵핑된다. <br>
  
  ```
  ├── train                    # 이미지 데이터 폴더
  │   ├── 002f99746285dfdd.jpg  # 이미지 파일 1
  │   ├── 008ccd231e1fea5d.jpg  # 이미지 파일 2
  │   ├── ...                   # 중간 생략
  │   ├── ff8a6a251ce51c95.jpg  # 이미지 파일 1569
  │   └── ffc22136f958deb1.jpg  # 이미지 파일 1570
  │
  ├── train.csv               # 이미지 파일명과 정답 클래스
  ```

 - 데이터 형태: .jpg 파일
 - 데이터 개수: train (`1,570`), test (`3,140`)
 - 카테고리 개수: 17

![image](https://github.com/user-attachments/assets/782fadcd-9bf5-4f40-aee5-d2af04ea7610)

학습, 평가 데이터 모두 개인정보는 검은색 박스로 가려져 있고, 학습 데이터와 달리 평가 데이터는 랜덤하게 회전, 반전 등이 적용 되었고 훼손된 이미지들이 존재한다.

| class | 문서 유형                          |
|------|------------------------------------|
| 0    | 계좌번호                          |
| 1    | 건강보험 임신출산 진료비 지급 신청서 |
| 2    | 자동차 계기판                      |
| 3    | 입퇴원 확인서                      |
| 4    | 진단서                            |
| 5    | 운전면허증                        |
| 6    | 진료비영수증                      |
| 7    | 통원/진료 확인서                  |
| 8    | 주민등록증                        |
| 9    | 여권                              |
| 10   | 진료비 납입 확인서                |
| 11   | 약제비 영수증                    |
| 12   | 처방전                            |
| 13   | 이력서                            |
| 14   | 소견서                            |
| 15   | 자동차 등록증                     |
| 16   | 자동차 번호판                     |

### 📊 EDA
 - **데이터 불균형**

   ![image](https://github.com/user-attachments/assets/99c9ac90-bf56-4c66-b610-6878532ce9db)

   학습 데이터의 클래스 별 데이터 개수가 불균형

<br>

 - **라벨링 오류**
   
   ![image](https://github.com/user-attachments/assets/26558456-d12f-4205-aa6a-1766e87251c5)

   ![image](https://github.com/user-attachments/assets/f101b927-fc54-40ce-82b0-6f4ffbf6a090)

   학습 데이터 중 라벨링이 잘못되어 있는 경우 발견 (진단서 --> 소견서, 진료 확인서 --> 입퇴원 확인서)

### 🔍 데이터 전처리
 
 1. 학습 데이터는 `1,570`개, 평가 데이터는 `3,140`개로 학습 데이터가 평가 데이터에 많이 부족하다. <br>
 2. 학습 데이터와 달리 평가 데이터는 회전, 반전, 노이즈 추가 등 원본 데이터와는 많이 다른 모습을 하고 있다. <br>
 따라서, 모델의 높은 성능을 위해서는 학습 데이터를 평가 데이터와 유사하도록 **변환**, **증강**을 해주었다. <br>

 - **데이터 증강**
   
   아래와 같은 순서대로 이미지 작업을 진행하였다. <br>
 > **1. Noise `Augraphy`**
 >
 > ![image](https://github.com/user-attachments/assets/97fb6268-e0e3-4906-bfcd-e9e33fe2892b)
 > 
 > <br>
 >
 > Augraphy에는 위와 같이 다양한 기법들이 존재하는데, 그 중 평가 이미지들과 가장 비슷한 **InkBleed**와 **VoronoiTessellation** 기법을 사용하였다.

 > **1-1. Mixup `cv2`**
 >
 > 평가 데이터를 살펴본 결과, 두 개의 이미지가 믹스업된 이미지가 있는 것을 발견하였다.
 > 따라서, 알파 값을 조정하여 랜덤으로 두개의 이미지를 적절히 믹스업하였다.

 > **2. Flip & Rotation 과 Padding `Albumentation`**
 >
 > <img src="https://github.com/user-attachments/assets/5d16fcaf-367b-4acc-bcfb-3e91c314fb4d" alt="0a0302cbe2dcdd40" width="30%">
 >
 > <br>
 >
 > 평가 데이터와 유사하도록 이미지를 여러 각도로 회전하고 반전하였으며, 나머지 부분은 하얀색 배경으로 채워넣었다.

### 🧠 모델링

 - 📐 평가지표

   - Macro F1
   <img src="https://github.com/user-attachments/assets/0cefe4a5-aa11-493f-9d1b-8f87f6f952f8" alt="0a0302cbe2dcdd40" width="50%">

 - **모델 구성**

   ![image](https://github.com/user-attachments/assets/0d4ea131-411d-4a1a-a0d4-f32ecde8a546)

   end-to-end 방법의 경우 GPU 리소스 문제로 불가능해 계층적 구조의 방법을 채택 <br>
   비교적 쉬운 작업(큰 세가지 카테고리로 분류)을 하는 `1차 분류기`의 성능은 **100%**가 나와야 함 <br>

   1차 분류기에서는 ResNet, CrossVIT, EfficientNet, CoAtNet 중, 가장 성능이 좋은 `EfficientNet`을 채택 <br>

   <br>

   | 모델          | 정확도 (%) |
   |---------------|------------------|
   | ResNet-50     | 98.5%            |
   | CrossViT-15   | 98.2%            |
   | EfficientNet-B0| 100.0%           |
   | CoAtNet-0     | 97.7%            |

   2차 분류기 중 난이도가 어려운 병원 문서 분류기에는 다양한 모델들을 사용하려고 노력하였다. <br>

   <br>

   **Focal Loss**

   CE(Cross Entropy)를 개선한 손실함수로, 오분류율이 높은 케이스에 가중치를 부여하여 상대적 난이도가 높은 문제를 더 잘 해결할 수 있게 만듬

   **TTA (Test-Time Augmentation)**

   TTA는 평가 단계에서 데이터를 다양한 방식으로 변형하여 모델에 입력한 후, 각 결과를 종합하여 최종 예측을 도출하는 기법

   - 문서 이미지를 회전, 크기 조정, 좌우 반전 등의 변환을 적용
   - 각 변환된 이미지에 대해 모델의 예측 결과를 도출하여 안정적인 결과 확보

   **Soft Voting을 통한 앙상블**

   여러 모델의 예측 확률을 평균 내어 최종 클래스를 결정하는 방식

   - 다양한 모델의 예측 확률 값을 조합
   - 각 모델의 개별 강점을 결합하여 최적의 예측 결과 도출

   TTA와 Soft Voting 기법을 활용한 앙상블을 통해 성능을 유의미하게 향상  

   | 모델 구성                    | F1 Score |
   |----------------------------|----------|
   | 기본                        | 0.9251     |
   | 기본 + TTA                  | 0.9273     |
   | 기본 + Soft Voting + TTA    | 0.9351     |

   <br>
   
   기존에 이미 회전, 반전을 통한 데이터 증강이 이루어졌기에 TTA를 통한 큰 성능 차이를 보이지 않은 것 같다. <br>
   회전 변형 증강을 거치지 않은 상태에서, 마지막 단계의 TTA를 통해서만 다양한 각도의 데이터를 다루었다면 더 빠른 학습이 가능하고, 드라마틱한 성능 향상으로 이어졌을 것 같다. <br>

## 🎯 결과 및 기대효과

- **데이터 증강 및 모델 성능 향상**: 데이터 증강과 TTA 기법의 효과적인 적용을 통해 모델의 일반화 능력 향상. 실생활에서 발생할 수 있는 다양한 문서 변형에 대해 대응할 수 있는 모델 구축 가능
- **문서 자동화 구축 가능성**: 금융, 의료, 보험 등 다양한 산업에서 대량의 문서 처리, 관리 자동화 및 오류 최소화를 통한 비용 절감 기여 가능
