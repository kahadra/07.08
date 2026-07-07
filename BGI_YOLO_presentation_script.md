# BGI-YOLO 세미나 발표 대본

- 발표 자료: `index.html`
- 기반 논문: `electronics-14-00060-v3.pdf`
- 목표 발표 시간: 30분
- 작성 원칙: 슬라이드에 있는 메시지는 빠짐없이 말하되, 보강 설명은 PDF 본문, 표, 그림, 결론에서 확인되는 내용만 사용한다.

## 시간 배분

| 슬라이드 | 제목 | 목표 시간 |
|---:|---|---:|
| 1 | Title | 0:40 |
| 2 | 발표 흐름 | 0:50 |
| 3 | Object Detection in Stationary Cameras | 1:40 |
| 4 | Problem 1: General Detectors Ignore Fixed Backgrounds | 1:50 |
| 5 | Problem 2: Limits of Background Subtraction-Based Detection | 2:00 |
| 6 | Background Image-Assisted Detection | 1:40 |
| 7 | Background Image Generation | 2:00 |
| 8 | Image-Level Fusion | 1:40 |
| 9 | Feature-Level Fusion | 1:50 |
| 10 | Compared Methods | 1:30 |
| 11 | Dataset Configuration | 1:50 |
| 12 | Implementation Details | 1:30 |
| 13 | Quantitative Results 1: WITHROBOT S1 | 1:50 |
| 14 | Quantitative Results 2: LLVIP | 1:40 |
| 15 | Precision & Recall Analysis | 1:40 |
| 16 | Qualitative Results | 1:50 |
| 17 | Strengths and Weaknesses | 2:00 |
| 18 | Why I Reviewed This Paper | 2:00 |
| 합계 |  | 30:00 |

---

## Slide 1. Title

**목표 시간: 0:40**

**PDF 근거:** 논문 제목, 저자, 게재 정보, 초록.

**발표 대본**

안녕하세요. 오늘 발표할 논문은 *BGI-YOLO: Background Image-Assisted Object Detection for Stationary Cameras*입니다.

이 논문은 2025년 *Electronics*에 게재된 논문이고, 핵심 주제는 정지 카메라 환경에서 반복적으로 등장하는 배경 이미지를 객체 검출 네트워크의 입력으로 함께 사용해 성능을 높일 수 있는가입니다.

보통 YOLO 계열 detector는 현재 프레임 한 장만 입력으로 받아 객체를 찾습니다. 그런데 CCTV나 도로 감시 카메라처럼 카메라가 고정되어 있는 경우에는 배경이 시간에 따라 크게 바뀌지 않습니다. 논문은 이 고정 배경 자체가 검출 성능을 높이는 추가 정보가 될 수 있다고 보고, 이를 YOLOv7 구조에 통합한 BGI-YOLO를 제안합니다.

오늘 발표에서는 문제 정의, 제안 방법, 실험 조건, 결과, 그리고 이 논문을 제가 왜 검토했는지 순서로 설명하겠습니다.

---

## Slide 2. 발표 흐름

**목표 시간: 0:50**

**PDF 근거:** 논문 구성은 Introduction, Related Works, Proposed Method, Experimental Results, Conclusions and Future Works 순서.

**발표 대본**

발표 흐름은 여섯 부분입니다.

먼저 이 논문이 출발하는 문제를 보겠습니다. 정지 카메라에서는 배경이 반복되는데, 일반 detector는 이 정보를 명시적으로 쓰지 않는다는 문제입니다. 다음으로 기존 background subtraction 기반 방식이 왜 충분하지 않은지 설명하겠습니다.

그 다음에는 제안 방법입니다. BGI-YOLO는 배경 이미지를 hard foreground mask로 만들지 않고, 현재 이미지와 함께 detector 입력 또는 중간 feature 단계에서 결합합니다. 논문에서는 image-level fusion과 feature-level fusion을 비교합니다.

이후 실험 세팅에서는 WITHROBOT S1과 LLVIP 데이터셋, 학습 조건, 비교 방법을 정리합니다. 결과 부분에서는 mAP, GFLOPs, precision, recall, 정성 결과를 순서대로 확인합니다.

마지막으로 장점과 한계를 정리하고, 이 논문이 제 관심 방향과 어떤 관계가 있는지, 즉 직접 차용할 방법론인지 아니면 비교 기준으로 봐야 하는지를 정리하겠습니다.

