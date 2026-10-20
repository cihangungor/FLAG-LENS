# Flaglens

Flaglens, CTF araçlarının ürettiği dosyalarda flag biçimlerini, parola/anahtar adaylarını, ipuçlarını ve kodlanmış metinleri arar. Networklens, Stegolens veya Reverselens gibi araçların raporları için kullanılabilir; girdileri kaynak araca göre ayırmak gerekmez.

Metin raporlarını tarar ve bilinen kodlama dönüşümlerini veya sınırlı anahtar denemelerini otomatik uygular. Bir ağ kaydını, görseli ya da veritabanını kendi başına çözümlemez.

## İndirme ve şifreli arşiv

GitHub'daki bu README herkese açıktır. Araç, ayrıca yayımlanan parola korumalı ZIP arşivinin içindedir. Arşiv parolasını proje sahibinden ayrı olarak alın; parola bu README'de bulunmaz.

Aşağıdaki örnekler arşivin adını flaglens_mvp.zip, içindeki üst klasörü flaglens_mvp olarak varsayar. Yayımlanan dosyanın adı farklıysa komutta değiştirin. Çıkardığınız klasörde pyproject.toml ve flaglens/ bulunmalıdır.

## İlk kurulum: Kali Linux

Python 3.11 veya üzeri ve internet bağlantısı gerekir. Terminalde:

~~~bash
sudo apt update
sudo apt install -y python3 pipx 7zip
pipx ensurepath
~~~

pipx komutun PATH'e eklenmesi için yeni terminal açmanızı isterse terminali kapatıp yeniden açın. ZIP dosyasını İndirilenler klasörüne kaydettiyseniz:

~~~bash
cd ~/Downloads
7z x flaglens_mvp.zip
cd flaglens_mvp
~~~

7z arşiv parolasını etkileşimli olarak sorar. Parolayı komut satırına yazmanız gerekmez. İçinde pyproject.toml bulunan klasöre girdikten sonra:

~~~bash
pipx install '.[full]'
flaglens --version
flaglens check --plain
~~~

pipx, flaglens komutunu kullanıcı hesabınıza kurar. Sonraki kullanımlarda PYTHONPATH yazmanız gerekmez. Raporlar çıkarılmış `flaglens_mvp/reports/` klasörüne yazılacağı için arşivden çıkardığınız proje klasörünü saklayın. Kontrolde selftest: OK beklenir; harici araç sayısı sisteminizde kurulu isteğe bağlı araçlara göre değişebilir.

Flaglens daha önce pipx ile kurulmuşsa, yeni arşivden çıkan sürüme geçmek için aynı klasörde şunu kullanın:

~~~bash
pipx install --force '.[full]'
~~~

