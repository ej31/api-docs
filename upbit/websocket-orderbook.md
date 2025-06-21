# Upbit WebSocket API - 호가(Orderbook) 정보

## 개요
WebSocket을 통해 실시간 호가 정보를 수신할 수 있는 API입니다.

## WebSocket 엔드포인트
```
wss://api.upbit.com/websocket/v1
```

## 요청 형식

### 기본 요청 구조
```json
[
  {
    "ticket": "test"
  },
  {
    "type": "orderbook",
    "codes": ["KRW-BTC", "KRW-ETH"],
    "level": 10000
  },
  {
    "format": "DEFAULT"
  }
]
```

### 주요 요청 필드

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `ticket` | String | 필수 | 요청 식별자 |
| `type` | String | 필수 | `"orderbook"` 고정 |
| `codes` | Array | 필수 | 마켓 코드 리스트 (대문자) |
| `level` | Double | 선택 | 호가 모아보기 단위 |
| `is_only_snapshot` | Boolean | 선택 | 스냅샷만 수신 (기본값: `false`) |
| `is_only_realtime` | Boolean | 선택 | 실시간 데이터만 수신 (기본값: `false`) |
| `format` | String | 선택 | 응답 형식 (DEFAULT/SIMPLE) |

### 호가 모아보기 단위 (level)
- **0 또는 생략**: 호가 모아보기 사용 안함
- **양수**: 해당 단위로 호가 모아보기 적용
- **예시**: `level: 1000` → 1000원 단위로 호가 모아보기

## 응답 구조

### 주요 응답 필드
| 필드명 | 타입 | 설명 |
|--------|------|------|
| `type` | String | `"orderbook"` 고정 |
| `code` | String | 마켓 코드 (예: KRW-BTC) |
| `timestamp` | Long | 타임스탬프 (밀리초) |
| `total_ask_size` | Double | 호가 매도 총 잔량 |
| `total_bid_size` | Double | 호가 매수 총 잔량 |
| `orderbook_units` | Array | 호가 정보 배열 |
| `level` | Double | 호가 모아보기 단위 |
| `stream_type` | String | 스트림 타입 (SNAPSHOT/REALTIME) |

### 호가 정보 (orderbook_units)
| 필드명 | 타입 | 설명 |
|--------|------|------|
| `ask_price` | Double | 매도호가 |
| `bid_price` | Double | 매수호가 |
| `ask_size` | Double | 매도 잔량 |
| `bid_size` | Double | 매수 잔량 |

## 응답 예시

### DEFAULT 포맷
```json
{
  "type": "orderbook",
  "code": "KRW-BTC",
  "timestamp": 1704096000123,
  "total_ask_size": 2.15486743,
  "total_bid_size": 1.87324521,
  "orderbook_units": [
    {
      "ask_price": 32290000.0,
      "bid_price": 32289000.0,
      "ask_size": 0.05423126,
      "bid_size": 0.12547892
    },
    {
      "ask_price": 32291000.0,
      "bid_price": 32288000.0,
      "ask_size": 0.08234567,
      "bid_size": 0.09876543
    }
  ],
  "level": 0,
  "stream_type": "REALTIME"
}
```

