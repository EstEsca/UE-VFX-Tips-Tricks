# Find Quat Between Vectors
개밟 환경 : UE5, 나이아가라 파티클 시스템
### 히스토리
플레이어 캐릭터의 손에 부착되어 카메라가 바라보는 방향을 기준으로 회전하는 파티클 이펙트를 개발하고 싶었다. 이펙트를 부착하는 소켓이 카메라 방향(또는 플레이어 캐릭터 방향)과 잘 정렬되어 있으면 좋았으련만, 소켓의 축은 그 어느것도 카메라 축과 일치하지 않았다.

이에 아래와 같은 방식으로 이펙트를 개발할 수 있었다.
1. World +X 축을 기준으로 회전하는 파티클을 만들고
2. 회전축과 임의의 벡터가 서로 일치하도록 해당 파티클의 위치 벡터를 사원수(``Quaternion``,``Quat``)을 활용해 회전
![FindQuatBetweenVectors_001](/assets/images/FindQuatBetweenVectors/FindQuatBetweenVectors_001.png)

*1번의 파티클을 회전시켜 2번의 움직임을 얻을 수 있다* 
### 사원수(Quat, Quaternion)
사원수는 회전을 나타내는 데 사용되는 수 또는 구조로, float4로 나타낼 수 있다. 각 구성요소는 꽤나 직관적인데, 그 구성요소는 아래와 같다.
- Quat.xyz = 회전축
- Quat.w = 회전의양(각도)

나이아가라 스크래치 패드 모듈에서 제공하는 ``FindQuatBetweenVectors``, ``Multiply Vector with Quaternion``과 같은 노드를 활용하면 회전 관련 연산을 쉽게 수행할 수 있다.

아래 예제는 다음을 구현한 것이다.
- 카메라 위치를 따라다니며,
- 카메라 전방 50유닛 앞에서
- 원운동을 하는 파티클

![FindQuatBetweenVectors](/assets/images/FindQuatBetweenVectors/rotation_overview.png)
![FindQuatBetweenVectors](/assets/images/FindQuatBetweenVectors/particle_system_overview.png)

### 예제 시스템 제작 #1 : 원운동
x축을 회전축으로 하는 원운동은 아래와 같은 offset 벡터로 나타낼 수 있다.

``Offset.y = cos(normalized_angle)``
``Offset.z = sin(normalized_angle)``

이를 나이아가라 시스템에서 구현하면 아래와 같다.

![FindQuatBetweenVectors](/assets/images/FindQuatBetweenVectors/initial_rotation_offset.png)
*Local Space Rotation Offset 부분*


### 예제 시스템 제작 #2 : 벡터 회전(축 일치)

임의의 ``벡터 A``를 회전축으로 갖는 회전운동이 ``벡터 B``를 새로운 회전축으로 갖는다고 하면, 새로운 회전운동은 다음과 같이 계산할 수 있다.
1. ``벡터 A``에서 ``벡터 B``로 향하는 ``Quat C``을 찾는다.
2. 회전운동을 나타내는 ``위치 벡터``에 ``Quat C``를 곱하여 회전축 ``벡터 A``를 회전한다(``벡터 A``와 ``벡터 B``를 일치시킨다).

이를 나이아가라 시스템에서 구현하면 아래와 같다.

![FindQuatBetweenVectors](/assets/images/FindQuatBetweenVectors/vector_rotation_overview.png)
*``OldAxis``는 1번에서 회전축이 되는 x축 벡터 <1, 0, 0>. ``NewAxis``는 카메라 벡터. ``VectorToRotate``는 1번에서 계산한 ``Offset`` 벡터.*
![FindQuatBetweenVectors](/assets/images/FindQuatBetweenVectors/vector_rotation_detail.png)

---
최종적으로 얻은 ``RotationOffset``을 활용해 파티클의 위치를 계산하는 과정은 아래와 같다.

![FindQuatBetweenVectors](/assets/images/FindQuatBetweenVectors/final_position.png)