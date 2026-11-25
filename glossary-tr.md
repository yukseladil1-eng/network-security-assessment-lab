# Laboratuvar terimleri — Türkçe okuma rehberi

Bu rehber, İngilizce rapordaki kavramları laboratuvar örnekleriyle açıklar. Yeni bir test sonucu veya ek güvenlik iddiası içermez.

| Terim | Anlamı ve bu projedeki karşılığı |
|---|---|
| Assessment | Güvenlik değerlendirmesi: kapsamı belirlemek, mevcut durumu incelemek, sorunları belgelemek ve düzeltmeleri kontrol etmek. |
| Finding | Bulgu: kanıtla desteklenen bir gözlem veya sorun. Her bulgu bir saldırı yaşandığını göstermez. |
| Risk / severity | Risk / önem derecesi: bir sorunun hangi koşullarda ne tür zarara yol açabileceği. Bu laboratuvarda veriler yapaydır; gerçek şirket kaybı hesaplanmamıştır. |
| Evidence | Kanıt: ekran görüntüsü, komut çıktısı veya test sonucu kaydı. Bir kişinin sonucu doğrulamasıyla ham çıktının saklanması aynı kanıt düzeyi değildir. |
| Remediation / retest | Düzeltme / tekrar test: sorunu gidermek ve ardından ilgili davranışı yeniden kontrol etmek. |
| VM | Sanal makine: fiziksel bilgisayar içinde çalışan ayrı işletim sistemi. Burada iki Ubuntu VM vardır. |
| Interface | Ağ arayüzü: sistemin bir ağa bağlandığı bağlantı noktası. Hedefte `enp0s1` test trafiği, `enp0s2` yönetim erişimi için kullanıldı. |
| Port / listener | Port bir hizmetin ağ bağlantılarında kullandığı numaradır. Listener, o numarada bağlantı bekleyen programdır. Portun içinde program veya dosya bulunmaz. |
| Open | Nmap'in test ettiği yoldan bir uygulamanın bağlantı kabul ettiğini belirlediği port durumu. Tek başına güvenlik açığı demek değildir. |
| Closed | Testte portun erişilebilir olduğu, ancak bağlantı kabul eden hizmet bulunmadığı yönünde yanıt alınması. |
| Filtered | Nmap'in filtreleme nedeniyle portun açık mı kapalı mı olduğunu belirleyememesi. Hizmetin durmuş olduğunu kanıtlamaz. |
| UFW / firewall | UFW, Linux güvenlik duvarı kurallarını yönetmek için kullanılan araçtır. Güvenlik duvarı, hangi ağ trafiğine izin verileceğini sınırlar. |
| Ingress / default deny | Gelen trafik / varsayılan engelleme: uygun izin kuralı bulunmayan gelen bağlantılar engellenir. Projedeki son UFW kaydı bu politikayı gösterir. |
| Least privilege | En az yetki: bir kullanıcıya veya bağlantıya yalnızca ihtiyaç duyduğu erişimi vermek. Geçici web kurallarının kaldırılması buna örnektir. |
| SSH / key-only | SSH, uzaktan yönetim için şifreli bağlantı sağlar. Buradaki key-only testi, izinli özel anahtarla girişin çalışması ve parola yöntemiyle denemenin reddedilmesidir; sonuç test edilen kullanıcı ve bağlantı için geçerlidir. |
| Public / private key | Açık anahtar sunucuda doğrulama için kullanılabilir. Özel anahtar sahibinde kalır; paylaşılmaz. Anahtarın dosya yolu ile dosyanın gizli içeriği farklı şeylerdir. |
| HTTP / plaintext | HTTP içeriği kendi başına şifrelemez. Plaintext, ağ trafiğini görebilen birinin okuyabileceği açık içeriktir. İstemcide sayfayı görmek tek başına paket yakalama kanıtı değildir. |
| TLS / HTTPS | TLS iletişimi şifreleyen protokoldür; HTTPS, HTTP'nin TLS üzerinden kullanılmasıdır. Şifreli içerik görmek, bütün TLS ayarlarının güvenli olduğunu tek başına göstermez. |
| Self-signed certificate | Kendi imzalı sertifika: güvenilir bir sertifika otoritesinin onayı yerine kendi anahtarıyla imzalanmıştır. Şifreleme yapılabilir; sunucu kimliğine güven ayrıca doğrulanmalıdır. |
| Packet capture / tcpdump | Paket yakalama / bu amaçla kullanılan araç. `tcpdump -A` paket verisini okunabilir karakterlerle göstermeye çalışır; şifreli verideki anlamsız karakterler tam bir kriptografi denetimi değildir. |
| Web root / directory exposure | Web sunucusunun yayımladığı dizin / bu dizindeki dosyaların açığa çıkması. Özel rapor bu dizinin dışına taşındı; aynı URL'nin 404 vermesi kontrol edildi. |
| 404 | İstenen adres için kaynağın bulunamadığını belirten HTTP yanıtı. Başka adreslerde kopya bulunmadığını kanıtlamaz. |
| `700` directory | Dizin sahibi okuyabilir, içeriğini değiştirebilir ve içine geçebilir; grup ve diğer kullanıcılar bu izinleri almaz. Dizinlerde `x`, arama/geçiş iznidir. |
| `600` file | Dosya sahibi okuyabilir ve yazabilir; grup ve diğer kullanıcılar bu izinleri almaz. Çalıştırma izni içermez. |
| Parent directory | Dosyaya ulaşırken geçilen üst dizin. `/home/student` zaten `750` olduğu için bazı kullanıcıların yolu daha önce de engelleniyordu. Sonraki `700/600` ayarlarını ilk ve tek engel olarak sunamayız. |
| Defense in depth | Birden fazla güvenlik önlemini birlikte kullanmak. Üst dizin kısıtlı olsa bile özel alt dizin ve dosyanın izinlerini daraltmak buna örnektir. |
| Attack surface | Saldırı yüzeyi: erişilebilen hizmetler ve kullanılabilecek giriş noktaları. Bir hizmeti durdurmak ile o hizmetin güvenlik duvarı iznini kaldırmak ayrı işlemlerdir. |

## Belgeleri nasıl okumalı?

Önce [README](README.md) ile projenin amacını ve sonuçlarını gör. Ardından [rapordaki](final-security-assessment-report.md) her bulgunun kanıtına ve tekrar test sınırlarına bak. [Kanıt dizini](evidence-register.md), hangi kaydın orijinal görüntü, hangisinin metin veya kullanıcı doğrulaması olduğunu açıklar.

Bu belgeler geçmiş testleri kaydeder. [Tekrar test rehberi](retest-guide.md), ileride VM'ler yeniden açıldığında mevcut durumun nasıl kontrol edilebileceğini açıklar; rehberin bulunması o testlerin bugün yeniden çalıştırıldığı anlamına gelmez.
