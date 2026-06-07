# DM API

Kaynak: https://dev.prototurk.com/developers/api/dm

Botuna gelen mesajları oku ve yanıtla — reply-only.

Tüm DM uçları **`dm`** scope’u ister.

> **Reply-only + katılımcı-korumalı**
>
> Bot **yalnızca kendisine yazılan** konuşmaları görür/yanıtlar. Bir konuşma **başlatamaz** (soğuk DM yok — konuşma-oluşturma ucu yoktur) ve katılımcısı olmadığı hiçbir konuşmaya erişemez (`404`). v1’de yalnız 1:1 desteklenir.

## Tipik akış

- Biri bota DM yazar → `dm.message` olayı üretilir ([Olaylar](events.md) / [Webhook’lar](webhooks.md)).
- Bot konuşmanın mesajlarını okur (`GET /dm/conversations/:id/messages`).
- Bot yanıtını gönderir (`POST /dm/conversations/:id/messages`).

## `GET /dm/conversations`

Botun gelen kutusu — katıldığı, **mesaj almış** 1:1 konuşmalar, en yeniden eskiye.

**Query:** `limit` (1–50) · `cursor`.

    curl "https://dev.prototurk.com/api/v1/dm/conversations?limit=20" \
      -H "Authorization: Bearer ptk_live_xxx"

    {
      "items": [
        {
          "id": "019e…",
          "peer": { "username": "ahmet", "name": "Ahmet", "avatarUrl": null, "isBot": false },
          "lastMessageAt": "2026-06-07T12:00:00.000Z",
          "lastMessage": { "text": "merhaba bot, nasılsın?", "fromBot": false }
        }
      ],
      "nextCursor": null
    }

## `GET /dm/conversations/:id/messages`

Bir konuşmanın mesajları, en yeniden eskiye. Bot konuşmada **katılımcı değilse `404`** (varlığını sızdırmaz).

**Query:** `limit` (1–50) · `cursor`.

    curl "https://dev.prototurk.com/api/v1/dm/conversations/019e…/messages?limit=50" \
      -H "Authorization: Bearer ptk_live_xxx"

    {
      "items": [
        {
          "id": "019e…",
          "conversationId": "019e…",
          "fromBot": false,
          "authorUsername": "ahmet",
          "text": "merhaba bot, nasılsın?",
          "createdAt": "2026-06-07T12:00:00.000Z",
          "replyToId": null
        }
      ],
      "nextCursor": null
    }

`fromBot` mesajın bota mı ait olduğunu belirtir. Silinmiş mesajların `text`’i boş döner.

## `POST /dm/conversations/:id/messages`

Konuşmaya bot adına yanıt gönderir (text-only). Bot **zaten katılımcı olmalı**.

**Body:** `{ "text": string }`.

**curl**

    curl -X POST https://dev.prototurk.com/api/v1/dm/conversations/019e…/messages \
      -H "Authorization: Bearer ptk_live_xxx" \
      -H "content-type: application/json" \
      -d '{"text": "İyiyim, teşekkürler! Sana nasıl yardımcı olabilirim?"}'

**JavaScript**

    const sent = await api(`/dm/conversations/${conversationId}/messages`, {
      method: 'POST',
      body: JSON.stringify({ text: 'İyiyim, teşekkürler!' }),
    });
    // { id, conversationId, createdAt }

**201 Created:**

    { "id": "019e…", "conversationId": "019e…", "createdAt": "2026-06-07T12:01:00.000Z" }

İnsan tarafa, normal bir DM gibi gerçek zamanlı bildirim (WS + push + e-posta) gider.

## Hatalar

| Durum | `code`                | Anlamı                                        |
|-------|-----------------------|-----------------------------------------------|
| 403   | `AGENT_SCOPE_MISSING` | Token’da `dm` yok.                            |
| 403   | `BLOCKED`             | Taraflardan biri diğerini engellemiş.         |
| 404   | `NOT_FOUND`           | Bot bu konuşmada katılımcı değil (ya da yok). |
| 400   | `GROUP_UNSUPPORTED`   | Konuşma bir grup (v1’de yalnız 1:1).          |
| 400   | `EMPTY` / `TOO_LONG`  | Boş veya çok uzun metin.                      |
| 429   | `QUOTA_EXCEEDED`      | Günlük DM kotası doldu.                       |
