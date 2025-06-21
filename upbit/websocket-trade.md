# Upbit WebSocket API - 체결(Trade) 정보

## 개요
WebSocket을 통해 실시간 체결 데이터를 수신할 수 있는 API입니다.

## WebSocket 엔드포인트
```
wss://api.upbit.com/websocket/v1
```

## 요청 형식

### 기본 요청 구조
```json
[
  {
    "ticket": "test example"
  },
  {
    "type": "trade",
    "codes": ["KRW-BTC", "KRW-ETH"]
  },
  {
    "format": "DEFAULT"
  }
]
```

### 주요 요청 필드
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `ticket` | String | 필수 | 요청 식별자 |
| `type` | String | 필수 | `"trade"` 고정 |
| `codes` | Array | 필수 | 조회할 마켓 코드 목록 (대문자) |
| `is_only_snapshot` | Boolean | 선택 | 스냅샷 데이터만 수신 |
| `is_only_realtime` | Boolean | 선택 | 실시간 데이터만 수신 |
| `format` | String | 선택 | 응답 형식 (DEFAULT/SIMPLE) |

## 응답 필드

### 주요 응답 필드
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `type` | 데이터 타입 | String | "trade" |
| `code` | 마켓 코드 | String | 예: "KRW-BTC" |
| `trade_price` | 체결 가격 | Double | |
| `trade_volume` | 체결량 | Double | |
| `ask_bid` | 매수/매도 구분 | String | ASK(매도)/BID(매수) |
| `prev_closing_price` | 전일 종가 | Double | |
| `change` | 전일 대비 | String | RISE/FALL/EVEN |
| `change_price` | 변화액 | Double | |
| `trade_date` | 체결 일자(UTC) | String | yyyyMMdd |
| `trade_time` | 체결 시각(UTC) | String | HHmmss |
| `trade_timestamp` | 체결 타임스탬프 | Long | 밀리초 |
| `timestamp` | 타임스탬프 | Long | 밀리초 |
| `sequential_id` | 체결 번호 | Long | 해당 일자의 체결 번호 |
| `stream_type` | 스트림 타입 | String | SNAPSHOT/REALTIME |

## 응답 예시

### DEFAULT 포맷
```json
{
  "type": "trade",
  "code": "KRW-BTC", 
  "trade_price": 100473000.00,
  "trade_volume": 0.00014208,
  "ask_bid": "BID",
  "prev_closing_price": 100450000.00,
  "change": "RISE",
  "change_price": 23000.00,
  "trade_date": "20240101",
  "trade_time": "120000",
  "trade_timestamp": 1704096000000,
  "timestamp": 1704096000123,
  "sequential_id": 15420000000,
  "stream_type": "REALTIME"
}
```

### SIMPLE 포맷
```json
{
  "ty": "trade",
  "cd": "KRW-BTC",
  "tp": 100473000.00,
  "tv": 0.00014208,
  "ab": "BID",
  "pcp": 100450000.00,
  "c": "RISE",
  "cp": 23000.00,
  "td": "20240101",
  "tt": "120000",
  "ttm": 1704096000000,
  "ttms": 1704096000123,
  "sid": 15420000000,
  "st": "REALTIME"
}
```

## Go 언어 구현 예시

### 구조체 정의
```go
package main

import (
    "encoding/json"
    "time"
)

// Trade 응답 구조체 (DEFAULT 포맷)
type TradeResponse struct {
    Type             string  `json:"type"`
    Code             string  `json:"code"`
    TradePrice       float64 `json:"trade_price"`
    TradeVolume      float64 `json:"trade_volume"`
    AskBid           string  `json:"ask_bid"`
    PrevClosingPrice float64 `json:"prev_closing_price"`
    Change           string  `json:"change"`
    ChangePrice      float64 `json:"change_price"`
    TradeDate        string  `json:"trade_date"`
    TradeTime        string  `json:"trade_time"`
    TradeTimestamp   int64   `json:"trade_timestamp"`
    Timestamp        int64   `json:"timestamp"`
    SequentialId     int64   `json:"sequential_id"`
    StreamType       string  `json:"stream_type"`
}

// Trade 응답 구조체 (SIMPLE 포맷)
type TradeResponseSimple struct {
    Type             string  `json:"ty"`
    Code             string  `json:"cd"`
    TradePrice       float64 `json:"tp"`
    TradeVolume      float64 `json:"tv"`
    AskBid           string  `json:"ab"`
    PrevClosingPrice float64 `json:"pcp"`
    Change           string  `json:"c"`
    ChangePrice      float64 `json:"cp"`
    TradeDate        string  `json:"td"`
    TradeTime        string  `json:"tt"`
    TradeTimestamp   int64   `json:"ttm"`
    Timestamp        int64   `json:"ttms"`
    SequentialId     int64   `json:"sid"`
    StreamType       string  `json:"st"`
}

// KST 시간 변환 함수
func (t *TradeResponse) GetTradeTimeKST() time.Time {
    return time.Unix(t.TradeTimestamp/1000, 0).In(time.FixedZone("KST", 9*60*60))
}

// 체결 정보 요청 생성
func createTradeRequest(codes []string) []interface{} {
    return []interface{}{
        map[string]string{"ticket": "trade_request"},
        map[string]interface{}{
            "type":  "trade",
            "codes": codes,
        },
        map[string]string{"format": "DEFAULT"},
    }
}

// 매수/매도 구분 한글 변환
func (t *TradeResponse) GetAskBidKorean() string {
    switch t.AskBid {
    case "ASK":
        return "매도"
    case "BID":
        return "매수"
    default:
        return "알 수 없음"
    }
}

// 전일 대비 상태 한글 변환
func (t *TradeResponse) GetChangeKorean() string {
    switch t.Change {
    case "RISE":
        return "상승"
    case "FALL":
        return "하락"
    case "EVEN":
        return "보합"
    default:
        return "알 수 없음"
    }
}
```

