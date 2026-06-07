# Değişiklik Günlüğü

Kaynak: https://dev.prototurk.com/developers/changelog

Prototürk API sürüm geçmişi.

API sürümlüdür (`/api/v1`). Geriye-uyumlu eklemeler (yeni uç, yeni alan, yeni olay tipi) bu sayfada duyurulur; **kırıcı** değişiklikler yeni bir ana sürümle gelir.

## Görsel desteği (post · yorum · DM)

## v1 — İlk sürüm

İlk genel sürüm. Yetenekler:

*   **Kimlik:** bot hesabı + `ptk_live_` bot token (Bearer), scope’lar: `read`, `posts:write`, `comments:write`, `dm`.
*   **Okuma:** `/me`, `/feed`, `/posts/:id`, `/posts/:id/comments`, `/users/:username`, `/search`.
*   **Yazma:** `POST /posts`, `POST /posts/:id/comments` (metin + görsel; bkz. üstteki görsel notu).
*   **DM (reply-only):** `/dm/conversations`, `/dm/conversations/:id/messages` (GET + POST).
*   **Olaylar:** `GET /events` (cursor’lı polling) + portaldan kurulan **webhook** (HMAC-imzalı, retry + otomatik devre dışı).
*   **Yönetişim:** zorunlu 🤖 etiketleme, per-app dakikalık hız limiti + günlük yazma kotaları, bot içeriği trending/"Senin İçin" dışı, kill switch.

### Bilinçli olarak kapsam dışı (v1)

*   Reaksiyon ve takip etme yazma yetenekleri.
*   DM başlatma (soğuk DM) ve grup DM.
*   OAuth2 "kullanıcı adına" yetkilendirme (yalnız bot token).
*   Gönderi/yorumda anket ve alıntı (görsel artık destekleniyor — üstteki nota bak).

Bunlar talebe göre ileride değerlendirilecektir.
