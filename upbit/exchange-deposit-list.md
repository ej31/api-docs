# 입금 리스트 조회 (Deposit List Inquiry)

## 개요
사용자의 입금 내역 리스트를 조회하는 API입니다. 다양한 필터 조건을 통해 원하는 입금 내역을 검색할 수 있습니다.

## API 정보
- **HTTP 메서드**: GET
- **엔드포인트**: `https://api.upbit.com/v1/deposits`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|-----|-----|
| currency | String | 선택 | 화폐 코드 |
| state | String | 선택 | 입금 상태 |
| uuids[] | Array | 선택 | 입금 UUID 목록 |
| txids[] | Array | 선택 | 입금 트랜잭션 ID 목록 |
| limit | Number | 선택 | 결과 개수 제한 (기본값: 100, 최대: 100) |
| page | Number | 선택 | 페이지 번호 (기본값: 1) |
| order_by | String | 선택 | 정렬 순서 |

### order_by 옵션
- `asc`: 오름차순
- `desc`: 내림차순 (기본값)

## 응답 필드

| 필드명 | 타입 | 설명 |
|-------|------|-----|
| type | String | 입출금 타입 |
| uuid | String | 고유 입금 ID |
| currency | String | 화폐 코드 |
| net_type | String | 입금 네트워크 |
| txid | String | 트랜잭션 ID |
| state | String | 입금 상태 |
| created_at | DateString | 입금 생성 시간 |
| done_at | DateString | 입금 완료 시간 |
| amount | NumberString | 입금 수량 |
| fee | NumberString | 입금 수수료 |
| transaction_type | String | 입금 타입 |

### 입금 상태 (state) 옵션
- `PROCESSING`: 진행 중
- `ACCEPTED`: 완료
- `CANCELLED`: 취소됨
- `REJECTED`: 거절됨
- `TRAVEL_RULE_SUSPECTED`: 추가 검증 대기 중
- `REFUNDING`: 환불 진행 중
- `REFUNDED`: 환불됨

### 거래 타입 (transaction_type) 옵션
- `default`: 일반 입금
- `internal`: 즉시 입금

## 주의사항
- 다양한 프로그래밍 언어 SDK를 제공합니다 (Node, Python, Ruby, Java)
- 페이지네이션을 통해 대량의 데이터를 효율적으로 조회할 수 있습니다
- 상태별, 화폐별 필터링이 가능합니다