---

## Slide 3. Object Detection in Stationary Cameras

**목표 시간: 1:40**

**PDF 근거:** Introduction; Figure 5; WITHROBOT S1 데이터셋 설명.

**발표 대본**

먼저 배경입니다. 이 슬라이드는 정지 카메라에서 객체 검출이 왜 중요한지 보여줍니다. 논문은 정지 감시 카메라가 공공장소, 교통 경로, 상업 시설 등 다양한 환경에서 빠르게 늘고 있고, 그 안에서 보행자나 차량 같은 동적 객체를 검출하고 추적하는 기술이 교통 제어, 범죄 예방, 이상행동 탐지에 중요하다고 설명합니다.

왼쪽의 Figure 5는 WITHROBOT S1 데이터셋 예시입니다. 이 데이터셋은 학교 주변 도로나 주거지 도로에 설치된 정지 감시 카메라에서 수집된 이미지로, 낮과 밤, 날씨 변화, 도로 환경이 포함되어 있습니다.

중요한 점은 정지 카메라에서는 시점이 고정되어 있다는 것입니다. 카메라가 움직이지 않으므로 배경은 비교적 반복적으로 등장합니다. 반대로 보행자, 차량, 킥보드, 자전거 탑승자 같은 foreground 객체는 시간에 따라 바뀝니다.

일반적인 딥러닝 기반 object detector는 다양한 배경에서 일반화되도록 학습됩니다. COCO 같은 대규모 benchmark 데이터셋도 여러 물체와 다양한 배경을 포함합니다. 이 자체는 범용 detector로서는 장점입니다. 하지만 정지 카메라에 적용할 때는, 해당 카메라에서 반복되는 고정 배경이라는 환경적 이점을 적극적으로 활용하지 못한다는 문제가 생깁니다.

이 논문은 바로 그 지점에서 출발합니다. 정지 카메라라면 현재 프레임만 볼 것이 아니라, 그 장면의 배경 이미지까지 같이 보면 객체 검출에 도움이 될 수 있다는 것입니다.

---

## Slide 4. Problem 1: General Detectors Ignore Fixed Backgrounds

**목표 시간: 1:50**

**PDF 근거:** Introduction; 일반 detector가 benchmark dataset 기반으로 일반화 중심 개발되었고 stationary camera의 background information을 활용하지 못한다는 설명.

**발표 대본**

첫 번째 문제는 일반 detector가 고정 배경을 무시한다는 점입니다.

일반 YOLO 계열 detector는 현재 이미지 한 장을 입력으로 받고, 그 안에서 bounding box와 class를 예측합니다. 이 구조는 매우 범용적입니다. 공원, 방, 도로, 주방처럼 배경이 계속 바뀌는 이미지에서도 작동해야 하기 때문에, 특정 카메라의 고정 배경을 모델 입력으로 가정하지 않습니다.

그런데 정지 카메라 환경에서는 상황이 다릅니다. 배경은 상대적으로 일정하고, 동적 객체만 프레임마다 바뀝니다. 이 경우 현재 이미지와 배경 이미지 사이의 차이는 단순한 노이즈가 아니라 객체 검출에 직접적으로 연결되는 단서가 됩니다.

예를 들어 같은 도로 장면에서 평소에는 비어 있던 영역에 보행자가 나타났다면, 현재 이미지와 배경 이미지의 차이는 그 보행자의 위치를 암시합니다. 하지만 일반 detector는 이 배경 이미지를 별도 입력으로 받지 않기 때문에, 이 차이를 명시적으로 사용할 수 없습니다.

따라서 논문이 보는 문제는 detector 성능 자체가 낮다는 것이 아니라, 정지 카메라라는 조건에서 사용할 수 있는 추가 정보를 기존 detector가 활용하지 않는다는 점입니다. 논문은 이 조건을 버리지 말고, detector 입력 구조 안으로 가져오자고 제안합니다.

---

## Slide 5. Problem 2: Limits of Background Subtraction-Based Detection

**목표 시간: 2:00**

**PDF 근거:** Related Works; Figure 10; background subtraction 기반 cascade 방식의 한계와 foreground mask 품질 의존성.

**발표 대본**

두 번째 문제는 기존 background subtraction 기반 방식의 한계입니다.

