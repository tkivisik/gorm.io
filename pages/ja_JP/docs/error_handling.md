---
title: エラーハンドリング
layout: page
---

Goではエラーハンドリングが重要です。

[即時メソッド](/docs/method_chaining.html#Immediate-Methods)の後ではエラーチェックを行ったほうが良いです。

## エラーハンドリング

Error handling in GORM is different than idiomatic Go code because of its chainable API, but still easy to implement.

If any error occurs, GORM will set `*gorm.DB`'s `Error` field, which you can check like this:

```go
if err := db.Where("name = ?", "jinzhu").First(&user).Error; err != nil {
    // エラーハンドリング...
}
```

もしくは

```go
if result := db.Where("name = ?", "jinzhu").First(&user); result.Error != nil {
    // エラーハンドリング...
}
```

## エラー

When processing data, it is common for multiple errors to occur. GORM provides an API to return all errors as a slice:

```go
// 1つ以上のエラーが起きた場合、`GetErrors`は`[]error`を返します
db.First(&user).Limit(10).Find(&users).GetErrors()

fmt.Println(len(errors))

for _, err := range errors {
  fmt.Println(err)
}
```

## RecordNotFoundエラー

GORM provides a shortcut to handle `RecordNotFound` errors. If there are several errors, it will check if any of them is a `RecordNotFound` error.

```go
// RecordNotFoundエラーを返すかチェックします
db.Where("name = ?", "hello world").First(&user).RecordNotFound()

if db.Model(&user).Related(&credit_card).RecordNotFound() {
  // レコードが見つかりません
}

if err := db.Where("name = ?", "jinzhu").First(&user).Error; gorm.IsRecordNotFoundError(err) {
  // レコードが見つかりません
}
```