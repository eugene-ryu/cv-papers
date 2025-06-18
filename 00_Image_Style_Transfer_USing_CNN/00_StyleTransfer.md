# Image Style Transfer Using Convolutional Neural Networks
<br><br>
![]('./imgs/title.png')<br><bR>
스타일 트랜스퍼 논문<br>
Contribution: content와 style을 분리해서 사용했으며, 기존 pretraned된 VGG19 모델을 사용했다는 것, 결과물이 이전 논문들 결과물 대비 매우 화질이 좋았다는 점 등이 있음<br><br><br>
Abstract에서 핵심 문장은 "A Neural Algorithm of Artistic Style that can seperate and recombine the image content and style of natural images". 컨텐츠와 스타일을 분리하고, 다시 조합하는 아키텍쳐를 정확하게 묘사함<br><br>

![](./imgs/fig2.png)<br>
StyleTransfer 알고리즘<br><br><br>

![](https://www.mdpi.com/agriengineering/agriengineering-04-00056/article_deploy/html/images/agriengineering-04-00056-g002.png)<br>
VGG!9 네트워크<br><br><br>


## Method
#### Deep image representations
<br>
오브젝트 인식과 로컬라이제이션에 트레이닝된 VGG19 네트워크(19-layer VGG network)를 사용함, 16개 convolutional과 5개의 풀링 레이어. FC(Fully connected layer)는 사용하지 않음<br>
원 논문에서는 프레임워크로 카페를 썼다고 했으나, 현실적으로 구현은 파이썬, 그리고 pytorch, torchvision을 통해서 자주 이루어짐<br>
기존 프레임워크 안에 있는 VGG19 네트워크에서는 맥스 풀링을 쓰지만, average pooling이 좀 더 나은 결과를 보여주었음<br><br><br>


#### Content representation
수식에서 input image는 $\vec{x}$로 표시됨<br> 
$N_l$은, 이미지의 가로길이 곱하기 세로 길이인 $M_l$의 각 feature map<br>
때문에 레이어 l(엘)의 행렬 $F^{l}$은 $N_l$ by $M_l$의 real-number matrix이다. $R^{N \times M}$<br>