정지 카메라에서 배경을 활용하는 전통적인 접근은 이미 있었습니다. 보통은 두 단계로 구성됩니다. 먼저 background subtraction을 해서 foreground mask나 ROI를 만들고, 그 다음 그 영역에 대해 object classification 또는 detection을 수행합니다. 논문에서는 이런 cascade 방식의 선행 연구들을 정리합니다.

문제는 전체 성능이 첫 번째 단계인 foreground segmentation 품질에 크게 의존한다는 점입니다. 배경 차분이 잘못되면 뒤의 detector가 보는 입력도 잘못됩니다. 특히 background subtraction에는 binary threshold, morphology 같은 후처리 파라미터가 들어가고, 이 파라미터는 환경마다 맞추기 어렵습니다.

왼쪽의 Figure 10이 이 문제를 보여줍니다. WITHROBOT S1 사례에서는 건물 벽의 밝은 영역과 그림자 영역이 배경과 크게 달라져서, 실제 이동 객체보다 조명 변화가 더 큰 차이로 나타납니다. 이 경우 difference image를 이진화할 때 보행자나 자전거 탑승자 영역이 제대로 foreground로 분리되지 않을 수 있습니다.

LLVIP 사례에서는 가로등이 없는 도로 영역이 차량 헤드라이트 때문에 밝아지면서 배경과 큰 차이를 만듭니다. 논문은 이 밝기 차이가 보행자 영역보다 더 강하게 나타날 수 있고, 그 결과 pedestrian 영역을 foreground로 정확히 잡는 데 실패할 수 있다고 설명합니다.

정리하면 background subtraction은 배경 정보를 쓰긴 하지만, hard mask를 먼저 만든다는 점 때문에 조명, 그림자, 헤드라이트, 복잡한 배경 변화에 취약합니다. BGI-YOLO는 이 hard mask 단계를 detector 내부 학습으로 대체하려는 접근입니다.

---

## Slide 6. Background Image-Assisted Detection

**목표 시간: 1:40**

**PDF 근거:** Proposed Method 3.2; Figure 2; end-to-end 방식과 image-level fusion 설명.

**발표 대본**

이제 핵심 아이디어입니다. BGI-YOLO는 foreground mask를 직접 만들지 않고, 현재 이미지와 background image를 함께 detector에 넣습니다.

Figure 2는 image-level fusion 구조를 보여줍니다. 현재 프레임과 배경 이미지를 channel axis 방향으로 결합한 뒤 YOLOv7에 입력합니다. 이 방식에서는 배경 차분 결과를 이진 mask로 만들지 않습니다. 대신 네트워크가 현재 이미지와 배경 이미지 사이의 관계를 end-to-end로 학습합니다.

논문이 강조하는 차이는 cascade 방식과의 차이입니다. 기존 방식은 background subtraction 결과를 먼저 만들고, 그 결과가 잘못되면 detector도 영향을 받습니다. 반면 BGI-YOLO는 배경 정보를 detector 입력의 일부로 제공하기 때문에, 네트워크가 어떤 차이를 객체 단서로 사용할지 학습할 수 있습니다.

이때 baseline detector는 YOLOv7입니다. 논문은 YOLOv7의 입력 단계 또는 중간 단계 구조를 수정해서, 배경 이미지와 현재 이미지가 함께 들어가도록 합니다. 그리고 이 결합 방식을 크게 두 가지, image-level fusion과 feature-level fusion으로 나누어 실험합니다.

슬라이드의 핵심 문장은 이것입니다. 배경 정보를 후처리 규칙으로 쓰는 것이 아니라 detector 내부 정보로 쓰자는 것입니다.

---

## Slide 7. Background Image Generation

**목표 시간: 2:00**

**PDF 근거:** Proposed Method 3.3; Figure 4; Figure 8; Implementation Details의 background selection 조건.

**발표 대본**

다음은 background image를 어떻게 만드는지입니다.

논문은 real-time object detection을 목표로 하기 때문에, 배경 모델링 자체는 가장 단순하고 계산량이 적은 basic background modeling을 사용합니다. 그중에서도 mean filtering이 아니라 median filtering을 선택합니다.

mean 방식은 여러 과거 프레임의 평균값으로 배경을 만들고, median filtering은 각 pixel 위치에서 여러 프레임 값의 중앙값을 선택합니다. 논문은 이미지 안에 객체가 많거나 조명과 배경의 작은 변화가 있을 때, median filtering 결과가 mean보다 더 선명하고 작은 움직임 변화에 덜 민감하다고 설명합니다.

