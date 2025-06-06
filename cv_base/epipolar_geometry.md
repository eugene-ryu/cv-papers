## Epipolar Geometry
<br>
<img src="https://www.cs.auckland.ac.nz/courses/compsci773s1t/lectures/773-GG/figures/epipolar.gif">
1. 정의: 두 개의 서로 다른 카메라에서 관찰된 3D 장면의 기하학적 관계를 설명하는 이론<br>
  1) Epipolar plane: 에피폴라 평면, 두 카메라 중심과 3D 점을 포함하는 평면<br>
  2) Epipolar line: 에피폴라 선, 한 이미지의 점에 대응하는 다른 이미지 상의 제약선<br>
  3) Fundamental matrix: 두 뷰 간의 기하학적 관계를 인코딩하는 3 by 3 행렬로 x^(T)Fx = 0 을 만족<br>
  4) Essential matrix: 필수 행렬, 캘리브레이션 된 카메라 간의 관계를 표현<br><br>

이 이론은 스테레오 매칭, 깊이 추정, 3D 재구성에 활용되며 대표적인 응용 사례로 SfM이 있다. 두 카메라의 시차(parallax)를 통해서 3D 구조를 복원할 수 있는 기반을 제공한다<br>
