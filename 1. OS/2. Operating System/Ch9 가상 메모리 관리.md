# Chapter9 가상 메모리 관리

[[_TOC_]]

## 1. 요구 페이징

- **요구 페이징**
  - 가져오기 정책에서 프로세스(사용자)가 요청할 때 메모리로 가져오는 방법
  - 가져오기 정책: 프로세스가 필요로 하는 데이터를 언제 가져올기 결정하는 것



### 1. 요구 페이지의 개요

- 컴퓨터를 오래 사용하면 느려지는 이유
  - 작업을 하지 않고 쉬는 프로세스나 좀비 프로세스가 메모리를 차지하여 메모리 관리가 복잡해지기 때문
  - 따라서 메모리에는 꼭 필요한 프로세스만 유지하는 것이 좋음



- 프로세스에 일부만 메모리에 가져오는 이유
  - 메모리의 효율적 관리를 위해(메모리가 꽉차면 관리하기 어려움)
  - 응답 속도의 향상을 위해(용량이 큰 프로세스를 전부 가져와 실행하면 응답이 늦어질 수 있음)
    - 예를 들어 포토샵 프로세서에서 사용자가 요구한 기능만을 메모리에 올려놓은 것



- **미리 가져오기**
  - 요구 페이징과 반대로 앞으로 필요할 것이라고 판단되는 페이지를 미리 가져오는 것(ex - 캐시)
    - 쓸데없는 데이터를 미리 가져올 경우 피해가 크므로 현대 운영체제는 요구 페이징을 기본으로 사용



- **스와핑**과 **게으른 스와퍼**
  - 스와핑: 프로세스를 구성하는 모든 페이지를 메모리에 올리는 것(**순수한 스와핑**)
  - 게으른 스와퍼: 사용자가 요구할 때 메모리에 올리는 것



### 2. 페이지 테이블 엔트리의 구조

- 페이지가 스왑 영역에 있게 되는 원인

  1. 요구 페이징으로 인해 처음부터 물리 메모리에 올라가지 못한 경우
  2. 메모리가 꽉 차서 스왑 영역으로 옮겨온 경우

  - 어떤 경우이든 테이블에는 페이지가 어디에 있는지 표시해야 하는데 이때 사용하는 비트가 **유효 비트**



- **페이지 테이블 엔트리**(**PTE**)
  - 페이지 테이블의 한 행을 의미
  - 구성
    - **페이지 번호**
      - 해당 페이지가 몇 번인지 알려주는 자료
      - 매핑 방식에 따라 포함되기도 하고 안되기도 함(직접 매핑에서는 불필요 연관 매핑은 필요)
    - **프레임 번호**(**주소 필드**)
      - 해당 페이지가 어느 프레임에 올라와 있는지 알려주는 자료(페이지 테이블의 핵심)
    - **플래그 비트**
      - **접근 비트**(**참조 비트**)
        - 페이지가 메모리에 올라온 후 사용한 적이 있는지 알려주는 비트
          - 즉 해당 메모리를 읽거나 실행 작업을 했다면 접근 비트는 1이됨
      - **변경 비트**(**더티 비트**)
        - 페이지가 메모리에 올라온 후 데이터의 변경이 있었는지 알려주는 비트
          - 즉 해당 메모리에 쓰기나 추가 작업을 했다면 변경 비트는 1이됨 
      - **유효 비트**(**현재 비트**)
        - 페이지가 실제 메모리에 있는지를 나타내는 비트
      - **읽기, 쓰기, 실행 비트**(**접근 권한 비트**)
        - 페이지에 대한 각각의 권한을 나타내는 비트로 권한 없는 프로세스의 접근을 차단
        - 세그먼테이션-페이징 혼용 기법에서는 테이블 크기를 줄이기 위해 세그먼테이션 테이블로 옮김



### 3. 페이지 부재

- **유효 비트**
  - 유효 비트값이 0일때는 페이지가 메모리에 있으므로 주소 필드에 프레임 번호가 저장
    - 1일때는 주소필드에 스왑 영역 내 페이지의 주소가 저장
    - 이렇게 프로세스가 요청했을 때 메모리에 없는 상황을 **페이지 부재**라고 함



