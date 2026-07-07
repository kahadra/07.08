# BGI-YOLO 세미나 발표 구성안

## 논문 정보

- **논문 제목**: *BGI-YOLO: Background Image-Assisted Object Detection for Stationary Cameras*
- **발표자**: 박천웅
- **소속**: IRV_lab
- **발표일**: 2026년 7월 8일
- **발표 분량**: 18장 / 약 30분
- **발표 방향**: 논문 리뷰 중심, 마지막에서 “이 논문을 검토한 이유”로 개인 연구와의 관계를 회수
- **논문 링크**: https://www.mdpi.com/2079-9292/14/1/60

---

## 전체 발표 흐름

| 번호 | 슬라이드 제목 | 예상 시간 | 핵심 역할 |
|---:|---|---:|---|
| 1 | Title | 0:30 | 발표 정보 제시 |
| 2 | 목차 | 0:40 | 발표 흐름 안내 |
| 3 | Background: Object Detection in Stationary Cameras | 1:30 | 정지 카메라 객체 검출 배경 |
| 4 | Problem 1: General Object Detectors Ignore Fixed Backgrounds | 2:00 | 일반 detector의 문제의식 |
| 5 | Problem 2: Limits of Background Subtraction-Based Detection | 2:00 | 기존 배경 차분 방식의 한계 |
| 6 | Main Idea: Background Image-Assisted Detection | 1:30 | BGI-YOLO의 핵심 아이디어 |
| 7 | Background Image Generation | 2:00 | 배경 이미지 생성 방식 |
| 8 | BGI-YOLO(1): Image-Level Fusion | 2:00 | image-level fusion 설명 |
| 9 | BGI-YOLO(2)(3): Feature-Level Fusion | 2:00 | feature-level fusion 설명 |
| 10 | Compared Methods | 1:40 | 비교군 정리 |
| 11 | Dataset Configuration | 2:00 | 데이터셋 구성 |
| 12 | Implementation Details | 1:30 | 학습 및 평가 조건 |
| 13 | Quantitative Results 1: WITHROBOT S1 | 2:00 | WITHROBOT S1 결과 |
| 14 | Quantitative Results 2: LLVIP | 2:00 | LLVIP 결과 |
| 15 | Precision & Recall Analysis | 1:40 | precision/recall 분석 |
| 16 | Qualitative Results | 2:00 | 정성 결과 사례 |
| 17 | Strengths and Weaknesses | 2:30 | 논문 자체의 장점과 약점 |
| 18 | Why I Reviewed This Paper | 2:30 | 이 논문을 검토한 이유 |

---

# 슬라이드별 상세 구성

## 1. Title

### 슬라이드 제목

**BGI-YOLO: Background Image-Assisted Object Detection for Stationary Cameras**

### 들어갈 내용

- 발표자: 박천웅
- 소속: IRV_lab
- 발표일: 2026년 7월 8일

### 이미지 자료

- 사용하지 않거나, 논문 첫 페이지의 제목부 일부를 작게 배치
- 깔끔하게 가려면 텍스트만 두는 편이 좋음

### 발표 포인트

- 논문 제목을 그대로 발표 제목으로 사용한다.
- 첫 장은 정보 전달용으로 간결하게 구성한다.

---

## 2. 목차

### 슬라이드 제목

**목차**

### 들어갈 내용

1. Problem Definition  
2. Proposed Method  
3. Experimental Setup  
4. Experimental Results  
5. Discussion  
6. Why I Reviewed This Paper

### 이미지 자료

- 없음
- 목차 슬라이드는 텍스트 중심으로 간단히 구성

### 발표 포인트

> 오늘 발표는 BGI-YOLO가 어떤 문제를 지적했고, 이를 어떻게 해결했으며, 실험 결과가 실제로 그 주장을 뒷받침하는지를 중심으로 구성했습니다.

---

## 3. Background: Object Detection in Stationary Cameras

### 슬라이드 역할

정지 카메라 환경에서 객체 검출이 왜 중요한지 설명하는 배경 슬라이드이다.

### 들어갈 내용

