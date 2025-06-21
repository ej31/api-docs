# Upbit WebSocket API: 내 자산 (MyAsset) 가이드

## 개요
WebSocket을 통해 실시간 자산 정보를 수신할 수 있는 Private 엔드포인트입니다.

## WebSocket 엔드포인트
```
wss://api.upbit.com/websocket/v1/private
```

## 인증 요구사항
- **Private 엔드포인트**: JWT 토큰 기반 인증 필수
- **Authorization 헤더**: `Bearer {JWT_TOKEN}` 형식
- 자세한 인증 방법은 [인증 가이드](./websocket-authentication.md) 참조

## 데이터 전송 특징
- **이벤트 기반**: 특정 이벤트(주문 체결, 입출금 등) 발생 시에만 데이터 전송
- **전체 자산**: 모든 보유 자산 정보를 한 번에 전송
- **초기 지연**: 최초 이용 시 데이터 전송에 수 분 소요 가능

## 요청 형식

### 기본 요청 구조
```json
[
  {
    "ticket": "test-myAsset"
  },
  {
    "type": "myAsset"
  }
]
```

### 주요 요청 필드
| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| `ticket` | String | 필수 | 요청 식별자 |
| `type` | String | 필수 | `"myAsset"` 고정 |

## 응답 구조

### 주요 응답 필드
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `type` | 데이터 타입 | String | "myAsset" |
| `assets` | 자산 리스트 | Array | 모든 보유 자산 |
| `stream_type` | 스트림 타입 | String | REALTIME |

### 자산 정보 (assets)
| 필드명 | 설명 | 타입 | 비고 |
|--------|------|------|------|
| `currency` | 화폐 코드 | String | KRW, BTC, ETH 등 |
| `balance` | 주문 가능 수량/금액 | String | 사용 가능한 자산 |
| `locked` | 주문 중 묶인 수량/금액 | String | 주문에 사용된 자산 |
| `avg_buy_price` | 매수 평균가 | String | 암호화폐의 경우 |
| `avg_buy_price_modified` | 매수 평균가 수정 여부 | Boolean | |
| `unit_currency` | 평가 화폐 단위 | String | 보통 "KRW" |

## 응답 예시

### 기본 응답
```json
{
  "type": "myAsset",
  "assets": [
    {
      "currency": "KRW",
      "balance": "1386929.37",
      "locked": "10329.67",
      "avg_buy_price": "0",
      "avg_buy_price_modified": false,
      "unit_currency": "KRW"
    },
    {
      "currency": "BTC",
      "balance": "0.12345678",
      "locked": "0.01000000",
      "avg_buy_price": "32500000.0",
      "avg_buy_price_modified": false,
      "unit_currency": "KRW"
    },
    {
      "currency": "ETH",
      "balance": "2.5",
      "locked": "0.0",
      "avg_buy_price": "2800000.0",
      "avg_buy_price_modified": false,
      "unit_currency": "KRW"
    }
  ],
  "stream_type": "REALTIME"
}
```

## Go 언어 구현 예시

### 구조체 정의
```go
package main

import (
    "crypto/sha256"
    "encoding/hex"
    "strconv"
    "github.com/golang-jwt/jwt/v4"
    "github.com/google/uuid"
    "github.com/gorilla/websocket"
    "net/http"
)

// 자산 정보 구조체
type Asset struct {
    Currency              string  `json:"currency"`
    Balance               string  `json:"balance"`
    Locked                string  `json:"locked"`
    AvgBuyPrice          string  `json:"avg_buy_price"`
    AvgBuyPriceModified  bool    `json:"avg_buy_price_modified"`
    UnitCurrency         string  `json:"unit_currency"`
}

// MyAsset 응답 구조체
type MyAssetResponse struct {
    Type       string  `json:"type"`
    Assets     []Asset `json:"assets"`
    StreamType string  `json:"stream_type"`
}

// 자산의 총 보유량 계산 (balance + locked)
func (a *Asset) GetTotalAmount() (float64, error) {
    balance, err := strconv.ParseFloat(a.Balance, 64)
    if err != nil {
        return 0, err
    }
    
    locked, err := strconv.ParseFloat(a.Locked, 64)
    if err != nil {
        return 0, err
    }
    
    return balance + locked, nil
}

// 자산의 평가금액 계산 (현재가 기준)
func (a *Asset) GetEstimatedValue(currentPrice float64) (float64, error) {
    total, err := a.GetTotalAmount()
    if err != nil {
        return 0, err
    }
    
    if a.Currency == "KRW" {
        return total, nil
    }
    
    return total * currentPrice, nil
}

// 수익률 계산 (%)
func (a *Asset) GetProfitRate(currentPrice float64) (float64, error) {
    avgPrice, err := strconv.ParseFloat(a.AvgBuyPrice, 64)
    if err != nil || avgPrice == 0 {
        return 0, err
    }
    
    return ((currentPrice - avgPrice) / avgPrice) * 100, nil
}

// Private WebSocket 클라이언트 (myOrder와 공통)
type PrivateWebSocketClient struct {
    accessKey string
    secretKey string
    conn      *websocket.Conn
}

func NewPrivateWebSocketClient(accessKey, secretKey string) *PrivateWebSocketClient {
    return &PrivateWebSocketClient{
        accessKey: accessKey,
        secretKey: secretKey,
    }
}

func (p *PrivateWebSocketClient) Connect() error {
    // JWT 토큰 생성
    payload := jwt.MapClaims{
        "access_key": p.accessKey,
        "nonce":      uuid.New().String(),
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, payload)
    tokenString, err := token.SignedString([]byte(p.secretKey))
    if err != nil {
        return err
    }
    
    // WebSocket 연결
    header := http.Header{}
    header.Set("Authorization", "Bearer "+tokenString)
    
    conn, _, err := websocket.DefaultDialer.Dial("wss://api.upbit.com/websocket/v1/private", header)
    if err != nil {
        return err
    }
    
    p.conn = conn
    return nil
}

func (p *PrivateWebSocketClient) SubscribeMyAsset() error {
    request := []interface{}{
        map[string]string{"ticket": "myAsset_" + uuid.New().String()},
        map[string]interface{}{
            "type": "myAsset",
        },
    }
    
    return p.conn.WriteJSON(request)
}

func (p *PrivateWebSocketClient) ReadAssetMessage() (*MyAssetResponse, error) {
    var asset MyAssetResponse
    err := p.conn.ReadJSON(&asset)
    return &asset, err
}

func (p *PrivateWebSocketClient) Close() error {
    if p.conn != nil {
        return p.conn.Close()
    }
    return nil
}
```

