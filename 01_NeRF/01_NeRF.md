# NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis

### Abstract
Nerf는 FC 네트워크를 사용하며, input으로는 continuous 5D를 받음. 지역 위치 정보인 (x,y,z)와 보는 방향을 나타내는 ($$\theta, \phi$$) 이렇게 5개를 받음<br>
output으로는 volume density와 해당 위치로부터 보았을 때 보여지는(view-dependent) 빛, 두 가지를 내놓음<br><br>

결과물은 상당히 사실적인 Novel View Synthesis(NVS)를 보여줌<br><br>
![](https://kimjy99.github.io/assets/img/nerf/nerf-fig1.webp)<br><br><br>
### Contribution
- 복잡한 기하의 연속 장면들을 나타내기 위한 접근법, 5D neural radiance fields은 재료, 기본 MLP 파라미터화<br>
- 기존 볼륨 렌더링 방식을 미분가능하게 만들어, 일반적인 RGB 이미지만으로도 3D 장면 표현을 학습, 최적화할 수 있게 함 (NeRF 이전의 volume rendering procedure는 미분이 안되는게 보통이었으나, NeRF에서 미분 가능한 volume rendering으로 확장. 볼륨 렌더링 적분을 수치적으로 근사하고 이 과정을 MLP와 결합해 최종 이미지와의 차이를 역전파로 최적화할 수 있게 만듬)<br>
- hierarchical sampling 전략을 통해, 신경망의 연산 자원을 실제로 오브젝트, 장면 정보가 있는 공간에 집중적으로 할당<br><br><br>
<br>

### Neural Radiance Field Scene Representation

continuous scenes은 5D vector-valued 함수로 나타냄<br>
- [input] 3D location: $${x} = (x,y,z)$$<br>
- [input] 2D viewing direction $$(\theta, \phi)$$<br><br>
- [output] color: $${c} = (r,g,b)$$<br>
- [output] volume density: $$\sigma$$<br><br><br>

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbkbbeN%2FbtrJWMpVa6w%2FAAAAAAAAAAAAAAAAAAAAAGU_KSl7ElNSLVg7meiW6mk3fsXbrd932fxWiIpwOoyU%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1751295599%26allow_ip%3D%26allow_referer%3D%26signature%3DXItb5muhF7UgAoiAeMqYnNVehhw%253D)<br><br><br>


방향은 3D unit vector $$\mathbf{d}$$<br><br>
아래는 MLP 네트워크의 수식<br>
$$\mathbf{F}_{\theta} : (\mathbf{x}, \mathbf{d}) \rightarrow (\boldsymbol{c}, \sigma)$$<br><br>
각 input coordinates로부터 거기에 상응하는 volume density와 방향에 따라 나타나는 색상으로 매핑되도록, 위의 수식에서 $$\theta$$는 최적화 해야하는 weights를 나타냄<br><br><br>

네트워크가 volume density $$\sigma$$를 위치인 x의 함수로만 예측하도록 제한하고, RGB 색상 c를 위치와 보는 방향의 함수로서 예측하도록 하여서 표현이 다중시점에서도 일관성을 갖도록 함<br><br>

이를 위해 MLP $$F_{\theta}$$는 input인 3D 좌표 x를 8개의 FC layer(Relu activation, 256 channels per layer)로 처리하고, output인 $$\sigma$$와 256 차원의 feature vector를 내놓음<br><br>
이 feature vectors는 카메라 광선의 보는 방향과 결합되서 한 추가 레이어(Relu activision, 128 channels)로 들어감. 그리고 보는 방향에 따른 RGB 색상을 output으로 내놓음<br><br>

Fig 4에서도 보여지듯이 view dependence 없이 x만 넣어주고 학습시킨 모델은 세부적인 부분들을 나타내기 어려워했다<br><br><br>

### Volume Rendering with Radiance Fields
volume density $$\sigma(x)$$는 위치 x에서의 무한소입자(infinitesimal particl)에서 광선이 끝나는 것에 대한 differential probability로 해석될 수 있다.<br><br>
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbNKGYk%2Fbtshz1WTEgR%2FAAAAAAAAAAAAAAAAAAAAAB1UyGB_E0O8W02quKb_x8ck4RMjsQaEkKYwPuKm6TeE%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DDgAFXoBpWvjG5RZ5ljbiBbfGXvs%253D)<br><br><br>

위는 volume rendering 공식<br>
- $$\mathbf C(r(t))$$: 결과 색상<br>
- $$[t_{n}, t_{f}]$$$: 경계 중 근거리 near bound가 t_n, 원거리 far bound가 t_f<br>
- T(t): transmittance, opacity, 투명도, 축적된 투과율을 나타내며, 광선이 t_n에서 t까지 다른 어떤 particle과 마주치는 일 없이 여행한 확률<br>
- r: ray, 광선<br>
- $$\sigma (r(t))$$: occupancy, volume density<br>
- $$c(r(t), d)$$: radiance, 방사 복사도, 색상, 위치 r(t)에서 방향 d로 방출되는 빛의 색상 (3채널 RGB)<br><br>

카메라 광선 r(t) = o + td인데, 이게 광선의 3차원 위치. o는 광선의 시작점, t는 광선위의 거리, d는 광선의 방향 unit vector<br><br>
연속된 neural radiance field로부터 view를 렌더링하는 건, 가상의 카메라의 각 픽셀을 통과하고 추적하며 적분 C(r)을 계산해야함. 이 continuous integral을 quadrature를 사용해 추정<br><br>

