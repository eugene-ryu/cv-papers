# NeRF: Representing Scenes as Neural Radiance Fields for View Synthesis

### Abstract
Nerf는 FC 네트워크를 사용하며, input으로는 continuous 5D를 받음. 지역 위치 정보인 (x,y,z)와 보는 방향을 나타내는 ($$\theta, \phi$$) 이렇게 5개를 받음<br>
output으로는 volume density와 해당 위치로부터 보았을 때 보여지는(view-dependent) radiance, 두 가지를 내놓음<br><br>

결과물은 상당히 사실적인 Novel View Synthesis(NVS)를 보여줌<br><br>
![](https://kimjy99.github.io/assets/img/nerf/nerf-fig1.webp)<br><br><br>

### Neural Radiance Field Scene Representation

continuous scenes은 5D vector-valued 함수로 나타냄<br>
- 3D location: $${x} = (x,y,z)$$<br>
- 2D viewing direction $$(\theta, \phi)$$<br><br>
- [output] color: $${c} = {r,g,b}$$<br>
- [output] volume density: $$\sigma$$<br><br><br>

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FbkbbeN%2FbtrJWMpVa6w%2FAAAAAAAAAAAAAAAAAAAAAGU_KSl7ElNSLVg7meiW6mk3fsXbrd932fxWiIpwOoyU%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1751295599%26allow_ip%3D%26allow_referer%3D%26signature%3DXItb5muhF7UgAoiAeMqYnNVehhw%253D)<br><br><br>

#### MLP Network
$$\boldsymbol{F}_{\theta} : (\boldsymbol{x}, \boldsymbol{d}) \rightarrow (\boldsymbol{c}, \sigma)$$<br><br>


<br><br><br>