- 객체 검출은 교통 감시, 범죄 예방, 이상행동 탐지 등 정지 카메라 환경에서 많이 사용됨
- 일반 DNN 기반 object detector는 다양한 배경에서 일반화되도록 설계됨
- 하지만 정지 카메라에서는 배경이 반복적으로 등장함
- 논문은 이 “고정 배경”을 검출 성능 향상에 활용할 수 있다고 봄

### 이미지 자료

- **Figure 5. Example images of the WITHROBOT S1 dataset**  
  또는  
- **Figure 6. Example images of LLVIP dataset**

### 이미지 배치

- 왼쪽: Figure 5 또는 Figure 6
- 오른쪽: 정지 카메라 환경의 특징 3개
  - Fixed viewpoint
  - Repeated background
  - Moving foreground objects

### 중복 방지

- Figure 5와 Figure 6을 모두 여기서 사용하지 않는다.
- 하나는 이 슬라이드에서 사용하고, 다른 하나는 Dataset 슬라이드에서 사용한다.

### 발표 포인트

> 정지 카메라 환경에서는 카메라의 시점이 고정되어 있고, 같은 배경이 반복적으로 등장합니다. 일반 detector는 이 배경을 명시적으로 활용하지 않지만, 이 논문은 바로 이 점을 추가 정보로 사용할 수 있다고 봅니다.

---

## 4. Problem 1: General Object Detectors Ignore Fixed Backgrounds

### 슬라이드 역할

일반 객체 검출기가 정지 카메라의 고정 배경 정보를 충분히 활용하지 못한다는 문제를 제시한다.

### 들어갈 내용

- 일반 detector는 현재 이미지 한 장만 입력으로 사용
- 정지 카메라 환경에서는 배경이 거의 고정되어 있음
- 하지만 기존 detector는 이 배경 정보를 명시적으로 활용하지 않음
- 따라서 정지 카메라의 환경적 이점을 버리고 있음

### 이미지 자료

- 논문 figure 사용 없음 권장

### 대신 사용할 도식

```text
Conventional YOLO
Current Image  →  Detector  →  Boxes

Stationary Camera Context
Current Image + Stable Background
```

### 이미지/도식 배치

- 상단: Conventional YOLO 구조
- 하단: Stationary camera context 구조
- 오른쪽: “고정 배경이 있는데도 사용하지 않는다”는 핵심 문장

### 발표 포인트

> 일반 YOLO 계열 detector는 현재 프레임만 보고 객체를 검출합니다. 하지만 정지 카메라 환경에서는 배경이 거의 고정되어 있기 때문에, 현재 프레임과 배경의 차이 자체가 중요한 단서가 될 수 있습니다.

---

## 5. Problem 2: Limits of Background Subtraction-Based Detection

### 슬라이드 역할

기존 background subtraction 기반 방식의 한계를 설명한다.

### 들어갈 내용

- 기존 cascade 방식:
  1. Background subtraction
  2. Foreground mask 생성
  3. Masked image에서 object detection
- 문제는 foreground mask 품질에 전체 성능이 의존한다는 점
- 조명 변화, 그림자, 날씨, 복잡한 배경에서 mask가 흔들릴 수 있음
- 파라미터 수동 조정도 필요함

### 이미지 자료

- **Figure 10. Limitation of background subtraction**

### 이미지 배치

- 상단: Figure 10 전체 또는 주요 부분 crop
- 하단: 실패 원인 요약
  - Illumination change
  - Shadow
  - Headlight
  - Binarization failure

### 중복 방지

- Figure 10은 여기서만 사용한다.
- 뒤에서 BS-OD 설명에는 Figure 9를 사용한다.

### 발표 포인트

> Background subtraction은 현재 이미지와 배경 이미지의 차이를 통해 foreground를 추출합니다. 하지만 조명 변화나 그림자처럼 객체가 아닌 차이도 크게 발생하기 때문에, foreground mask가 잘못 만들어질 수 있습니다. 이 경우 뒤의 detector가 아무리 좋아도 입력 자체가 오염됩니다.

---

## 6. Main Idea: Background Image-Assisted Detection

### 슬라이드 역할

