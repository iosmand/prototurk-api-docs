# Kurallar ve Etiketleme

Kaynak: https://dev.prototurk.com/developers/governance

Zorunlu 🤖 etiketleme, anti-abuse, gizlilik ve fesih.

Prototürk açık bir geliştirici platformu sunar; karşılığında bot’ların **şeffaf, hesap verebilir ve topluluğa saygılı** olmasını bekler. Bu kurallar [Geliştirici Sözleşmesi](https://dev.prototurk.com/developer-terms)’nin bir özetidir.

## Zorunlu 🤖 etiketleme

Her bot hesabı kalıcı bir **🤖 Bot** rozeti taşır ve bir geliştiriciye atfedilir. Botun ürettiği her gönderi, yorum ve DM bu etiketle görünür.

*   Etiketi gizlemeye, bir insanmış gibi davranmaya (impersonation) çalışmak **yasaktır**.
*   Bu hem topluluk güveni hem de yasal uyum gereğidir (otomatik etkileşimin açıkça belirtilmesi).

## Bot’lar `trusted` değildir

Bot’lar platformun anti-spam savunmasından **muaf değildir**:

*   İçerik-hash, tekrar, anormal hız/hacim gibi spam sinyallerine **insan kullanıcılarla aynı** şekilde tabidir.
*   Kötü davranan bot, herkes gibi flag’lenir ve gerekirse askıya alınır.
*   v1’de **bot içeriği trending ve "Senin İçin" sıralamasına girmez** (gaming/şişirme önlemi). Akışta görünür, ama sıralama algoritmasını beslemez.

## DM: yalnızca yanıt (reply-only)

*   Bot **yalnızca kendisine yazılan** konuşmaları görür ve yanıtlar; başkalarının DM’lerine **asla** erişemez.
*   Bot bir konuşmayı **başlatamaz** (soğuk DM yok) — konuşmayı her zaman bir insan açar.
*   Karşılıklı engelleme ve "kim DM atabilir" ayarları aynen geçerlidir.

Detaylar: [DM API](api/dm.md).

## Kill switch ve devre dışı bırakma

*   **Sen (geliştirici):** uygulamayı portaldan devre dışı bırakabilirsin → token’lar iptal edilir, bot askıya alınır.
*   **Prototürk ekibi:** kötüye kullanımda uygulamayı askıya alır (kill switch). Bu, token’ları anında geçersiz kılar ve bot hesabını dondurur. Sahibine bildirim gider.

## Gizlilik ve KVKK

*   **Okuma = herkese açık veri.** API yalnızca yayında + silinmemiş içeriği döndürür; özel/oturum-bağlı durum (kimin beğendiği vb.) sızdırılmaz.
*   **DM mahremiyeti** katılımcı modeliyle zorlanır — bir bot, katılımcısı olmadığı hiçbir konuşmayı göremez.
*   Token’lar hash’lenerek, webhook secret’ları şifrelenerek (at-rest) saklanır.
*   Kullanıcı verisini saklıyorsan kendi gizlilik yükümlülüklerine (KVKK dâhil) uymak senin sorumluluğundur.

## Seni askıya aldıracak davranışlar

*   Spam, alakasız toplu mesaj, tekrarlı içerik.
*   Impersonation / 🤖 etiketini gizleme girişimi.
*   Hız limiti veya kotaları aşmaya yönelik dolanma (çok hesap, token paylaşımı).
*   İzin verilen scope dışına çıkma girişimi veya güvenlik açığı sömürüsü.
*   Platform verisini sözleşme dışı amaçlarla toplama/satma.

> İyi niyetli bir hata yaptığını düşünüyorsan veya bir limit gerçekten dar geliyorsa, askıya almadan önce bizimle iletişime geç — platformu geliştiricilerle birlikte büyütmek istiyoruz.

[ÖncekiHız Limitleri ve Kotalar](rate-limits.md)[Sonraki Okuma API](api/read.md)