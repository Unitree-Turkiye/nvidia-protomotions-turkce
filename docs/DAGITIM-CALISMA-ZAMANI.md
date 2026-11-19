> Bu belge NVlabs/ProtoMotions deposundaki `deployment/README.md` dosyasının Türkçe çevirisidir (Apache-2.0, değişiklik: Türkçeleştirme). Bu belgedeki göreli yollar ve komutlar orijinal deponun `deployment/` klasörüne göredir.

# ProtoMotions dağıtımı (deployment)

---

> **Durum: deneysel prototipleme.** Bu dizindeki her şey, aşağıda *Desteklenen* başlığı altında sayılan üst düzey Python modülleri dışında, prototipleme kodudur. Kendi dağıtım yolunu kuran kişiler için başvuru malzemesi olarak depoda tutulur. Resmi destek kapsamında yer almaz. API'leri, CMake hedefleri ve dosya düzenleri haber verilmeden değişebilir ya da kaldırılabilir. Projenin uyumluluk taahhütlerinin dışındadır ve sürekli entegrasyon (CI) onu derlemez, çalıştırmaz (CI'da yalnız `protomotions/tests` altındaki Python testleri çalışır; bunlar dışa aktarıcıları ve `native/` altında aşağıda adı geçen üç dosyayı kapsar, çalışma zamanını kapsamaz). Birkaç örnek ve araç gerçek donanımla konuşur (üretici SDK'sı üzerinden bir Unitree G1, ağ üzerinden bir Pico kumandası) ve robotu hareket ettirir: çalıştırmadan önce onları okuyun.

### Desteklenen

Üst düzey Python modülleri desteklenen dışa aktarım yoludur ve CI onları `protomotions/tests` üzerinden test eder:

- `contract.py`: dışa aktarılan her modelle birlikte gelen `DeploymentContract` yan dosyası
- `export_to_onnx.py`: dışa aktarıcı
  (`python deployment/export_to_onnx.py --checkpoint <dir>/inference_last.ckpt`;
  simülatörün kendi gövde durumunu okuyan bir politika için `--host simulation` ekleyin)
- `export_fk_to_onnx.py`: politika grafiğinin izlediği çekirdek nesnenin aynısından `fk.onnx` ve `fk.yaml` yazan ileri kinematik (FK) dışa aktarıcısı
  (`python deployment/export_fk_to_onnx.py --checkpoint <dir>/inference_last.ckpt`)
- `sample_context.py`, `tracker_inputs.py`, `input_history.py`, `metatron_policy.py`, `motion_utils.py`, `state_utils.py`: dışa aktarıcının girdileri ve yardımcıları
- `test_tracker_mujoco.py`: dışa aktarılmış bir izleyicinin elle yapılan MuJoCo kontrolü (CI değil, kullanıcı çalıştırır)

`native/` altındaki bazı dosyalar dışarıdan içe aktarılır. Bu yüzden içinde bulundukları ağaç deneysel olsa da serbestçe değiştirilemez ya da kaldırılamaz: `export_to_onnx.py --leapp`, paketini `native/examples/policy_artifact.py` dosyasını içe aktaran `native/tools/leapp_export.py` üzerinden yazar. `--leapp-graph-history` ise `native/tools/leapp_graph_history.py` üzerinden yazar. CI'ın pytest çalıştırması iki LEAPP modülünü (`protomotions/tests/test_metatron_generic_export.py`, `test_leapp_graph_history.py`), `native/examples/metatron_v1_contract.py` dosyasını (`test_metatron_generic_export.py`, `test_scene_boxes.py`) ve `native/tools/make_metatron_fk_fixture.py` dosyasını (`test_fk_exporter.py`) içe aktarır.

### Deneysel

- `native/include/`, `native/src/`, `native/tests/`, `native/CMakeLists.txt`, `native/cmake/`: dışa aktarılmış bir ONNX modelini ve yan dosyasını yükleyip bir simülatörü ya da robot arka ucunu süren, bağımsız bir C++20 çalışma zamanı
- `native/tools/`: belirli politikalar (Metatron v1, GPC, LEAPP) için dışa aktarım ve dönüştürme araçları
- `native/examples/`: operatör girdisi ve robot arka uçları dahil baştan sona çalıştırmalar
- `native/bench/`, `native/docs/`, `native/assets/`, `native/scripts/`: zamanlama düzenekleri, tasarım notları, küçük örnek varlıklar ve kurulum yardımcısı `install.sh`

`native/assets/styles` altındaki stil ön ayarları, iç kullanımdaki bir hareket yakalama klibinden türetilmiş öznitelik vektörleridir. *Deneysel* altındaki hiçbir şey bir sürüm çıktısı (release artifact) değildir. Dışa aktarılmış bir politikayı desteklenen yoldan çalıştırmak için yukarıdaki dışa aktarıcıyı ve onunla gelen yan dosyayı kullanın.

---

Bu dizin hem **politika dışa aktarım araçlarını** hem de **bağımsız bir C++20 dağıtım çalışma zamanını** içerir. ProtoMotions'ta eğitin, modeli girdi/eylem sözleşmesiyle birlikte dışa aktarın, simülasyonda doğrulayın, sonra aynı denetleyiciyi bir robot arka ucuna bağlayın. Eğitim, C++ derlemesine ya da robot SDK'larına bağlı değildir.

Çalışma zamanı NVIDIA'nın iç dağıtım deposundan alındı. Entegrasyon değişiklikleri [SOURCE.md](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/SOURCE.md) dosyasında listelenir. Bu dizinin üst düzeyindeki mevcut izleyici dışa aktarıcısı ve yardımcıları yerinde kalır.

## Buradan başlayın

- Çekirdeği aşağıdaki komutlarla derleyip test edin. Robot ya da GPU gerekmez.
- 39 girdili güncel Metatron v1 politikası için [politika hazırlığı](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/metatron_v1_policy.md) ve [yerel simülasyon/dağıtım](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/metatron_v1_native.md) belgelerini okuyun. Xbox ile hareket izleme ve yalnız tekmede çalan robot TTS işareti de orada anlatılır.
- GPC sabit şekilli örnekleme motorları için [GPC TensorRT](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/gpc_tensorrt.md) belgesini okuyun.

GPC **yalnız kod** olarak gelir: kontrol noktası, ONNX ağırlıkları, TensorRT planı, indirme adresi ya da önceden eğitilmiş model dağıtılmaz. Uyumlu modelinizi ve yapılandırmanızı kendiniz sağlayın. Mevcut üst düzey izleyici araçları değişmeden kalır. Bu çalışma zamanı bir BeyondMimic denetleyicisi ya da başlatıcı entegrasyonu eklemez.

**Donanım güvenliği:** simülatörde geçen bir test fiziksel doğrulama yerine geçmez. Başlatıcılar durmuş/sönümleme kipinde açılır, `--arm` bayrağını asla kendiliğinden eklemeyin. Robot güvenceye alınmışken eklem sırasını, birimleri, kazançları, sınırları, algılayıcı tazeliğini, komut bekçisini (watchdog) ve fiziksel durdurma denetimini doğrulayın. Metatron v1 sönümleme için X, boşta kipe geri dönmek için Y düğmesini kullanır. Denetim eşlemesinin tamamı için onun kılavuzuna bakın.

## Neler destekleniyor

| Yol | Uygulama | Kapsam ve sınırlar |
| --- | --- | --- |
| Çekirdek çalışma zamanı | C++ çalışma zamanı, güvenlik denetçisi, denetleyici yöneticisi, C ABI | Arka uçtan bağımsız; üretici SDK'ları ve çıkarım kütüphaneleri isteğe bağlı |
| MuJoCo | Yerel arka uç ve eşzamansız görüntüleyici | Simülasyon, duman testleri ve algılayıcı/eylem sözleşmesi doğrulaması |
| Unitree G1 | İsteğe bağlı SDK2 arka ucu ve G1 başlatıcıları | 29 gövde DoF'u; SDK2/CycloneDDS ve operatör doğrulaması gerektirir |
| Metatron v1 | Özel yerel TensorRT politika motoru + politikanın dışa aktardığı `fk.onnx` dosyasından kurulan FK motoru | Birebir 39 girdili G1 sözleşmesi; başlatıcı boşta, yönlendirme ve toparlanma kiplerini sunar |
| GPC koşulsuz öncülü (prior) | Python bağdaştırıcısı, sabit şekilli TensorRT çalıştırıcısı ve kıyaslama | Belirli 29 DoF, 64 gözlem, sekiz jetonlu sözleşme; argmax, Gumbel-max ve kesin top-p |
| Operatör girdisi | Linux evdev Xbox, isteğe bağlı SDL2 ve Pico/XRoboToolkit arayüzleri | Kullanılabilirlik ve tuş atamaları başlatıcıya bağlıdır; Pico SDK dışarıdan gelir |

Arayüzler genişletilebilir. Yine de bir SOMA ya da rastgele bir politika çıktısını bir G1 başlatıcısına kopyalamak o robot ya da sözleşme için destek **eklemez**. GPC yönlendirmesi ve genel yerel AMP bağdaştırıcıları birer genişletme fırsatıdır, hazır gelen tak çalıştır yollar değildir. Ayrıntılı kılavuzlarda kaydedilmiş ana kod zamanlama/doğrulama sonuçları kendi özgün makinelerini ve çıktılarını anlatır. Bu klonun ya da sizin donanımınızın sertifikası değildir.

## Kod nasıl çalışır

```text
robot SDK / MuJoCo -> RobotBackend -> RobotState
                                       |
operator InputSource -> task goals -> Controller / policy adapter
                                       |
                              ActuatorCommand
                                       |
                     SafetySupervisor -> RobotBackend
```

Taşıma ve algılayıcı dönüşümü arka ucun, politika ve görevin anlamı denetleyicinin işidir. `DeploymentRuntime` yaşam döngüsünü yönetir ve durumu ve komutları göndermeden önce doğrular. `ControllerManager` denetleyicileri çevrim sınırlarında değiştirir. `FixedRateScheduler` zamanlamayı politika çıkarımından ayrı yönetir. Geçersiz, bayat ya da desteklenmeyen durum/komutlar ve taşıma hataları güvenlik yoluna girer. Rastgele önceki eylemlere sessizce geri dönülmez. Bkz. [mimari](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/architecture.md) ve [paket boşluğu güvenliği](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/packet_gap_safety.md).

Dışa aktarılan grafikler gözlem işlemeyi, normalleştirmeyi, ağ çıkarımını, eylem son işlemesini ve robotun kendi gövdesinin ileri kinematiğini üstlenir. Robot tarafındaki bir ana makine (host) ham algılayıcı okumalarını (eklem konumları, IMU yönelimi, konumlandırmadan gelen çapa konumu), görev hedeflerini ve geçmişleri verir. Gövde pozlarını asla vermez (simülasyon ana makinesi verebilir, aşağıdaki *Hangi ana makine* bölümüne bakın). Politika grafiği tam olarak dört denetim çıktısını (`actions`, `joint_pos_targets`, `stiffness_targets`, `damping_targets`) döndürür ve her biri bir komuttur. Gövde pozlarına politika çağrısının *dışında* ihtiyaç duyan bir ana makine ikinci, küçük bir grafik olan `fk.onnx` dosyasını çalıştırır. Metatron v1 yerel döngüsü böyle yapar: görev hedeflerini kurmak için her çağrıdan önce güncel ölçüm üzerinde FK çalıştırır. `export_fk_to_onnx.py` bu grafiği, politika grafiğinin izlediği aynı `BakedAnchorFK` çekirdek nesnesinden, kontrol noktasının yanındaki aynı yapılandırma paketinden yeniden kurarak izler. Önceki bağımsız FK dışa aktarıcısı ileri kinematiği MJCF üzerinde kendi dördey (quaternion) dönüşümüyle yeniden çalıştırıyordu. Bu, politikanın çekirdeğinden zamanla sapabilecek ikinci bir uygulamaydı. `fk.onnx` ise politikanın çekirdeğinin *kendisidir*. Böylece tek bir FK uygulaması kalır ve iki grafik yapısı gereği aynı sonucu verir. Girdileri `noisy_dof_pos`, `noisy_anchor_pos` ve `noisy_anchor_rot` (toplu boyut 1) aynı yollar için politika grafiğinin adlarını taşır. Bu sayede ana makine tek bir tensörü tek bir adla iki grafiğe birden verir. Çıktıları `body_pos` ve `body_rot` (`xyzw`, robotun gövde sırası) olur. Yan dosyası `fk.yaml`, yerel araçların politika manifestosuyla karşılaştırabilmesi için gövde sırasını, eklem sırasını ve çapayı kaydeder.

**Hangi ana makine.** `export_to_onnx.py --host`, grafiği neyin besleyeceğini adlandırır ve dışa aktarıcı o ana makinenin besleyemeyeceği bir bağlamayı reddeder:

- `robot` (varsayılan): bir robotta ne varsa o, tarif onu hangi ad alanından okursa okusun: eklem okumaları, IMU yönelimi, çapa için bir konumlandırma sonucu, politikanın kendi geçmiş eylemleri, hedef akışları ve sahneye koşullu bir politika için ana makinenin tarif ettiği sahne kutuları (`scene` sağlayıcısı; depoyla gelen G1 izleyicisi girdilerini `current.*` altında, Metatron tarifi `noisy.*` altında bağlar; kural ad alanına göre değil, türe göre işler). Robotun gövde pozları grafiğin içinde tarifin FK çekirdeğiyle hesaplanır, asla dışarıdan verilmez. Robot için hazırlanan bir politika, simülatörde test edilirken bile bu şekilde dışa aktarılır.
- `simulation`: buna ek olarak simülatörün kendi gövde durumu `current.*` altında (her gövdenin pozu ve hızı, çapanın altındaki zemin yüksekliği; `contract.py` içindeki `simulated_*` türleri). Bu yol, kendi simülatörünü çalıştıran ve algılayıcılar yerine gövde durumunu veren bir oyun motoru ya da herhangi bir düzenek içindir. Yan dosya bu durumda `host: simulation` kaydını taşır. Depoyla gelen SOMA izleyicisi (`motion_tracker/soma-bones`) böyle bir politikadır: gözlemleri simülatörün gövde durumunu okur, bu yüzden yalnız `--host simulation` ile dışa aktarılır. Deponun başvuru izleyici ana makinesi (`test_tracker_mujoco.py`) yalnız robot girdileri verir ve bir simülasyon yan dosyasını baştan reddeder. Böyle bir grafiğin ana makinesi motorun kendisidir.

| Dizin/dosya | Sorumluluk | Durum |
| --- | --- | --- |
| `export_to_onnx.py`, `export_fk_to_onnx.py`, `tracker_inputs.py`, `contract.py` | İzleyici dışa aktarımı, [genel Metatron dışa aktarımı](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/metatron_generic_export.md) ve FK grafiği `fk.onnx` | Desteklenen |
| `test_tracker_mujoco.py`, `motion_utils.py`, `state_utils.py` | Mevcut izleyici oynatma ve ortak yardımcılar | Desteklenen |
| `native/include/protomotions/deployment/`, `native/src/` | Yerel arayüzler, çalışma zamanı, güvenlik, robot/girdi ve çıkarım bağdaştırıcıları | Deneysel |
| `native/examples/` | Başlatıcılar ve ortak politika/görev bağdaştırıcıları | Deneysel |
| `native/tools/` | Dışa aktarım, motor hazırlığı, eşleme/denklik ve girdi tanılama | Deneysel |
| `native/tests/` | Yerel ve Python sözleşme testleri ile küçük test verileri | Deneysel |
| `native/bench/` | Çevrim dışı denklik ve gecikme programları | Deneysel |
| `native/scripts/` | Açık kurulum yardımcısı `install.sh`. Metatron paketinin indirilmesi burada değil, deponun kökündeki `scripts/download_metatron.py` dosyasındadır ve `data/pretrained_models/metatron/G1` içine yazar. | Deneysel |

## Derleme ve test

### Çekirdek: GPU ya da robot SDK'sı gerekmez

CMake >=3.20, bir C++20 derleyicisi ve `nlohmann/json` başlıkları gerekir (örneğin Ubuntu'nun `nlohmann-json3-dev` paketi). Buradaki komutlar **ProtoMotions kökünden** başlar:

```bash
cmake -S deployment/native -B deployment/native/build -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=ON
cmake --build deployment/native/build -j4
ctest --test-dir deployment/native/build --output-on-failure
```

Sistem dışı bir JSON kurulumu için `-DNLOHMANN_JSON_INCLUDE_DIR=/path/to/prefix/include` ekleyin. CMake hiçbir şey indirmez. Çekirdeği açık bir öneke kurmak için:

```bash
bash deployment/native/scripts/install.sh --prefix /path/to/deployment-install
```

CMake kullanan projeler `find_package(ProtoMotionsDeployment CONFIG REQUIRED)` çağırıp `ProtoMotions::Deployment` hedefine bağlanabilir. Çekirdek C ABI sürüm 2'dir. İstemcileri eşleşen başlıklarla yeniden derleyin. Bu CMake projesi ayrı derlenir. ProtoMotions Python paketini kurmak onu derlemez ya da kurmaz.

### İsteğe bağlı arka uçlar ve çıkarım

İlgili seçenekleri ayrı bir derleme dizinine ekleyin:

| CMake seçeneği | Gereken dış bağımlılıklar |
| --- | --- |
| `PROTOMOTIONS_BUILD_MUJOCO=ON` | MuJoCo başlıkları/kütüphanesiyle `MUJOCO_ROOT` |
| `PROTOMOTIONS_BUILD_UNITREE_SDK2=ON` | `UNITREE_SDK2_ROOT`, isteğe bağlı olarak `UNITREE_DDS_ROOT` |
| `PROTOMOTIONS_BUILD_METATRON_NATIVE=ON` | Aşağıdaki TensorRT ve CUDA kökleri; Metatron v1 başlatıcısı güncel derlemede MuJoCo, Unitree SDK2 ve evdev de ister |
| `PROTOMOTIONS_BUILD_GPC_TENSORRT=ON` | TensorRT ve CUDA; kesin top-p için isteğe bağlı `PROTOMOTIONS_BUILD_GPC_TRT_PLUGIN=ON` |
| `PROTOMOTIONS_BUILD_EVDEV_INPUT=ON` | Linux; Linux'ta varsayılan olarak açık |
| `PROTOMOTIONS_BUILD_SDL2_INPUT=ON` | SDL2 geliştirme dosyaları ve pkg-config |
| `PROTOMOTIONS_BUILD_PICO_INPUT=ON` | Dış SDK `XROBOTOOLKIT_ROOT` |

GPU hedefleri `TENSORRT_ROOT`, isteğe bağlı `TENSORRT_LIBRARY_ROOT`, `CUDA_RUNTIME_ROOT` ve `CUDA_TOOLKIT_INCLUDE_ROOT` değişkenlerini kullanır. Adına karşın son değişken `include/` dizininin kendisi değil, **`include/` dizinini içeren** araç takımı önekidir. Hedef çalışma zamanıyla eşleşen başlıkları ve kütüphaneleri kullanın. İçe alınan top-p eklentisi derlemesi şu an **sm_120** hedefler. Öteki GPU'lar kopyalanmış bir motorla değil, açıkça ayarlanmış ve doğrulanmış bir CUDA derlemesiyle kullanılır. Politikaya özel derleme kılavuzlarına bakın.

### Python araçları ve testleri

Eşleşen ProtoMotions bağımlılıklarına sahip bir Python ortamı kullanın ve [requirements-deployment.txt](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/requirements-deployment.txt) dosyasını kurun. CUDA/TensorRT ORT için CPU ORT wheel'i **yerine** uyumlu bir `onnxruntime-gpu` kurulumu kullanın. Dışa aktarım ayrıca PyTorch ve ONNX ister. TensorRT hazırlığı onun Python bağlamalarını ister. Hiçbir çalışma zamanı başlatıcısı bağımlılıkları sizin yerinize kurmaz.

İçe alınan araçlar, `PYTHONPATH` üzerinde `deployment/native` dizininden çözümlenen kendi bağımsız `examples.*` / `tools.*` içe aktarımlarını korur. Ayrıntılı çalışma zamanı kılavuzlarındaki komutlar için bu çalışma dizinini ve yol sırasını kullanın. Böylece komutlar ProtoMotions'ın eğitim `examples/` dizinini değil, çalışma zamanının kendi `native/examples/` dizinini çözümler:

```bash
cd deployment/native
export PROTOMOTIONS_ROOT="$(cd ../.. && pwd)"
export PYTHONPATH="$PWD:$PROTOMOTIONS_ROOT${PYTHONPATH:+:$PYTHONPATH}"
python -m pytest tests -q
```

G1 MJCF dosyasını yükleyen testler onun Git LFS ağ (mesh) yüklerini ister. Önce eşleşen varlıkları indirin. LFS işaretçi metni kullanılabilir bir ağ dosyası yerine geçmez. Özel politika çıktıları, üretici SDK'ları ve GPU planları pakette gelmez. Bu çıktılar yokken onlara bağlı kontroller bir modeli doğrulayamaz.

## Eğitilmiş bir politikayı dağıtıma eşleyin

1. Kontrol noktasını ve kaydedilmiş yapılandırma paketini bir arada tutun. Güncel dışa aktarıcı `export_to_onnx.py`, JSON paketini (`resolved_configs.json`) okur. Eski yerel çıktılar `resolved_configs*.pt` dosyalarından dışa aktarılmıştı. Dışa aktarımı rastgele daha yeni bir klonla değil, **o çıktıyı yükleyebilen eğitim kaynağı sürümüyle** yapın. GPC dışa aktarıcısının sabit mimari/API varsayımları vardır. Tarihsel model sınıfları seçilen kaynakta bulunmalıdır.
2. Metatron v1 ve GPC, ONNX grafikleri ile JSON manifestoları kullanır. Metatron v1 ayrıca [LEAPP paket dışa aktarımını ve yerel hazırlığı](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/leapp.md) destekler. Bu, doğrulanmış ONNX grafiğini korur ve YAML arayüzünü ekler. İlgili hazırlık kılavuzlarına bakın. Dış ONNX ağırlık yan dosyalarını modelle birlikte taşıyın. Bu depo GPC ağırlıklarını sağlamaz ya da indirmez. Genel Metatron tarifi için `export_to_onnx.py --leapp` grafiği, LEAPP arayüzünü ve türlü dağıtım sözleşmesini birlikte yazar. `--leapp-graph-history` ise geçmişleri grafiğin üstlendiği bir LEAPP paketi yazar. Paket yazıcıları `native/tools/leapp_export.py` ve `native/tools/leapp_graph_history.py` dosyalarıdır ve `protomotions/tests` tarafından sınanır. O tarifin gerçek girdi kümesi, sabit 39 girdili yerel v1 arayüzünden farklıdır. Bkz. [genel Metatron dışa aktarımı](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/metatron_generic_export.md).
3. Girdileri **anlamsal ad, şekil, veri türü ve zaman kuralına** göre bağlayın. Sıralı eklem/gövde listelerini, ana makinenin leğen IMU'su ve bel eklemlerinden türettiği gövde çapası pozunu, dördey sırasını (dağıtım sınırında `xyzw`), birimleri, geçmiş uzunluklarını, sıfırlama dolgusunu ve politika periyodunu kontrol edin. Tensör boyutlarının eşleşmesi tek başına yetmez.
4. Çıktı eylem alanını kontrol edin. İşlenmiş bir konum hedefine eylem ofseti/ölçeği iki kez uygulanmamalıdır. Ham eylem geçmişi ile işlenmiş hedef geçmişi sözleşmeden sözleşmeye farklıdır. Manifestonun kuralını koruyun ve yalnız uygun kabul edilmiş komut sınırında güncelleyin. Metatron v1 ham geçmişi, kabul edilen hedeften kendi afin eylem eşlemesiyle açıkça yeniden kurar.
5. Dışa aktarılmış çıkarımı, sıfırlamalar, birden çok hedef kipi ve sıralı geçmiş dahil, eğitim başvurusuyla karşılaştırın. Ardından gözetimli herhangi bir fiziksel denemeden önce kapalı döngü bir MuJoCo çalıştırmasını test edin.

Yeni bir robot için `RobotBackend` uygulayın ve onun yeteneklerini, eklem eşlemesini, algılayıcı çerçevelerini, sınırlarını ve acil durum çıktısını tanımlayın. Yeni bir politika için `Controller` (ya da `PolicyRuntime` ile `ArtifactController`) uygulayın ve sözleşme/görev mantığını orada tutun. Yeni bir operatör cihazı için `InputSource` uygulayın ve `InputRouter` bağlantısını başlangıçta kurun. Çalışma zamanına politikaya özel dallar eklemek yerine çalışma zamanı/güvenlik kodunu yeniden kullanın. Her genişletme odaklı sözleşme testleri ister.

## TensorRT iyileştirme ve doğrulama

İyileştirmeyi doğrulanmış bir çıktı hattı olarak ele alın:

```text
checkpoint + saved config -> ONNX + manifest -> parity
   -> target-machine TensorRT plan -> native parity -> warmed latency -> rollout
```

- FP32 ile başlayın. Planları hedeflenen GPU/çalışma zamanı üzerinde derleyin. `.plan` dosyaları ve ORT TensorRT önbellekleri farklı çıktılardır ve genelde makineler arasında taşınamaz.
- Metatron için `native/tools/rewrite_metatron_graphsafe.py` ve `native/tools/validate_metatron_graphsafe.py`, grafik güvenli adayı hazırlar ve denetler. `native/tools/build_metatron_tensorrt.py`, `--optimization-level` (0-5) seçeneğini sunar. FK motorunu `fk.onnx` ve yan dosyası `fk.yaml` üzerinden `native/tools/build_metatron_fk_tensorrt.py` ile derleyin. Bu araç statik bir FP32 planı derler ve `--manifest` verildiğinde yan dosyanın gövde sırasını, eklem sırasını ve çapasını politika manifestosunun `robot_contract` bölümüyle karşılaştırır, uyuşmazlığı reddeder. `native/tools/make_metatron_fk_fixture.py` denklik test verisini aynı iki dosyadan yazar. `body_rot` matristen dördeye dönüşümle elde edildiği için denklik kontrolü `q` ve `-q` değerlerini aynı dönme olarak kabul etmelidir. Komutların tamamı için [Metatron v1 yerel kılavuzunu](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/metatron_v1_native.md) izleyin.
- GPC için `native/tools/prepare_gpc_policy.py` sabit şekilli örnekleme kiplerini dışa aktarır, derler, doğrular ve kıyaslar. Kesin top-p, eşleşen eklentiyi ister. Başlangıç çizgisi için TF32'yi kapalı tutun: kayan nokta hataları küçük görünse bile küçük logit değişimleri ayrık jetonları çevirebilir.
- CUDA grafikleri, ısınmadan sonra kalıcı tamponlar ve sabit şekil/adreslerle başlatma yükünü azaltır. Örnekleme gürültüsü açık bir girdi olarak kalır. Yakalamayı yalnız bunu destekleyen sözleşmelerde açın ve grafik ile akış yollarını karşılaştırın.
- FP16/BF16 sürümlerini güvenli sayılan yükseltmeler olarak değil, ayrı adaylar olarak test edin. Daha düşük duyarlıklı bir motoru kabul etmeden önce aileye özgü denklik araçlarını yeniden çalıştırın.
- Isınmış yerel H2D/çıkarım/D2H zamanlamasını ve durumdan komuta tam gecikmeyi ayrı ayrı ölçün. İkincisine algılayıcı tazeliğini ve komut süre sınırlarını dahil edin. Düşük salt politika gecikmesi bir donanım güvenliği ya da kararlılık sonucu yerine geçmez.

Üretilen modelleri, planları, önbellekleri, kayıtları ve üretici SDK'larını Git dışında tutun. Sabitlenmiş Metatron v1 ağırlıkları NVIDIA'nın iç kullanımındadır ve bu depoyla dağıtılmaz. Çekirdeği derlemek ya da yalnız kaynak gerektiren testleri çalıştırmak için gerekmezler.

## Ek okuma

[Mimari](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/architecture.md) ·
[Metatron v1 sözleşmesi](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/metatron_v1_policy.md) ·
[Metatron v1 yerel çalışma zamanı](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/metatron_v1_native.md) ·
[GPC](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/docs/gpc_tensorrt.md) · [Kıyaslamalar](https://github.com/NVlabs/ProtoMotions/blob/main/deployment/native/bench/README.md)

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