과정은 세 단계로 볼 수 있습니다. 먼저 과거 프레임 중 객체가 적고 작은 이미지들을 선택합니다. 이 선택의 목적은 visible background 영역을 최대화하는 것입니다. 그 다음 각 pixel 위치마다 여러 프레임의 값을 모으고, 마지막으로 그 위치의 median value를 background pixel로 사용합니다.

구현 세부도 중요합니다. WITHROBOT S1에서는 각 scene마다 bounding box 한 변 길이가 0.07보다 작고 이미지 안 객체 수가 4개 미만인 프레임을 모아 median filter를 적용했습니다. LLVIP에서는 전체적으로 객체 크기가 크기 때문에, scene별로 객체 수가 3개 미만인 이미지를 모아 배경을 만들었습니다.

이렇게 생성된 배경 이미지는 학습 전에 한 번 만들어지고, 이후 BGI-YOLO 학습에 사용됩니다. Figure 8은 이렇게 얻은 WITHROBOT S1과 LLVIP 배경 이미지 예시입니다.

---

## Slide 8. Image-Level Fusion

**목표 시간: 1:40**

**PDF 근거:** Proposed Method 3.2; Evaluation Results; Table 2, Table 3; image-level fusion이 가장 좋은 성능과 낮은 비용을 보인다는 결과.

**발표 대본**

BGI-YOLO(1)은 image-level fusion입니다. 슬라이드에 있는 것처럼 가장 단순한 구조이지만, 실험에서는 가장 좋은 성능을 보였습니다.

구조는 현재 이미지와 배경 이미지를 입력 단계에서 channel-wise로 결합하는 방식입니다. 현재 RGB 이미지가 3채널이고 배경을 gray로 사용하면, 네트워크 입력은 현재 이미지 3채널에 gray background 1채널이 추가된 형태로 볼 수 있습니다. RGB background를 쓰면 배경도 3채널로 들어갑니다.

논문은 이 방식을 가장 기본적인 data fusion 형태라고 설명합니다. 원본 데이터를 그대로 결합하기 때문에 정보 손실이 적고, YOLOv7 입력 단계의 채널 수만 수정하면 됩니다. 이후 detector 전체는 각 dataset 조건에서 다시 학습됩니다.

결과적으로 BGI-YOLO(1)-Gray가 WITHROBOT S1과 LLVIP 모두에서 최고 mAP를 보였습니다. WITHROBOT S1에서는 YOLOv7보다 5.6%p 높고, LLVIP에서는 2.5%p 높습니다. GFLOPs도 103.2에서 103.4로 증가해 약 0.19% 증가 수준입니다.

따라서 이 슬라이드의 메시지는 명확합니다. 이 문제에서는 복잡한 결합보다, aligned image 두 장을 입력 단계에서 직접 결합하는 단순한 방식이 가장 효율적이었다는 것입니다.

---

## Slide 9. Feature-Level Fusion

**목표 시간: 1:50**

**PDF 근거:** Proposed Method 3.2; Figure 3; Evaluation Results에서 feature-level fusion의 한계와 GFLOPs 증가 분석.

**발표 대본**

다음은 BGI-YOLO(2)와 BGI-YOLO(3), 즉 feature-level fusion입니다.

이 방식은 현재 이미지와 배경 이미지를 입력에서 바로 붙이지 않습니다. 대신 두 이미지를 각각 backbone에 통과시킨 뒤, YOLOv7의 feature pyramid인 P3, P4, P5 단계에서 같은 level의 feature map을 결합합니다.

BGI-YOLO(2)는 feature map을 concatenate하는 방식이고, BGI-YOLO(3)는 element-wise addition을 사용하는 방식입니다. 논문에서는 이 둘이 heterogeneous data fusion에서 흔히 쓰이는 방식이기 때문에 후보로 검토했다고 설명합니다. 처음에는 image-level fusion보다 더 좋은 결과를 기대할 수 있었던 구조입니다.

하지만 결과는 반대였습니다. BGI-YOLO(1)이 두 데이터셋 모두에서 BGI-YOLO(2), BGI-YOLO(3)보다 좋은 성능을 보였습니다. 논문은 그 이유를 두 가지로 해석합니다.

