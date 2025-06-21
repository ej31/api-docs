# Upbit WebSocket API: 캔들(Candle) 정보

## 개요
WebSocket을 통해 실시간 캔들 데이터를 수신할 수 있는 API입니다.

## 중요 사항
- **베타 서비스**: 현재 베타 버전으로 제공
- **체결 기반**: 체결이 발생한 경우에만 캔들 생성
- **전송 주기**: 1초마다 데이터 전송 (체결 시 값 변경 시)

## WebSocket 엔드포인트
```
wss://api.upbit.com/websocket/v1
```

## 지원되는 캔들 타입

| 타입 | 설명 | 비고 |
|------|------|------|
| `candle.1s` | 1초봉 | |
| `candle.1m` | 1분봉 | |
| `candle.3m` | 3분봉 | |
| `candle.5m` | 5분봉 | |
| `candle.10m` | 10분봉 | |
| `candle.15m` | 15분봉 | |
| `candle.30m` | 30분봉 | |
| `candle.60m` | 60분봉 | |
| `candle.240m` | 240분봉 (4시간봉) | |

## 요청 형식

### 기본 요청 구조
```json
[
  {
    "ticket": "test"
  },
  {
    "type": "candle.1m",
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
| `type` | String | 필수 | 캔들 타입 (예: "candle.1m") |
| `codes` | Array | 필수 | 마켓 코드 리스트 (대문자) |
| `is_only_snapshot` | Boolean | 선택 | 스냅샷 데이터만 수신 |
| `is_only_realtime` | Boolean | 선택 | 실시간 데이터만 수신 |
| `format` | String | 선택 | 응답 형식 (DEFAULT/SIMPLE) |

## 응답 필드

### 주요 응답 필드
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `type` | 캔들 타입 | String | 예: "candle.1m" |
| `code` | 마켓 코드 | String | 예: "KRW-BTC" |
| `candle_date_time_utc` | UTC 기준 캔들 시각 | String | ISO 8601 형식 |
| `candle_date_time_kst` | KST 기준 캔들 시각 | String | ISO 8601 형식 |
| `opening_price` | 시가 | Double | |
| `high_price` | 고가 | Double | |
| `low_price` | 저가 | Double | |
| `trade_price` | 종가 | Double | |
| `timestamp` | 타임스탬프 | Long | 밀리초 |
| `candle_acc_trade_price` | 캔들 기간 중 누적 거래대금 | Double | |
| `candle_acc_trade_volume` | 캔들 기간 중 누적 거래량 | Double | |
| `unit` | 분 단위 | Integer | 예: 1, 3, 5, 10, 15, 30, 60, 240 |
| `prev_closing_price` | 전일 종가 | Double | |
| `change_price` | 전일 대비 변화액 | Double | |
| `change_rate` | 전일 대비 변화율 | Double | |
| `converted_trade_price` | 종가 환산화폐 단위 | Double | |
| `stream_type` | 스트림 타입 | String | SNAPSHOT/REALTIME |

## 응답 예시

### 1분봉 DEFAULT 포맷
```json
{
  "type": "candle.1m",
  "code": "KRW-BTC",
  "candle_date_time_utc": "2024-01-01T12:00:00Z",
  "candle_date_time_kst": "2024-01-01T21:00:00+09:00",
  "opening_price": 32285000.0,
  "high_price": 32290000.0,
  "low_price": 32280000.0,
  "trade_price": 32287000.0,
  "timestamp": 1704096000000,
  "candle_acc_trade_price": 1500000000.50,
  "candle_acc_trade_volume": 46.75234156,
  "unit": 1,
  "prev_closing_price": 32280000.0,
  "change_price": 7000.0,
  "change_rate": 0.0002168,
  "converted_trade_price": null,
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
    "strconv"
    "strings"
)

