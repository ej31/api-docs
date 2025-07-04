# 입금 주소 생성 요청 (Deposit Address Generation Request)

## 개요
새로운 입금 주소 생성을 요청하는 API입니다. 가상화폐 입금을 위한 주소를 생성할 때 사용합니다.

## API 정보
- **HTTP 메서드**: POST
- **엔드포인트**: `https://api.upbit.com/v1/deposits/generate_coin_address`

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|-----|-----|
| currency | String | 필수 | 화폐 코드 |
| net_type | String | 필수 | 입금 네트워크 |

## 응답 필드

### 응답 타입 1 (초기 요청 응답)
| 필드명 | 타입 | 설명 |
|-------|------|-----|
| success | Boolean | 요청 성공 여부 |
| message | String | 요청 결과 메시지 |

### 응답 타입 2 (주소 생성 완료 시)
| 필드명 | 타입 | 설명 |
|-------|------|-----|
| currency | String | 화폐 코드 |
| net_type | String | 입금 네트워크 |
| deposit_address | String | 입금 주소 |
| secondary_address | String | 보조 입금 주소 |

## 주의사항

### 비동기 처리
- **중요**: 입금 주소의 생성은 서버에서 비동기적으로 이뤄집니다
- 초기 응답에서는 주소가 즉시 제공되지 않을 수 있습니다
- 주소가 생성되지 않은 경우, 일정 시간 후 재시도가 필요합니다

### 재시도 로직
- 반복 호출 시 이전에 생성된 주소가 반환될 수 있습니다
- 주소가 아직 생성되지 않은 경우 적절한 대기 시간을 두고 재요청하세요

### 네트워크 지원
- 각 화폐별로 지원하는 네트워크가 다를 수 있습니다
- 올바른 네트워크 타입을 지정해야 합니다

## 사용 예시 플로우
1. 주소 생성 요청 API 호출
2. 초기 응답 확인 (success: true)
3. 일정 시간 후 개별 입금 주소 조회 API로 생성된 주소 확인
4. 주소가 null인 경우 재시도