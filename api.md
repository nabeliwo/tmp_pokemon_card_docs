# API 設計

## 方針

- ID は基本 UUID を使う
- ページネーションはレスポンスボディに pagination という属性を入れる
- ここには一旦認証周りの話は書かないが、全ての API のリクエストヘッダに `Authorization` を付ける

## 資産

### GET `/api/assets/by_day`

日ごとの総資産を返す API。総資産タブのグラフ表示用。

```typescript
type Parameter = {
  from?: string // ex. '2022-10-01' デフォルトは半年前
  to?: string // ex. '2022-10-01' デフォルトは今日
}

type Response = Array<{
  date: string // ex. '2022-10-01'
  assets: number
}>
```

### GET `/api/assets/by_month`

月ごとの総資産を返す API。総資産タブのテーブル表示用。

```typescript
type Parameter = {
  page?: number // デフォルトは1
  per_page?: number // デフォルトは20
}

type Response = {
  pagination: {
    page: number
    per_page: number
    page_count: number
    total_count: number
  }
  total_assets: number // 現在の総資産額。最新月の総資産額とイコールなのでいらないかも
  assets_by_month: Array<{
    date: string // ex. '2022-10'
    assets: number
    month_on_month: number // ex. 1000, -1000
  }>
}
```

## 通知

### GET `/api/notifications`

通知一覧を返す API。

```typescript
type Response = Array<{
  id: string
  date: string // ex. '2022-10-10'
  unread: boolean // true だと日付の横に赤いバッジを表示する
  cards: Array<{
    id: string
    name: string
    image_url: string
    expansion_mark: string // ex. 's11a'
    card_list: string // ex. '023/028'
    amount: number // ex. 1000, -1000
  }>
}>
```

### PATCH `/api/notifications`

通知を既読にする API。

```typescript
type RequestBody = {
  id: string
  unread: string
}
```

## カード

### GET `/api/cards`

カード一覧を取得する API。検索で使う。
一旦、カード名検索のみで考える。(絞り込みやカード名以外での検索は考慮しない)

```typescript
type Parameter = {
  q: string // カード名
  page?: number // デフォルトは1
  per_page?: number // デフォルトは20
}

type Response = {
  pagination: {
    page: number
    per_page: number
    page_count: number
    total_count: number
  }
  cards: Array<{
    id: string
    name: string
    image_url: string
    expansion_mark: string // ex. 's11a'
    card_list: string // ex. '023/028'
    price: number
    price_fluctuation: number // 価格変動 ex. 1000, -1000
    rise_and_fall_rate: number // 騰落率 ex. 10.95, -10.95
  }>
}
```

## 所持カード

### GET `/api/my_cards`

自分のカードリストに入っているカードの一覧を取得する API。

```typescript
type Parameter = {
  page?: number // デフォルトは1
  per_page?: number // デフォルトは20
}

type Response = {
  pagination: {
    page: number
    per_page: number
    page_count: number
    total_count: number
  }
  cards: Array<{
    id: string
    card_id: string
    name: string
    image_url: string
    expansion_mark: string // ex. 's11a'
    card_list: string // ex. '023/028'
    price: number
    condition: string // 状態。カードラッシュの状態基準に従う
    price_difference: number // 入手時との価格差 ex. 1000, -1000
  }>
}
```

### POST `/api/my_cards`

自分のカードリストにカードを追加する API。

```typescript
type RequestBody = {
  card_id: string
  condition: string
  price_at_acquision: number // 入手時の価格
  count: string // 複数枚同時に登録できる。状態と入手時の価格が同じじゃなきゃいけないからいらないかも？1枚ずつ登録で十分かも
}
```

### DELETE `/api/my_cards`

自分のカードリストからカードを削除する API。

```typescript
type RequestBody = {
  id: string
}
```