BGI-YOLO의 핵심 아이디어를 처음 제시한다.

### 들어갈 내용

- BGI-YOLO는 foreground mask를 직접 만들지 않음
- 현재 이미지와 background image를 함께 입력
- YOLO가 두 이미지 사이의 차이를 end-to-end로 학습
- 배경 정보를 후처리 규칙이 아니라 detector 내부의 정보로 사용

### 이미지 자료

- **Figure 2. Image-level fusion framework for BGI-YOLO**

### 이미지 배치

- 중앙 크게: Figure 2
- 우측 또는 하단 강조 문구:

```text
Input image + Background image → YOLOv7
```

### 중복 방지

- Figure 2는 이 슬라이드에서만 사용한다.
- Slide 8에서는 Figure 2를 다시 쓰지 않고, 별도의 간단 도식으로 설명한다.

### 발표 포인트

> BGI-YOLO의 핵심은 배경 차분 결과를 hard mask로 만들지 않는다는 점입니다. 대신 현재 이미지와 배경 이미지를 함께 detector에 넣고, 네트워크가 두 이미지의 관계를 직접 학습하게 합니다.

---

## 7. Background Image Generation

### 슬라이드 역할

BGI-YOLO에서 사용하는 background image가 어떻게 생성되는지 설명한다.

### 들어갈 내용

- 여러 과거 프레임을 사용
- 객체가 적고 작은 프레임을 선택
- 각 pixel 위치에서 median value 계산
- background image는 학습 전에 한 번 생성됨
- mean보다 median이 작은 움직임이나 객체에 덜 민감하고 선명한 배경을 생성한다고 설명함

### 이미지 자료

- **Figure 4. Background generation using median filtering**
- 보조: **Figure 8. Example background image generated using the median filter method**

### 이미지 배치

- 왼쪽: Figure 4
- 오른쪽: Figure 8에서 WITHROBOT S1 또는 LLVIP 배경 예시 하나만 crop

### 중복 방지

- Figure 4는 과정 설명용으로 여기서만 사용한다.
- Figure 8은 결과 예시용으로 여기서만 사용한다.

### 발표 포인트

> 논문은 여러 프레임을 이용해 pixel-wise median 방식으로 background image를 생성합니다. 평균값을 쓰면 움직이는 객체나 작은 변화가 배경에 섞일 수 있는데, median 방식은 이런 영향이 상대적으로 작다고 설명합니다.

---

## 8. BGI-YOLO(1): Image-Level Fusion

### 슬라이드 역할

논문에서 가장 좋은 성능을 보인 BGI-YOLO(1)을 설명한다.

### 들어갈 내용

- 현재 이미지와 배경 이미지를 channel axis 방향으로 결합
- YOLOv7 입력 채널 수를 확장
- 가장 단순한 fusion 방식
- 정보 손실이 적음
- 실험 결과에서 가장 좋은 성능을 보임

### 이미지 자료

- Figure 2 재사용하지 않음 권장
- 직접 만든 채널 결합 도식 사용

### 대신 사용할 도식

```text
Input image:       RGB, 3ch
Background image:  RGB or Gray
Fusion:            Channel-wise concatenation
Detector:          YOLOv7
```

또는

```text
[Current RGB] + [Background Gray] → [4-channel input] → YOLOv7
```

### 이미지 배치

- 왼쪽: 채널 결합 도식
- 오른쪽: Image-level fusion의 특징
  - Simple
  - Low cost
  - Best mAP in experiments

### 발표 포인트

> BGI-YOLO(1)은 현재 이미지와 배경 이미지를 입력 단계에서 단순히 결합합니다. 구조는 가장 단순하지만, 실험에서는 이 방식이 가장 좋은 성능을 보였습니다.

---

## 9. BGI-YOLO(2)(3): Feature-Level Fusion

### 슬라이드 역할

feature-level fusion 방식인 BGI-YOLO(2), BGI-YOLO(3)을 설명한다.

### 들어갈 내용

- current image와 background image를 각각 backbone에 통과
- P3, P4, P5 feature map을 같은 level끼리 결합
- BGI-YOLO(2): concatenate
- BGI-YOLO(3): add
- dual backbone 구조라 연산량 증가