첫째, YOLOv7 backbone의 capacity가 이 문제의 복잡도보다 충분히 크기 때문에, 입력 단계에서 여러 채널을 함께 보는 convolution만으로도 fusion 역할을 효과적으로 수행했을 가능성이 있습니다. 둘째, 이 문제는 현재 이미지와 배경 이미지의 차이 정보를 적극적으로 쓰는 것이 중요한데, feature-level fusion은 이미지 차이를 직접적으로 활용하지 못하는 한계가 있을 수 있습니다.

또한 비용도 큽니다. dual backbone을 쓰는 BGI-YOLO(2), (3)은 baseline YOLOv7 대비 GFLOPs가 약 64~66% 증가하지만, BGI-YOLO(1)은 약 0.19% 증가에 그칩니다.

---

## Slide 10. Compared Methods

**목표 시간: 1:30**

**PDF 근거:** Evaluation Results; Figure 9; YOLOv7, BS-OD, BGI-YOLO(1)-(3), RGB/Gray background 비교.

**발표 대본**

이 슬라이드는 실험에서 비교한 방법들을 정리합니다.

첫 번째는 baseline YOLOv7입니다. 논문에서는 COCO로 pre-trained된 YOLOv7 weight를 초기값으로 사용하고, 각 데이터셋에서 다시 학습한 결과를 baseline으로 둡니다.

두 번째는 BS-OD입니다. Figure 9처럼 background subtraction으로 foreground mask를 만들고, 그 mask로 input image를 가린 뒤 YOLOv7로 객체를 검출합니다. 배경 이미지는 BGI-YOLO와 동일하게 median filter로 만든 것을 사용합니다. 차분 과정에서는 input image와 background image의 difference image에 adaptive threshold를 적용해 binary image를 만들고, dilation을 반복해서 foreground mask를 얻습니다.

세 번째 그룹은 BGI-YOLO 변형입니다. BGI-YOLO(1)은 image-level fusion, BGI-YOLO(2)는 feature-level concatenate, BGI-YOLO(3)는 feature-level add입니다. 그리고 각 BGI-YOLO 후보에 대해 background image를 RGB 3채널로 넣는 경우와 gray 1채널로 넣는 경우를 모두 비교합니다.

중요한 비교축은 이것입니다. BS-OD는 hard foreground mask를 만들고, BGI-YOLO는 mask를 만들지 않습니다. 배경 정보를 detector가 학습할 입력으로 주는지가 핵심 차이입니다.

---

## Slide 11. Dataset Configuration

**목표 시간: 1:50**

**PDF 근거:** Experimental Results 4.1; Table 1; Figure 5, Figure 6.

**발표 대본**

실험 데이터셋은 두 개입니다.

첫 번째는 WITHROBOT S1입니다. 이 데이터셋은 WITHROBOT Inc.가 학교 근처 도로와 주거지 도로에 설치된 정지 감시 카메라에서 수집한 private dataset입니다. 총 12,194장, 18개 scene으로 구성되어 있고, class는 vehicle, pedestrian, kickboard, cyclist 네 종류입니다.

이 데이터는 2020년 11월부터 2023년 4월까지 긴 기간에 걸쳐 수집되었기 때문에, 낮과 밤, 날씨 변화, 그리고 건물 공사나 새 카메라 설치 같은 느린 배경 변화도 포함합니다. 논문은 도로 안전과 보안을 목표로 하기 때문에, 도로 영역 밖의 객체나 detection zone 밖의 보행자 등은 gray-filled rectangle로 표시해 “don’t care” 영역으로 제외했다고 설명합니다.

두 번째는 LLVIP입니다. LLVIP는 visible-infrared paired dataset이지만, 이 논문에서는 fixed single visible camera 조건만 고려하기 위해 visible image만 사용했습니다. 또한 daytime data는 fixed single camera로 촬영되지 않았기 때문에 제외하고, nighttime data만 사용했습니다. 이 데이터셋은 pedestrian 한 class 중심이고, 차량 헤드라이트로 인한 illumination variation이 큰 것이 특징입니다.

Table 1을 보면 WITHROBOT S1은 train 9,051장, test 3,143장이고, LLVIP는 train 11,600장, test 2,930장입니다.

---

## Slide 12. Implementation Details

**목표 시간: 1:30**

