> Bu belge NVlabs/ProtoMotions deposundaki `docs/source/robots/g1.rst` dosyasının Türkçe çevirisidir (Apache-2.0, değişiklik: Türkçeleştirme).

# Unitree G1

29 DoF'lu G1 modeli için `--robot-name g1` seçin. Varsayılan varlık `mjcf/g1_holo_compat.xml` dosyasıdır. Başka bir G1 varlığının yalnız 29 eyleyicisi var diye aynı gövde çerçevelerine, ellere ya da eylemsizliklere sahip olduğunu varsaymayın.

## Varsayılan sözleşme

* Yapılandırma: `protomotions/robot_configs/g1.py` (`G1RobotConfig`).
* Yapı: 33 gövde, 29 eklem serbestlik derecesi; +X ileri, Z yukarı.
* Gözlem çapası (anchor): `torso_link`; fiziksel kök (root): `pelvis`.
* Hedef gövdeler: `torso_link`, `head`, `right_ankle_roll_link`, `left_ankle_roll_link`, `left_wrist_yaw_link`, `right_wrist_yaw_link`. Eller, her G1 el türünde bulunan bir çerçeve olan bilek bağlantısından izlenir. `left_rubber_hand` ve `right_rubber_hand` varlıkta bileklerin katı alt öğeleri olarak kalır.

Yapılandırma eğik bir ayakta duruş pozu ile eyleyici ailesine göre PD kazançlarını, armatürleri, hız sınırlarını ve tork sınırlarını tanımlar. Kazanç formülleri 10 Hz doğal frekans ve 2 sönüm oranı kullanır, ekleme özgü çarpanlarla birlikte. Bunlar simülasyon denetleyicisinin varsayılanlarıdır ve her G1 donanım türünün ayarları olarak okunmamalıdır. Eylem eşlemesini deneyler ayrıca tanımlar.

## Hareket ve dağıtım

Kaynak insan eklem koordinatlarını değil, G1'e yeniden hedeflenmiş (retargeted) hareketi kullanın. Dışa aktarırken ya da simülatör değiştirirken kontrol noktasının kaydettiği varlığı ve algılayıcı/gözlem sözleşmesini koruyun. Seçilen el gövdesini değiştirmek hedef geometrisini değiştirir.

* [SEED G1 CSV veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/seed_g1_csv_preparation.html): G1 CSV hareketini dönüştürün.
* [PyRoki ile yeniden hedefleme](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/retargeting_pyroki.html): insan hareketini G1'e yeniden hedefleyin.
* [G1 gerçek robota dağıtım](G1-GERCEK-ROBOTA-DAGITIM.md): dışa aktarım ve dağıtım iş akışı.
* [Önceden eğitilmiş modeller](https://nvlabs.github.io/ProtoMotions/getting_started/pretrained_models.html): G1 politikaları ve örnek hareketler.

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