### 이미지 자료

- **Figure 3. Feature-level fusion framework for BGI-YOLO**

### 이미지 배치

- 중앙: Figure 3
- 오른쪽: 비교 박스

```text
BGI-YOLO(2): Concat
BGI-YOLO(3): Add
Cost: Dual backbone
```

### 중복 방지

- Figure 3은 여기서만 사용한다.

### 발표 포인트

> BGI-YOLO(2)와 BGI-YOLO(3)은 입력 단계가 아니라 feature 단계에서 현재 이미지와 배경 이미지의 정보를 결합합니다. 하지만 두 이미지를 각각 backbone에 통과시키기 때문에 연산량이 크게 증가합니다.

---

## 10. Compared Methods

### 슬라이드 역할

실험에서 비교한 방법들을 정리한다.

### 들어갈 내용

비교 대상 5종:

1. Baseline YOLOv7
2. BS-OD
3. BGI-YOLO(1): Image-level fusion
4. BGI-YOLO(2): Feature-level fusion + concat
5. BGI-YOLO(3): Feature-level fusion + add

추가 비교:

- RGB background
- Gray background

### 이미지 자료

- **Figure 9. Object detection process based on background subtraction (BS-OD)**

### 이미지 배치

- 왼쪽: Figure 9
- 오른쪽: 비교군 목록
- 하단 강조:

```text
BGI-YOLO does not generate a hard foreground mask.
```

### 발표 포인트

> 논문은 baseline YOLOv7뿐 아니라, background subtraction을 이용한 BS-OD도 비교군으로 사용합니다. BS-OD는 foreground mask를 만든 뒤 YOLOv7을 적용하는 방식이고, BGI-YOLO는 이 mask 생성 단계를 detector 내부 학습으로 대체하려는 방향입니다.

---

## 11. Dataset Configuration

### 슬라이드 역할

실험 데이터셋의 구성과 성격을 설명한다.

### 들어갈 내용

### WITHROBOT S1

- 12,194 images
- 18 scenes
- vehicle, pedestrian, kickboard, cyclist
- day/night, weather variation 포함

### LLVIP

- visible image만 사용
- pedestrian class 중심
- 대부분 야간 이미지

### 이미지 자료

- **Table 1. Train/test data split**
- **Figure 5 또는 Figure 6 중 Slide 3에서 쓰지 않은 것**

### 이미지 배치

- 왼쪽: Table 1
- 오른쪽: Figure 5 또는 Figure 6
- 하단: dataset 차이 요약

### 중복 방지

- Slide 3에서 Figure 5를 사용했다면 여기서는 Figure 6을 사용한다.
- Slide 3에서 Figure 6을 사용했다면 여기서는 Figure 5를 사용한다.

### 발표 포인트

> WITHROBOT S1은 차량, 보행자, 킥보드, 자전거 탑승자 등 여러 class를 포함하는 정지 카메라 데이터셋입니다. LLVIP는 visible image만 사용했고, 주로 야간 보행자 검출 성능을 확인하는 데 사용되었습니다.

---

## 12. Implementation Details

### 슬라이드 역할

학습 및 평가 조건을 간단히 정리한다.

### 들어갈 내용

- YOLOv7 COCO pre-trained weight에서 재학습
- input image size: 416 × 416
- learning rate: 0.001
- batch size: 16
- WITHROBOT S1: 50 epochs
- LLVIP: 100 epochs
- metric: mAP
- framework: PyTorch 1.12.1

### 이미지 자료

- Figure/Table 사용하지 않음 권장

### 대신 사용할 구성

실험 세팅 카드 4개:

```text
Model: YOLOv7
Input: 416×416
Training: 50 / 100 epochs
Metric: mAP
```

### 이미지 배치

- 카드형 요약 4개 또는 6개
- 표를 직접 만들어도 좋음

### 발표 포인트