**PDF 근거:** Experimental Results 4.2; 구현 환경, 학습 파라미터, augmentation, metric.

**발표 대본**

구현 조건도 확인하겠습니다.

실험은 Ubuntu 20.04 환경에서 수행되었습니다. 하드웨어는 Intel Core i9-10980XE CPU, NVIDIA GeForce RTX 3090 GPU, 64GB RAM이고, 구현은 PyTorch 1.12.1을 사용했습니다.

모델은 COCO dataset으로 pre-trained된 YOLOv7 weight를 초기값으로 사용해 재학습했습니다. 입력 이미지 크기는 416 x 416이고, learning rate는 0.001, batch size는 16입니다. WITHROBOT S1에서는 50 epochs, LLVIP에서는 100 epochs 학습했습니다.

학습 augmentation으로는 HSV-hue, HSV-saturation, HSV-value augmentation, translation, scale, left-right flip, mosaic, mixup이 사용되었습니다. 평가는 object detection에서 널리 쓰이는 mean Average Precision, 즉 mAP로 수행했습니다.

이 슬라이드에서 중요한 해석은, BGI-YOLO가 단순히 기존 YOLO 결과를 후처리하는 방식이 아니라는 점입니다. 입력 구조를 바꾸고, 그 구조에 맞춰 detector를 다시 학습합니다. 그래서 이 논문은 background-aware input을 갖는 detector 재학습 논문으로 보는 것이 정확합니다.

---

## Slide 13. Quantitative Results 1: WITHROBOT S1

**목표 시간: 1:50**

**PDF 근거:** Table 2; Evaluation Results의 WITHROBOT S1 해석.

**발표 대본**

이제 정량 결과입니다. 먼저 WITHROBOT S1입니다.

Table 2에서 baseline YOLOv7의 mAP는 76.7이고 GFLOPs는 103.2입니다. BS-OD는 같은 GFLOPs 103.2에서 mAP가 64.5로 떨어집니다. 즉, background subtraction을 앞에 붙였다고 해서 성능이 올라가지 않았고, 오히려 foreground mask 품질 문제가 전체 detector 성능을 낮춘 결과로 볼 수 있습니다.

가장 좋은 결과는 BGI-YOLO(1)-Gray입니다. mAP는 82.3이고 GFLOPs는 103.4입니다. baseline YOLOv7 대비 mAP가 5.6%p 높고, BS-OD 대비로는 17.8%p 높습니다. 연산량은 103.2에서 103.4로 거의 증가하지 않았습니다.

반면 feature-level fusion은 성능과 비용 측면에서 모두 불리합니다. BGI-YOLO(2)-Gray는 75.0, BGI-YOLO(3)-Gray는 72.5로, baseline보다도 낮습니다. GFLOPs는 각각 171.9와 169.3으로 크게 증가합니다.

논문의 해석은, 이 문제에서는 image-level fusion이 current image와 background image의 차이를 더 직접적으로 활용할 수 있고, YOLOv7의 convolution이 multi-channel 정보를 효과적으로 결합했을 가능성이 있다는 것입니다.

---

## Slide 14. Quantitative Results 2: LLVIP

**목표 시간: 1:40**

**PDF 근거:** Table 3; Evaluation Results의 LLVIP 해석.

**발표 대본**

다음은 LLVIP 결과입니다.

LLVIP에서도 전체 경향은 WITHROBOT S1과 같습니다. baseline YOLOv7의 mAP는 91.4이고, BS-OD는 84.8입니다. 즉 야간 보행자 데이터셋에서도 background subtraction 기반 방식은 baseline보다 낮은 성능을 보였습니다.

가장 좋은 결과는 역시 BGI-YOLO(1)-Gray입니다. mAP는 93.9이고 GFLOPs는 103.4입니다. baseline YOLOv7보다 2.5%p 높고, BS-OD보다 9.1%p 높습니다.

BGI-YOLO(1)-RGB도 93.8로 거의 비슷하게 높지만, gray background가 93.9로 가장 높습니다. 논문은 전체적으로 gray background image를 사용한 후보들이 RGB background보다 약 1~7%p 정도 더 좋은 경향을 보였다고 설명합니다. 이유는 object detection에 필요한 핵심 정보가 shape를 나타내는 edge information인데, gray image가 이 정보를 충분히 담는 반면 RGB는 color information까지 포함해 edge detection에 간섭할 수 있기 때문이라고 해석합니다.