- **페이지 부재**

  - 프로세스가 요청했을 때 메모리에 없는 상황

  - 작업 과정

    1. PTE i번을 조회했을 때, 유효 비트가 1인 경우 페이지 부재가 발생

    2. 메모리 관리자는 스왑 영역의 n번에 있는 페이지를 메모리의 비어 있는 프레임 m번에 가져옴(**스왑인**)
       - 이때 메모리가 비어있지 않다면 프레임 중 하나를 스왑 영역에 보내야 가져올 수 있음
         - 어떤 페이지를 스왑 영역에 내볼낼지 결정하는 알고리즘을 **페이지 교체 알고리즘**이라 함
         - 또한 보내지는 페이지를 **대상 페이지**라고 함
         - 당연히 대상 페이지의 PTE는 갱신함
    3. 프레임 m에 페이지가 들어오면 PTE i번의 유효 비트를 0으로 바꾸고 주소 필드 n을 m으로 바꿈
    4. 프레임 m으로 접근하여 해당 데이터를 프로세스에 넘김



- 세그먼테이션 오류와 페이지 부재
  - 세그먼테이션 오류는 사용자의 프로세스가 주어진 메모리 공간을 벗어나거나 접근 권한이 없는 곳에 접근할 때 발생
    - 결과는 강제 종료임
  - 페이지 부재는 해당 페이지가 물리 메모리에 없을 때 발생함
    - 결과는 스왑 영역의 페이지를 메모리에 가져오고 PTE을 갱신함



### 4. 지역성

- **지역성**
  - 기억장치에 접근하는 패턴이 메모리 전체에 고루 분포되는 것이 아니라 특정 영역에 집중되는 성질
    - 페이지 교체 알고리즘을 이용하여 쫒아낼 페이지를 찾는데 사용
  - **공간의 지역성**
    - 현재 위치에서 가까운 데이터에 접근할 확률이 먼 곳에 접근할 확률보다 높다는 것
  - **시간의 지역성**
    - 현재를 기준으로 가장 가까운 시간에 접근한 데이터가 먼 시간에 접근한 데이터보다 사용할 확률이 높다는 것
  - **순차적 지역성**
    - 여러 작업이 순서대로 진행되는 경향이 있다는 것
    - 문헌에 따라서는 공간의 지역성의 특수한 경우로 봄



- 지역성은 캐시와 페이지 교체 알고리즘에서 고려됨



## 2. 페이지 교체 알고리즘

- 메모리가 꽉 찼을 때 어떤 페이지를 스왑 영역으로 내보낼지 결정하는 재배치 정책



### 1. 페이지 교체 알고리즘의 개요

- 페이지 교체 알고리즘
  - 메모리에서 앞으로 사용할 확률이 적은 페이지를 대상 페이지로 선정함
  - 페이지 부재를 줄이고 시스템의 성능 향상이 목적



#### 1.1 페이지 교체 알고리즘의 종류

- 페이지 교체 알고리즘의 종류
  - 무작위 알고리즘: 말그대로 무작위로 대상 페이지를 선정
  - 최적 알고리즘: 미래의 메모리 접근 패턴을 살펴보고 대상 페이지를 선정하므로 가장 좋지만 현실적으로 구현이 불가
  - NRU, LFU, NUR 알고리즘: 최적 알고리즘에 근접하는 성능을 보여줌

| 종료               | 알고리즘  | 특징                                                         |
| ------------------ | --------- | ------------------------------------------------------------ |
| 간단한 알고리즘    | 무작위    | 무작위로 대상 페이지를 선정하여 스왑 영역으로 보냄           |
|                    | FIFO      | 처음 메모리에 올라온 페이지를 스왑 영역으로 보냄             |
| 이론적 알고리즘    | 최적      | 미래의 접근 패턴을 보고 대상 페이지를 선정하여 스왑 영역으로 보냄 |
| 최적 근접 알고리즘 | LRU       | 시간적으로 멀리 떨어진 페이지를 스왑 영역으로 보냄           |
|                    | LFU       | 사용 빈도가 작은 페이지를 스왑 영역으로 보냄                 |
|                    | NUR       | 최근에 사용한 적이 없는 페이지를 스왑 영역으로 보냄          |
|                    | FIFO 변형 | FIFO 알고리즘을 변형하여 성능을 높임                         |



#### 1.2 페이지 교체 알고리즘의 성능 평가 기준

- 알고리즘의 성능 평가 기준
  - 페이지 부재의 횟수, 평균 대기 시간, 전체 작업 시간, 페이지 교체 횟수 등 다양한 기준이 존재
    - 주로 페이지 교체 횟수를 비교
  - 페이지 교체 알고리즘은 성능뿐만 아니라 유지 비용도 고려해야 함



### 2. 무작위 페이지 교체 알고리즘

- 무작위로 선정하므로 지역성이 전혀 고려되지 않으므로 자주 사용하는 페이지가 선정되기도 함
  - 따라서 알고리즘의 성능이 좋지 않아 자주 사용되지 않음