> 실험은 COCO로 pre-trained된 YOLOv7을 기반으로 재학습하는 방식입니다. 따라서 이 논문은 YOLO를 그대로 쓰는 방식이 아니라, 입력 구조를 바꾸고 다시 학습하는 detector 개선 논문으로 봐야 합니다.

---

## 13. Quantitative Results 1: WITHROBOT S1

### 슬라이드 역할

WITHROBOT S1 데이터셋에서의 정량 결과를 설명한다.

### 들어갈 내용

- YOLOv7: 76.7 mAP
- BS-OD: 64.5 mAP
- BGI-YOLO(1)-Gray: 82.3 mAP
- baseline 대비 +5.6%p
- BS-OD 대비 +17.8%p
- GFLOPs: 103.2 → 103.4로 거의 증가 없음

### 이미지 자료

- **Table 2. Detection performance comparisons using the WITHROBOT S1 dataset**

### 이미지 배치

- 중앙: Table 2
- 강조 표시:
  - YOLOv7 76.7
  - BS-OD 64.5
  - BGI-YOLO(1)-Gray 82.3
  - GFLOPs 103.4

### 중복 방지

- Table 2는 여기서만 사용한다.

### 발표 포인트

> WITHROBOT S1에서 가장 좋은 결과는 BGI-YOLO(1)-Gray입니다. mAP는 YOLOv7 대비 5.6%p 높고, GFLOPs는 거의 증가하지 않았습니다. 반면 BS-OD는 baseline보다 낮아, 단순 background subtraction이 항상 도움이 되지는 않음을 보여줍니다.

---

## 14. Quantitative Results 2: LLVIP

### 슬라이드 역할

LLVIP 데이터셋에서의 정량 결과를 설명한다.

### 들어갈 내용

- YOLOv7: 91.4 mAP
- BS-OD: 84.8 mAP
- BGI-YOLO(1)-Gray: 93.9 mAP
- baseline 대비 +2.5%p
- BS-OD 대비 +9.1%p
- WITHROBOT S1과 유사한 경향

### 이미지 자료

- **Table 3. Detection performance comparisons using the LLVIP dataset**

### 이미지 배치

- 중앙: Table 3
- 오른쪽 또는 하단 요약:

```text
Same trend as WITHROBOT S1
BGI-YOLO(1)-Gray > YOLOv7 > BS-OD
```

### 중복 방지

- Table 3은 여기서만 사용한다.

### 발표 포인트

> LLVIP에서도 같은 경향이 나타납니다. BGI-YOLO(1)-Gray가 baseline YOLOv7보다 높고, BS-OD는 오히려 낮습니다. 즉, 단순 foreground mask보다 배경 이미지를 detector 입력으로 주는 방식이 더 안정적이었다고 볼 수 있습니다.

---

## 15. Precision & Recall Analysis

### 슬라이드 역할

mAP 외에 precision과 recall 관점에서 결과를 해석한다.

### 들어갈 내용

- mAP뿐 아니라 precision/recall도 개선됨
- WITHROBOT S1:
  - Precision: 81.4 → 82.7
  - Recall: 70.4 → 77.5
- LLVIP:
  - Precision: 89.0 → 93.5
  - Recall: 87.2 → 89.0
- 단순히 false positive만 줄인 것이 아니라 missed detection도 줄인 것으로 해석 가능

### 이미지 자료

- **Table 4. Precision/Recall WITHROBOT S1**
- **Table 5. Precision/Recall LLVIP**

### 이미지 배치

- 좌측: Table 4
- 우측: Table 5
- 하단 강조:

```text
Precision and Recall both improved.
```

### 중복 방지

- Table 4, Table 5는 여기서만 사용한다.

### 발표 포인트

> BGI-YOLO(1)은 mAP뿐 아니라 precision과 recall도 함께 개선했습니다. 이는 단순히 더 보수적으로 잡아서 false positive를 줄인 것이 아니라, 놓치는 객체도 줄였다는 방향으로 해석할 수 있습니다.

---

## 16. Qualitative Results

### 슬라이드 역할

정성적 검출 결과를 통해 실제 개선 사례를 보여준다.

### 들어갈 내용

