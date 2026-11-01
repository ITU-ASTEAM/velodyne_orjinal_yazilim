# velodyne_orjinal_yazilim

Velodyne VLP-16 lidarı için bize verilen orijinal yazılım: **VeloView 5.1.0**, Ubuntu 18.04 / x86_64 için derlenmiş hazır paket.

**VeloView**, Velodyne lidarların verisini gerçek zamanlı görmek, kaydetmek ve yeniden oynatmak için kullanılan açık kaynak bir masaüstü uygulamasıdır. Kitware tarafından ParaView üzerine geliştirilmiştir; proje bugün **LidarView** adıyla sürdürülüyor.

Paket kurulum gerektirmez: gereken kütüphaneler (Qt, ParaView 5.9, Python 3.9, numpy, Boost, libpcap) `lib/` klasöründe gelir.

## Ne işe yarar

- **Lidarı ROS'tan bağımsız test etmek:** VLP-16 açılıyor mu, ağ bağlantısı doğru mu, nokta bulutunda montaj ya da kablo kaynaklı bir sorun var mı; ROS tarafına geçmeden önce buradan görülür.
- **Kayıt almak:** Lidarın ham paketleri `.pcap` dosyası olarak kaydedilir; saha testinden sonra aynı veri tekrar tekrar incelenebilir.
- **Kaydı oynatmak:** `.pcap` dosyası VeloView'da açılıp kare kare izlenebilir. `PacketFileSender` ise kaydı ağ üzerinden göndererek gerçek bir lidar bağlıymış gibi davranır; böylece lidar olmadan sürücü ve algılama kodu denenebilir.

Lidarı ROS içinde görmek için `velodyne` sürücüsü ve [ituas_sim_2025 reposundaki `tools/rviz`](https://github.com/ITU-ASTEAM/ituas_sim_2025/tree/main/tools/rviz) ayarları kullanılır.

## Çalıştırma

```bash
cd VeloView-5.1.0-Ubuntu18.04-x86_64
./bin/VeloView
```

Uygulama açıldıktan sonra **Open Sensor Stream** ile canlı veriyi açın ve sensör olarak **VLP-16**'yı seçin. Kayıtlı bir dosya için **Open** ile `.pcap` dosyasını açın.

## Kaydı ağ üzerinden oynatmak: PacketFileSender

`PacketFileSender`'ın kütüphane arama yolu paketin `lib/` klasörünü göstermiyor; doğrudan çalıştırılınca `libpcap.so.1` ve `libboost_program_options.so.1.76.0` bulunamıyor. Bu yüzden `lib/` yolu elle verilmelidir:

```bash
cd VeloView-5.1.0-Ubuntu18.04-x86_64
LD_LIBRARY_PATH="$PWD/lib" ./bin/PacketFileSender kayit.pcap --loop
```

| Seçenek | Varsayılan | Açıklama |
|---|---|---|
| `--ip` | `127.0.0.1` | Paketlerin gönderileceği adres |
| `--lidarPort` | `2368` | Lidar paketlerinin portu |
| `--GPSPort` | `8308` | Konum/GPS paketlerinin portu |
| `--loop` | kapalı | Kayıt bitince baştan başlar |
| `--speed` | `1` | Oynatma hızı |

Varsayılan ayarlarla paketler aynı bilgisayara gider; VeloView ya da ROS sürücüsü bu portları dinliyorsa gerçek lidar bağlıymış gibi veri alır.

## Ağ ayarları

| Ayar | Varsayılan değer |
|---|---|
| Lidar IP adresi | `192.168.1.201` |
| Veri portu (UDP) | `2368` |
| Konum/GPS portu (UDP) | `8308` |

- Bilgisayarın kablolu ağ arayüzüne aynı alt ağdan sabit bir IP verilmelidir (örneğin `192.168.1.100/24`).
- Bağlantıyı kontrol etmek için: `ping 192.168.1.201`
- Lidarın ayar arayüzü: `http://192.168.1.201/` (https değil)

Varsayılan değerler lidarın ayar arayüzünden değiştirilmiş olabilir; emin olmak için önce ayar arayüzüne bakın.

## Klasör yapısı

Yollar `VeloView-5.1.0-Ubuntu18.04-x86_64/` klasörüne göredir.

| Yol | İçerik |
|---|---|
| `bin/VeloView` | Uygulamanın kendisi |
| `bin/PacketFileSender` | `.pcap` kaydını ağ üzerinden gönderen araç |
| `bin/qt.conf`, `bin/VeloView.conf` | Göreli yol ayarları; bu sayede klasör başka bir yere taşınabilir |
| `lib/` | Paketle gelen kütüphaneler |
| `plugins/` | Qt eklentileri |
| `share/` | Sensör kalibrasyon dosyaları (`VLP-16.xml` vb.) ve kullanım kılavuzu (`share/doc/VeloView_User_Guide.pdf`) |

## Ubuntu 24.04'te durum

2026-11-01'de Ubuntu 24.04 üzerinde denendi:

- `VeloView` eksik kütüphane olmadan açılıyor (`./bin/VeloView --version` çalıştı). Grafik arayüz ve canlı lidarla denenmedi.
- `PacketFileSender` yalnızca yukarıdaki `LD_LIBRARY_PATH` ile çalışıyor.
- Paketin eski kopyalarında dosyalar çalıştırma iznini kaybetmişti. Bu repoda `bin/VeloView` ve `bin/PacketFileSender` çalıştırılabilir olarak kayıtlı. Başka bir kopyada `Permission denied` alırsanız: `chmod +x bin/VeloView bin/PacketFileSender`

## Lisans

VeloView Apache 2.0 lisanslıdır (bkz. [LICENSE](LICENSE)). Paketle gelen kütüphaneler kendi lisanslarıyla dağıtılır: Qt (LGPL), ParaView ve VTK (BSD), Python (PSF), numpy (BSD), Boost (Boost Software License), libpcap (BSD).

## Kaynaklar

- LidarView proje sayfası: <https://lidarview.kitware.com/>
- Kaynak kod (Kitware GitLab): <https://gitlab.kitware.com/LidarView/lidarview>
- Kullanım kılavuzu: `VeloView-5.1.0-Ubuntu18.04-x86_64/share/doc/VeloView_User_Guide.pdf`
