> Bu belge NVlabs/ProtoMotions deposundaki `docs/source/getting_started/quickstart.rst` dosyasının Türkçe çevirisidir (Apache-2.0, değişiklik: Türkçeleştirme).

# Hızlı Başlangıç

Bu kılavuz, önceden eğitilmiş modelleri çalıştırmanıza ve kendi ajanlarınızı eğitmeye başlamanıza yardım eder.

## Önceden eğitilmiş modelleri çalıştırın

Birkaç robot ve denetleyici ailesi için önceden eğitilmiş kontrol noktaları (checkpoint) sunuyoruz. Kısa bir genel bakış ve her kontrol noktasının yanında duran ayrıntılı model kartlarının bağlantıları için [önceden eğitilmiş modeller](https://nvlabs.github.io/ProtoMotions/getting_started/pretrained_models.html) kataloğuna bakın.

Aşağıdaki komutlar önerilen IsaacLab kontrol noktalarını kullanır. Uyumlu simülatörlere ya da donanıma aktarılması beklenen tek model, alan rastgeleleştirmesiyle (domain randomization) eğitilmiş G1 dağıtım izleyicisidir.

**Örnek hareket verisi:**

Robot modelleriyle test için küçük örnek hareket dosyaları sunuyoruz:

* `data/motion_for_trackers/g1_random_subset_tiny.pt` - G1 için yeniden hedeflenmiş (retargeted) AMASS verisinin küçük bir alt kümesi
* `data/motion_for_trackers/g1_bones_seed_mini.pt` - G1 için yeniden hedeflenmiş BONES-SEED hareketlerinin küçük bir alt kümesi
* `data/motion_for_trackers/soma23_bones_seed_mini.pt` - SOMA 23 gövdeli insansı için BONES-SEED hareketlerinin küçük bir alt kümesi
* `data/motion_for_trackers/anatomic_soma_bones_seed_mini.pt` - Anatomik SOMA insansısına yeniden hedeflenmiş on BONES-SEED klibi
* `data/motion_for_trackers/crouch_soma23.pt` - GPC ve PEFT örnekleri için küçük bir SOMA çömelme veri seti
* `data/motion_for_trackers/h1_2_random_subset_tiny.pt` - H1-2 için yeniden hedeflenmiş AMASS verisinin küçük bir alt kümesi

SMPL hareket verisi için AMASS'tan kendi MotionLib'inizi üretmek üzere [AMASS veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/amass_preparation.html) belgesine bakın. Yerel GPU belleğiniz AMASS'ın tüm hareket kütüphanesini yüklemeye yetmiyorsa, kütüphaneyi küçük bir alt kümeye indiren basit bir `scripts/subset_motion_lib.py` betiği bulunur.

**Çıkarımı çalıştırın:**

```bash
# Run G1 on BONES-SEED motions
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/inference_last.ckpt \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --simulator isaaclab

# Headless validation on a server or VM
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/inference_last.ckpt \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --simulator isaaclab \
    --num-envs 100 \
    --headless \
    --full-eval

# Run SOMA 23-body humanoid on BONES-SEED motions
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/soma-bones/inference_last_lab.ckpt \
    --motion-file data/motion_for_trackers/soma23_bones_seed_mini.pt \
    --simulator isaaclab

# Run the SOMA BONES-SEED FSQ tracker used by GPC
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/soma_bones_fsq/inference_last.ckpt \
    --motion-file data/motion_for_trackers/soma23_bones_seed_mini.pt \
    --simulator isaaclab

# Run the anatomical SOMA BONES-SEED FSQ tracker (IsaacLab or MuJoCo)
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/anatomic_soma_bones_fsq/inference_last.ckpt \
    --motion-file data/motion_for_trackers/anatomic_soma_bones_seed_mini.pt \
    --simulator mujoco --num-envs 1

# Run the SOMA GPC prior with its SOMA BONES-SEED FSQ decoder
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/gpc_prior/soma_bones/inference_last.ckpt \
    --motion-file data/motion_for_trackers/crouch_soma23.pt \
    --simulator isaaclab \
    --num-envs 1

# Run the anatomical SOMA GPC prior with its anatomical FSQ decoder
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/gpc_prior/anatomic_soma_bones/inference_last.ckpt \
    --motion-file data/motion_for_trackers/anatomic_soma_bones_seed_mini.pt \
    --simulator isaaclab \
    --num-envs 1

# Test the domain-randomized G1 tracker in MuJoCo
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/inference_last.ckpt \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --simulator mujoco \
    --num-envs 1
```

> **Not:** Ekransız makinelerde `--full-eval` ile doğrulama genelde daha kolaydır, çünkü hareket kümesini değerlendirdikten sonra çıkar ve metrikleri yazdırır. `--full-eval` olmadan çıkarım siz durdurana kadar sürekli çalışır.
>
> Simülatörler arası aktarım, model kartı politikanın aktarıma yönelik tam alan rastgeleleştirme tarifiyle eğitildiğini söylüyorsa beklenmelidir. Ayrıntılar için [önceden eğitilmiş modeller](https://nvlabs.github.io/ProtoMotions/getting_started/pretrained_models.html) ve [alan rastgeleleştirmesi](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/domain_randomization.html) belgelerine bakın.

## İlk ajanınızı eğitin

### DeepMimic ile hareket taklidi eğitimi

MLP politikası kullanan bir hareket taklidi ajanı eğitin. Aşağıdaki komut depoyla gelen G1 hareket dosyalarından birini kullandığı için yeni bir klonda doğrudan çalışır:

```bash
python protomotions/train_agent.py \
    --robot-name g1 \
    --simulator isaaclab \
    --experiment-path examples/experiments/mimic/mlp.py \
    --experiment-name g1_mimic_example \
    --motion-file data/motion_for_trackers/g1_random_subset_tiny.pt \
    --num-envs 4096 \
    --batch-size 16384 \
    --ngpu 1
```

Hareket dosyası, eğittiğiniz robot için hazırlanmış olmalıdır. `data/motion_for_trackers/` içindeki her dosya tek bir robota aittir ve ortam, gövde ve serbestlik derecesi (DoF) sayıları robotunkinden farklı olan bir dosyayı reddeder. Kendi hareket kütüphanenizi kurmak için, örneğin SMPL insansısı için AMASS hareketleri, [AMASS veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/amass_preparation.html) belgesine bakın.

## Simülatör ve robot seçimi

### Simülatör seçimi

`--simulator` bağımsız değişkenini kullanın:

* `isaacgym` - NVIDIA IsaacGym eski GPU arka ucu
* `isaaclab` - NVIDIA IsaacLab/IsaacSim (eğitim için önerilir)
* `newton` - NVIDIA Newton 1.0.0 (MuJoCo Warp üzerine kurulu)
* `mujoco` - Yalnız CPU üzerinde MuJoCo (tek ortam, hızlı test ve hata ayıklama için)

### Robot seçimi

`--robot-name` bağımsız değişkenini kullanın:

| Robot | Açıklama |
| --- | --- |
| `smpl` | SMPL insansısı (dijital insan) |
| `smplx` | Elli SMPL-X insansısı |
| `g1` | Unitree G1 insansı robot |
| `h1_2` | Unitree H1 insansı robot (sürüm 2) |
| `amp` | AMP insansısı |
| `soma23` | SOMA 23 gövdeli insansı (dijital insan) |

Kendi robotunuzu eklemek için [özel robot](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/custom_robot.html) belgesine bakın.

## Deney yönetimi

`--experiment-name`, sonuçların nereye kaydedileceğini belirler. Var olan bir deney adıyla eğitim başlatırsanız eğitim son kontrol noktasından kendiliğinden sürer.

Sonuçlar şuraya kaydedilir:

```text
results/<experiment_name>/
├── config.yaml                      # CLI arguments and wandb ID
├── resolved_configs.json            # The config bundle (tagged JSON) - source of truth
├── experiment_config.py             # Copy of experiment file
├── last.ckpt                        # Latest model checkpoint
├── score_based.ckpt                 # Best-performing checkpoint (by eval score)
├── epoch_100.ckpt                   # Intermediate checkpoints (if configured)
└── env_<task_id>.ckpt               # Environment state for exact resume
```

> **Not:** Sürdürme (deney adı aynıysa) kaydedilmiş yapılandırmaları birebir kullanır. Sürdürme sırasında komut satırı geçersiz kılmaları yok sayılır. Bu tasarım, kümelerde çok GPU'lu çalıştırmaların kendiliğinden sürdürülmesine yardım eder.
>
> Yapılandırma değişiklikleri için yeni bir deney adı kullanın. Bulutta ya da kümede eğitirken kaynak kodu yeni bir dizine kopyalayıp orada istediğiniz deney adıyla da eğitebilirsiniz.

> **Uyarı:** **resolved_configs.json dosyasını elle düzenlemeyin.** Bu dosya tek doğru kaynaktır: yüklemede gerçek kurucular üzerinden yeniden kurulan, düz metin, etiketli bir JSON ağacıdır. Elle düzenlenen bir paket alanı, sınıfı ve dosyayı adlandıran bir hatayla durur. Yapılandırma değişiklikleri için `--overrides` (küçük değişiklikler) ya da hem `resolved_configs.json` hem `experiment_config.py` dosyasını yazan `--create-config-only` kullanın. Büyük değişikliklerde ikisini de kontrol noktası dizininize kopyalayın, yoksa çıkarım eski tarifin kancasını yeni yapılandırmalarla çalıştırır. [Yapılandırma](https://nvlabs.github.io/ProtoMotions/user_guide/configuration.html) belgesine bakın.

## Eğitim yapılandırması

Sık kullanılan yapılandırma seçenekleri:

```bash
python protomotions/train_agent.py \
    --robot-name g1 \
    --simulator isaaclab \
    --experiment-path examples/experiments/mimic/mlp.py \
    --experiment-name my_experiment \
    --motion-file data/motion_for_trackers/g1_random_subset_tiny.pt \
    --num-envs 4096 \
    --batch-size 16384 \
    --ngpu 1 \
    --training-max-steps 10000000
```

### Yapılandırma geçersiz kılmaları

Yapılandırma değerlerini çalışma anında değiştirmek için `--overrides` kullanın:

```bash
--overrides "agent.num_mini_epochs=4" "env.max_episode_length=500"
```

**Desteklenen geçersiz kılma biçimi:** `config_type.field.subfield=value`

Desteklenen yapılandırma türleri: `env`, `simulator`, `robot`, `agent`, `terrain`, `motion_lib`, `scene_lib`

**Değerler:** sağ taraf bir Python sabiti (literal) olarak okunur, sonra hedef alanın bildirdiği türe göre denetlenir. Skaler değerler (`int`, `float`, `bool`, `str`), `None` ve liste ya da demet sabitleri kabul edilir. Sözlük sabiti, türü bir dataclass olan alanı doldurur. Geçerli sabit olmayan metin dize olarak tutulur. Yanlış türde bir değer ya da yapılandırmanın bildirmediği bir alan adı, yapılandırmalar birleştirilirken, herhangi bir simülatör ya da ortam kurulmadan önce, alanı adlandıran bir hatayla reddedilir.

```bash
--overrides "env.max_episode_length=500" \
    'robot.contact_bodies=["left_ankle_roll_link","right_ankle_roll_link"]'
```

Robot gövdelerinin listesini, bu G1 örneğindeki gibi, robotun kendi gövde adlarıyla yazın.

**Sınırlamalar:** bir geçersiz kılma tek seferde tek alan ayarlar. Yeni bir ödül terimi ya da farklı bir gözlem kümesi gibi aynı anda birkaç bileşene dokunan bir değişiklik yeni bir deney dosyasına aittir. Bu, farklı deney yapılandırmalarını yönetmek ve izlemek için de iyi bir uygulamadır.

Yapılandırma sistemiyle ilgili ayrıntılar için [yapılandırma](https://nvlabs.github.io/ProtoMotions/user_guide/configuration.html) belgesine bakın.

## Weights & Biases ile kayıt

Önce wandb kimlik doğrulamasını kurun:

```bash
wandb login
```

Ardından deney takibini açın:

```bash
python protomotions/train_agent.py \
    ... \
    --use-wandb \
    --wandb-project my_project
```

Varsayılan W&B projesi `physical_animation` adını taşır.

İzlenecek temel metrikler:

* `eval/gt_error/mean` - Mimic MLP tarifindeki ortalama konum izleme hatası. Metrik adları deneyin değerlendirici yapılandırmasından gelir.
* `eval/success_rate` - Değerlendirilen denemelerden yapılandırılmış başarısızlık kurallarını geçenlerin oranı. Bu oran, düşmeden tamamlanan hareketlerin oranıyla her zaman aynı olmaz.
* `info/episode_reward` - Eğitim ödülü (öncelikli örneklemeden etkilenir).
* `actor/clip_frac` - PPO olasılık oranı kırpılan örneklerin oranı.
* `actor/grad_norm_before_clip` / `critic/grad_norm_before_clip` - Kırpmadan önceki gradyan büyüklükleri.

> **İpucu:** Weights & Biases'ın temel metrik grafiklerinin ötesinde pek çok yararlı özelliği vardır. Çalıştırmaları herhangi bir yapılandırma parametresine göre arayıp süzebilir, yan yana karşılaştırabilir ve özel panolar kurabilirsiniz. Deney takibinden en iyi verimi almak için arayüzü incelemeye biraz zaman ayırın.

## Değerlendirme

Eğitilmiş bir ajanı değerlendirin:

```bash
# Evaluate G1 pretrained model
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/inference_last.ckpt \
    --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
    --simulator isaaclab

# Evaluate SOMA pretrained model
python protomotions/inference_agent.py \
    --checkpoint data/pretrained_models/motion_tracker/soma-bones/inference_last_lab.ckpt \
    --motion-file data/motion_for_trackers/soma23_bones_seed_mini.pt \
    --simulator isaaclab

# Or evaluate your own trained model. Pass a motion file made for the robot
# you trained, or omit --motion-file to reuse the training motion file.
python protomotions/inference_agent.py \
    --checkpoint results/my_experiment/last.ckpt \
    --motion-file data/motion_for_trackers/g1_random_subset_tiny.pt \
    --simulator isaaclab
```

### Klavye denetimleri

Görselleştirme sırasında:

| Tuş | Açıklama |
| --- | --- |
| `J` | Tüm robotlara fiziksel kuvvet uygular (dayanıklılık testi) |
| `R` | Görevi sıfırlar |
| `O` | Kamerayı değiştirir (varlıklar arasında döner) |
| `L` | Video kaydını açar ya da kapatır |
| `Q` | Çıkar |

## Sonraki adımlar

* [AMASS veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/amass_preparation.html) - AMASS hareket verisini hazırlayın
* [Eğitimler](https://nvlabs.github.io/ProtoMotions/tutorials/index.html) - Baştan sona iş akışı eğitimleri
* [Kavramlar](https://nvlabs.github.io/ProtoMotions/concepts/index.html) - Temel soyutlamaları öğrenin
* [Yapılandırma](https://nvlabs.github.io/ProtoMotions/user_guide/configuration.html) - Yapılandırma sistemine ayrıntılı bakış

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