// Candle 응답 구조체
type CandleResponse struct {
    Type                   string  `json:"type"`
    Code                   string  `json:"code"`
    CandleDateTimeUTC      string  `json:"candle_date_time_utc"`
    CandleDateTimeKST      string  `json:"candle_date_time_kst"`
    OpeningPrice           float64 `json:"opening_price"`
    HighPrice              float64 `json:"high_price"`
    LowPrice               float64 `json:"low_price"`
    TradePrice             float64 `json:"trade_price"`
    Timestamp              int64   `json:"timestamp"`
    CandleAccTradePrice    float64 `json:"candle_acc_trade_price"`
    CandleAccTradeVolume   float64 `json:"candle_acc_trade_volume"`
    Unit                   int     `json:"unit"`
    PrevClosingPrice       float64 `json:"prev_closing_price"`
    ChangePrice            float64 `json:"change_price"`
    ChangeRate             float64 `json:"change_rate"`
    ConvertedTradePrice    *float64 `json:"converted_trade_price"`
    StreamType             string  `json:"stream_type"`
}

// KST 시간 파싱 함수
func (c *CandleResponse) GetCandleTimeKST() (time.Time, error) {
    return time.Parse(time.RFC3339, c.CandleDateTimeKST)
}

// UTC 시간 파싱 함수
func (c *CandleResponse) GetCandleTimeUTC() (time.Time, error) {
    return time.Parse(time.RFC3339, c.CandleDateTimeUTC)
}

// 캔들 타입에서 분 단위 추출
func (c *CandleResponse) GetIntervalMinutes() int {
    // "candle.1m" -> 1, "candle.60m" -> 60
    parts := strings.Split(c.Type, ".")
    if len(parts) != 2 {
        return 0
    }
    
    timeStr := strings.TrimSuffix(parts[1], "m")
    if timeStr == "1s" {
        return 0 // 초봉의 경우
    }
    
    interval, err := strconv.Atoi(timeStr)
    if err != nil {
        return 0
    }
    return interval
}

// 캔들 요청 생성 함수
func createCandleRequest(candleType string, codes []string) []interface{} {
    return []interface{}{
        map[string]string{"ticket": "candle_request"},
        map[string]interface{}{
            "type":  candleType,
            "codes": codes,
        },
        map[string]string{"format": "DEFAULT"},
    }
}

// 가격 변화율 백분율 변환
func (c *CandleResponse) GetChangeRatePercent() float64 {
    return c.ChangeRate * 100
}

// 캔들 상태 문자열 반환
func (c *CandleResponse) GetCandleState() string {
    if c.TradePrice > c.OpeningPrice {
        return "상승"
    } else if c.TradePrice < c.OpeningPrice {
        return "하락"
    }
    return "보합"
}

// 변동성 계산 (고가-저가)/시가 * 100
func (c *CandleResponse) GetVolatility() float64 {
    if c.OpeningPrice == 0 {
        return 0
    }
    return ((c.HighPrice - c.LowPrice) / c.OpeningPrice) * 100
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

    // 1분봉 캔들 요청
    request := createCandleRequest("candle.1m", []string{"KRW-BTC", "KRW-ETH"})
    if err := conn.WriteJSON(request); err != nil {
        log.Fatal("요청 전송 실패:", err)
    }

    // 응답 수신 및 처리
    for {
        var candle CandleResponse
        if err := conn.ReadJSON(&candle); err != nil {
            log.Fatal("응답 수신 실패:", err)
        }

        // KST 시간으로 변환
        kstTime, err := candle.GetCandleTimeKST()
        if err != nil {
            log.Printf("시간 파싱 오류: %v", err)
            continue
        }
        
        fmt.Printf("[%s] %s %s캔들:\n", 
            kstTime.Format("2006-01-02 15:04:05"), 
            candle.Code,
            candle.Type,
        )
        fmt.Printf("  OHLC: %.0f/%.0f/%.0f/%.0f\n", 
            candle.OpeningPrice, candle.HighPrice, 
            candle.LowPrice, candle.TradePrice,
        )
        fmt.Printf("  거래량: %.4f개, 거래대금: %.0f원\n", 
            candle.CandleAccTradeVolume, candle.CandleAccTradePrice,
        )
        fmt.Printf("  변화: %s (%.2f%%), 변동성: %.2f%%\n\n", 
            candle.GetCandleState(),
            candle.GetChangeRatePercent(),
            candle.GetVolatility(),
        )
    }
}
```

### 캔들 데이터 저장 예시
```go
package main

