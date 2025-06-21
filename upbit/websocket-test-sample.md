# Upbit WebSocket API 테스트 및 요청 예제

## 테스트 방법

### wscat 사용
```shell
$ npm install -g wscat
$ wscat -c wss://api.upbit.com/websocket/v1
```

### telsocket 사용
```shell
$ telsocket -url wss://api.upbit.com/websocket/v1
```

## 기본 요청 예시

### 원화-비트코인(KRW-BTC) 마켓 실시간 정보 요청
```json
[{"ticket":"test"},{"type":"ticker","codes":["KRW-BTC"]}]
```

### 여러 마켓 체결 정보 요청 (간소화된 포맷)
```json
[
    {"ticket":"test"},
    {"type":"trade","codes":["KRW-BTC","BTC-BCH"]},
    {"format":"SIMPLE"}
]
```

## 다양한 요청 유형

### 1. 복수 마켓 체결 정보
```json
[
    {"ticket":"UNIQUE_TICKET"},
    {"type":"trade","codes":["KRW-BTC","BTC-XRP"]}
]
```

### 2. 복수 마켓 호가 정보
```json
[
    {"ticket":"UNIQUE_TICKET"},
    {"type":"orderbook","codes":["KRW-BTC","BTC-XRP"]}
]
```

### 3. 혼합 정보 요청
```json
[
    {"ticket":"UNIQUE_TICKET"},
    {"type":"trade","codes":["KRW-BTC"]},
    {"type":"orderbook","codes":["KRW-ETH"]},
    {"type":"ticker", "codes":["KRW-EOS"]}
]
```

### 4. Private 데이터 요청 (내 주문)
```json
[
    {"ticket":"UNIQUE_TICKET"},
    {"type":"myOrder","codes":["KRW-BTC"]}
]
```

### 5. Private 데이터 요청 (내 자산)
```json
[
    {"ticket":"UNIQUE_TICKET"},
    {"type":"myAsset"}
]
```

## Go 언어 테스트 예제

```go
package main

import (
    "encoding/json"
    "fmt"
    "github.com/gorilla/websocket"
    "log"
)

func main() {
    // WebSocket 연결
    conn, _, err := websocket.DefaultDialer.Dial("wss://api.upbit.com/websocket/v1", nil)
    if err != nil {
        log.Fatal("WebSocket 연결 실패:", err)
    }
    defer conn.Close()

    // 요청 메시지 생성
    request := []interface{}{
        map[string]string{"ticket": "test"},
        map[string]interface{}{
            "type":  "ticker",
            "codes": []string{"KRW-BTC"},
        },
    }

    // 요청 전송
    if err := conn.WriteJSON(request); err != nil {
        log.Fatal("메시지 전송 실패:", err)
    }

    // 응답 수신
    for {
        var response map[string]interface{}
        if err := conn.ReadJSON(&response); err != nil {
            log.Fatal("메시지 수신 실패:", err)
        }
        
        // 응답 출력
        jsonData, _ := json.MarshalIndent(response, "", "  ")
        fmt.Println(string(jsonData))
    }
}
```

## 주의사항
- **고유 티켓**: 요청 시 `ticket` 필드는 고유한 값으로 설정
- **타입 지정**: `type` 필드로 정보 유형 구분
- **코드 형식**: 마켓 코드는 대문자로 입력 (예: "KRW-BTC")
- **시간대 처리**: 모든 시간 데이터는 KST 기준으로 처리
- **연결 유지**: 장시간 연결 시 ping/pong 메커니즘 구현 권장