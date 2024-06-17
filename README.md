# 제로베이스 백엔드 22기 개인 프로젝트

> 작성자: 장진영(danceurdance@naver.com)

> 프로젝트명: Team Run!

1. 시나리오:

   1. 사용자는 클럽에 가입신청을 할 수 있고 클럽은 사용자에게 가입초대를 할 수 있다.
   2. 사용자는 월간 개인 챌린지를 설정할 수 있다.
   3. 사용자는 월간 러닝 스케쥴을 설정할 수 있다.
   4. 사용자 일간 달리기 기록은 러닝 스케쥴에 맞춰 주어진 평가지표에 따라 생성된다.
   5. 클럽장은 달마다 팀 챌린지 수립할 수 있고 그에 대한 리워드를 설정할 수 있다.
   6. 클럽장은 비슷한 티어의 클럽을 검색할 수 있고 동반 챌린지를 해서 경쟁할 수 있다.
   7. 클럽 챌린지에 대한 멤버들 수행상황을 실시간으로 리더보드에 반영한다.
   8. 월 마지막 날 24:00에 개인/팀 챌린지는 자동 종료된다.
   9. 위와 함께 사용자에 대해서 월간 달리기 기록이 주어진 평가지표에 따라 생성된다.
   10. 위와 함께 클럽 티어를 업데이트하고 멤버들에게 리워드를 수여한다.

2. 구현 정보: 

   ***(구성: 비즈니스 모듈 → Kafka ← Metric)***

   1. User, Club 엔티티의 ‘이번달…’ 칼럼에 대한 쿼리문은 달을 기준으로 한다. 
   2. 평가 및 평가지표와 관련된 부분은 독립된 Metric 모듈을 만들어 관리한다.
   3. 위 모듈은 Kafka를 활용하여 비즈니스 모듈과 통합한다.
   4. 러닝 스케쥴의 종료 시간에 맞춰 그 결과를 Kafka에 전송한다.
   5. 개인/클럽 챌린지의 종료 시간에 맞춰 그 결과를 Kafka에 전송한다.
   6. Metric은 주어진 평가지표(거리, 속도 등)을 기준으로 개인/팀을 평가하고 일/월간 달리기 기록을 Kafka에 전송한다.
   7. 비즈니스 모듈은 위 정보를 실시간으로 리더보드에 반영한다.
   8. 비즈니스 모듈은 위 정보를 클럽 티어 업데이트 및 멤버 리워드에 반영한다.

3. 논리 데이터 모델 

| Anchor | Description            | Logical Type      | Example Value                   | Physical Column | Physical Type |
| ------ | ---------------------- | ----------------- | ------------------------------- | --------------- | ------------- |
| User   | 개인 신상 정보         | [String, Int]     | 장진영, 32세 등                 |                 |               |
| User   | 건강 및 체력 관련 정보 | [String, Int]     | 키 168cm, 몸무게 58kg 등        |                 |               |
| User   | 소속된 클럽            | Club(N:1)         | 동탄 조깅 클럽                  |                 |               |
| User   | 이번달 러닝 스케쥴     | Running Day(1:N)  | 8월 7일 수요일 오전 6시~7시 3km |                 |               |
| User   | 이번달 개인 챌린지     | Challenge(1:N)    | 1km당 평균 6분 내외             |                 |               |
| User   | 달리기 기록            | Track Record(1:N) | 총 달린 거리 4km, 속도 260m/min |                 |               |
| User   | 이번달 리워드          | Reward(N:1)       | 완주자 메달                     |                 |               |

| Anchor | Description        | Logical Type        | Example Value                         | Physical Column | Physical Type |
| ------ | ------------------ | ------------------- | ------------------------------------- | --------------- | ------------- |
| Club   | 클럽 이름          | String              | 동탄 조깅 클럽                        |                 |               |
| Club   | 클럽 정보          | [String, Int]       | 클럽장 장진영, 지역 동탄, 인원 7명 등 |                 |               |
| Club   | 클럽 멤버          | User(1:N)           | 장진영, 장원석 등                     |                 |               |
| Club   | 클럽 티어          | String              | 플라티넘                              |                 |               |
| Club   | 이번달 팀 챌린지   | Team Challenge(N:M) | 6월 달 100km 달리기                   |                 |               |
| Club   | 이번달 클럽 이벤트 | Event(N:M)          | 6월 9일 퇴근후 한강 런 모임           |                 |               |
| Club   | 이번달 리더보드    | LeaderBoard(1:1)    | 1위 장진영 40km, 2위 장원석 35km      |                 |               |

| Anchor         | Description      | Logical Type | Example Value                 | Physical Column | Physical Type |
| -------------- | ---------------- | ------------ | ----------------------------- | --------------- | ------------- |
| Team Challenge | 팀 챌린지 이름   | String       | 동탄 100Sprint                |                 |               |
| Team Challenge | 팀 챌린지 정보   | String       | 운동하는 달 7월 총 거리 100Km |                 |               |
| Team Challenge | 팀 챌린지 참여자 | Club(M:N)    | 6월 달 100km 달리기           |                 |               |

| Anchor             | Description        | Logical Type | Example Value       | Physical Column | Physical Type |
| ------------------ | ------------------ | ------------ | ------------------- | --------------- | ------------- |
| Personal Challenge | 개인 챌린지 이름   | String       | 할수있다!           |                 |               |
| Personal Challenge | 개인 챌린지 정보   | String       | 1km당 평균 6분 내외 |                 |               |
| Personal Challenge | 개인 챌린지 참여자 | User(N:1)    | 장진영              |                 |               |

| Anchor | Description | Logical Type | Example Value                                         | Physical Column | Physical Type |
| ------ | ----------- | ------------ | ----------------------------------------------------- | --------------- | ------------- |
| Reward | 리워드 이름 | String       | 금메달, 완주자 메달 등                                |                 |               |
| Reward | 리워드 정보 | String       | 6월달 클럽 목표 거리에 가장 많은 기여를 멤버에게 수여 |                 |               |