- 정량 결과 외에 실제 detection 예시 확인
- WITHROBOT S1에서는 작은 보행자, 가려진 차량 등에서 개선
- LLVIP에서는 야간 보행자 검출 개선
- BGI-YOLO(1)이 현재 프레임만 보는 YOLOv7보다 배경 정보를 활용해 놓친 객체를 더 잘 잡는 사례 제시

### 이미지 자료

- **Figure 11. Qualitative results using the WITHROBOT S1 dataset**  
  또는  
- **Figure 12. Qualitative results using the LLVIP dataset**

### 추천

- 본문에서는 **Figure 11** 사용 권장
- Figure 11은 class가 다양해서 설명하기 좋음
- Figure 12는 backup 또는 appendix 성격으로 보관

### 이미지 배치

- 중앙 크게: Figure 11
- 오른쪽: 개선 사례 3개
  - Small object
  - Occluded object
  - Dark area object

### 중복 방지

- Figure 11 또는 Figure 12 중 하나만 사용한다.
- 둘 다 넣으면 슬라이드가 산만해질 수 있다.

### 발표 포인트

> 정성 결과에서는 작은 객체나 일부 가려진 객체에서 BGI-YOLO가 baseline보다 더 잘 검출하는 사례를 보여줍니다. 이 결과는 배경 정보가 단순 수치상 성능뿐 아니라 실제 검출 사례에서도 의미가 있음을 보여주는 근거로 볼 수 있습니다.

---

## 17. Strengths and Weaknesses

### 슬라이드 역할

논문 자체의 장점과 객관적 약점을 정리한다.

### 들어갈 내용

### Strengths

- 정지 카메라라는 환경 조건을 명확히 활용함
- 구조가 단순함
- image-level fusion은 연산량 증가가 거의 없음
- background subtraction의 hard mask 문제를 피함
- mAP, precision, recall에서 모두 개선 확인

### Weaknesses

- stationary camera assumption이 강함
- background image 품질에 의존함
- background image를 한 번 생성하는 방식이라 장기적 배경 변화 대응은 제한적일 수 있음
- 최신 YOLO 계열이나 transformer detector와의 비교는 부족함
- feature-level fusion은 구조 대비 성능과 효율이 좋지 않음
- 실제 deployment 지표, 예를 들어 latency, false alarm, 장기 운용 안정성 분석은 제한적임

### 이미지 자료

- 이미지 없음 권장
- 또는 직접 만든 요약 표 사용

### 이미지 배치

| Strengths | Weaknesses |
|---|---|
| Simple structure | Stationary camera assumption |
| Low extra computation | Background quality dependency |
| Avoids hard foreground mask | Limited long-term robustness |
| Improves mAP/precision/recall | Limited comparison with newer detectors |

### 발표 포인트

> 이 논문의 장점은 명확합니다. 정지 카메라라는 조건을 잘 활용했고, 특히 image-level fusion은 단순하면서도 연산 증가가 거의 없습니다. 다만 이 방식은 정지 카메라와 안정적인 배경 이미지에 강하게 의존하며, 장기 운용 환경이나 최신 detector와의 비교는 충분히 다뤄지지 않았습니다.

---

## 18. Why I Reviewed This Paper

### 슬라이드 제목

**Why I Reviewed This Paper**  
또는  
**이 논문을 검토한 이유**

### 슬라이드 역할

발표 초반에 생략한 개인 연구와의 관계를 결말에서 회수한다.

### 들어갈 내용

- 이 논문은 “정지 카메라에서 배경 정보가 유효한가?”라는 질문을 직접 다룸
- 단순한 background subtraction이 아니라 detector 입력 단계에서 배경 정보를 활용함
- 다만 이 논문은 YOLO 구조 변경과 재학습을 전제로 함
- 따라서 비학습적 현장 적응이나 이벤트 판정 문제의 직접 해결책은 아님
- 의미는 “직접 차용할 방법론”보다 “비교 기준이 되는 학습 기반 접근”에 있음

### 이미지 자료

- 이미지 없음 권장
- 직접 만든 2열 비교 표 사용

### 이미지 배치

