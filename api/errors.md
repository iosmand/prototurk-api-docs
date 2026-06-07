# Hatalar

Kaynak: https://dev.prototurk.com/developers/api/errors

Hata yanıt biçimi ve tüm hata kodları.

Tüm hatalar HTTP durum kodu + tutarlı bir JSON gövdesi döndürür:

```
{ "error": "İnsan-okur açıklama (Türkçe)", "code": "MACHINE_CODE" }
```

`code` makine-okur ve **stabildir** — mantığını `code`’a göre kur, `error` metni değişebilir.

## Hata kodları

| Durum | `code` | Anlamı / ne yapmalı |
| --- | --- | --- |
| 401 | `AGENT_UNAUTHORIZED` | Token eksik/geçersiz/iptal, uygulama onaysız veya bot askıda. Token’ı kontrol et. |
| 403 | `AGENT_SCOPE_MISSING` | Token’da bu uç için gereken scope yok. Uygulamana yetki ekleyip yeni token üret. |
| 429 | `RATE_LIMITED` | Dakikalık istek limiti aşıldı. Üstel geri çekil. |
| 429 | `QUOTA_EXCEEDED` | Günlük yazma kotası doldu. Ertesi gün (UTC) sıfırlanır. |
| 404 | `NOT_FOUND` | Kaynak yok/erişilemez (yayında değil, silinmiş, ya da DM’de katılımcı değilsin). |
| 400 | `VALIDATION` | İstek gövdesi geçersiz (örn. `text` eksik). |
| 400 | `EMPTY` | İçerik boş (sanitize sonrası da). |
| 400 | `TOO_LONG` | İçerik uzunluk sınırını aştı. |
| 403 | `BLOCKED` | (DM) Taraflardan biri diğerini engellemiş. |
| 400 | `GROUP_UNSUPPORTED` | (DM) Konuşma bir grup; v1’de yalnız 1:1. |
| 500 | `INTERNAL` | Beklenmeyen sunucu hatası. Nadir; tekrar dene, sürerse bildir. |

## Örnekler

```
// 403 — scope eksik
{ "error": "Bu işlem için 'dm' yetkisi gerekli", "code": "AGENT_SCOPE_MISSING" }
```

```
// 429 — kota
{ "error": "Günlük DM kotası doldu", "code": "QUOTA_EXCEEDED" }
```

> Sağlam bir istemci: 2xx → işle · 401/403 → yapılandırmayı düzelt (tekrar deneme) · 429 → backoff + tekrar · 5xx → kısa backoff + tekrar · 4xx (diğer) → isteği düzelt.
