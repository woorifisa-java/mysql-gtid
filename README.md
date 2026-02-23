# MySQL-GTID
MySQL Replication GTID 실습입니다.<br>
기존 방식인 File/Position 방식과 GTID 방식의 차이를 실습을 통한 비교입니다.

---

## GTID(Global Transaction Identifier)란?
### MySQL  복제 환경에서 소스(Master)와 복제본(Replica)의 모든 트랜젝션에 부여되는 고유한 식별자

- 핵심
  - 복제 환경에서 각 Transaction에 고유한 ID를 부여하여 자동 장애 조치(Failover), 편리한 복제 구성 및 관리, 데이터 무결성을 보장
  - server_uuid와 transaction_id 조합으로 생성
 
- 장점
  - 손쉬운 장애 조치 및 복구(Failover)
  - 복제 구조 단순화 및 자동화
  - 데이터 일관성 보장
  - 복제 위치 오류 감소   

---

## File/Position 방식과 GTID 방식 비교
| 항목 | 전통적 방식 (File/Position) | GTID 기반 복제 |
|---|---|---|
| 복제 위치 추적 | binlog 파일명 + 위치를 수동으로 설정 | GTID 기반으로 자동 추적 |
| 장애 복구 | 복잡하며 수동 작업 필요 | 자동 추적으로 비교적 간단 |
| 마스터 전환 | Replica 재설정 등 수동 작업 필요 | `AUTO_POSITION=1`로 쉽게 처리 |
| 운영 안정성 | binlog 삭제 시 복제 실패 가능 | 상대적으로 안정적 |

---
# MySQL-GTID
MySQL Replication GTID 실습입니다.<br>
기존 방식인 File/Position 방식과 GTID 방식의 차이를 실습을 통한 비교입니다.

---

## GTID(Global Transaction Identifier)란?
### MySQL  복제 환경에서 소스(Master)와 복제본(Replica)의 모든 트랜젝션에 부여되는 고유한 식별자

- 핵심
  - 복제 환경에서 각 Transaction에 고유한 ID를 부여하여 자동 장애 조치(Failover), 편리한 복제 구성 및 관리, 데이터 무결성을 보장
 
- 장점
  - 손쉬운 장애 조치 및 복구(Failover)
  - 복제 구 단순화 및 자동화
  - 데이터 일관성 보장
  - 복제 위치 오류 감소   

---

## File/Position 방식과 GTID 방식 비교
| 항목 | 전통적 방식 (File/Position) | GTID 기반 복제 |
|---|---|---|
| 복제 위치 추적 | binlog 파일명 + 위치를 수동으로 설정 | GTID 기반으로 자동 추적 |
| 장애 복구 | 복잡하며 수동 작업 필요 | 자동 추적으로 비교적 간단 |
| 마스터 전환 | Replica 재설정 등 수동 작업 필요 | `AUTO_POSITION=1`로 쉽게 처리 |
| 운영 안정성 | binlog 삭제 시 복제 실패 가능 | 상대적으로 안정적 |

---

## 직접 해보기 GTID

이거 이전은 강의에서 했던 실습과 똑같이 만듦.

**GTID를 쓰지 않을 때 위치(binlog pos) 기반으로 복제를 추적**

- **Read_Source_Log_Pos**
    - Replica의 IO Thread가 Source의 binlog를 어디까지 읽었는지
- **Exec_Source_Log_Pos**
    - Replica의 SQL Thread가 그중 어디까지 실제로 적용했는지
<img width="827" height="428" alt="image" src="https://github.com/user-attachments/assets/b1135c89-23bf-4cbe-90f9-6553f1c89519" />

### Source

<img width="521" height="427" alt="image" src="https://github.com/user-attachments/assets/e6b19662-22cb-4788-8ba8-02c0b8fb5d5a" />


**gtid_mode=ON**

- GTID 기반 복제 활성화
- 트랜잭션마다 UUID:번호 형식의 ID 부여

**enforce_gtid_consistency=ON**

- GTID와 충돌나는 SQL 실행 금지 (안전 모드)

### Replica

<img width="526" height="445" alt="image" src="https://github.com/user-attachments/assets/1eaf5f5a-00c0-4cdd-b472-16f4e737a45f" />


**log_replica_updates=ON**

- Replica가 Source에서 받은 트랜잭션도 자기 binlog에 기록함 (이 Replica를 나중에 또 다른 Replica의 Source로 쓰게 하기 위해서 - 체인 복제)

이렇게 추가해주고 dump 적용해서 확인.

잘 복제됨.

<img width="406" height="175" alt="image" src="https://github.com/user-attachments/assets/82c910b0-70b8-4912-85a7-1f7bb18afc73" />


<img width="662" height="416" alt="image" src="https://github.com/user-attachments/assets/f7d36b19-a262-4dd3-b8fb-b145793999fc" />


- **SOURCE_AUTO_POSITION=1**
    - replica가 binlog 파일/위치 대신 GTID 집합 기준으로 source에 요청하겠다는 의미
    - replica가 Executed_Gtid_Set 까지의 GTID들은 이미 실행했고, 이후의 트랜잭션만 요구 → source에서 이후의 gtid만 binlog에서 찾아서 보내주겠다.

### 이제 REPLICA RESET 시나리오 가정

기존에는 

<img width="513" height="501" alt="image" src="https://github.com/user-attachments/assets/6b1a2596-2b6a-48b6-9729-b44a855b7caa" />


<img width="902" height="127" alt="image" src="https://github.com/user-attachments/assets/884e432e-a073-4aca-b0ee-8f9eb89aad75" />


여기까지 데이터가 복제되어있는 상태. (22번까지)

### Replica

Replica 멈춤

<img width="468" height="83" alt="image" src="https://github.com/user-attachments/assets/799e3ef4-ee40-4fb7-b302-9cc3f8ba5d13" />


### Source

Source에 새로운 데이터 추가

<img width="740" height="206" alt="image" src="https://github.com/user-attachments/assets/fcedf338-0833-4a10-ba82-6e6ff8505d7a" />


### Replica

Replica에서 RESET

<img width="468" height="77" alt="image" src="https://github.com/user-attachments/assets/d777fad1-c13b-4094-865d-39ee3ff274bd" />


GTID 방식으로 다시 연결 및 복제

<img width="627" height="231" alt="image" src="https://github.com/user-attachments/assets/3413a614-46d0-48f7-91f9-e96e63dc00b0" />


이후  `SHOW REPLICA STATUS\G` 해보면

<img width="900" height="133" alt="image" src="https://github.com/user-attachments/assets/4c48740e-d6c9-45fc-b306-27b177483eca" />


<img width="536" height="505" alt="image" src="https://github.com/user-attachments/assets/f3eabec9-08c5-42d9-a438-4a76e4687d48" />