| BGI-YOLO | My Direction |
|---|---|
| YOLO 재학습 | YOLO 이후 비학습 보정 |
| 객체 검출 성능 향상 | 이벤트 후보 판정 |
| background image를 입력으로 사용 | background 변화/차이를 규칙 판단에 사용 |
| detector 개선 | detector 결과 해석 |

### 발표 포인트

> 이 논문을 검토한 이유는 BGI-YOLO가 제 연구의 직접적인 방법론이기 때문은 아닙니다. 오히려 정지 카메라 환경에서 배경 정보를 학습 기반 detector에 통합한 대표적인 접근이기 때문입니다. 이 논문은 배경 정보가 객체 검출에 유효할 수 있음을 보여주지만, YOLO 구조 변경과 재학습을 전제로 합니다. 따라서 제 관심사인 비학습적 현장 적응이나 이벤트 판정과는 해결 방식이 다릅니다. 이 점에서 BGI-YOLO는 제 연구의 기반이라기보다, 비교하고 구분해야 할 관련 연구로 볼 수 있습니다.

---

# 논문 Figure/Table 분배 요약

| 자료 | 사용할 슬라이드 | 용도 | 사용 여부 |
|---|---:|---|---|
| Figure 1. YOLOv7 Architecture | 사용하지 않음 | YOLOv7 구조 설명 | 생략 권장 |
| Figure 2. Image-level fusion | 6 | BGI-YOLO 핵심 아이디어 | 사용 |
| Figure 3. Feature-level fusion | 9 | BGI-YOLO(2)(3) 설명 | 사용 |
| Figure 4. Median filtering background generation | 7 | 배경 이미지 생성 과정 | 사용 |
| Figure 5. WITHROBOT S1 examples | 3 또는 11 | 정지 카메라 환경/데이터셋 예시 | 선택 사용 |
| Figure 6. LLVIP examples | 3 또는 11 | 야간/조명 변화 데이터셋 예시 | 선택 사용 |
| Figure 7. Object class samples | 사용하지 않음 | class sample 설명 | 생략 가능 |
| Figure 8. Generated background image | 7 | median filter 결과 예시 | 사용 |
| Figure 9. BS-OD process | 10 | 비교군 BS-OD 설명 | 사용 |
| Figure 10. Background subtraction limitation | 5 | 기존 방식의 실패 원인 | 사용 |
| Figure 11. WITHROBOT qualitative results | 16 | 정성 결과 | 사용 권장 |
| Figure 12. LLVIP qualitative results | 예비 자료 | 정성 결과 보조 | 생략 또는 backup |
| Table 1. Train/test split | 11 | 데이터셋 구성 | 사용 |
| Table 2. WITHROBOT results | 13 | 정량 결과 1 | 사용 |
| Table 3. LLVIP results | 14 | 정량 결과 2 | 사용 |
| Table 4. Precision/Recall WITHROBOT | 15 | 세부 지표 | 사용 |
| Table 5. Precision/Recall LLVIP | 15 | 세부 지표 | 사용 |

---

# 최종 권장 자료 배치

## 본문에 넣을 자료

- Figure 2
- Figure 3
- Figure 4
- Figure 5 또는 Figure 6
- Figure 8
- Figure 9
- Figure 10
- Figure 11
- Table 1
- Table 2
- Table 3
- Table 4
- Table 5

## 생략 권장 자료

- Figure 1: YOLOv7 구조 설명이 길어짐
- Figure 7: class samples는 중요도가 낮음
- Figure 12: Figure 11과 역할이 겹치므로 본문에서는 생략 권장

---

# 발표 전체 결론 문장

> BGI-YOLO는 정지 카메라에서 배경 정보가 객체 검출 성능 향상에 유효하다는 점을 보여준다. 그러나 이 방식은 YOLO 구조 변경과 재학습을 전제로 하므로, 비학습적 현장 적응이나 이벤트 판정 문제를 직접 해결하지는 않는다. 따라서 이 논문은 직접 차용할 방법론이라기보다, 학습 기반 배경 활용 방식과 비학습적 이벤트 판정 접근을 구분하기 위한 관련 연구로 검토할 가치가 있다.
