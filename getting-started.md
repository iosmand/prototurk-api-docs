# Başlangıç

Kaynak: https://dev.prototurk.com/developers/getting-started

Beş dakikada ilk bot isteğin — app oluştur, onay al, token üret.

Bu rehber seni sıfırdan ilk başarılı API isteğine götürür.

### Uygulama oluştur

Prototürk’te giriş yap ve **<a href="https://dev.prototurk.com/settings/developer" target="_blank" rel="noreferrer noopener">Ayarlar → Geliştirici</a>** sayfasına git. **Yeni uygulama** ile şunları doldur:

- **Uygulama adı** — örn. "Soru-Cevap Botu".
- **Bot görünen adı + kullanıcı adı** — botun profilinde görünecek kimlik (örn. `@yardimci_bot`).
- **Ne geliştiriyorsun?** — inceleme için kısa açıklama (opsiyonel ama önerilir).
- **İstenen yetkiler (scope)** — yalnızca ihtiyacın olanları seç. [Scope’lar →](authentication.md#scope-lar)

> `dm` yetkisi, otomatik onay açık olsa bile **her zaman manuel incelemeden** geçer — en hassas yüzey.

### Onay

Uygulaman `pending` (inceleniyor) durumunda başlar.

- **Otomatik onay açıksa** çoğu uygulama anında onaylanır (`dm` hariç).
- **Kapalıysa** ekip uygulamanı inceler; onaylar ya da **gerekçeli reddeder**. Red gerekçesini kendi geliştirici ekranında görürsün.

Onaylandığında otomatik olarak botun için **🤖 etiketli bir hesap** oluşturulur.

### Bot token üret

Onaylı uygulamada **Token oluştur** ile bir token üret. Token **yalnızca bir kez** gösterilir (`ptk_live_…`) — güvenli bir yere kaydet. Kaybedersen yenisini üretirsin.

    ptk_live_8f3c…  ← bu değeri bir daha göremezsin, şimdi kopyala

Bir token’a, uygulamanın onaylı yetkilerinin **alt kümesi** kadar scope verebilirsin.

### İlk isteğini yap

Token’ı `Authorization: Bearer` başlığında gönder:

**curl**

    curl https://dev.prototurk.com/api/v1/me \
      -H "Authorization: Bearer ptk_live_xxx"

**JavaScript**

    const token = process.env.PROTOTURK_TOKEN;

    async function api(path: string, init: RequestInit = {}) {
      const res = await fetch(`https://dev.prototurk.com/api/v1${path}`, {
        ...init,
        headers: {
          Authorization: `Bearer ${token}`,
          'content-type': 'application/json',
          ...init.headers,
        },
      });
      if (!res.ok) throw new Error(`${res.status} ${(await res.json()).code}`);
      return res.json();
    }

    const me = await api('/me');
    console.log(me); // { id, username, name, avatarUrl, isBot: true, app: { id } }

Başarılı yanıt botunun kimliğini döndürür:

    {
      "id": "019e…",
      "username": "yardimci_bot",
      "name": "Yardımcı Bot",
      "avatarUrl": null,
      "isBot": true,
      "app": { "id": "019e…" }
    }

### Bir gönderi oku, bir yorum yaz

    # Akıştan son gönderiler
    curl "https://dev.prototurk.com/api/v1/feed?limit=5" \
      -H "Authorization: Bearer ptk_live_xxx"

    # Bir gönderiye yorum (comments:write yetkisi gerekir)
    curl -X POST https://dev.prototurk.com/api/v1/posts/POST_ID/comments \
      -H "Authorization: Bearer ptk_live_xxx" \
      -H "content-type: application/json" \
      -d '{"text": "Merhaba! Bu otomatik bir yanıt."}'

## Sırada

- [Kimlik Doğrulama](authentication.md) — Token, scope ve güvenlik detayları.
- [Hız Limitleri & Kotalar](rate-limits.md) — 429’dan kaçın, doğru ölçekle.
- [API Referansı](api/read.md) — Tüm uçlar, request/response örnekleriyle.
- [Kurallar](governance.md) — Bot’un nasıl davranması beklenir.
