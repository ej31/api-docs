# Upbit WebSocket API 요청 포맷

## 기본 요청 구조

WebSocket 요청은 JSON 배열로 구성되며, 다음 주요 필드로 이루어집니다:

### 1. Ticket Field (필수)
- **목적**: 요청자 식별
- **필수 필드**
- **권장**: 유니크한 값 사용 (예: UUID)

```json
{
  "ticket": "unique_ticket_value"
}
```

### 2. Type Field (필수)
- **목적**: 수신할 시세 정보 지정
- **여러 타입 동시 요청 가능**
- **선택적 필드**: 
  - `is_only_snapshot`: 스냅샷 데이터만 수신
  - `is_only_realtime`: 실시간 데이터만 수신

```json
{
  "type": "ticker",
  "codes": ["KRW-BTC"],
  "is_only_snapshot": false,
  "is_only_realtime": false
}
```

### 3. Format Field (선택)
- **목적**: 응답 데이터 형식 지정
- **옵션**:
  - `DEFAULT`: 기본 형식 (모든 필드 포함)
  - `SIMPLE`: 축약형 (트래픽 절감)

```json
{
  "format": "DEFAULT"
}
```

## 완전한 요청 형식 예시

### 기본 요청
```json
[
  {
    "ticket": "unique_ticket_value"
  },
  {
    "type": "ticker",
    "codes": ["KRW-BTC"]
  },
  {
    "format": "DEFAULT"
  }
]
```

### 복합 요청 (여러 타입)
```json
[
  {
    "ticket": "multi_request_ticket"
  },
  {
    "type": "ticker",
    "codes": ["KRW-BTC", "KRW-ETH"]
  },
  {
    "type": "trade",
    "codes": ["KRW-BTC"]
  },
  {
    "format": "SIMPLE"
  }
]
```

### 스냅샷 전용 요청
```json
[
  {
    "ticket": "snapshot_only"
  },
  {
    "type": "orderbook",
    "codes": ["KRW-BTC"],
    "is_only_snapshot": true
  }
]
```

### 실시간 전용 요청
```json
[
  {
    "ticket": "realtime_only"
  },
  {
    "type": "trade",
    "codes": ["KRW-BTC"],
    "is_only_realtime": true
  }
]
```

## Go 언어 구현 예시

```go
package main

import (
    "encoding/json"
    "github.com/gorilla/websocket"
    "github.com/google/uuid"
)

// WebSocket 요청 구조체
type WebSocketRequest struct {
    Ticket string `json:"ticket,omitempty"`
    Type   string `json:"type,omitempty"`
    Codes  []string `json:"codes,omitempty"`
    Format string `json:"format,omitempty"`
    IsOnlySnapshot bool `json:"is_only_snapshot,omitempty"`
    IsOnlyRealtime bool `json:"is_only_realtime,omitempty"`
}

func createTickerRequest(codes []string) []WebSocketRequest {
    return []WebSocketRequest{
        {Ticket: uuid.New().String()},
        {
            Type:  "ticker",
            Codes: codes,
        },
        {Format: "DEFAULT"},
    }
}

func sendRequest(conn *websocket.Conn, request []WebSocketRequest) error {
    return conn.WriteJSON(request)
}
```

## 주요 특징

- **JSON 배열 형식**: 모든 요청은 JSON 배열로 구성
- **각 필드는 별도 객체**: 각 설정은 독립적인 JSON 객체
- **유연한 요청 구조**: 필요에 따라 필드 조합 가능
- **실시간/스냅샷 선택**: 데이터 수신 방식 선택 가능

## 주의사항

- **고유 티켓**: `ticket` 값은 요청마다 고유해야 함
- **대문자 코드**: 마켓 코드는 반드시 대문자 사용 (예: "KRW-BTC")
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리
- **배열 순서**: JSON 배열의 순서는 ticket → type → format 순서 권장
- **포맷 영향**: format 설정은 현재 연결의 모든 응답에 영향을 줄 수 있음