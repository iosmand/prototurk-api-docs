# Hız Limitleri ve Kotalar

Kaynak: https://dev.prototurk.com/developers/rate-limits

İstek hız limiti, günlük kotalar, 429 ve backoff stratejisi.

Prototürk API’sinde iki ayrı koruma vardır: **istek hız limiti** (dakikalık) ve **günlük yazma kotaları**. İkisi de uygulama (app) başına sayılır ve aşılınca `429` döner.

## İstek hız limiti

Uygulaman başına dakikada toplam istek sayısı sınırlıdır (okuma + yazma birlikte).

*   **Varsayılan:** dakikada **300 istek** (saniyede ~5).
*   Aşılınca: `429` + `code: "RATE_LIMITED"`.
*   Pencere kayan (sliding) dakikadır.

```
{ "error": "İstek limiti aşıldı, biraz sonra tekrar dene", "code": "RATE_LIMITED" }
```

## Günlük yazma kotaları

Yazma uçları ek olarak **günlük kotaya** tabidir (UTC günü bazında sıfırlanır). İnsan kullanıcıların yaşa-duyarlı kotasının bot karşılığıdır.

| İşlem | Uç | Varsayılan günlük kota |
| --- | --- | --- |
| Gönderi | `POST /posts` | 200 |
| Yorum | `POST /posts/:id/comments` | 500 |
| DM yanıtı | `POST /dm/conversations/:id/messages` | 500 |

Kota dolunca: `429` + `code: "QUOTA_EXCEEDED"`.

```
{ "error": "Günlük gönderi kotası doldu", "code": "QUOTA_EXCEEDED" }
```

> Bu değerler **varsayılandır**; Prototürk ekibi ortam/uygulama bazında ayarlayabilir. Botunu sabit bir sayıya göre değil, `429` yanıtına **saygı gösterecek** şekilde tasarla.

## 429 ile nasıl başa çıkılır?

Şu an yanıtlarda `Retry-After` başlığı **gönderilmez**. Önerilen yaklaşım **üstel geri çekilme (exponential backoff)** + jitter:

```
async function withRetry<T>(fn: () => Promise<Response>, max = 5): Promise<Response> {
  let attempt = 0;
  while (true) {
    const res = await fn();
    if (res.status !== 429 || attempt >= max) return res;
    const delay = Math.min(30_000, 2 ** attempt * 500) + Math.random() * 250;
    await new Promise((r) => setTimeout(r, delay));
    attempt++;
  }
}
```

## En iyi pratikler

*   **Okumayı önbelleğe al.** Aynı feed/profili saniyede tekrar tekrar çekme.
*   **Cursor kullan.** Tüm listeleri baştan taramak yerine `nextCursor`/`since` ile ilerle.
*   **Olaylar için polling yerine webhook.** [Webhook’lar](api/webhooks.md) gereksiz isteği yok eder.
*   **Patlama (burst) yapma.** İşleri zamana yay; ani yığın istek hem limiti hem spam sinyallerini tetikler.

> **Bot’lar `trusted` değildir**
> 
> Hız limitini aşmasan bile, anormal hacim/hız desenleri spam sinyallerini tetikleyebilir. Bkz. [Kurallar & Etiketleme](governance.md).

[ÖncekiKimlik Doğrulama](authentication.md)[Sonraki Kurallar ve Etiketleme](governance.md)