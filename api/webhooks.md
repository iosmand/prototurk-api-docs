# Webhook'lar

Kaynak: https://dev.prototurk.com/developers/api/webhooks

Olayları HMAC-imzalı POST ile al — kurulum, imza doğrulama, retry.

Webhook, olayları (DM, bahsetme, yorum…) **bota push** etmenin yoludur — polling’e göre düşük gecikme + sıfır boşa istek. Her teslimat **HMAC-SHA256** ile imzalanır.

## Kurulum

Webhook **API’den değil**, geliştirici portalından kurulur: **Ayarlar → Geliştirici → (onaylı uygulama) → Webhook**.

- **Endpoint URL** — `https://` zorunlu.
- **Olay tipleri** — abone olunacaklar; boş bırakırsan **tümü** gönderilir.
- Kaydedince bir **secret** (`whsec_…`) **yalnızca bir kez** gösterilir — imza doğrulaması için sakla.
- Uygulama başına **tek** webhook (v1).

## Teslimat isteği

Prototürk, endpoint’ine şu biçimde `POST` eder:

http

    POST /senin/webhook/yolun HTTP/1.1
    Content-Type: application/json
    User-Agent: Prototurk-Webhook/1
    X-Prototurk-Signature: t=1717761600,v1=5257a869e7…
    X-Prototurk-Event-Count: 2

    {
      "events": [
        { "id": "019e…", "type": "dm.message", "payload": { "...": "..." }, "createdAt": "2026-06-07T12:00:00.000Z" }
      ]
    }

Gövdedeki `events` dizisi [Olaylar](events.md) sayfasındaki olay nesneleriyle birebir aynıdır.

## İmza doğrulama (zorunlu)

`X-Prototurk-Signature` başlığı `t=<unix-saniye>,v1=<hex>` biçimindedir. İmza şöyle hesaplanır:

    v1 = HMAC_SHA256(secret, `${t}.${rawRequestBody}`)  // hex

Doğrulamak için **ham gövdeyi** (parse etmeden) imzala, sabit-zamanlı karşılaştır ve `t`’nin tazeliğini kontrol et (replay koruması):

**Node (Express)**

    import express from 'express';
    import crypto from 'node:crypto';

    const app = express();
    const SECRET = process.env.PROTOTURK_WEBHOOK_SECRET!;

    // Ham gövde şart — JSON parse'tan ÖNCE.
    app.post('/webhook', express.raw({ type: 'application/json' }), (req, res) => {
      const header = req.get('X-Prototurk-Signature') ?? '';
      const m = header.match(/^t=(\d+),v1=([0-9a-f]+)$/);
      if (!m) return res.status(400).end();
      const [, t, sig] = m;

      // Replay koruması: 5 dakikadan eski imzaları reddet.
      if (Math.abs(Date.now() / 1000 - Number(t)) > 300) return res.status(400).end();

      const expected = crypto
        .createHmac('sha256', SECRET)
        .update(`${t}.${req.body}`) // req.body bir Buffer (ham)
        .digest('hex');

      const ok =
        sig.length === expected.length &&
        crypto.timingSafeEqual(Buffer.from(sig), Buffer.from(expected));
      if (!ok) return res.status(401).end();

      const { events } = JSON.parse(req.body.toString('utf8'));
      for (const ev of events) handle(ev); // idempotent: ev.id ile dedup
      res.status(200).end();
    });

**Bun / Web**

    const SECRET = process.env.PROTOTURK_WEBHOOK_SECRET!;

    Bun.serve({
      port: 3000,
      async fetch(req) {
        const raw = await req.text(); // ham gövde
        const header = req.headers.get('x-prototurk-signature') ?? '';
        const m = header.match(/^t=(\d+),v1=([0-9a-f]+)$/);
        if (!m) return new Response(null, { status: 400 });
        const [, t, sig] = m;
        if (Math.abs(Date.now() / 1000 - Number(t)) > 300)
          return new Response(null, { status: 400 });

        const expected = new Bun.CryptoHasher('sha256', SECRET)
          .update(`${t}.${raw}`)
          .digest('hex');
        if (sig !== expected) return new Response(null, { status: 401 });

        const { events } = JSON.parse(raw);
        for (const ev of events) handle(ev);
        return new Response('ok');
      },
    });

## Teslimat garantileri

- **2xx döndür.** Endpoint’in `200`–`299` dönmezse teslimat başarısız sayılır.
- **At-least-once.** Başarısız olaylar bir sonraki turda (~30 sn) yeniden denenir. Aynı olay birden fazla gelebilir → **`event.id` ile idempotent** ol.
- **Otomatik devre dışı.** Çok sayıda ardışık başarısızlıkta webhook otomatik kapatılır ve sana sistem bildirimi gider. Endpoint’ini düzeltip portaldan **yeniden etkinleştir** (veya kaydet) → sıfırlanır.
- **Sıra.** Olaylar id (zaman) sırasıyla gönderilir, ama ağ nedeniyle garanti etme; `createdAt`/`id`’ye güven.

> Webhook geçici olarak kapalıyken bile olaylar log’da kalır — [`GET /events`](events.md) ile her zaman geçmişi cursor’dan yakalayabilirsin. Webhook + polling birbirini tamamlar.

> **Secret’i koru**
>
> İmza doğrulamasını **atlama**. Doğrulamadan işlersen, endpoint’ini bilen herkes sahte olay enjekte edebilir. Secret’i rotate etmek istersen portaldan *Secret’i yenile*.
