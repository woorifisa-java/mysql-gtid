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
## 기존 File/Position 방식
## GTID 방식