import (
    "database/sql"
    "log"
    "time"
    _ "github.com/lib/pq"
)

type CandleCollector struct {
    db   *sql.DB
    conn *websocket.Conn
}

func NewCandleCollector(dbConnStr string) (*CandleCollector, error) {
    db, err := sql.Open("postgres", dbConnStr)
    if err != nil {
        return nil, err
    }
    
    conn, _, err := websocket.DefaultDialer.Dial("wss://api.upbit.com/websocket/v1", nil)
    if err != nil {
        return nil, err
    }
    
    return &CandleCollector{db: db, conn: conn}, nil
}

func (cc *CandleCollector) SaveCandle(candle *CandleResponse) error {
    // KST 시간으로 파싱
    kstTime, err := candle.GetCandleTimeKST()
    if err != nil {
        return err
    }
    
    utcTime, err := candle.GetCandleTimeUTC()
    if err != nil {
        return err
    }
    
    query := `
        INSERT INTO upbit_candle_1m (
            code, candle_date_time_utc, candle_date_time_kst,
            opening_price, high_price, low_price, trade_price,
            timestamp, candle_acc_trade_price, candle_acc_trade_volume, unit
        ) VALUES ($1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11)
        ON CONFLICT (code, candle_date_time_kst) 
        DO UPDATE SET
            opening_price = EXCLUDED.opening_price,
            high_price = EXCLUDED.high_price,
            low_price = EXCLUDED.low_price,
            trade_price = EXCLUDED.trade_price,
            candle_acc_trade_price = EXCLUDED.candle_acc_trade_price,
            candle_acc_trade_volume = EXCLUDED.candle_acc_trade_volume
    `
    
    _, err = cc.db.Exec(query,
        candle.Code,
        utcTime,
        kstTime,
        candle.OpeningPrice,
        candle.HighPrice,
        candle.LowPrice,
        candle.TradePrice,
        time.Unix(candle.Timestamp/1000, 0),
        candle.CandleAccTradePrice,
        candle.CandleAccTradeVolume,
        candle.Unit,
    )
    
    return err
}

func (cc *CandleCollector) Start(candleType string, codes []string) {
    request := createCandleRequest(candleType, codes)
    if err := cc.conn.WriteJSON(request); err != nil {
        log.Fatal("요청 전송 실패:", err)
    }

    for {
        var candle CandleResponse
        if err := cc.conn.ReadJSON(&candle); err != nil {
            log.Printf("응답 수신 실패: %v", err)
            continue
        }

        if err := cc.SaveCandle(&candle); err != nil {
            log.Printf("캔들 데이터 저장 실패: %v", err)
        } else {
            log.Printf("캔들 저장 완료: %s %s", candle.Code, candle.CandleDateTimeKST)
        }
    }
}
```

## 주의사항

- **베타 서비스**: 정식 서비스 전이므로 안정성 주의
- **체결 기반**: 체결이 없으면 캔들이 생성되지 않음
- **시간대**: KST 기준으로 캔들 시간 처리해야 함
- **데이터 저장**: `upbit_candle_1m` 테이블 스키마 준수
- **중복 처리**: 동일 시간대 캔들 업데이트 가능성 고려
- **연결 유지**: 장시간 연결 시 ping/pong 메커니즘 구현 권장
- **에러 처리**: 네트워크 오류나 연결 끊김에 대한 적절한 예외 처리 필요
- **타임스탬프**: 밀리초 단위이므로 초 단위 변환 시 1000으로 나누기
- **시간 형식**: ISO 8601 형식의 시간 문자열 파싱 주의