### 3. FIFO 페이지 교체 알고리즘

- FIFO 알고리즘
  - 시간상으로 메모리에 가장 먼저 들어온 페이지를 대상 페이지로 선정
  - 알고리즘 구현은 큐로 함
    - 메모리의 맨 위는 가장 먼저, 가장 아래는 가장 최근 삽입된 것으로 메모리가 꽉차면 위부터 나감
  - 구현이 쉽고 시간의 지역성이 고려되기는 했지만 단순히 시간만을 기준이므로 성능이 떨어짐
    - 이를 개선한 알고리즘이 **2차 기회 페이지 교체 알고리즘**



### 4. 최적 페이지 교체 알고리즘

- 최적 알고리즘
  - 앞으로 사용하지 않을 페이지를 대상 페이지로 선정
    - 즉 메모리가 앞으로 사용할 페이지를 미리 살펴보고 페이지 교체 시점부터 사용 시점까지 가장 멀리 있는 페이지를 대상 페이지로 선정하는 것
  - 그러나 이러한 예측은 거의 불가능에 가깝기에 최적 알고리즘에 근접한 최적 근접 알고리즘(LRU, LFU, NUR)이 등장
    - 과거를 보고 미래를 예측하는 것
    - **LRU 알고리즘**
      - 페이지에 접근한 시간을 기준으로 대상 페이지를 선정
    - **LFU 알고리즘**
      - 페이지가 사용된 횟수를 기준으로 대상 페이지를 선정



### 5. LRU 페이지 교체 알고리즘

- LRU 알고리즘
  - 메모리에 올라온 후 가장 오랫동안 사용되지 않은 페이지를 대상 페이지로 선정



#### 5.1 페이지 접근 시간에 기반한 구현

- 예를 들어 접근(읽기, 쓰기 등의 연산)한 숫자(시간)를 초로 적어두고 이 시간이 가장 작은 페이지를 대상 페이지로 선정하는 것
  - 메모리 접근 패턴을 변경하면 FIFO 알고리즘만큼 느려지기도 하고, 최적 알고리즘만큼 빨라지기도 함



#### 5.2 카운터에 기반한 구현

- 숫자를 시간이 아닌 카운터를 사용하는 것
  - 그러나 숫자든 카운터든 모두 추가적인 메모리 공간을 필요로 하는 것이 단점(공간 낭비)



#### 5.3 참조 비트 시프트 방식

- 각 페이지에 일정한 크기의 참조 비트를 만들어 사용
  - 초기값은 0이며 접근할 때마다 1로 바뀌고, 주기적으로 오른쪽으로 한 칸씩 이동
  - 예를 들어 페이지 A에 접근하면 A의 참조 비트의 맨 앞은 1이 되고, 다시 B를 참조하면 모든 참조 비트를 오른쪽으로 한 칸 밀어낸 뒤 맨 앞을 1로 채움
  - 역시 참조 비트를 유지하기 위한 메모리 공간을 필요로 하는 것이 단점(공간 낭비)



### 6. LFU 페이지 교체 알고리즘

- LFU 알고리즘
  - 페이지가 몇 번 사용되었는지를 기준으로 대상 페이지를 선정
    - 즉 프레임에 있는 페이지마다 그 사용 횟수를 세어 가장 적은 페이지를 대상 페이지로 선정
  - 일반적으로 LRU와 성능이 비슷하고 단점도 같음



### 7. NUR 페이지 교체 알고리즘

- NUR 알고리즘
  - 참조 비트와 변경 비트를 이용하여 대상 페이지를 선정
    - NRU, NFU와 성능이 거의 비슷하면서 공간 낭비를 해결한 알고리즘(2bit만 사용)
  - 원리
    - 95번 접근한 페이지와 91번 접근한 페이지를 쫒아내야 할 때, 어떤 페이지를 쫒아내도 상관없음
      - 어차피 확률은 비슷하기 때문
  - 과정
    - 모든 페이지의 초기 상태는 (0, 0) => (참조 비트, 변경 비트)
    - 참조 또는 변경이 발생함에 따라 (0, 1), (1, 0), (1, 1)이 됨
    - 이후 (0, 0) => (0, 1) => (1, 0) => (1, 1) 순으로 쫓아냄(앞에서부터)
      - 즉 참조 비트가 0인 것을 먼저 찾고 같은 순위가 어러 개이면 무작위로 선정함
    - 모든 페이지가 (1, 1)이 되면 (0, 0)으로 초기화함



### 8. FIFO 변형 알고리즘