### SIMPLE 포맷
```json
{
  "ty": "orderbook",
  "cd": "KRW-BTC",
  "tms": 1704096000123,
  "tas": 2.15486743,
  "tbs": 1.87324521,
  "obu": [
    {
      "ap": 32290000.0,
      "bp": 32289000.0,
      "as": 0.05423126,
      "bs": 0.12547892
    }
  ],
  "lv": 0,
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

// 호가 단위 정보 구조체
type OrderbookUnit struct {
    AskPrice float64 `json:"ask_price"`
    BidPrice float64 `json:"bid_price"`
    AskSize  float64 `json:"ask_size"`
    BidSize  float64 `json:"bid_size"`
}

// 호가 단위 정보 구조체 (SIMPLE 포맷)
type OrderbookUnitSimple struct {
    AskPrice float64 `json:"ap"`
    BidPrice float64 `json:"bp"`
    AskSize  float64 `json:"as"`
    BidSize  float64 `json:"bs"`
}

// Orderbook 응답 구조체 (DEFAULT 포맷)
type OrderbookResponse struct {
    Type            string          `json:"type"`
    Code            string          `json:"code"`
    Timestamp       int64           `json:"timestamp"`
    TotalAskSize    float64         `json:"total_ask_size"`
    TotalBidSize    float64         `json:"total_bid_size"`
    OrderbookUnits  []OrderbookUnit `json:"orderbook_units"`
    Level           float64         `json:"level"`
    StreamType      string          `json:"stream_type"`
}

// Orderbook 응답 구조체 (SIMPLE 포맷)
type OrderbookResponseSimple struct {
    Type            string                `json:"ty"`
    Code            string                `json:"cd"`
    Timestamp       int64                 `json:"tms"`
    TotalAskSize    float64               `json:"tas"`
    TotalBidSize    float64               `json:"tbs"`
    OrderbookUnits  []OrderbookUnitSimple `json:"obu"`
    Level           float64               `json:"lv"`
    StreamType      string                `json:"st"`
}

// KST 시간 변환 함수
func (o *OrderbookResponse) GetTimestampKST() time.Time {
    return time.Unix(o.Timestamp/1000, 0).In(time.FixedZone("KST", 9*60*60))
}

// 호가 정보 요청 생성
func createOrderbookRequest(codes []string, level float64) []interface{} {
    typeField := map[string]interface{}{
        "type":  "orderbook",
        "codes": codes,
    }
    
    if level > 0 {
        typeField["level"] = level
    }
    
    return []interface{}{
        map[string]string{"ticket": "orderbook_request"},
        typeField,
        map[string]string{"format": "DEFAULT"},
    }
}

// 최우선 호가 정보 가져오기
func (o *OrderbookResponse) GetBestQuote() (askPrice, bidPrice, askSize, bidSize float64) {
    if len(o.OrderbookUnits) > 0 {
        unit := o.OrderbookUnits[0]
        return unit.AskPrice, unit.BidPrice, unit.AskSize, unit.BidSize
    }
    return 0, 0, 0, 0
}

// 호가 스프레드 계산
func (o *OrderbookResponse) GetSpread() float64 {
    if len(o.OrderbookUnits) > 0 {
        unit := o.OrderbookUnits[0]
        return unit.AskPrice - unit.BidPrice
    }
    return 0
}

// 호가 스프레드 비율 계산 (%)
func (o *OrderbookResponse) GetSpreadRate() float64 {
    if len(o.OrderbookUnits) > 0 {
        unit := o.OrderbookUnits[0]
        if unit.BidPrice > 0 {
            return ((unit.AskPrice - unit.BidPrice) / unit.BidPrice) * 100
        }
    }
    return 0
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

    // Orderbook 요청 (1000원 단위 모아보기)
    request := createOrderbookRequest([]string{"KRW-BTC", "KRW-ETH"}, 1000)
    if err := conn.WriteJSON(request); err != nil {
        log.Fatal("요청 전송 실패:", err)
    }

    // 응답 수신 및 처리
    for {
        var orderbook OrderbookResponse
        if err := conn.ReadJSON(&orderbook); err != nil {
            log.Fatal("응답 수신 실패:", err)
        }

        // KST 시간으로 변환
        kstTime := orderbook.GetTimestampKST()
        
        // 최우선 호가 정보
        askPrice, bidPrice, askSize, bidSize := orderbook.GetBestQuote()
        spread := orderbook.GetSpread()
        spreadRate := orderbook.GetSpreadRate()
        
        fmt.Printf("[%s] %s 호가 정보:\n", 
            kstTime.Format("15:04:05"), orderbook.Code)
        fmt.Printf("  매도: %.0f원 (%.4f개)\n", askPrice, askSize)
        fmt.Printf("  매수: %.0f원 (%.4f개)\n", bidPrice, bidSize)
        fmt.Printf("  스프레드: %.0f원 (%.4f%%)\n", spread, spreadRate)
        fmt.Printf("  총 매도량: %.4f개, 총 매수량: %.4f개\n\n", 
            orderbook.TotalAskSize, orderbook.TotalBidSize)
    }
}
```