### 사용 예시
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

    // Trade 요청
    request := createTradeRequest([]string{"KRW-BTC", "KRW-ETH"})
    if err := conn.WriteJSON(request); err != nil {
        log.Fatal("요청 전송 실패:", err)
    }

    // 응답 수신 및 처리
    for {
        var trade TradeResponse
        if err := conn.ReadJSON(&trade); err != nil {
            log.Fatal("응답 수신 실패:", err)
        }

        // KST 시간으로 변환
        kstTime := trade.GetTradeTimeKST()
        
        fmt.Printf("[%s] %s: %.0f원 (%.4f개) - %s (%s)\n", 
            kstTime.Format("15:04:05"), 
            trade.Code, 
            trade.TradePrice,
            trade.TradeVolume,
            trade.GetAskBidKorean(),
            trade.GetChangeKorean(),
        )
    }
}
```

### 실시간 체결 데이터 저장 예시
```go
package main

import (
    "database/sql"
    "log"
    "time"
    _ "github.com/lib/pq"
)

type TradeCollector struct {
    db   *sql.DB
    conn *websocket.Conn
}

func NewTradeCollector(dbConnStr string) (*TradeCollector, error) {
    db, err := sql.Open("postgres", dbConnStr)
    if err != nil {
        return nil, err
    }
    
    conn, _, err := websocket.DefaultDialer.Dial("wss://api.upbit.com/websocket/v1", nil)
    if err != nil {
        return nil, err
    }
    
    return &TradeCollector{db: db, conn: conn}, nil
}

func (tc *TradeCollector) SaveTrade(trade *TradeResponse) error {
    query := `
        INSERT INTO upbit_trades (
            code, trade_price, trade_volume, ask_bid, 
            trade_timestamp, sequential_id, created_at
        ) VALUES ($1, $2, $3, $4, $5, $6, $7)
    `
    
    _, err := tc.db.Exec(query,
        trade.Code,
        trade.TradePrice,
        trade.TradeVolume,
        trade.AskBid,
        time.Unix(trade.TradeTimestamp/1000, 0),
        trade.SequentialId,
        time.Now(),
    )
    
    return err
}

func (tc *TradeCollector) Start(codes []string) {
    request := createTradeRequest(codes)
    if err := tc.conn.WriteJSON(request); err != nil {
        log.Fatal("요청 전송 실패:", err)
    }

    for {
        var trade TradeResponse
        if err := tc.conn.ReadJSON(&trade); err != nil {
            log.Printf("응답 수신 실패: %v", err)
            continue
        }

        if err := tc.SaveTrade(&trade); err != nil {
            log.Printf("체결 데이터 저장 실패: %v", err)
        }
    }
}
```

## 주의사항

- **마켓 코드**: 반드시 대문자로 입력 (예: "KRW-BTC")
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리해야 함
- **체결 번호**: `sequential_id`는 해당 일자 내에서만 유니크함
- **실시간성**: 체결이 발생할 때마다 즉시 데이터 전송
- **스냅샷**: 연결 직후 최근 체결 데이터를 스냅샷으로 제공
- **데이터 순서**: 체결 시간 순서대로 데이터 제공
- **연결 유지**: 장시간 연결 시 ping/pong 메커니즘 구현 권장
- **에러 처리**: 네트워크 오류나 연결 끊김에 대한 적절한 예외 처리 필요