#### 8.1 2차 기회 페이지 교체 알고리즘

- 2차 기회 알고리즘
  - 큐를 사용하고, 특정 페이지에 접근하여 페이지 부재 없이 성공할 경우 해당 페이지를 큐의 맨 뒤로 이동하여 대상 페이지에서 제외
    - 즉 성공한 페이지를 큐의 맨 뒤로 옮겨 기회를 한 번 더 주는 것
  - 일반적으로 최적 근접 알고리즘보다는 성능이 떨어지고, 큐를 유지하는 비용이 큼
    - 또한 페이지가 성공하면 큐의 중간에 있는 값을 맨 뒤로 이동하는 작업이 존재



#### 8.2 시계 알고리즘

- 시계 알고리즘
  - 2차 기회 알고리즘과 유사하나 원형 큐를 사용하는 것이 가장 큰 차이
  - 스왑 영역으로 보낼 대상 페이지를 가리키는 포인터를 사용
    - 이 포인터가 맨 바닥으로 내려가면 다시 큐의 처음을 가리키게 됨
    - 메모리가 비어있을 때는 큐의 처음만 가리키고 있음
    - 가리키는 메모리에 쫓겨나는 경우 아래로 한 칸 이동함
      - 이때 참조 비트가 1인 비트는 건너 뛰면서 0으로 바꿈
  - NUR보다 더 적은 메모리를 사용하지만 알고리즘이 복잡하고 계산량이 많음



## 3. 스레싱과 프레임 할당

### 1. 스레싱

#### 1.1 스레싱의 개념

- **스레싱**: 잦은 페이지 부재로 작업이 멈춘 것 같은 상태



#### 1.2 물리 메모리의 크기와 스레싱

- 스레싱은 메모리의 크기가 일정할 경우 멀티 프로그램의 수와 밀접한 관계가 있음
  - 프로그램 수가 증가하면 CPU 사용률도 높아지다가 메모리가 꽉차면 작업 시간보다 스왑 영역으로 보내고 받아오는 작업이 빈번해지는데 이 지점이 **스레싱 발생 지점**
    - 물리 메모리 크기를 늘리면 스레싱 발생 지점을 늦출 수 있음
      - 단, 일정 이상 커지면 작업 속도에 영향 없음



#### 1.3 스레싱과 프레임 할당

- 스레싱은 각 프로세스에 프레임 할당 문제와도 연관
  - 프로세스에 너무 적은 프레임을 할당하면 페이지 부재가 많아짐
  - 너무 많은 프로세스를 할당하면 페이지 부재는 적지만 전체적인 성능이 낮아짐
  - 따라서 적절히 프레임을 나눠주는 정책이 필요하고 크게 **정적 할당**과 **동적 할당**이 있음



### 2. 정적 할당

- **정적 할당**: 프로세스 실행 초기에 프레임을 나눠준 후 그 크기를 고정하는 것(균등 할당, 비례 할당이 존재)



#### 2.1 균등 할당

- **균등 할당**
  - 프로세스의 크기와 상관없이 사용 가능한 프레임을 모든 프로세스에 동일하게 할당
  - 프로세스 크기가 큰 경우 페이지 부재가 빈번히 발생하고, 작은 경우 메모리가 낭비됨



#### 2.2 비례 할당

- **비례 할당**
  - 프로세스 크기에 비례하여 프레임을 할당하는 방식
  - 균등 할당보다 현실적인 방법이나 두 가지의 문제가 존재
    1. 프로세스가 실행 중에 필요로 하는 프레임을 유동적으로 반영하지 못함
       - 예를 들어 동영상 플레이어는 프로그램 자체는 작으나 실행중에는 매우 많은 메모리가 필요
    2. 사용하지 않을 메모리를 처음부터 미리 확보하여 공간을 낭비
       - 예를 들어 큰 프로세스를 실행하면서 당장 필요 없는프레임을 미리 할당하게 됨



### 3. 동적 할당

- **동적 할당**: 시시각각 변하는 요청을 수용하는 방식(작업집합 모델, 페이지 부재 빈도 방식이 존재)
  - 정적 할당은 실행 동안 변경이 안됨



#### 3.1 작업집합 모델