### 사용 예시
```go
package main

import (
    "fmt"
    "log"
    "strconv"
)

func main() {
    // API 키 설정 (실제 키로 교체)
    accessKey := "your_access_key"
    secretKey := "your_secret_key"
    
    // Private WebSocket 클라이언트 생성
    client := NewPrivateWebSocketClient(accessKey, secretKey)
    
    // 연결
    if err := client.Connect(); err != nil {
        log.Fatal("WebSocket 연결 실패:", err)
    }
    defer client.Close()
    
    // 내 자산 정보 구독
    if err := client.SubscribeMyAsset(); err != nil {
        log.Fatal("구독 요청 실패:", err)
    }
    
    fmt.Println("내 자산 정보를 실시간으로 수신합니다...")
    fmt.Println("주의: 최초 데이터 수신까지 수 분이 소요될 수 있습니다.")
    
    // 메시지 수신 루프
    for {
        assetData, err := client.ReadAssetMessage()
        if err != nil {
            log.Printf("메시지 수신 실패: %v", err)
            continue
        }
        
        fmt.Printf("\n=== 자산 정보 업데이트 ===\n")
        
        var totalValue float64
        
        for _, asset := range assetData.Assets {
            balance, _ := strconv.ParseFloat(asset.Balance, 64)
            locked, _ := strconv.ParseFloat(asset.Locked, 64)
            total := balance + locked
            
            if total > 0 {
                fmt.Printf("\n[%s]\n", asset.Currency)
                
                if asset.Currency == "KRW" {
                    fmt.Printf("  사용가능: %.2f원\n", balance)
                    fmt.Printf("  주문중: %.2f원\n", locked)
                    fmt.Printf("  총 보유: %.2f원\n", total)
                    totalValue += total
                } else {
                    avgPrice, _ := strconv.ParseFloat(asset.AvgBuyPrice, 64)
                    fmt.Printf("  사용가능: %.8f개\n", balance)
                    fmt.Printf("  주문중: %.8f개\n", locked)
                    fmt.Printf("  총 보유: %.8f개\n", total)
                    fmt.Printf("  매수평균가: %.0f원\n", avgPrice)
                    
                    // 현재가 정보가 있다면 평가금액 계산
                    // 실제로는 ticker API로 현재가를 가져와야 함
                    // 여기서는 예시로 고정값 사용
                    var currentPrice float64
                    switch asset.Currency {
                    case "BTC":
                        currentPrice = 100000000 // 1억원 가정
                    case "ETH":
                        currentPrice = 3000000   // 300만원 가정
                    }
                    
                    if currentPrice > 0 {
                        estimatedValue := total * currentPrice
                        fmt.Printf("  평가금액: %.0f원\n", estimatedValue)
                        totalValue += estimatedValue
                        
                        if avgPrice > 0 {
                            profitRate := ((currentPrice - avgPrice) / avgPrice) * 100
                            profitAmount := (currentPrice - avgPrice) * total
                            fmt.Printf("  수익률: %.2f%% (%.0f원)\n", profitRate, profitAmount)
                        }
                    }
                }
            }
        }
        
        fmt.Printf("\n총 자산 평가액: %.0f원\n", totalValue)
    }
}
```