Kurulum ve PATH hakkında ayrıntı için [pipx belgelerine](https://pipx.pypa.io/latest/how-to/install-pipx.html), 7z paketi için [Kali 7zip sayfasına](https://www.kali.org/tools/7zip/) bakabilirsiniz.

## İlk deneme

Çıkarılmış proje klasöründeyken kodlama örneğini çalıştırın:

~~~bash
flaglens report examples/encoding_v020.txt --format FLAG
~~~

Bu örnekte 21 adet FLAG{...} eşleşmesi beklenir. Komut kısa raporu ekrana basar ve dosya yollarını sonunda gösterir. Dosyaları görmek için:

~~~bash
ls reports/
~~~

Her çalıştırma, hedef adı, mod ve zaman damgasını içeren yeni bir dosya grubu oluşturur. Örneğin:

- `reports/encoding_v020_normal_<zaman>_report.md` — yarışmacı veya yapay zekâ için kısa özet
- `reports/encoding_v020_normal_<zaman>_report.json` — kısa özetin `brief` alanı ve tüm bulgular
- `reports/encoding_v020_normal_<zaman>_findings.duckdb` — ayrıntılı bulguların veritabanı

Dosyaları açarken komutun yazdığı tam yolu kullanın. Raporları başka yere koymak isterseniz yalnız `flaglens report` için isteğe bağlı `--out /path/to/prefix` seçeneği korunur.

## Kendi raporlarınızı tarama

Örneğin başka bir aracın ürettiği sonuç klasörünü taramak için üç rapor komutundan birini seçin:

~~~bash
flaglens quick report "/path/to/outputs" --format ISUCTF
flaglens report "/path/to/outputs" --format ISUCTF --wordlist default
flaglens deep report "/path/to/outputs" --format ISUCTF --wordlist default
~~~

`/path/to/outputs` yerine kendi dosya veya klasör yolunuzu yazın. `quick report` hızlı ve seçici tarama yapar. `report` normal tarama ve kısa rapor üretir. `deep report` tüm CTF profilleriyle daha geniş tarama yapar. Üçü de kısa Markdown raporunu, tüm bulguları içeren JSON'u ve DuckDB dosyasını otomatik olarak çıkarılmış proje klasöründeki `reports/` dizinine kaydeder. Önceki raporlar ezilmez.

Flag önekini bilmiyorsanız `--format ISUCTF` bölümünü çıkarın; araç genel CTF benzeri desenleri de arar. `--format` verdiğinizde yalnız o biçime uyan flag'ler eşleşir. `--wordlist default`, Vigenere ve tekrarlayan anahtarlı XOR için paket içindeki anahtar listesini dener.

Sadece terminalde hızlı bakış için:

~~~bash
flaglens quick "/path/to/outputs" --format ISUCTF
~~~

Daha geniş ve gürültülü inceleme için:

~~~bash
flaglens deep "/path/to/outputs" --format ISUCTF --wordlist default
~~~

`quick` ve `deep` komutlarını `report` sözcüğü olmadan çalıştırırsanız yalnız terminal bulgularını görürsünüz; rapor dosyası oluşturulmaz. `deep report` kendi taramasını yapar; önceden çalıştırılmış bir `deep` komutunun sonucunu devralmaz.

Bir değerin olası kodlamasını incelemek için:

~~~bash
flaglens identify 'RkxBR3tCQVNFNjR9' --format FLAG
~~~

Komutların bütün seçenekleri `flaglens report --help`, `flaglens quick --help`, `flaglens deep --help` ve `flaglens identify --help` ile görülebilir.

## Kısa raporu yorumlama

Rapor yalnız anlamlı bulgu seçilen dosyaları listeler. Her satır bulgunun kaynak dosyasını ve satırını gösterir; aynı değer farklı raporlarda geçiyorsa bir kez yazılır ve tekrar sayısı eklenir.

- Eşleşen flag sayısı, seçilen --format veya genel desenle bulunan sonuçları gösterir. Flag adayı yarışmanın doğru cevabı olarak doğrulanmış değildir.
- Tam tarama, desteklenen dosyaların ve çözmeye alınacak değerlerin sınır aşmadan işlendiğini gösterir. Kısmi tarama, atlanan dosyalar, okunamayan hedefler veya çözme bütçesi gibi bir sınır olduğunu gösterir.
- Dosya izi, raporlarda o dosyanın yolunun bulunduğunu söyler; dosyanın ikili içeriği incelenmiş anlamına gelmez.
- Raporun üstündeki eşleşen flag: 0, özellikle kısmi taramada, klasörde hiç flag olmadığına dair kesin sonuç değildir.

Bir ağ raporunda parola, dosya izi veya öncelikli URL bulunduğunda kısa rapor bunları kaynak dosya ve satırla gösterir. Flag eşleşmesi sıfır olsa bile bu izler ayrıca incelenebilir; bir görselin içeriği Flaglens tarafından taranmaz.

Yapay zekâya ön bağlam vermek için kısa Markdown raporunu veya JSON içindeki brief alanını gönderin. Ayrıntılı araştırmada JSON'daki findings alanını ya da DuckDB dosyasını kullanın.

## Sınırlar ve dosya türleri

.txt, .log, .json, .csv, .xml, .md ve benzeri metin dosyaları taranır. JSON ve CSV alanları yapısal olarak ayrıştırılmaz; içerikleri metin olarak aranır. PDF, SQLite/DuckDB, PNG ve diğer ikili dosyalar için içerik çıkarıcı yoktur. Bu dosyalar atlanan dosya sayısına girer; içeriklerini taratmak için önce metin raporuna dönüştürün.

Raporlarda çözmeye alınan farklı değer sınırı varsayılan olarak 800'dür. Büyük ham çıktılarda daha fazlasını denemek için:

~~~bash
flaglens deep report "/path/to/outputs" --decode-limit 2000
~~~

Daha yüksek sınır daha uzun sürebilir. `--decode-limit 0` yalnız doğrudan bulunanları raporlar. `--summary-limit 20` kısa raporda en fazla 20 farklı anlamlı bulgu gösterir; daha fazlası varsa üst bölümde sayısı belirtilir. Normal rapordaki isteğe bağlı `--out` bir dosya yolu değil, üç çıktı dosyasının ortak ad önekidir.

Flaglens Base64/Base32/Base85, hex, URL/HTML/Unicode kaçışları, Morse, ROT/Caesar/Atbash/Affine, bazı XOR ve Vigenere denemeleri, JWT yükü ve metne dönüştürülmüş sıkıştırılmış veri gibi yöntemleri dener. Her şifrelemeyi otomatik çözmez; parola gerektiren güçlü kriptografi için doğru anahtar gerekir.