따라서 LLVIP 결과는 BGI-YOLO의 주장이 특정 private dataset에만 국한되지 않고, 야간 보행자 데이터셋에서도 같은 방향으로 나타난다는 근거가 됩니다.

---

## Slide 15. Precision & Recall Analysis

**목표 시간: 1:40**

**PDF 근거:** Table 4, Table 5; Evaluation Results의 false negative/false positive 해석.

**발표 대본**

mAP만 보면 성능이 오른 것은 확인할 수 있지만, 어떤 방향으로 좋아졌는지는 precision과 recall을 같이 봐야 합니다.

Table 4는 WITHROBOT S1에서 baseline YOLOv7과 BGI-YOLO(1)을 비교합니다. precision은 81.4에서 82.7로 올라가고, recall은 70.4에서 77.5로 올라갑니다. 특히 recall 상승폭이 큽니다. 이는 놓치는 객체, 즉 false negative가 줄어드는 방향으로 해석할 수 있습니다.

Table 5는 LLVIP 결과입니다. precision은 89.0에서 93.5로 올라가고, recall은 87.2에서 89.0으로 올라갑니다. 여기서는 precision 개선이 더 두드러집니다. 즉 false positive도 줄어드는 방향입니다.

논문은 이 결과를 BGI-YOLO(1)이 false negatives, 즉 missing과 false positives, 즉 false detections를 동시에 줄인 것으로 설명합니다. 이 점이 중요합니다. 단순히 threshold를 보수적으로 조정하면 precision은 올라가도 recall은 떨어질 수 있습니다. 반대로 더 많이 잡으려 하면 recall은 올라가도 precision이 떨어질 수 있습니다.

그런데 이 결과에서는 두 지표가 모두 개선됩니다. 따라서 background image를 입력으로 제공한 것이 단순한 검출 기준 변화가 아니라, detector가 객체와 배경의 관계를 더 잘 구분하는 데 도움을 준 것으로 해석할 수 있습니다.

---

## Slide 16. Qualitative Results

**목표 시간: 1:50**

**PDF 근거:** Figure 11, Figure 12; 정성 결과 설명.

**발표 대본**

정성 결과를 보겠습니다. 슬라이드에는 Figure 11, WITHROBOT S1의 예시가 들어가 있습니다.

논문은 Figure 11에서 세 가지 유형의 개선을 설명합니다. 첫 번째 row는 작은 객체입니다. Ground truth에는 cyclist와 pedestrian이 모두 있는데, baseline YOLOv7은 cyclist는 검출하지만 cyclist 뒤의 매우 작은 pedestrian은 놓칩니다. 반면 BGI-YOLO(1)은 cyclist와 pedestrian을 모두 검출합니다.

두 번째 row는 occluded object입니다. Ground truth에는 cyclist 하나와 vehicle 두 대가 있습니다. baseline YOLOv7은 도로 위 cyclist와 vehicle 하나는 검출하지만, 건물에 의해 일부 가려진 vehicle은 놓칩니다. BGI-YOLO(1)은 cyclist와 두 vehicle을 모두 검출합니다.

세 번째 row는 매우 작고 어두운 영역에 있는 객체입니다. 차량과 cyclist는 카메라와 가깝고 가로등 아래에 있어서 비교적 검출이 쉽지만, pedestrian은 카메라에서 멀고, 크기가 작고, 주변 조명이 없는 어두운 영역에 있습니다. baseline YOLOv7은 pedestrian을 놓치지만 BGI-YOLO(1)은 이를 포함해 모든 객체를 검출합니다.

논문은 Figure 12의 LLVIP 결과에서도 BGI-YOLO(1)이 false positive가 적고 pedestrian detection이 더 정확하다고 설명합니다. 따라서 정성 결과는 정량 지표의 개선이 실제 검출 사례에서도 확인된다는 보조 근거입니다.

---

## Slide 17. Strengths and Weaknesses

**목표 시간: 2:00**

**PDF 근거:** Abstract, Evaluation Results, Conclusions and Future Works.

**발표 대본**

이제 논문 자체의 장점과 한계를 정리하겠습니다.

먼저 장점입니다. 첫째, 정지 카메라라는 환경 조건을 명확히 활용합니다. 일반 detector가 놓치는 고정 배경 정보를 detector 입력으로 가져온다는 문제 설정이 분명합니다.

