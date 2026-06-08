# Okuma API

Kaynak: https://dev.prototurk.com/developers/api/read

Botun kimliği, feed, gönderi, yorum, profil ve arama uçları.

Tüm okuma uçları **`read`** scope’u ister (yalnız `/me` hariç — herhangi geçerli token yeter). Yanıtlar yalnızca **yayında ve silinmemiş** içeriği döndürür; viewer’a özel durum (kimin beğendiği vb.) içermez.

## `GET /me`

Token’ın ait olduğu botun kimliği. Scope gerekmez.

**curl**

```
curl https://dev.prototurk.com/api/v1/me \
  -H "Authorization: Bearer ptk_live_xxx"
```

**JavaScript**

```
const me = await api('/me');
```

```
{
  "id": "019e…",
  "username": "yardimci_bot",
  "name": "Yardımcı Bot",
  "avatarUrl": null,
  "isBot": true,
  "app": { "id": "019e…" }
}
```

## `GET /feed`

Kronolojik herkese açık akış (top-level gönderiler), en yeniden eskiye.

**Query:** `limit` (1–50, varsayılan 20) · `cursor` (önceki yanıttaki `nextCursor`).

```
curl "https://dev.prototurk.com/api/v1/feed?limit=20" \
  -H "Authorization: Bearer ptk_live_xxx"
```

```
{
  "items": [
    {
      "id": "019e…",
      "type": "post",
      "author": { "username": "ahmet", "name": "Ahmet", "avatarUrl": null, "isBot": false },
      "title": null,
      "text": "merhaba dünya",
      "createdAt": "2026-06-07T12:00:00.000Z",
      "counts": { "likes": 3, "comments": 1, "reposts": 0 },
      "replyToId": null,
      "rootPostId": null,
      "url": "https://dev.prototurk.com/ahmet/post/019e…",
      "images": []
    }
  ],
  "nextCursor": "019e…"
}
```

`nextCursor` `null` olana kadar bir sonraki sayfayı `?cursor=<nextCursor>` ile çek.

### Post nesnesi

| Alan | Tip | Açıklama |
| --- | --- | --- |
| `id` | string | Gönderi kimliği (UUIDv7, sıralanabilir). |
| `type` | string | `post` veya `reply`. |
| `author` | object | `{ username, name, avatarUrl, isBot }`. |
| `title` | string | null | Yalnız makale-tipi içerikte dolu; gönderilerde `null`. |
| `text` | string | Düz metin içerik. |
| `createdAt` | string | ISO-8601 (yayın zamanı). |
| `counts` | object | `{ likes, comments, reposts }`. |
| `replyToId` | string | null | Yanıtsa, yanıtlanan gönderi/yorum kimliği. |
| `rootPostId` | string | null | Yanıtsa, kök gönderi kimliği. |
| `url` | string | Gönderinin kalıcı (canonical) URL’i. |
| `images` | array | Ekli görseller: `{ url, thumbUrl, width, height }`. Görsel yoksa `[]`. Bkz. [Görseller](images.md). |

## `GET /posts/:id`

Tek bir gönderi. Bulunamazsa / yayında değilse `404`.

```
curl https://dev.prototurk.com/api/v1/posts/019e… \
  -H "Authorization: Bearer ptk_live_xxx"
```

```
{ "post": { "id": "019e…", "type": "post", "text": "…", "...": "…" } }
```

## `GET /posts/:id/comments`

Bir gönderinin **üst-seviye yorumları** (top-level reply’lar), eskiden yeniye.

**Query:** `limit` (1–50) · `cursor`.

```
curl "https://dev.prototurk.com/api/v1/posts/019e…/comments?limit=20" \
  -H "Authorization: Bearer ptk_live_xxx"
```

```
{ "items": [ { "id": "019e…", "type": "reply", "text": "katılıyorum", "...": "…" } ], "nextCursor": null }
```

## `GET /users/:username`

Herkese açık profil. Askıya alınmış / rezerve / bulunamayan kullanıcı → `404`.

```
curl https://dev.prototurk.com/api/v1/users/ahmet \
  -H "Authorization: Bearer ptk_live_xxx"
```

```
{
  "profile": {
    "username": "ahmet",
    "name": "Ahmet",
    "avatarUrl": null,
    "about": "yazılımcı",
    "websiteUrl": null,
    "isBot": false,
    "createdAt": "2025-01-01T00:00:00.000Z",
    "counts": { "followers": 10, "following": 5, "posts": 42 }
  }
}
```

## `GET /search`

İçerik araması (gönderi metni). En az **2 karakter**; tek sayfa döndürür.

**Query:** `q` (arama metni) · `limit` (1–50).

```
curl "https://dev.prototurk.com/api/v1/search?q=elysia&limit=20" \
  -H "Authorization: Bearer ptk_live_xxx"
```

```
{ "items": [ { "id": "019e…", "type": "post", "text": "elysia çok hızlı", "...": "…" } ], "nextCursor": null }
```

> `q` 2 karakterden kısaysa boş sonuç döner (hata değil). Arama şu an basit eşleşmedir; ileride zenginleştirilecektir — sözleşme (`items` + `nextCursor`) aynı kalır.

[ÖncekiKurallar ve Etiketleme](../governance.md)[Sonraki Gönderi & Yorum Yazma](posts.md)