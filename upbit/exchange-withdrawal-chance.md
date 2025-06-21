# 출금 가능 정보 (Withdrawal Possibility Information)

## 개요
해당 통화의 가능한 출금 정보를 확인하는 API입니다. 사용자의 보안 등급, 출금 한도, 수수료 등의 정보를 제공합니다.

## API 정보
- **HTTP Method**: GET
- **Endpoint**: `https://api.upbit.com/v1/withdraws/chance`
- **인증**: API Key 필요

## 요청 파라미터

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `currency` | String | 필수 | 화폐 코드 |
| `net_type` | String | 선택 | 출금 네트워크 |

## 응답 필드

### member_level (사용자 정보)
| 필드 | 타입 | 설명 |
|------|------|------|
| `security_level` | Number | 사용자 보안 등급 |
| `email_verified` | Boolean | 이메일 인증 여부 |
| `two_factor_auth_verified` | Boolean | 2단계 인증 활성화 여부 |

### currency (화폐 정보)
| 필드 | 타입 | 설명 |
|------|------|------|
| `code` | String | 화폐 코드 |
| `withdraw_fee` | NumberString | 출금 수수료 |
| `wallet_support` | Array | 입출금 지원 여부 |

### withdraw_limit (출금 한도)
| 필드 | 타입 | 설명 |
|------|------|------|
| `minimum` | NumberString | 최소 출금 수량 |
| `remaining_daily_fiat` | NumberString | 남은 일일 출금 한도 |
| `can_withdraw` | Boolean | 출금 지원 여부 |

## 응답 예시

```json
{
  "member_level": {
    "security_level": 3,
    "email_verified": true,
    "two_factor_auth_verified": true
  },
  "currency": {
    "code": "BTC",
    "withdraw_fee": "0.0005",
    "wallet_support": ["deposit", "withdraw"]
  },
  "withdraw_limit": {
    "minimum": "0.001",
    "remaining_daily_fiat": "1000000",
    "can_withdraw": true
  }
}
```

## 주요 변경사항
❗️ **v1.4.1부터**: "카카오페이 인증" 채널이 중단되었습니다. `kakao_pay_auth_verified` 필드가 제거되었습니다.

## 주의사항
- 출금하기 전에 반드시 이 API를 통해 출금 가능 여부를 확인해야 합니다.
- 보안 등급에 따라 출금 한도가 달라질 수 있습니다.
- 이메일 인증 및 2단계 인증이 필요할 수 있습니다.
- 네트워크별로 출금 수수료가 다를 수 있습니다.