둘째, image-level fusion 구조가 매우 단순합니다. 현재 이미지와 배경 이미지를 channel 방향으로 결합하고 YOLOv7을 재학습합니다. 그럼에도 WITHROBOT S1과 LLVIP에서 모두 baseline보다 높은 mAP를 보였습니다.

셋째, 연산량 증가가 거의 없습니다. BGI-YOLO(1)-Gray는 baseline 103.2 GFLOPs에서 103.4 GFLOPs로 증가해 약 0.19% 증가 수준입니다. 반대로 feature-level fusion은 64~66% 정도 GFLOPs가 증가했지만 성능은 더 낮았습니다.

넷째, background subtraction의 hard mask 문제를 피합니다. Figure 10에서 보듯 조명, 그림자, 헤드라이트가 foreground mask를 왜곡할 수 있는데, BGI-YOLO는 그런 mask를 직접 만들지 않고 detector가 학습하도록 합니다.

한계도 분명합니다. 이 방식은 stationary camera assumption이 강합니다. 논문 결론에서도 non-stationary camera scenario나 moving background scenario로 확장하는 것은 future work로 남깁니다. 또한 배경 이미지를 median filter로 한 번 생성해 사용하므로, sudden lighting changes, dynamic shadows, rain, snow 같은 환경 변화에 대해 더 robust한 background modeling이 필요하다고 논문이 직접 언급합니다.

또 최신 YOLO 버전이나 transformer-based detector에서의 검증도 future work로 제시됩니다. 따라서 현재 논문은 아이디어의 유효성을 보여주는 단계이지, 모든 운용 환경에서 완성된 deployment 검증이라고 보기는 어렵습니다.

---

## Slide 18. Why I Reviewed This Paper

**목표 시간: 2:00**

**PDF 근거:** 논문 전체의 문제 설정과 결론. 개인 연구 방향과의 연결은 슬라이드 내용 범위 안에서만 설명.

**발표 대본**

마지막으로 이 논문을 검토한 이유입니다.

이 논문이 중요한 이유는 “정지 카메라에서 배경 정보가 객체 검출에 유효한가?”라는 질문을 직접 다루기 때문입니다. 결과적으로 논문은 background image를 detector 입력으로 통합하면 YOLOv7 baseline보다 높은 성능을 얻을 수 있고, 특히 image-level fusion은 거의 추가 연산 없이 성능을 개선할 수 있음을 보였습니다.

하지만 제 방향과 비교하면 차이도 분명합니다. BGI-YOLO는 YOLO 구조를 바꾸고 재학습하는 detector 개선 논문입니다. 목표는 객체 검출 성능, 즉 mAP와 precision, recall을 높이는 것입니다. 배경 이미지는 detector 입력으로 들어가고, 네트워크가 이를 학습합니다.

반면 슬라이드 오른쪽의 제 방향은 YOLO 이후의 비학습 보정, 또는 detector 결과 해석에 가깝습니다. 관심은 객체 검출 모델 자체를 다시 학습하는 것보다, 검출 결과와 배경 변화 정보를 이용해 이벤트 후보를 어떻게 판단할지에 있습니다.

따라서 BGI-YOLO는 제가 그대로 가져와 적용할 방법론이라기보다는, 정지 카메라에서 배경 정보를 학습 기반 detector에 통합한 대표적인 비교 기준으로 볼 수 있습니다. 즉 “배경 정보가 유효하다”는 큰 문제의식은 공유하지만, 해결 방식은 다릅니다.

정리하면, BGI-YOLO는 단순 background subtraction보다 안정적인 학습 기반 배경 활용 방식의 가능성을 보여줍니다. 동시에 YOLO 구조 변경과 재학습을 전제로 하기 때문에, 비학습적 현장 적응이나 이벤트 판정 문제의 직접적인 해답은 아닙니다. 이 구분을 명확히 하는 것이 이 논문을 검토한 핵심 이유입니다.

---

## 마무리 한 문장

BGI-YOLO는 정지 카메라에서 배경 정보가 객체 검출 성능 향상에 유효하다는 점을 보여주지만, 그 방식은 detector 구조 변경과 재학습을 전제로 하므로, 비학습적 이벤트 판정 접근과는 비교하고 구분해야 할 학습 기반 관련 연구로 보는 것이 적절합니다.