- **작업집합 모델**
  - 최근 일정 시간 동안 참조된 페이지들을 집합으로 만들고, 이 집합에 있는 페이지들을 물리 메모리에 유지하여 프로세스의 실행을 도움
    - 지역성 이론을 바탕으로 하며, 가장 최근에 접근한 프레임이 이후에도 또 참조될 가능성이 높다는 가정에서 출발
  - 물리 메모리에 유지할 페이지의 크기를 **작업집합 크기**라고 함
    - 작업집합의 크기는 최대 페이지의 수뿐만 아니라 얼마나 자주 작업집합을 갱신할 것인지도 의미
  - 작업집합에 포함되는 페이지의 범위를 **작업집합 윈도우**라고 함
    - 현재 시점에 최대 어느 범위까지의 페이지를 살펴볼 것인가를 결정하는 것으로 성능에 큰 영향
    - 너무 크면 불필요한 페이지까지 메모리에 남고, 너무 작으면 필요한 페이지가 스왑 영역에 가게 됨



#### 3.2 페이지 부재 빈도

- 작업집합 모델은 어떤 프레임을 메모리에 유지해야 하는지는 알 수 있지만, 어떤 프로세스에 프레임을 얼마나 할당해야 하는지는 알 수 없음
  - 즉 작업집합은 프로세스의 성능을 높이는 방법이지만 스레싱 문제를 해결하지 못함



- **페이지 부재 빈도를 이용**
  - 페이지 부재 횟수를 기록하여 페이지 부재 비율을 계산하는 방식으로 부재 비율의 상한과 하한을 설정
    - 상한선을 초과하면 할당한 프레임이 적다는 것이므로 프레임을 추가하여 늘림
    - 하한 밑으로 내려가면 메모리가 낭비된다는 것이므로 할당한 프레임을 회수함



## 4. [심화학습] 프레임 관련 이슈

### 1. 전역 교체와 지역 교체

- 페이지를 교체할 때 방식
  - **전역 교체**: 전체 프레임을 대상으로 교체 알고리즘을 적용
    - **장점**
      - 전체 시스템의 입장에서는 효율적
    - **단점**
      - 다른 프로세스를 스레싱 도달 지점으로 만들 수 있음
        - 즉 스레싱이 증가함
  - **지역 교체**: 현재 실행 중인 프로세스의 프레임을 대상으로 교체 알고리즘을 적용
    - **장점**
      - 자신에게 할당된 프레임의 전채 개수가 변화가 없기 때문에 페이지 교체가 다른 프로세스에 영향을 미치지 않음
        - 만약 영향을 미치면 프레임을 빼앗긴 프로세스는 스레싱 발생 지점에 도달할 수 있음
        - 즉 스레싱을 줄일 수 있음
    - **단점**
      - 자주 사용하는 페이지가 스왑 영역으로 옮겨져 시스템의 효율이 떨어짐



### 2. 페이지 테이블 크기

- 32bit CPU의 윈도우 NT에서 예시
  - 가상 공간의 크기는 2\*\*32B(약 4GB), 페이지 1개의 크기는 2\*\*12(4,096)B
  - 두 개를 나눈 2\*\*20B, 즉 1,048,576개의 행이 있고 이를 테이블로 나타내려면 1,048,576개의 각 행과 각 행을 표현할 주소 공간 20bit가 필요
    - 따라서 페이지 테이블이 차지하는 공간은 약 2.62MB
    - 이는 프로세스 1개의 페이지 테이블



- VAX 운영체제에서 예시
  - 가상 공간의 크기는 2\*\*32B(약 4GB), 페이지 1개의 크기는 2\*\*(512)B
  - 두 개를 나눈 2\*\*23(8,388,608)개의 행이 있고 이를 테이블로 나타내기 위한 주소 공간은 23bit
    - 따라서 페이지 테이블이 차지하는 공간은 약 24.11MB(8,388,608 X 23bit)
    - 이는 프로세스 1개의 페이지 테이블



- 페이지의 크기가 크면 테이블 크기를 줄일 수 있겠지만, 내부 단편화로 공간이 낭비됨



### 3. 쓰기 시점 복사

- 운영체제는 필요로 하는 순간까지 새로운 프레임 확보를 미룸
  - 그동안은 하나의 프레임을 공유하여 여러 프로세스를 실행함
  - 프로세스는 변수를 공유할 수 없으므로 변수를 변경하는 경우 새로운 프레임이 생성됨
    - 예를 들어 프로세스 A, B가 같은 google 페이지를 공유하고 있다가 프로세스 B가 naver로 이동하는 경우 
    - B의 변수 Home이 google에서 naver로 변경되야 하므로 변수는 새로운 프레임에 복사되고 내용이 naver로 바뀜
    - 이렇게 데이터에 변화가 있을 때까지 복사를 미루는 방식을 **쓰기 시점 복사**라고 함
      - 쓰기 시점 복사는 요구 페이징과 닮은 점이 많음
