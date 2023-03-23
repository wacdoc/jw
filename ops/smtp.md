# Mbangun server ngirim mail SMTP dhewe

## pambuka

SMTP bisa langsung tuku layanan saka vendor awan, kayata:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali push email awan](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Sampeyan uga bisa mbangun server mail dhewe - ngirim tanpa watesan, biaya sakabèhé murah.

Ing ngisor iki, kita nduduhake langkah demi langkah carane nggawe server mail dhewe.

## Pilihan server

Server SMTP sing dadi tuan rumah dhewe mbutuhake IP umum kanthi port 25, 456, lan 587 mbukak.

Awan umum sing umum digunakake wis ngalangi port kasebut kanthi standar, lan bisa uga dibukak kanthi nerbitake pesenan kerja, nanging angel banget.

Aku nyaranake tuku saka host sing mbukak port kasebut lan ndhukung nyetel jeneng domain mbalikke.

Kene, aku nyaranake [Contabo](https://contabo.com) .

Contabo minangka panyedhiya hosting sing adhedhasar ing Munich, Jerman, didegaké ing 2003 kanthi rega sing kompetitif.

Yen sampeyan milih Euro minangka mata uang tuku, rega bakal luwih murah (server kanthi memori 8GB lan 4 CPU biaya kira-kira 529 yuan saben taun, lan biaya instalasi dhisikan gratis kanggo setahun).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Nalika nggawe pesenan, komentar `prefer AMD` , lan server kanthi CPU AMD bakal duwe kinerja sing luwih apik.

Ing ngisor iki, aku bakal njupuk Contabo's VPS minangka conto kanggo nduduhake carane nggawe server mail sampeyan dhewe.

## Konfigurasi sistem Ubuntu

Sistem operasi ing kene yaiku Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Yen server ing ssh nampilake `Welcome to TinyCore 13!` (minangka ditampilake ing tokoh ngisor), iku tegese sistem durung diinstal. Mangga pedhot sambungan ssh lan ngenteni sawetara menit kanggo mlebu maneh.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Nalika `Welcome to Ubuntu 22.04.1 LTS` katon, initialization lengkap, lan sampeyan bisa nerusake karo langkah ing ngisor iki.

### [Opsional] Miwiti lingkungan pangembangan

Langkah iki opsional.

Kanggo penak, aku sijine instalasi lan konfigurasi sistem piranti lunak ubuntu ing [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Jalanake printah ing ngisor iki kanggo nginstal kanthi siji klik.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Pangguna Cina, gunakake printah ing ngisor iki, lan basa, zona wektu, lan sapiturute bakal disetel kanthi otomatis.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo ngaktifake IPV6

Aktifake IPV6 supaya SMTP uga bisa ngirim email nganggo alamat IPV6.

sunting `/etc/sysctl.conf`

Owahi utawa tambahake baris ing ngisor iki

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Tindakake [tutorial contabo: Nambahake konektivitas IPv6 menyang server sampeyan](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Sunting `/etc/netplan/01-netcfg.yaml` , tambahake sawetara baris kaya sing ditampilake ing gambar ing ngisor iki (Contabo VPS file konfigurasi standar wis duwe garis kasebut, mung uncomment).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Banjur `netplan apply` kanggo nggawe konfigurasi sing diowahi ditrapake.

Sawise konfigurasi sukses, sampeyan bisa nggunakake `curl 6.ipw.cn` kanggo ndeleng alamat IPv6 jaringan eksternal sampeyan.

## Kloning ops repositori konfigurasi

```
git clone https://github.com/wactax/ops.soft.git
```

## Gawe sertifikat SSL gratis kanggo jeneng domain sampeyan

Ngirim email mbutuhake sertifikat SSL kanggo enkripsi lan teken.

Kita nggunakake [acme.sh](https://github.com/acmesh-official/acme.sh) kanggo ngasilake sertifikat.

acme.sh minangka alat tandha sertifikat otomatis open source,

Ketik ops.soft gudang konfigurasi, mbukak `./ssl.sh` , lan folder `conf` bakal digawe ing **direktori ndhuwur** .

Temokake panyedhiya DNS saka [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , edit `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Banjur mbukak `./ssl.sh 123.com` kanggo ngasilake sertifikat `123.com` lan `*.123.com` kanggo jeneng domain sampeyan.

Run pisanan bakal kanthi otomatis nginstal [acme.sh](https://github.com/acmesh-official/acme.sh) lan nambah tugas dijadwal kanggo nganyari maneh otomatis. Sampeyan bisa ndeleng `crontab -l` , ana baris kaya ing ngisor iki.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Path kanggo sertifikat sing digawe kaya `/mnt/www/.acme.sh/123.com_ecc。`

Pembaruan sertifikat bakal nelpon skrip `conf/reload/123.com.sh` , sunting skrip iki, sampeyan bisa nambah printah kayata `nginx -s reload` kanggo refresh cache sertifikat aplikasi sing gegandhengan.

## Mbangun server SMTP karo chasquid

[chasquid](https://github.com/albertito/chasquid) minangka server SMTP open source sing ditulis ing basa Go.

Minangka sulih kanggo program mail server kuna kayata Postfix lan Sendmail, chasquid prasaja lan luwih gampang kanggo nggunakake, lan iku uga luwih gampang kanggo pembangunan secondary.

Run `./chasquid/init.sh 123.com` bakal diinstal kanthi otomatis kanthi siji klik (ganti 123.com nganggo jeneng domain sing sampeyan kirim).

## Konfigurasi Email Signature DKIM

DKIM digunakake kanggo ngirim teken email kanggo nyegah layang saka dianggep minangka spam.

Sawise printah kasebut sukses, sampeyan bakal dijaluk nyetel rekaman DKIM (kaya sing ditampilake ing ngisor iki).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Cukup tambahake rekaman TXT menyang DNS sampeyan (kaya ing ngisor iki).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Deleng status layanan & log

 `systemctl status chasquid` Ndeleng status layanan.

Kahanan operasi normal kaya sing ditampilake ing gambar ing ngisor iki

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` utawa `journalctl -xeu chasquid` bisa ndeleng log kesalahan.

## Konfigurasi jeneng domain mbalikke

Jeneng domain mbalikke kanggo ngidini alamat IP ditanggulangi menyang jeneng domain sing cocog.

Nyetel jeneng domain mbalikke bisa nyegah email saka dikenali minangka spam.

Nalika mail ditampa, server panampa bakal nindakake analisis jeneng domain mbalikke ing alamat IP saka server ngirim kanggo konfirmasi apa server ngirim duwe jeneng domain mbalikke bener.

Yen server ngirim ora duwe jeneng domain mbalikke utawa yen jeneng domain mbalikke ora cocog alamat IP saka server ngirim, server panampa bisa ngenali email minangka spam utawa nolak.

Dolan maring [https://my.contabo.com/rdns](https://my.contabo.com/rdns) lan atur kaya ing ngisor iki

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Sawise nyetel jeneng domain mbalikke, elinga kanggo ngatur resolusi maju saka jeneng domain ipv4 lan ipv6 menyang server.

## Sunting jeneng host chasquid.conf

Ngowahi `conf/chasquid/chasquid.conf` kanggo Nilai saka jeneng domain mbalikke.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Banjur mbukak `systemctl restart chasquid` kanggo miwiti maneh layanan.

## Gawe serep conf menyang repositori git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Contone, aku nggawe serep folder conf menyang proses github dhewe kaya ing ngisor iki

Nggawe gudang pribadi dhisik

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Ketik direktori conf lan kirim menyang gudang

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Tambah pangirim

mlayu

```
chasquid-util user-add i@wac.tax
```

Bisa nambah pangirim

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Priksa manawa sandhi wis disetel kanthi bener

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Sawise nambahake pangguna, `chasquid/domains/wac.tax/users` bakal dianyari, elinga ngirim menyang gudang.

## DNS nambah rekaman SPF

SPF (Sender Policy Framework) minangka teknologi verifikasi email sing digunakake kanggo nyegah penipuan email.

Iki verifikasi identitas pangirim email kanthi mriksa manawa alamat IP pangirim cocog karo cathetan DNS saka jeneng domain sing diklaim, nyegah penipu ngirim email palsu.

Nambah cathetan SPF bisa nyegah email saka dikenali minangka spam sabisa.

Yen server jeneng domain sampeyan ora ndhukung jinis SPF, tambahake rekaman jinis TXT.

Contone, SPF saka `wac.tax` kaya ing ngisor iki

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF kanggo `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Elinga yen aku wis `include:_spf.google.com` kene, iki amarga aku bakal ngatur `i@wac.tax` minangka alamat ngirim ing kothak layang Google mengko.

## Konfigurasi DNS DMARC

DMARC minangka singkatan saka (Otentikasi Pesen, Pelaporan & Kesesuaian adhedhasar Domain).

Iki digunakake kanggo njupuk bouncing SPF (bisa uga disebabake kesalahan konfigurasi, utawa wong liya nyamar dadi sampeyan ngirim spam).

Tambah rekaman TXT `_dmarc` ,

Isine kaya ing ngisor iki

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Makna saben paramèter kaya ing ngisor iki

### p (Kebijakan)

Nuduhake carane nangani email sing gagal verifikasi SPF (Sender Policy Framework) utawa DKIM (DomainKeys Identified Mail). Parameter p bisa disetel menyang salah siji saka telung nilai:

* ora ana: Ora ana tumindak sing ditindakake, mung asil verifikasi diwenehake menyang pangirim liwat mekanisme laporan email.
* Quarantine: Lebokake email sing durung lulus verifikasi menyang folder spam, nanging ora bakal langsung nolak email kasebut.
* nolak: Langsung nolak email sing gagal verifikasi.

### fo (Pilihan Gagal)

Nemtokake jumlah informasi sing bali dening mekanisme laporan. Bisa disetel menyang salah siji saka nilai ing ngisor iki:

* 0: Laporan asil validasi kanggo kabeh pesen
* 1: Mung laporan pesen sing gagal verifikasi
* d: Mung laporan kegagalan verifikasi jeneng domain
* s: mung laporan gagal verifikasi SPF
* l: Mung laporake kegagalan verifikasi DKIM

### rua & ruf

* rua (Nglaporake URI kanggo laporan Agregat): Alamat email kanggo nampa laporan gabungan
* ruf (Nglaporake URI kanggo laporan Forensik): alamat email kanggo nampa laporan rinci

## Tambah cathetan MX kanggo nerusake email menyang Google Mail

Amarga aku ora bisa nemokake kothak layang perusahaan gratis sing ndhukung alamat universal (Catch-All, bisa nampa email apa wae sing dikirim menyang jeneng domain iki, tanpa watesan awalan), aku nggunakake chasquid kanggo nerusake kabeh email menyang kothak layang Gmail.

**Yen sampeyan duwe kothak layang bisnis mbayar dhewe, aja ngowahi MX lan skip langkah iki.**

Sunting `conf/chasquid/domains/wac.tax/aliases` , atur kothak layang sing diterusake

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` nuduhake kabeh email, `i` minangka awalan alamat email saka pangguna sing ngirim digawe ing ndhuwur. Kanggo nerusake email, saben pangguna kudu nambah baris.

Banjur tambahake rekaman MX (Aku langsung ngarahake alamat jeneng domain mbalikke ing kene, kaya sing ditampilake ing baris pisanan ing gambar ing ngisor iki).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Sawise konfigurasi rampung, sampeyan bisa nggunakake alamat email liyane kanggo ngirim email menyang `i@wac.tax` lan `any123@wac.tax` kanggo ndeleng apa sampeyan bisa nampa email ing Gmail.

Yen ora, priksa log chasquid ( `grep chasquid /var/log/syslog` ).

## Kirim email menyang i@wac.tax nganggo Google Mail

Sawise Google Mail nampa surat kasebut, mesthine aku ngarep-arep bisa mangsuli nganggo `i@wac.tax` tinimbang i.wac.tax@gmail.com.

Bukak [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) banjur klik "Tambah alamat email liyane".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Banjur, ketik kode verifikasi sing ditampa dening email sing diterusake.

Pungkasan, bisa disetel minangka alamat pangirim standar (bebarengan karo pilihan kanggo mbales nganggo alamat sing padha).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Kanthi cara iki, kita wis ngrampungake panyiapan server mail SMTP lan ing wektu sing padha nggunakake Google Mail kanggo ngirim lan nampa email.

## Kirimi email test kanggo mriksa apa konfigurasi wis sukses

Ketik `ops/chasquid`

Run `direnv allow` kanggo nginstal dependensi (direnv wis diinstal ing proses initialization siji-tombol sadurunge lan pancing wis ditambahake menyang cangkang)

banjur mlayu

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Tegese paramèter kaya ing ngisor iki

* pangguna: jeneng pangguna SMTP
* pass: sandi SMTP
* kanggo: panampa

Sampeyan bisa ngirim email test.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Disaranake nggunakake Gmail kanggo nampa email test kanggo mriksa apa konfigurasi wis sukses.

### enkripsi standar TLS

Kaya sing ditampilake ing gambar ing ngisor iki, ana kunci cilik iki, tegese sertifikat SSL wis kasil diaktifake.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Banjur klik "Show Original Email"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Kaya sing dituduhake ing gambar ing ngisor iki, kaca email asli Gmail nampilake DKIM, tegese konfigurasi DKIM wis sukses.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Priksa Ditampa ing header saka email asli, lan sampeyan bisa ndeleng sing alamat pangirim IPV6, kang tegese IPV6 uga kasil diatur.