Deterministic quadrature, 보통 이산화된 voxel grids를 렌더링할 때 씀, 이는 항상 같은 위치에서만 샘플을 뽑기에 해상도 제한, aliasing 등 문제 발생<br>
그래서 대신 $$[t_{n}, t_{f}]$$를 N개의 균일한 간격의 bins로 분할, 각 bin 내에서 무작위로 uniform dist.를 따라 하나의 샘플을 추출하는 stratified sampling 방식 사용.<br><br>
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbTHjDG%2FbtshPpuIQsA%2FAAAAAAAAAAAAAAAAAAAAAMxU2OwsQKvM4eaUoBFL2_VjHdX0-Er8MfMIT2JeFWYC%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DJNxanzwKllO14A1LvKgFvTq79oM%253D)<br><br><br>
- $$t_{i}$$: i번째 bin안에서 무작위로 뽑힌 샘플의 위치, 광선상의 거리<br>
- U[a,b]: 구간 a,b에서 uniform distribution으로 무작위 샘플을 뽑음<br>
- $$t_{n}$$: 광선이 장면과 교차하는 시작점, near bound<br>
- N: 샘플링 구간내 bin의 개수<br>
- i: 샘플 인덱스<br><br>

stratified sampling은 구간을 나눈 뒤, 각 구간에서 무작위로 샘플을 뽑아 적분을 근사함. 균일하게 전체 구간을 커버하면서도 샘플 위치에 무작위성이 들어가 MLP가 더 다양한 위치에서 값을 예측하게 해줌, 결과적으로 더 부드럽고 노이즈가 적은 결과물이 나옴<br><br>

적분을 근사하려고 샘플들의 이산 집합을 쓰긴 하지만 stratified sampling은 continuous scene 표현이 가능하게 해줌. 왜냐면 결과적으로 MLP의 최적화 과정 중에서 continuous positions로 평가되니까<br><br>

우리는 이 샘플들을 볼륨 렌더링에서 논의된 quadrature rule과 함께 C(r)을 측정하는데에 씀<br><br><br>
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fb3vomA%2FbtshChdyr9O%2FAAAAAAAAAAAAAAAAAAAAAF6QDa--tPnuhUzYpuJQj5w8b6-8DrevoSTOJUTkEH_g%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DFc98ckHZkVjm2fB8iUipDAIvi9A%253D)
<br><br><br>
- $$\delta_{i}=t_{i+1}-t_{i}$$ :인접한 샘플들간의 거리<br>
- exp(-something): 흡수량이 많을수록 투과도는 급격히 감소<br>
- $$1 - exp(\sigma_{i) \delta_{i})$$: 알파값, i번째 샘플에서 빛이 흡수되어 방출되는 비율<br>
      $$\sigma_{i} \delta_{i}$$가 클수록 (즉, 밀도가 높거나 간격이 넓을수록) 이 값은 1에 가까워짐 (거의 다 흡수, 불투명) 반대로 이 값이 작을수록 (밀도가 낮거나 간격이 좁으면) 이 값은 0에 가까워짐(거의 투명)<br><br>
- $$C_{i}$$: MLP의 출력, i번째 샘플 위치에서 방출되는 RGB 색상<br><br>

각 샘플의 기여도 = (그 위치까지 도달할 확률) x (거기서 흡수되어 방출될 확률) x (그 위치의 색상)<br><br><br>

### Optimizing a Neural Radiance Field
SOTA 달성을 위해 고화질의 복잡한 장면들 표현을 가능하게 해주는 2가지 개선점들<br><br>
첫번째, MLP가 고주파 함수를 표현하는데 도움이 되는 input coordinate의 positional encoding<br>
두번째, hierarchical sampling과정, 이게 고주파 표현에서 효율적인 샘플링을 가능하게 함<br><br><br>

### Positional encoding
신경망들이 universal function approximators 임에도 네트워크 $$F_{theta}$$를 입력좌표 $$xyz \theta \phi$$에 대해 직접적으로 동작시켜보는 건 렌더링 결과가 좋지 못함<br>
Rahaman et al.에선 추가적으로 네트워크에 넣기 전에 inputs를 고주파 함수를 사용해 더 높은 차원의 space로 매핑했는데, 고주파 variation을 담은 데이터를 네트워크는 더 잘 fitting<br><br>

이러한 것들을 활용해서 $$f_{\theta}$$를 두 개 함수의 합성으로 재구성함<br><br>
$$F_{\theta}=F_{\theta}^{`}\circ \gamma $$<br><br>
하나는 학습시키고 다른 하나는 학습을 안 하게 했는데 퍼포먼스가 상당히 향상됨, gamma는 R에서 R^{2L}로 매핑하는 것이고, F_{\theta}^{`}는 일반적인 MLP<br><br><br>
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FVHhK3%2FbtshRySxXA3%2FAAAAAAAAAAAAAAAAAAAAAOijxIw_7xqktsgUXG0Vum3JucbRDxY2Uq2-cmmCS7Sd%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1753973999%26allow_ip%3D%26allow_referer%3D%26signature%3DCXgTSRZRmUYV6mPRt%252BtVXLfJr2U%253D)<br><br><br>

























