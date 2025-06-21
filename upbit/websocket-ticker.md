# Upbit WebSocket API - 현재가(Ticker) 정보

## 개요
WebSocket을 통해 실시간 암호화폐 현재가 정보를 수신할 수 있는 API입니다.

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
    "type": "ticker",
    "codes": ["KRW-BTC", "KRW-ETH"],
    "is_only_snapshot": false,
    "is_only_realtime": false
  },
  {
    "format": "DEFAULT"
  }
]
```

### 주요 요청 파라미터
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `ticket` | String | 필수 | 요청 식별자 |
| `type` | String | 필수 | `"ticker"` 고정 |
| `codes` | Array | 필수 | 마켓 코드 리스트 (대문자) |
| `is_only_snapshot` | Boolean | 선택 | 스냅샷 데이터만 수신 |
| `is_only_realtime` | Boolean | 선택 | 실시간 데이터만 수신 |
| `format` | String | 선택 | 응답 형식 (DEFAULT/SIMPLE) |

## 응답 필드

### 주요 응답 필드
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `type` | 데이터 타입 | String | "ticker" |
| `code` | 마켓 코드 | String | 예: "KRW-BTC" |
| `trade_price` | 현재가 | Double | 최근 체결 가격 |
| `change` | 전일 대비 상태 | String | RISE/FALL/EVEN |
| `change_rate` | 전일 대비 등락율 | Double | 소수점 형태 |
| `change_price` | 전일 대비 등락액 | Double | |
| `signed_change_rate` | 부호가 있는 등락율 | Double | |
| `signed_change_price` | 부호가 있는 등락액 | Double | |
| `trade_volume` | 가장 최근 거래량 | Double | |
| `acc_trade_volume` | 누적 거래량(UTC 0시 기준) | Double | |
| `acc_trade_volume_24h` | 24시간 누적 거래량 | Double | |
| `acc_trade_price` | 누적 거래대금(UTC 0시 기준) | Double | |
| `acc_trade_price_24h` | 24시간 누적 거래대금 | Double | |
| `trade_date` | 최근 거래 일자(UTC) | String | yyyyMMdd |
| `trade_time` | 최근 거래 시각(UTC) | String | HHmmss |
| `trade_timestamp` | 체결 타임스탬프 | Long | 밀리초 |
| `ask_bid` | 매수/매도 구분 | String | ASK/BID |
| `acc_ask_volume` | 누적 매도량 | Double | |
| `acc_bid_volume` | 누적 매수량 | Double | |
| `highest_52_week_price` | 52주 신고가 | Double | |
| `highest_52_week_date` | 52주 신고가 달성일 | String | |
| `lowest_52_week_price` | 52주 신저가 | Double | |
| `lowest_52_week_date` | 52주 신저가 달성일 | String | |
| `trade_status` | 거래 상태 | String | |
| `market_state` | 마켓 경고 구분 | String | |
| `market_state_for_ios` | iOS 앱용 마켓 상태 | String | |
| `is_trading_suspended` | 거래 정지 여부 | Boolean | |
| `delisting_date` | 상장폐지일 | String | |
| `market_warning` | 마켓 경고 구분 | String | |
| `timestamp` | 타임스탬프 | Long | 밀리초 |
| `stream_type` | 스트림 타입 | String | SNAPSHOT/REALTIME |

## 응답 예시

### DEFAULT 포맷
```json
{
  "type": "ticker",
  "code": "KRW-BTC",
  "opening_price": 32285000,
  "high_price": 32400000,
  "low_price": 32100000,
  "trade_price": 32287000,
  "prev_closing_price": 32280000,
  "change": "RISE",
  "change_rate": 0.0002168050,
  "signed_change_rate": 0.0002168050,
  "change_price": 7000,
  "signed_change_price": 7000,
  "trade_volume": 0.03073852,
  "acc_trade_volume": 2624.53894846,
  "acc_trade_volume_24h": 3711.41927673,
  "acc_trade_price": 84795524793.68954,
  "acc_trade_price_24h": 119897717952.65657,
  "trade_date": "20240101",
  "trade_time": "120000",
  "trade_timestamp": 1704096000000,
  "ask_bid": "BID",
  "acc_ask_volume": 1274.29425598,
  "acc_bid_volume": 1350.24469248,
  "highest_52_week_price": 50000000,
  "highest_52_week_date": "2023-12-15",
  "lowest_52_week_price": 20000000,
  "lowest_52_week_date": "2023-01-01",
  "market_state": "ACTIVE",
  "is_trading_suspended": false,
  "delisting_date": null,
  "market_warning": "NONE",
  "timestamp": 1704096000123,
  "stream_type": "REALTIME"
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

// Ticker 응답 구조체
type TickerResponse struct {
    Type                    string  `json:"type"`
    Code                    string  `json:"code"`
    OpeningPrice           float64 `json:"opening_price"`
    HighPrice              float64 `json:"high_price"`
    LowPrice               float64 `json:"low_price"`
    TradePrice             float64 `json:"trade_price"`
    PrevClosingPrice       float64 `json:"prev_closing_price"`
    Change                 string  `json:"change"`
    ChangeRate             float64 `json:"change_rate"`
    SignedChangeRate       float64 `json:"signed_change_rate"`
    ChangePrice            float64 `json:"change_price"`
    SignedChangePrice      float64 `json:"signed_change_price"`
    TradeVolume            float64 `json:"trade_volume"`
    AccTradeVolume         float64 `json:"acc_trade_volume"`
    AccTradeVolume24h      float64 `json:"acc_trade_volume_24h"`
    AccTradePrice          float64 `json:"acc_trade_price"`
    AccTradePrice24h       float64 `json:"acc_trade_price_24h"`
    TradeDate              string  `json:"trade_date"`
    TradeTime              string  `json:"trade_time"`
    TradeTimestamp         int64   `json:"trade_timestamp"`
    AskBid                 string  `json:"ask_bid"`
    AccAskVolume           float64 `json:"acc_ask_volume"`
    AccBidVolume           float64 `json:"acc_bid_volume"`
    Highest52WeekPrice     float64 `json:"highest_52_week_price"`
    Highest52WeekDate      string  `json:"highest_52_week_date"`
    Lowest52WeekPrice      float64 `json:"lowest_52_week_price"`
    Lowest52WeekDate       string  `json:"lowest_52_week_date"`
    MarketState            string  `json:"market_state"`
    IsTradingSuspended     bool    `json:"is_trading_suspended"`
    DelistingDate          *string `json:"delisting_date"`
    MarketWarning          string  `json:"market_warning"`
    Timestamp              int64   `json:"timestamp"`
    StreamType             string  `json:"stream_type"`
}

// KST 시간 변환 함수
func (t *TickerResponse) GetTradeTimeKST() time.Time {
    return time.Unix(t.TradeTimestamp/1000, 0).In(time.FixedZone("KST", 9*60*60))
}

// 현재가 정보 요청 생성
func createTickerRequest(codes []string) []interface{} {
    return []interface{}{
        map[string]string{"ticket": "ticker_request"},
        map[string]interface{}{
            "type":  "ticker",
            "codes": codes,
        },
        map[string]string{"format": "DEFAULT"},
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

    // Ticker 요청
    request := createTickerRequest([]string{"KRW-BTC", "KRW-ETH"})
    if err := conn.WriteJSON(request); err != nil {
        log.Fatal("요청 전송 실패:", err)
    }

    // 응답 수신 및 처리
    for {
        var ticker TickerResponse
        if err := conn.ReadJSON(&ticker); err != nil {
            log.Fatal("응답 수신 실패:", err)
        }

        // KST 시간으로 변환
        kstTime := ticker.GetTradeTimeKST()
        
        fmt.Printf("[%s] %s: %.0f원 (%.2f%%)\n", 
            kstTime.Format("2006-01-02 15:04:05"), 
            ticker.Code, 
            ticker.TradePrice,
            ticker.SignedChangeRate*100,
        )
    }
}
```

## 주의사항

- **마켓 코드**: 반드시 대문자로 입력 (예: "KRW-BTC")
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리해야 함
- **데이터 타입**: 숫자 필드는 모두 float64/Double 타입
- **스트림 타입**: SNAPSHOT(초기 데이터) / REALTIME(실시간 업데이트) 구분
- **연결 유지**: 장시간 연결 시 ping/pong 메커니즘 구현 권장
- **에러 처리**: 네트워크 오류나 연결 끊김에 대한 적절한 예외 처리 필요