> Bu belge NVlabs/ProtoMotions deposundaki `docs/source/tutorials/workflows/g1_deployment.rst` dosyasının Türkçe çevirisidir (Apache-2.0, değişiklik: Türkçeleştirme).

# G1 Tüm Gövde İzleyicisi: Veriden Gerçek Robota

ProtoMotions, Unitree G1 insansı robot için hareket verisinden gerçek robota dağıtıma kadar eksiksiz ve tümüyle yeniden üretilebilir bir hat sunar. Her adım (veri hazırlama, yeniden hedefleme, pekiştirmeli öğrenme eğitimi, ONNX dışa aktarımı, MuJoCo doğrulaması ve gerçek robota dağıtım) tümüyle açık kaynaklı kod ve veriyle, en az bağımlılıkla yürür.

Tasarım ilkeleri:

* **Modüler ve genel:** her aşama, girdisi ve çıktısı açık, kendi içinde tamamlanmış bir betiktir.
* **Yalın ve okunur:** dağıtım kodu ağır çatılardan uzak durur. Başvuru MuJoCo test betiği (`deployment/test_tracker_mujoco.py`), algılayıcı ve bağlam yardımcıları için içe aktarma yolunda `protomotions` paketinin yanında `mujoco`, `onnxruntime`, `numpy`, `pyyaml` ve `torch` ister. Simülatör arka ucu ve GPU gerektirmez.
* **Yeniden üretilebilir:** önceden eğitilmiş genel hareket izleyicisi, 24 A100 GPU ile tüm [BONES-SEED](https://huggingface.co/datasets/bones-studio/seed) veri seti (yaklaşık 142 bin yeniden hedeflenmiş G1 hareketi, bkz. [SEED G1 CSV veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/seed_g1_csv_preparation.html)) üzerinde eğitildi. Bununla birlikte, çok daha az GPU ve daha küçük bir hareket alt kümesiyle yapılan eğitim de pek çok yaygın hareket (yürüme, dönme, jestler vb.) için makul bir genel tüm gövde izleyicisi üretir.

> **Not:** Önceden eğitilmiş bir kontrol noktası ve onun dışa aktarılmış ONNX modeli `data/pretrained_models/motion_tracker/g1-bones-deploy/` içinde gelir. Böylece eğitimi atlayıp doğrudan dağıtıma geçebilirsiniz. Örnek G1 hareketleri depoda başka yerlerde durur: tekil klipler `data/g1-kimodo-generated/proto/` altında (`output_walk.motion`, `output_wave.motion`, `output_dance.motion`, `output_high5.motion` ve `output_jumpjack.motion`), küçük kütüphane ise `data/motion_for_trackers/g1_bones_seed_mini.pt` dosyasında. Test betiği bu kütüphaneyi `--motion-index` ile dizinler.

## Hattın genel görünümü

```text
BONES-SEED CSVs (open-source, ~142K retargeted G1 motions)
     |
     v  data/scripts/convert_g1_csv_to_proto.py
ProtoMotions .motion files (30 fps)
     |
     v  protomotions/train_agent.py (IsaacGym/IsaacLab, multi-GPU)
Trained checkpoint (last.ckpt + resolved_configs.json + experiment_config.py)
     |
     v  deployment/export_to_onnx.py (CPU, no simulator)
Unified ONNX model + YAML metadata
     |
     +---> deployment/test_tracker_mujoco.py  (reference MuJoCo test)
     |
     +---> robojudo/ integration              (MuJoCo sim + real G1)
```

## Tasarım notları

**Birleşik ONNX modeli.** Dışa aktarılan ONNX, gözlem hesabını, aktör ağını ve eylem işlemeyi (tanh + PD ofset/ölçek) tek bir modelde toplar. Dağıtım çatıları eklem konumlarını ve hızlarını, gövde (torso) yönelimini, leğen (pelvis) açısal hızını ve gelecekteki hareket referans karelerini verir. Gözlem fonksiyonlarını yeniden yazmaları gerekmez. Gövde yönelimi bir IMU okuması değildir: varsayılan dağıtım yolu yalnız leğen IMU'sunu okur ve gövde yönelimini bu IMU ile eklem kodlayıcılarından ileri kinematikle (forward kinematics) hesaplar (bkz. [Sensör gereksinimleri](#sensör-gereksinimleri)). Bu, eğitim ile dağıtım arasında fark çıkma olasılığını en aza indirir.

**Önce MuJoCo doğrulaması.** Gerçek donanıma dokunmadan önce ONNX modelini bağımsız bir MuJoCo betiğinde (`deployment/test_tracker_mujoco.py`) doğruluyoruz. Betik hiçbir simülatör arka ucu kullanmaz ve yalnız birkaç küçük ProtoMotions modülüne dayanır (`deployment/tracker_inputs.py` ve `deployment/contract.py` dosyalarının arkasındaki algılayıcı ve bağlam sözlükleri). Bu betik **dağıtım sözleşmesi** işlevi görür: davranışını yeniden üreten her çatı politikayı doğru sürer.

**Önbelleğe alınmış 50 fps hareket.** Referans hareketler kendi FPS değerlerinden (genelde 30) denetim hızına (50 Hz) eğitimdeki SLERP/lerp ara değerlemesinin aynısıyla yeniden örneklenir, sonra bir `.pt` dosyasına önbelleğe alınır. Sonraki çalıştırmalar PyTorch hesabı olmadan, salt NumPy dizi dizinlemesiyle ilerler.

**Yön hizalama.** Politika, robotun gerçek gövde yönü ile hareketin 0. karedeki yönü arasında yalnız sapma (yaw) açısından oluşan bir yön ofseti hesaplar. Bu hep açıktır ve hem simülasyonu (ofset birim dönüşüme yakındır) hem gerçek donanımı (robot açıldığında herhangi bir yöne bakıyor olabilir) karşılar.

**RoboJuDo entegrasyonu.** G1 için dağıtım çatısı olarak [RoboJuDo](https://github.com/HansZ8/RoboJuDo) kullanıyoruz. Temiz, okunur ve hafiftir. Modüler bir Policy / Environment / Controller mimarisi vardır ve ağır bağımlılık taşımaz. ProtoMotions izleyicisi, RoboJuDo çekirdeğinde değişiklik gerekmeden tek bir Policy sınıfı olarak bağlanır.

## Adım 1: Hareket verisini hazırlayın

BONES-SEED G1 CSV dosyalarını ProtoMotions `.motion` dosyalarına dönüştürmek ve bunları bir MotionLib `.pt` dosyasında paketlemek için [SEED G1 CSV veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/seed_g1_csv_preparation.html) belgesini izleyin.

İsterseniz depoda hazır duran örnek G1 hareketlerini kullanın: `data/g1-kimodo-generated/proto/output_walk.motion` (ve kardeşleri) ya da `data/motion_for_trackers/g1_bones_seed_mini.pt`.

## Adım 2: Eğitin (ya da önceden eğitilmişi kullanın)

**Önceden eğitilmiş kontrol noktasıyla** (başlangıç için önerilir):

Önceden eğitilmiş model `data/pretrained_models/motion_tracker/g1-bones-deploy/` içindedir. Kontrol noktasını, çözümlenmiş yapılandırmaları ve önceden dışa aktarılmış bir ONNX modelini içerir. Adım 3'e geçin.

**Sıfırdan eğitim:**

```bash
python protomotions/train_agent.py \
    --robot-name g1 \
    --simulator isaaclab \
    --experiment-path data/pretrained_models/motion_tracker/g1-bones-deploy/experiment_config.py \
    --experiment-name g1_bm_tracker \
    --motion-file /path/to/bones_seed_g1_motions.pt \
    --num-envs 4096 \
    --batch-size 16384
```

Deney yapılandırması (`data/pretrained_models/motion_tracker/g1-bones-deploy/experiment_config.py`), BeyondMimic tarzı ayarları bizim küçük değişikliklerimizle kullanır:

* **Gözlemler:** indirgenmiş koordinatlarda öz algı (gürültülü) + çok ufuklu indirgenmiş koordinat hedef pozları (adımlar [1, 2, 4, 8]) + önceki işlenmiş eylemler. L2C2 düzgünlük düzenlemesi için temiz (gürültüsüz) karşılıklar da eklenir.
* **Ödüller:** BeyondMimic'in yönden bağımsız göreli gövde izlemesi (konum + yönelim + doğrusal/açısal hız, bölge ağırlıklı) ile birlikte küresel çapa (anchor) yönelimi, eylem hızı cezası ve yumuşak eklem sınırı cezası.
* **Eylem işleme:** BeyondMimic PD eylem yapılandırması (`make_bm_pd_action_config`).
* **Alan rastgeleleştirmesi:** gözlem gürültüsü, sürtünme rastgeleleştirmesi, kütle merkezi rastgeleleştirmesi, itme bozucuları.

## Adım 3: ONNX dışa aktarın

Politikayı birleşik bir ONNX modeline dışa aktarın. Bunun için PyTorch, ProtoMotions paketi ve `onnxruntime` gerekir. Simülatör ya da GPU gerekmez. `onnxruntime` isteğe bağlı değildir: betik grafiği yazdıktan hemen sonra YAML yan dosyası için girdi adlarını okumak üzere onu açar. `onnxruntime` kurulu değilse çalışma `.onnx` yazıldıktan sonra durur ve YAML üretilmez (`--no-validate` yalnız sonraki doğrulama geçişini atlar).

```bash
python deployment/export_to_onnx.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/inference_last.ckpt \
    --output /path/to/export_dir
```

Bu komut `--output` dizininde şunları üretir:

* `unified_pipeline.onnx`: ONNX modeli
* `unified_pipeline.yaml`: her girdi ve çıktıyı, zamanlamayı, kuralları ve çerçeve gereksinimlerini belgeleyen zengin üst veri

`--output` vermezseniz dosyalar kontrol noktasının yanına, `<checkpoint_dir>/compiled_models/` içine yazılır. Depoyla gelen paket için bu `data/pretrained_models/motion_tracker/g1-bones-deploy/compiled_models/` dizinidir ve aşağıda anlatılan önceden dışa aktarılmış çiftin üzerine yazar. Onları korumak için `--output` verin.

Varsayılan `--host robot` bu izleyici için doğru seçimdir: gözlemleri yalnız eklem kodlayıcılarını, IMU'yu, kendi geçmiş işlenmiş eylemlerini ve referans hareketi okur. Bunları bir robot sağlayabilir. Gözlemleri simülatörün kendi gövde durumunu okuyan bir politika (örneğin depoyla gelen SOMA izleyicisi) varsayılan ayarla reddedilir. Böyle bir politika bir oyun motoru, tarayıcıda MuJoCo ya da simülasyondan simülasyona (sim-to-sim) kontrol için `--host simulation` ile dışa aktarılır. Yan dosya bu durumda `host: simulation` kaydını taşır.

Kendi eğittiğiniz bir model için `--checkpoint` bayrağını `results/<experiment>/last.ckpt` dosyasına yönlendirin, dışa aktarım bu dosyayla da yürür. Daha ince olan `inference_last.ckpt` (yalnız ağırlıklar, iyileştirici durumu yok), yalnız tarif ajan yapılandırmasında `save_inference_checkpoint=True` ayarladığında yazılır. Depoyla gelen dağıtım tarifi bu ayarı kullanmaz.

YAML üst verisi makinenin okuyabileceği bir dağıtım sözleşmesi işlevi görür. Her girdi için beklenen şekilleri, kaynak ifadelerini ve en önemlisi açısal hız girdilerinin referans çerçeve kuralını belgeler (bkz. aşağıdaki [açısal hız çerçevesi kuralı](#önemli-açısal-hız-çerçevesi-kuralı)).

> **Not:** Önceden dışa aktarılmış bir ONNX modeli `data/pretrained_models/motion_tracker/g1-bones-deploy/compiled_models/unified_pipeline.onnx` yolunda hazır gelir.

## Adım 4: MuJoCo'da test edin

Başvuru MuJoCo test betiği ONNX modelini en az bağımlılıkla doğrular. Bu betik **dağıtım sözleşmesidir**: davranışını tutturan her çatı politikayı doğru sürer.

```bash
# First run: resample motion to 50fps and cache it next to the clip
python deployment/test_tracker_mujoco.py \
    --onnx data/pretrained_models/motion_tracker/g1-bones-deploy/compiled_models/unified_pipeline.onnx \
    --motion data/g1-kimodo-generated/proto/output_walk.motion \
    --cache-motion --render

# Subsequent runs use the cached motion (skips the SLERP resampling step)
python deployment/test_tracker_mujoco.py \
    --onnx data/pretrained_models/motion_tracker/g1-bones-deploy/compiled_models/unified_pipeline.onnx \
    --motion data/g1-kimodo-generated/proto/output_walk.50fps.pt \
    --render
```

Ek test kipi:

```bash
# Headless benchmark
python deployment/test_tracker_mujoco.py \
    --onnx ... --motion ... --no-realtime
```

Betik şunları üstlenir: MJCF yükleme ve yamalama, PD yapılandırması, yön hizalama, ivme kırpma ve gerçek zamanlı adımlama. EMA süzgeci kullanılmaz: yan dosyadaki `action_ema_alpha` her zaman `1.0` değerindedir (süzme yok) ve dışa aktarıcı başka bir değer ayarlamaz.

## Adım 5: RoboJuDo ile dağıtın (simülasyon)

[RoboJuDo](https://github.com/HansZ8/RoboJuDo), insansı robotlar için MuJoCo simülasyonu ve Unitree gerçek robot desteği olan, tak çalıştır bir dağıtım çatısıdır. ProtoMotions BM izleyici desteği RoboJuDo'nun ana koduna alındı, yama gerekmez.

**Kurulum:**

```bash
git clone https://github.com/HansZ8/RoboJuDo.git
cd RoboJuDo
git checkout release   # contains the ProtoMotions tracker integration
pip install -e .
git lfs pull           # download mesh assets
```

**İzleyiciyi MuJoCo simülasyonunda çalıştırın:**

İzleyicinin kendi başlatıcısı vardır. Genel `scripts/run_pipeline.py` yalnız `-c` kabul eder. `scripts/run_tracker_pipeline.py` buna ONNX ve hareket bayraklarını ekler.

```bash
python scripts/run_tracker_pipeline.py -c g1_protomotions_tracker \
    --onnx-path /path/to/unified_pipeline.onnx \
    --motion-path /path/to/motion.motion
```

RoboJuDo'nun ana koddaki entegrasyonu (`release` dalı) şunları sağlar:

* `ProtoMotionsTrackerPolicy`: ONNX'i ve önbelleğe alınmış 50 fps hareketi yükler. Eylem geçmişi geri beslemesiyle PD hedeflerini, yön hizalamayı ve hareketin yumuşak giriş/çıkış (fade-in/fade-out) desteğini sağlar.
* Sanal askı (virtual gantry): gerçek robota dağıtım geçişleri için yay-sönümleyici güvenlik koşumu.
* Harmanlayarak giriş ve çıkış (blend-in/blend-out): boşta pozu ile politika denetimi arasında yumuşak geçişler.
* `-c`, `--onnx-path`, `--motion-path` ve `--motion-index` (çok hareketli bir `.pt` kütüphanesindeki dizin) bayraklarıyla `scripts/run_tracker_pipeline.py`. `release` dalında bunların dışında izleyici bayrağı bulunmaz.

> **Not:** `release` dalındaki izleyici politikası grafiğe yalnız eski `historical.processed_actions` girdisini verir ve tanımadığı grafik girdilerini sessizce atlar. `g1-bones-deploy` içinde gelen ONNX güncel sözleşmeyle yeniden dışa aktarıldı ve `processed_action_history` girdisini ister (YAML dosyasında `output_key: joint_pos_targets`, `steps_back: [1]` ve `fill_before_available: zeros` yazar). Bu yüzden onu `--onnx-path` ile verdiğinizde onnxruntime eksik zorunlu girdi hatasıyla durur. Entegrasyon bu girdiyi besleyene kadar depoyla gelen modeli `deployment/test_tracker_mujoco.py` ile çalıştırın ya da RoboJuDo'yu izleyicinin kendi paketli kopyasıyla (`assets/models/g1/protomotions_tracker/`) kullanın. Bu kopya, YAML dosyası hala `output_key: robot_action` ile `historical.processed_actions` adını taşıyan eski bir dışa aktarımdır. Orada bilinen bir sapma var: RoboJuDo, geri beslenen hedefleri döndürüldükleri haliyle değil, ivme kırpmasından sonra kaydeder (aşağıdaki sözleşme maddesi 2). Bu yüzden eklem hedeflerinin `deployment/test_tracker_mujoco.py` hedeflerinden biraz daha sarsıntılı olmasını bekleyin. İzlemenin kendisi bundan pek etkilenmez.

**Kendi dağıtım çatınızı getirmek:** temel sözleşme şudur:

1. YAML yan dosyasında belgelenen ONNX girdilerini verin (eklem konumları ve hızları, gövde yönelimi, leğenin yerel açısal hızı, önceki işlenmiş eylemler, gelecekteki hareket referansları).
2. Her eylem geçmişi girdisine, grafiğin önceki çağrılardaki kendi çıktısını, döndürüldüğü haliyle, herhangi bir kırpma ya da yumuşatmadan önce kaydedilmiş olarak verin: `processed_action_history` (ya da eski `historical.processed_actions`) için `joint_pos_targets`, `raw_action_history` için `actions`. YAML girdisi bu çıktıyı adlandırır (`output_key`), her yuvanın ne kadar geriye uzandığını söyler (`steps_back`: `[1]` önceki çağrı, `[2]` ondan önceki) ve sıfırlamadan sonra o kadar çağrı çalışana kadar ne verileceğini belirtir (`fill_before_available`). Paketli G1 YAML dosyası üçünü de taşır (`output_key: joint_pos_targets`, `steps_back: [1]`, `fill_before_available: zeros`). Yalnız RoboJuDo'nun kendi paketli kopyası gibi eski dışa aktarımlar `output_key: robot_action` yazar ve bu `joint_pos_targets` anlamına gelir. `steps_back` yoksa iki güncel girdi için `[1]`, `historical.processed_actions` için `[2]` kullanın, dolgu sıfır olsun. `deployment/tracker_inputs.py` içindeki `ActionFeedback` bunların hepsini yapar.
3. ONNX çıktılarını (PD konum hedefleri) robotunuzun eyleyicilerine uygulayın.
4. YAML `control` bölümünde belgelenen ivme kırpmasını uygulayın (`pd_target_max_accel`; `null` kırpma yok demektir). Orada `action_ema_alpha` her zaman `1.0` olduğu için uygulanacak EMA bulunmaz.
5. MuJoCo davranışının kendi çatınız ile `deployment/test_tracker_mujoco.py` arasında eşleştiğini doğrulayın.

## Adım 6: Gerçek G1 robota dağıtın

> **Uyarı:** Gerçek robota dağıtmadan önce [RoboJuDo README](https://github.com/HansZ8/RoboJuDo#alert--disclaimer-%EF%B8%8F%EF%B8%8F%EF%B8%8F) içindeki güvenlik uyarılarını okuyun. Acil durdurmanın çalıştığını her zaman doğrulayın. Politikalar denge kaybında sert hareketlere yol açabilir.

Genel gerçek robot kurulumu (Ethernet bağlantısı, güvenlik duvarı yapılandırması, Unitree SDK kurulumu) için [RoboJuDo gerçek robot kılavuzuna](https://github.com/HansZ8/RoboJuDo#run-robojudo-on-real-robot-) bakın.

Robot bağlandıktan sonra şunu çalıştırın:

```bash
cd RoboJuDo
python scripts/run_tracker_pipeline.py -c g1_protomotions_tracker_real \
    --onnx-path /path/to/unified_pipeline.onnx \
    --motion-path /path/to/motion.motion
```

`g1_protomotions_tracker_real` yapılandırması, simülasyon yapılandırmasını Unitree gerçek robot ortamı (`UnitreeCppEnv`) ve açık güvenlik denetimleriyle genişletir. Yapılandırmadaki `net_if` alanını ağ arayüzünüze ayarlayın.

### Güvenlik

Robotu **acil durdurmak** için Unitree kumandasındaki **A düğmesine** istediğiniz an basın. Kumandayı her zaman elinizde tutun ve bir dağıtım oturumuna başlamadan önce acil durdurmanın çalıştığını doğrulayın.

### Dağıtım aşamaları

Gerçek robota dağıtım, boşta durum ile politika denetimi arasında güvenle geçmek için dört aşama kullanır. Her aşama gereklidir: aşamaları atlamak ya da aceleye getirmek, robota ya da çevreye zarar verebilecek ani eklem hareketleri riski taşır.

**Aşama 1: Yükselme (ramp-up).** Eklemler, robotun o anki pozundan hareket klibinin ilk karesine yavaşça ara değerlenir. Bu aşamada robot bir askıda tutulmalıdır. Devam etmeden önce geçişin bitmesini bekleyin.

**Aşama 2: Harmanlayarak giriş (blend-in).** Politika devreye girer ama hedef olarak yalnız ilk kareyi alır. Bu, politikaya o pozu kendi başına korumasını söyler. Denetim birkaç saniye içinde durağan pozdan politikaya doğrusal olarak geçer. Bu aşamada operatör robotu askıda yavaşça indirir ve robot kendi dengesiyle yere basar. Aşama 2'nin sonunda robot tümüyle özerktir ve askı bırakılabilir.

**Aşama 3: İzleme (tracking).** Politika hareketin tamamını alır ve onu yeniden kurmaya başlar. Robot hareket klibini baştan sona uygular.

**Aşama 4: Harmanlayarak çıkış (blend-out).** Hareket bittiğinde ya da operatör Unitree kumandasından yumuşak çıkışı tetiklediğinde süreç tersine döner: PD hedefleri politika çıktısından başlangıç pozuna doğru harmanlanır. Robot yumuşakça dengeli bir ayakta duruş pozuna döner. Operatör bu aşamada askıyı yeniden destek konumuna kaldırır, böylece harmanlayarak çıkış bittiğinde robot tümüyle desteklenir. Harmanlayarak çıkıştan sonra hat bir sonraki hareket için harmanlayarak girişe dönebilir ya da oturum sona erdirilebilir.

### Sensör gereksinimleri

ProtoMotions izleyici politikası gerçek G1 robotla uyumludur. Tüm ONNX girdileri fiziksel G1 üzerindeki algılayıcılara karşılık gelir:

* `dof_pos` / `dof_vel`: eklem kodlayıcılarından
* `anchor_rot` (gövde): leğen IMU'su + eklem kodlayıcılarından ileri kinematikle hesaplanır
* `root_local_ang_vel` (leğen): doğrudan leğen IMU jiroskopundan (zaten gövdeye yerel çerçevede)
* Gelecekteki hareket referansları: önbelleğe alınmış hareket dosyasından

Eğitim sırasındaki alan rastgeleleştirmesi (gözlem gürültüsü, sürtünme rastgeleleştirmesi, itme bozucuları), gerçek dünyadaki algılayıcı gürültüsüne ve dinamik farklarına karşı dayanıklılık sağlar.

## Önemli: açısal hız çerçevesi kuralı

ONNX modeli `root_local_ang_vel` değerini **leğen gövdesinin yerel çerçevesinde** bekler. Farklı kaynaklar açısal hızı farklı çerçevelerde verir:

| Kaynak | Çerçeve | Ne yapmalı |
| --- | --- | --- |
| MuJoCo `data.cvel[body+1, 0:3]` | Dünya | `quat_rotate_inverse(pelvis_rot, ang_vel)` uygulayın |
| MuJoCo `data.qvel[3:6]` (serbest eklem) | Yerel | Doğrudan kullanın, döndürme yok |
| Gerçek robot IMU jiroskopu | Yerel | Doğrudan kullanın, döndürme yok |

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
