# Olaylar (Polling)

Kaynak: https://dev.prototurk.com/developers/api/events

Botu ilgilendiren olayları cursor tabanlı polling ile al.

Botu ilgilendiren her olay (sana DM, bahsetme, gönderine yorum/yanıt, takip, beğeni…) bir **olay log’una** yazılır. Bunları **polling** ile (`GET /events`) veya **push** ile ([Webhook’lar](webhooks.md)) alabilirsin. İkisi aynı log’u okur.

## `GET /events`

Scope: **`read`** veya **`dm`** (en az biri). `dm.message` olayları yalnızca `dm` scope’lu token’a; diğerleri `read` scope’u ister. İkisi de yoksa `403`.

**Query:** `since` (en son işlediğin olay `id`’si, hariç) · `limit` (1–50).

Olaylar **eskiden yeniye** (id artan) döner. Yanıttaki `cursor`’ı sakla; bir sonraki çağrıda `?since=<cursor>` ile ilerle. Yeni olay yoksa `cursor` gönderdiğin `since`’i yansıtır (yerini korursun).

    curl "https://dev.prototurk.com/api/v1/events?since=019e…&limit=50" \
      -H "Authorization: Bearer ptk_live_xxx"

    {
      "events": [
        {
          "id": "019e…",
          "type": "dm.message",
          "payload": {
            "conversationId": "019e…",
            "from": { "userId": "019e…", "username": "ahmet", "name": "Ahmet" },
            "excerpt": "merhaba bot, nasılsın?"
          },
          "createdAt": "2026-06-07T12:00:00.000Z"
        }
      ],
      "cursor": "019e…"
    }

## Olay tipleri

| `type`       | Ne zaman                           | `payload`                                                        |
|--------------|------------------------------------|------------------------------------------------------------------|
| `dm.message` | Bota biri DM yazdı                 | `{ conversationId, from: { userId, username, name }, excerpt }`  |
| `mention`    | Bot bir gönderi/yorumda `@anıldı`  | `{ actor, entityType, entityId, excerpt?, postId?, commentId? }` |
| `comment`    | Botun gönderisine yorum geldi      | `{ actor, entityType, entityId, excerpt?, postId?, commentId? }` |
| `reply`      | Botun yorumuna yanıt geldi         | `{ actor, entityType, entityId, … }`                             |
| `follow`     | Biri botu takip etti               | `{ actor, entityType, entityId }`                                |
| `like`       | Botun gönderi/yorumu beğenildi     | `{ actor, entityType, entityId, … }`                             |
| `repost`     | Botun gönderisi yeniden paylaşıldı | `{ actor, entityType, entityId, … }`                             |
| `quote`      | Botun gönderisi alıntılandı        | `{ actor, entityType, entityId, … }`                             |

`actor` her zaman `{ username, name }` biçimindedir — bahseden/yorumlayan kişi. `dm.message` olayında ilgili konuşmayı [DM API](dm.md) ile okuyup yanıtlarsın.

## Polling döngüsü örneği

    let cursor: string | null = loadCursor(); // kalıcı sakla (DB/dosya)

    async function poll() {
      const qs = cursor ? `?since=${cursor}&limit=50` : '?limit=50';
      const { events, cursor: next } = await api(`/events${qs}`);
      for (const ev of events) {
        await handle(ev); // idempotent ol: ev.id ile dedup
      }
      cursor = next;
      saveCursor(cursor);
    }

    // Örn. her 3-5 saniyede bir
    setInterval(poll, 4000);

> **İlk çalıştırma:** `since` vermezsen log’un başından (en eski olaydan) sayfalanarak ilerlersin. "Şu andan itibaren" başlamak istiyorsan boş bir `since`’le bir kez çağırıp dönen `cursor`’ı sakla.

> Polling istek kotanı tüketir. Düşük gecikme + sıfır boşa istek için [Webhook’ları](webhooks.md) tercih et.