### 자산 모니터링 시스템 예시
```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "sync"
    "time"
)

type AssetMonitor struct {
    assets     map[string]*Asset
    mutex      sync.RWMutex
    client     *PrivateWebSocketClient
    priceCache map[string]float64 // 간단한 가격 캐시
}

func NewAssetMonitor(accessKey, secretKey string) *AssetMonitor {
    return &AssetMonitor{
        assets:     make(map[string]*Asset),
        client:     NewPrivateWebSocketClient(accessKey, secretKey),
        priceCache: make(map[string]float64),
    }
}

func (am *AssetMonitor) Start() error {
    if err := am.client.Connect(); err != nil {
        return err
    }
    
    if err := am.client.SubscribeMyAsset(); err != nil {
        return err
    }
    
    go am.receiveMessages()
    return nil
}

func (am *AssetMonitor) receiveMessages() {
    for {
        assetData, err := am.client.ReadAssetMessage()
        if err != nil {
            log.Printf("메시지 수신 오류: %v", err)
            time.Sleep(time.Second)
            continue
        }
        
        am.updateAssets(assetData.Assets)
    }
}

func (am *AssetMonitor) updateAssets(newAssets []Asset) {
    am.mutex.Lock()
    defer am.mutex.Unlock()
    
    // 이전 자산과 비교하여 변화 감지
    for _, asset := range newAssets {
        prevAsset, exists := am.assets[asset.Currency]
        
        if exists {
            // 잔고 변화 확인
            prevBalance, _ := strconv.ParseFloat(prevAsset.Balance, 64)
            currBalance, _ := strconv.ParseFloat(asset.Balance, 64)
            
            if prevBalance != currBalance {
                diff := currBalance - prevBalance
                fmt.Printf("[잔고 변화] %s: %.8f → %.8f (%.8f)\n", 
                    asset.Currency, prevBalance, currBalance, diff)
            }
            
            // 주문중 금액 변화 확인
            prevLocked, _ := strconv.ParseFloat(prevAsset.Locked, 64)
            currLocked, _ := strconv.ParseFloat(asset.Locked, 64)
            
            if prevLocked != currLocked {
                diff := currLocked - prevLocked
                fmt.Printf("[주문중 변화] %s: %.8f → %.8f (%.8f)\n", 
                    asset.Currency, prevLocked, currLocked, diff)
            }
        } else {
            // 새로운 자산 추가
            balance, _ := strconv.ParseFloat(asset.Balance, 64)
            if balance > 0 {
                fmt.Printf("[새 자산] %s: %.8f개\n", asset.Currency, balance)
            }
        }
        
        am.assets[asset.Currency] = &asset
    }
}

func (am *AssetMonitor) GetTotalValue() float64 {
    am.mutex.RLock()
    defer am.mutex.RUnlock()
    
    var totalValue float64
    
    for currency, asset := range am.assets {
        total, err := asset.GetTotalAmount()
        if err != nil {
            continue
        }
        
        if currency == "KRW" {
            totalValue += total
        } else {
            // 현재가 조회 (실제로는 REST API 또는 WebSocket ticker 사용)
            if price, exists := am.priceCache[currency]; exists {
                totalValue += total * price
            }
        }
    }
    
    return totalValue
}

func (am *AssetMonitor) PrintPortfolio() {
    am.mutex.RLock()
    defer am.mutex.RUnlock()
    
    fmt.Println("\n=== 포트폴리오 현황 ===")
    
    for currency, asset := range am.assets {
        total, _ := asset.GetTotalAmount()
        if total > 0 {
            balance, _ := strconv.ParseFloat(asset.Balance, 64)
            locked, _ := strconv.ParseFloat(asset.Locked, 64)
            
            fmt.Printf("%s: 보유 %.8f, 주문중 %.8f\n", 
                currency, balance, locked)
            
            if currency != "KRW" {
                avgPrice, _ := strconv.ParseFloat(asset.AvgBuyPrice, 64)
                if avgPrice > 0 {
                    fmt.Printf("  매수평균가: %.0f원\n", avgPrice)
                }
            }
        }
    }
    
    fmt.Printf("\n총 평가액: %.0f원\n", am.GetTotalValue())
}

func (am *AssetMonitor) UpdatePrice(currency string, price float64) {
    am.mutex.Lock()
    am.priceCache[currency] = price
    am.mutex.Unlock()
}

func (am *AssetMonitor) Close() error {
    return am.client.Close()
}
```

## 주의사항

- **인증 필수**: Private 엔드포인트이므로 JWT 토큰 인증 필요
- **이벤트 기반**: 자산 변화 시에만 데이터 전송
- **초기 지연**: 최초 이용 시 데이터 전송에 수 분 소요 가능
- **전체 자산**: 모든 보유 자산이 한 번에 전송됨
- **연결 유지**: 주기적 ping/pong 권장
- **문자열 숫자**: 정확도를 위해 숫자 필드가 문자열로 제공됨
- **시간대**: 모든 시간 데이터는 KST 기준으로 처리
- **가격 정보**: 현재가 정보는 별도로 ticker API 등을 통해 조회 필요
- **에러 처리**: 네트워크 오류나 인증 실패에 대한 적절한 예외 처리 필요
- **보안**: API 키는 안전한 방법으로 관리해야 함
- **정확성**: 실시간 자산 계산 시 부동소수점 연산 정확도 주의