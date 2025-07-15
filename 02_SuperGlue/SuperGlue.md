## SuperGlue: Learning Feature Matching with Graph Neural Networks
<br><br>
28 Mar 2020<br><br>
![](https://miro.medium.com/v2/resize:fit:1100/format:webp/1*2tEinOcDv2T4GiyDZfxPhA.png)<br><br>
<br>
SuperGlue는 크게 Attentional Graph Neural Network와 optimal matching layer로 구성<br><br>
- Attentional GNN의 keypoint encoder는 추출된 두 이미지의 keypoint poisitions **p**와 visual descriptors **d**를 받아서 하나의 벡터로 만듬. 그리고 self와 cross-attention을 번갈아 L번 쓰면서 더 강력한 representation **f**로 만듬. context 집계, keypoint의 기하학적 배치와 외형 정보를 동시에 고려.<br><br>
- Optimal matching layer는 M by N 크기의 score matrix, 두 이미지의 특징점간 유사도 행렬을 만드는데 dustbin과 함께 만듬. dustbin score는 매칭되지 않는 keypoint(짝이 없는 점)를 처리하기 위해 추가한 것인데, occlusion, noise, 검출실패 등의 이슈가 있는 것들을 dustbin 에 할당. 그리하여 (M+1) by (N+1) 사이즈가 된다. 그리고 T번 동안 Sinkhorn algorithm을 사용해 optimal partial assignment를 찾음<br><br>

### Sinkhorn algorithm
엔트로피 정규화(Entropic Regularization)를 적용한 Optimal Transport(OT) 문제를 빠르고 효율적으로 푸는 반복적 최적화 방법. Matrix scaling이 본 알고리즘의 핵심 과정<br>
  1) 초기 행렬 K 준비<br>
  2) 두 개의 대각행렬(diagonal matrix) D1, D2를 찾아 D1KD2의 행 합과 열 합이 각각 주어진 분포 p,q와 같아지도록 반복 스케일링<br>
  ㄴ 행 합이 p가 되도록 각 행을 스케일링 (즉, 각 행을 p와 현재 행 합의 비율만큼 곱함)<br>
  ㄴ 열 합이 q가 되도록 각 열을 스케일링 (즉, 각 열을 q와 현재 열 합의 비율만큼 곱함)<br>
  ㄴ 이 과정을 원하는 정확도가 나올때까지 반복<br><bR>

이 과정을 통해 K는 최종적으로 doubly stochastic matrix(각 행과 열의 합이 1인 행렬) 또는 원하는 행/열 합을 갖는 행렬로 변환<br><br>

- Optimal Transport(OT, 최적수송) 문제 등에서 행렬이 두 분포(e.g. 두 이미지의 특징점 분포 등)를 동시에 만족하는 확률적 매핑(transport plan)이 되기 위해 필요<br>