### 호가 데이터 분석 예시
```go
package main

import (
    "fmt"
    "sort"
)

type OrderbookAnalyzer struct {
    orderbook *OrderbookResponse
}

func NewOrderbookAnalyzer(orderbook *OrderbookResponse) *OrderbookAnalyzer {
    return &OrderbookAnalyzer{orderbook: orderbook}
}

// 호가 분포 분석
func (oa *OrderbookAnalyzer) AnalyzeDistribution() {
    fmt.Printf("=== %s 호가 분포 분석 ===\n", oa.orderbook.Code)
    
    // 매도호가 분석
    fmt.Println("매도호가 (높은 가격부터):")
    askPrices := make([]float64, 0)
    for _, unit := range oa.orderbook.OrderbookUnits {
        if unit.AskPrice > 0 {
            askPrices = append(askPrices, unit.AskPrice)
        }
    }
    sort.Slice(askPrices, func(i, j int) bool {
        return askPrices[i] > askPrices[j]
    })
    
    for i, price := range askPrices {
        if i >= 5 { // 상위 5개만 표시
            break
        }
        // 해당 가격의 잔량 찾기
        for _, unit := range oa.orderbook.OrderbookUnits {
            if unit.AskPrice == price {
                fmt.Printf("  %d. %.0f원: %.4f개\n", i+1, price, unit.AskSize)
                break
            }
        }
    }
    
    // 매수호가 분석
    fmt.Println("매수호가 (높은 가격부터):")
    bidPrices := make([]float64, 0)
    for _, unit := range oa.orderbook.OrderbookUnits {
        if unit.BidPrice > 0 {
            bidPrices = append(bidPrices, unit.BidPrice)
        }
    }
    sort.Slice(bidPrices, func(i, j int) bool {
        return bidPrices[i] > bidPrices[j]
    })
    
    for i, price := range bidPrices {
        if i >= 5 { // 상위 5개만 표시
            break
        }
        // 해당 가격의 잔량 찾기
        for _, unit := range oa.orderbook.OrderbookUnits {
            if unit.BidPrice == price {
                fmt.Printf("  %d. %.0f원: %.4f개\n", i+1, price, unit.BidSize)
                break
            }
        }
    }
}

// 호가 불균형 계산
func (oa *OrderbookAnalyzer) CalculateImbalance() float64 {
    if oa.orderbook.TotalAskSize+oa.orderbook.TotalBidSize == 0 {
        return 0
    }
    
    // 매수 우세: 양수, 매도 우세: 음수
    return (oa.orderbook.TotalBidSize - oa.orderbook.TotalAskSize) / 
           (oa.orderbook.TotalBidSize + oa.orderbook.TotalAskSize) * 100
}
```

## 주의사항

- **마켓 코드**: 반드시 대문자로 입력 (예: "KRW-BTC")
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리해야 함
- **호가 순서**: 매도호가는 낮은 가격부터, 매수호가는 높은 가격부터 정렬
- **호가 개수**: 일반적으로 상하 각각 15개 호가 제공
- **모아보기**: `level` 설정으로 호가 단위 조정 가능
- **실시간성**: 호가 변동 시 즉시 업데이트
- **연결 유지**: 장시간 연결 시 ping/pong 메커니즘 구현 권장
- **데이터 검증**: 호가 데이터의 유효성 검증 권장 (가격 > 0, 잔량 >= 0)
- **에러 처리**: 네트워크 오류나 연결 끊김에 대한 적절한 예외 처리 필요