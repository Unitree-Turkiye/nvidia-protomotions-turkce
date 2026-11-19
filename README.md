<div align="center">

# NVIDIA ProtoMotions 3 Türkçe: Unitree G1 ve İnsansı Robotlar için GPU Hızlandırmalı Simülasyon ve Eğitim Çatısı

**ProtoMotions 3: A GPU-Accelerated Framework for Simulated Humanoids**

**Fizik tabanlı hareket öğrenimi · Isaac Gym, IsaacLab, Newton ve MuJoCo · Unitree G1 için simülasyondan gerçek robota dağıtım**

**Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org · [www.robotlar.org](https://www.robotlar.org)**

👉 **Unitree G1 ürün sayfası: [robotlar.org/tr/insansi-robotlar/unitree-g1](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · G1 EDU için teklif: [robotlar.org/tr/fiyat/unitree-g1-edu-fiyat](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat)**

📖 **Türkçe rehber: [NVIDIA ProtoMotions insansı simülasyon rehberi](https://www.robotlar.org/tr/rehber/nvidia-protomotions-insansi-simulasyon-rehberi)**

[Genel Bakış](#genel-bakış) · [Neler Yapabilirsiniz](#protomotions3-ile-neler-yapabilirsiniz) · [Simden Gerçeğe](#-simülasyondan-gerçek-robota) · [Belgeler](#-belgeler) · [Simülatör Seçimi](#-simülatör-seçimi-ve-türkçe-kaynaklar) · [G1 ile Başlangıç](#-unitree-g1-ile-protomotions-türkiyede-nereden-başlanır) · [Unitree Ürünleri](#-unitree-ürün-ailesi-türkiye)

[![Lisans](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/NVlabs/ProtoMotions/blob/main/LICENSE.md)
[![Belgeler](https://img.shields.io/badge/docs-online-green.svg)](https://nvlabs.github.io/ProtoMotions/)

[![Newton](https://img.shields.io/badge/Newton-1.0.0-brightgreen.svg)](https://pypi.org/project/newton/1.0.0/)
[![IsaacLab](https://img.shields.io/badge/IsaacLab-3.0-blue.svg)](https://github.com/isaac-sim/IsaacLab/commit/4ecd0b036da19ff6ad2bb4d621f886b63e9f6db8)
[![IsaacGym](https://img.shields.io/badge/IsaacGym-Preview_4-blue.svg)](https://developer.nvidia.com/isaac-gym)
[![MuJoCo](https://img.shields.io/badge/MuJoCo-3.0+-orange.svg)](https://github.com/google-deepmind/mujoco)

</div>

> Bu depo, NVIDIA'nın (NVlabs) resmi [NVlabs/ProtoMotions](https://github.com/NVlabs/ProtoMotions) projesinin belgelerinin Türkçe çevirisidir. Kaynak kod, önceden eğitilmiş modeller, hareket dosyaları ve görseller orijinal depoda kalır. Burada yalnız Türkçe anlatım bulunur ve görseller orijinal depodan gösterilir.
>
> **Kaynak sürüm:** `main` dalı, commit `13de0cc` (9 Kasım 2026).
>
> **Lisans dağılımı:** kaynak kod ve belgeler Apache-2.0, NVIDIA'nın sağladığı önceden eğitilmiş modeller NVIDIA Open Model License kapsamındadır. Veri setleri ve robot varlıkları kendi koşullarını korur. Ayrıntı [Lisans](#lisans) bölümünde.
>
> **Kodu çalıştırmak için orijinal depoyu klonlayın:** [github.com/NVlabs/ProtoMotions](https://github.com/NVlabs/ProtoMotions)
>
> ProtoMotions, NVIDIA'nın projesidir. Çeviri ve Türkçe teknik destek: Unitree Türkiye resmi distribütörü [Robotlar.org](https://www.robotlar.org).

---

## 📰 Haberler

- **28 Ekim 2026: Metatron yayımlandı.** Metatron, Unitree G1 için geliştirilen üretken tüm gövde politikasıdır: hareket izleme, yönlendirme (steering), yol izleme, seyrek gövde hedeflerine uzanma, sahne etkileşimi ve düştükten sonra ayağa kalkma işlerini tek bir ağ üstlenir. [**Makaleyi**](https://research.nvidia.com/labs/par/projects/metatron/assets/metatron-paper.pdf?v=de2411a21822) ve [**proje sayfasını**](https://research.nvidia.com/labs/par/projects/metatron/) okuyun, [**etkileşimli demoyla**](https://nvlabs.github.io/ProtoMotions/demos/metatron/) tarayıcınızda deneyin, [**G1 Racer**](https://nvlabs.github.io/ProtoMotions/demos/g1-racer/) ile altı tanesini yarıştırın, ağırlıkları ve yerel paketi `python scripts/download_metatron.py` ile [**Hugging Face**](https://huggingface.co/nvidia/Metatron-v1) üzerinden indirin ya da [**Metatron G1 öğrenci kılavuzuyla**](https://nvlabs.github.io/ProtoMotions/user_guide/metatron_student.html) kendinizinkini eğitin.
- **28 Ekim 2026: `main` büyük bir yeniden düzenleme aldı.** Ortam artık modüler durum sahipleri ve görev denetim bileşenleri üzerinde çalışan kısa bir açık adım aşamaları listesidir. Aralarındaki sınırlar denetlendiği için bir yerdeki değişiklik başka bir yeri sessizce bozamaz. Deneyler daha temiz bir [**yapılandırma sistemi**](https://nvlabs.github.io/ProtoMotions/user_guide/configuration.html) ve [**gözlemleri, ödülleri ve sonlandırmaları**](https://nvlabs.github.io/ProtoMotions/concepts/environment_context.html) adlandırılmış bağlam yollarına bağlı saf tensör çekirdekleri olarak tanımlamanın bildirimsel bir yolunu kullanır. Algılayıcılar kanal başına bir kez, her biri [**ölçüm gürültüsü ve bir durum kestirimi aşamasıyla**](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/domain_randomization.html#privileged-clean-and-noisy-sources) tanımlanır. Bu da alan rastgeleleştirmesine (domain randomization) ve simülasyondan gerçeğe (sim-to-real) aktarıma ilkeli bir zemin verir. Önceki sürümlerden kaydedilmiş yapılandırmaların yeniden üretilmesi gerekir ve Genesis arka ucu kaldırıldı. Tam liste [**değişiklik günlüğünde**](https://nvlabs.github.io/ProtoMotions/changelog.html).

---

## Genel Bakış

**ProtoMotions3**, fiziksel olarak benzetilen dijital insanları ve insansı robotları eğitmek için GPU hızlandırmalı bir simülasyon ve öğrenme çatısıdır. Amacımız; **animasyon**, **robotik** ve **pekiştirmeli öğrenme** alanındaki araştırmacılar ve uygulayıcılar için çeşitli benzetilmiş insansı öğrenme görevlerine ve ortamlarına yönelik **hızlı bir prototipleme platformu** sunmak ve topluluklar arasındaki çalışmaları birbirine bağlamaktır.

ProtoMotions3'ün özünde **modülerlik**, **genişletilebilirlik** ve **ölçeklenebilirlik** yer alır. Proje **topluluk odaklıdır** ve serbest [Apache-2.0 lisansıyla](https://github.com/NVlabs/ProtoMotions/blob/main/LICENSE.md) yayımlanır.

Hareket taklidi öğrenimi için hafif bir çatı olan kardeş depomuz **[MimicKit](https://github.com/xbpeng/MimicKit/tree/main)**'e de göz atın.

---

## ProtoMotions3 ile neler yapabilirsiniz

### 🎭 Üretken politikalar

Tek bir referans klibi izlemek yerine görevi bitirmek için kendi "hareketini" seçen üretken bir politika eğitin. [**MaskedMimic**](https://research.nvidia.com/labs/par/maskedmimic/), birkaç hedef gövde, bir yol ya da sahnedeki nesneler gibi kısmi hedeflerden bir hareketi tamamlayan tek bir politika öğrenir. GPC tarifi yeniden kullanılabilir ayrık bir gizil öncül (latent prior) eğitir, sonra onu küçük PEFT bağdaştırıcılarıyla yeni görevlere uyarlar. Bkz. [**GPC ve PEFT kılavuzu**](https://nvlabs.github.io/ProtoMotions/user_guide/gpc.html). Yayımladığımız Unitree G1 politikası [**Metatron**](https://research.nvidia.com/labs/par/projects/metatron/) bu yolla eğitildi: bir hareket izleme öğretmeninden damıtılmış, MaskedMimic tarzı bir öğrencidir ve öğrendiği öncül izleme, yönlendirme, gövde hedefi, sahne ve toparlanma hedeflerini tüm gövde hareketine çevirir. [**Tarayıcı demosunda**](https://nvlabs.github.io/ProtoMotions/demos/metatron/) çalıştırın, [**G1 Racer**](https://nvlabs.github.io/ProtoMotions/demos/g1-racer/) ile yarıştırın, ağırlıkları [**Hugging Face**](https://huggingface.co/nvidia/Metatron-v1) üzerinden indirin ya da kendinizinkini eğitmek için [**Metatron G1 öğrenci kılavuzunu**](https://nvlabs.github.io/ProtoMotions/user_guide/metatron_student.html) izleyin.

<table align="center">
<tr>
<td align="center"><img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/gpc_parkour.gif" alt="GPC ile insan ve sahne etkileşimi" height="200"/></td>
<td align="center"><img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/racer.gif" alt="G1 Racer" height="200"/></td>
</tr>
<tr>
<td align="center"><sub>GPC: insan ve sahne etkileşimi</sub></td>
<td align="center"><sub><a href="https://nvlabs.github.io/ProtoMotions/demos/g1-racer/">G1 Racer</a>: altı robot, toplu çalışan tek bir Metatron politikası, tarayıcınızda</sub></td>
</tr>
<tr>
<td align="center" colspan="2"><img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/metatron.gif" alt="Gerçek Unitree G1 üzerinde Metatron" width="640"/></td>
</tr>
<tr>
<td align="center" colspan="2"><sub>Gerçek G1 üzerinde <a href="https://nvlabs.github.io/ProtoMotions/demos/metatron/">Metatron</a>: oturma ve kalkma, tüm gövde teleoperasyonu, yerden toparlanma</sub></td>
</tr>
</table>

### 🏃 Büyük ölçekli hareket öğrenimi

Tümüyle fiziksel olarak benzetilen karakterinize, herkese açık [**AMASS**](https://amass.is.tue.mpg.de/) insan animasyon veri setinin tamamından (**40+ saat**) hareket becerilerini 4 A100 ile **12 saat** içinde öğretin.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/smpl_mlp_094132.gif" alt="SMPL hareketi 1" height="180">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/smpl_mlp_094428.gif" alt="SMPL hareketi 2" height="180">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/smpl_mlp_095344.gif" alt="SMPL hareketi 3" height="180">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/smpl_mlp_095848.gif" alt="SMPL hareketi 4" height="180">
</p>

### 📈 Ölçeklenebilir çok GPU'lu eğitim

Her GPU'nun hareketlerin bir alt kümesini üstlendiği eğitimle daha da büyük veri setlerine ölçekleyin. Örneğin [**SOMA**](https://github.com/NVlabs/SOMA-X) iskelet biçimindeki [**BONES**](https://huggingface.co/datasets/bones-studio/seed) veri setiyle, GPU başına **13 bin hareketle** **24 A100** üzerinde eğitim yaptık. Veri setini ve önceden eğitilmiş modelleri bugün denemek için [Hızlı Başlangıç](docs/HIZLI-BASLANGIC.md) ve [SEED BVH veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/seed_bvh_preparation.html) belgelerine bakın.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/soma_regen_markers.gif" alt="SOMA hareketi, işaretçilerle" height="180">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/soma_regen_2.gif" alt="SOMA hareketi 2" height="180">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/soma_regen_3.gif" alt="SOMA hareketi 3" height="180">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/soma_regen_5.gif" alt="SOMA hareketi 5" height="180">
</p>

### 🔄 Tek komutla yeniden hedefleme

[AMASS](https://amass.is.tue.mpg.de/) veri setinin tamamını, yerleşik [**PyRoki**](https://github.com/chungmin99/pyroki) tabanlı iyileştiriciyle tek komutta sevdiğiniz robota aktarın (yeniden hedefleyin, retarget).

> **Not:** v3 itibarıyla yeniden hedefleme için [PyRoki](https://github.com/chungmin99/pyroki) kullanıyoruz. Önceki sürümler [Mink](https://github.com/kevinzakka/mink) kullanıyordu.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/retargeting-g1.gif" alt="Unitree G1'e yeniden hedefleme" height="280">
</p>

### 🤖 Her robotu eğitin

Tek bir komut bağımsız değişkenini değiştirerek robotunuza AMASS motor becerilerini **12 saatte** öğretin:
`--robot-name=smpl` → `--robot-name=h1_2` ve yeniden hedeflenmiş hareketleri hazırlayın (bkz. [buradaki belge](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/retargeting_pyroki.html))

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/h1_2_gym.gif" alt="H1_2 AMASS eğitimi" height="280">
</p>

### 🔬 Simülasyondan simülasyona (Sim2Sim) test

Robot denetim politikalarını **H1_2** ya da **G1** üzerinde farklı fizik motorlarında (NVIDIA Newton, MuJoCo CPU) tek tıkla test edin (`--simulator=isaacgym` → `--simulator=newton` → `--simulator=mujoco`). Aşağıda gösterilen politikalar yalnız gerçek donanımdan alınabilecek gözlemleri kullanır.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/h12-g1-newton-sim2sim.gif" alt="H1_2 ve G1 sim2sim" height="280">
</p>

### 🤖 Simülasyondan gerçek robota

Simülasyonda eğitin, gerçek donanıma dağıtın. ProtoMotions, [**BONES-SEED**](https://huggingface.co/datasets/bones-studio/seed) veri setinin tamamı (yaklaşık 142 bin hareket) üzerinde tek bir Genel İzleme Politikası (General Tracking Policy) eğitir ve bu politika Unitree G1 insansı robota sıfır atışla (zero-shot) doğrudan aktarılır.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/g1_deploy_1.gif" alt="Unitree G1 dağıtımı 1" height="240">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/g1_deploy_2.gif" alt="Unitree G1 dağıtımı 2" height="240">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/real_robot_2.gif" alt="Gerçek Unitree G1 robot" height="240">
</p>

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/sit-sim2real.gif" alt="G1 sahne kutuları ve gövde hedefleriyle mobilyaya oturuyor, MuJoCo'da ve gerçek robotta" width="400">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/run-sim2real.gif" alt="Gerçek G1 farklı yönlere yönlendiriliyor" width="400">
</p>

Dağıtım hattımız gözlem hesabının içine gömüldüğü tek bir ONNX modeli dışa aktarır. Böylece dağıtım çatılarının yalnız ham algılayıcı sinyallerini vermesi yeter: gözlem fonksiyonlarını yeniden yazmak ya da eğitimin iç yapısını eşlemek gerekmez. Unitree G1 üzerinde başarılı [**RoboJuDo**](https://github.com/HansZ8/RoboJuDo) çatısıyla test ettik. Bunun için tek bir politika dosyası eklemek yetti, RoboJuDo çekirdeğinde zorunlu değişiklik gerekmedi.

📖 [**Tam dağıtım eğitimi**](docs/G1-GERCEK-ROBOTA-DAGITIM.md): veri hazırlamadan gerçek robota, tümüyle yeniden üretilebilir.

Metatron gibi büyük üretken politikaların dağıtımı, küçük bir izleme ağınınkinden farklı bir problemdir: birkaç yüz megabaytlık ağırlığa sahip bir dönüştürücü (transformer) her adımda kendi politika periyodu içinde bir komut döndürmek zorundadır. Bu yüzden gecikme, genel amaçlı bir Python yığınına bırakılmak yerine baştan sona mühendislikle tasarlanmalıdır. Bunun için kendi dağıtım çalışma zamanımızın da prototipini geliştiriyoruz: MuJoCo ve Unitree G1 arka uçları, bir güvenlik denetçisi, bir politika eşleme iş akışı ve bir TensorRT derleme ve doğrulama hattı olan bağımsız bir C++ çalışma zamanı. Bkz. [**Dağıtım çalışma zamanı (deneysel)**](docs/DAGITIM-CALISMA-ZAMANI.md). O klasör, kendi dağıtım yolunu kuranlar için prototipleme başvuru kodudur, desteklenen bir sürüm çıktısı değildir.

### 🎨 Yüksek doğrulukta görüntü oluşturma

Politikanızı [**IsaacSim 5.0+**](https://developer.nvidia.com/isaac-sim) içinde test edin. IsaacSim, [**Omniverse NuRec**](https://developer.nvidia.com/blog/reconstruct-a-scene-in-nvidia-isaac-sim-using-only-a-smartphone/) ile güzel işlenmiş Gauss sıçratma (Gaussian splatting) arka planları yüklemenize izin verir (bu işlenmiş sahne şimdilik fiziksel etkileşime açık değildir).

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/g1-neurc.gif" alt="G1 ve NuRec sahnesi" height="280">
</p>

### 🎬 Kimodo ile hareket hazırlama

[**Kimodo**](https://research.nvidia.com/labs/sil/projects/kimodo/) (NVIDIA'nın metinden harekete üretim modeli) ile bir metin isteminden herhangi bir hareket üretin ve hem SOMA animasyon karakteri hem de Unitree G1 robot için bu hareketi uygulayan fizik tabanlı bir politikayı ProtoMotions ile eğitin. Bu yolla eğitilen politikalar doğrudan gerçek donanıma dağıtılabilir.

Kimodo çıktılarını ProtoMotions biçimine dönüştürmek için [Kimodo veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/kimodo_preparation.html) belgesine bakın.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/aibm-vaulting.gif" alt="Engelden atlama" height="240">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/g1_robot_walking.gif" alt="Yürüyen G1 robot" height="240">
</p>

> *Görsel kaynağı: [NVIDIA Human Motion Modeling Research](https://research.nvidia.com/labs/sil/human_motion_modeling/)*

### 🏗️ Yordamsal sahne üretimi

Ölçeklenebilir **Sentetik Veri Üretimi (SDG)** için çok sayıda sahneyi yordamsal olarak üretin: bir tohum hareket kümesiyle başlayın, hareketleri çeşitlendirilmiş sahnelere uyarlamak için pekiştirmeli öğrenmeyi kullanın.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/augmented_combined.gif" alt="Çeşitlendirilmiş sahneler ve hareketler" height="280">
</p>

### ⛰️ Arazide yol bulma

Robotunuzu zorlu arazilerde yürümesi için eğitin!

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/smpl_terrain.gif" alt="SMPL arazi" height="280">
</p>

### 🎯 Özel ortamlar

Yeni bir göreviniz mi var? Onu modüler bileşenlerden kurun, tek parça bir ortam sınıfına gerek yok. **Yönlendirme (steering)** görevi şöyle kurulur:

| Katman | Dosya | Ne yapar |
|-------|------|-------------|
| **Denetim** | [`steering_control.py`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/envs/control/steering_control.py) | Görev durumunu (hedef yön, hız, bakış yönü) yönetir. Belirli aralıklarla yeni yön hedefleri örnekler. |
| **Gözlem** | [`obs/steering.py`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/envs/obs/steering.py) | Saf tensör çekirdeği: hedefleri robotun yerel çerçevesine dönüştürür → 5 boyutlu öznitelik vektörü. |
| **Ödül** | [`rewards/task.py`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/envs/rewards/task.py) | `compute_heading_velocity_rew`: yön eşleme (0.7) ve bakış eşleme (0.3) ödüllerini harmanlar. |
| **Deney** | [`steering/mlp.py`](https://github.com/NVlabs/ProtoMotions/blob/main/examples/experiments/steering/mlp.py) | Bileşenleri bağlam yolları üzerinden `MdpComponent` örnekleri olarak birbirine bağlar. |

Her parça bağımsız bir fonksiyon ya da sınıftır. Deney yapılandırması onları [`MdpComponent`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/envs/mdp_component.py) ve [`FieldPath`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/envs/context_views.py) tanımlayıcılarıyla eksiksiz bir göreve bağlar.

<p align="center">
  <img src="https://media.githubusercontent.com/media/NVlabs/ProtoMotions/main/data/static/g1_steering.gif" alt="G1 yönlendirme" height="280">
</p>

### 🧪 Yeni pekiştirmeli öğrenme algoritmaları

Yeni bir pekiştirmeli öğrenme algoritması denemek mi istiyorsunuz? Modüler tasarımımızdan yararlanarak **ADD** gibi algoritmaları ProtoMotions'ta yaklaşık 50 satır kodla uygulayın:

📄 [`protomotions/agents/mimic/agent_add.py`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/agents/mimic/agent_add.py)

### 🔧 Özel simülatörler

Kendi simülatörünüzü mü kullanmak istiyorsunuz? Farklı simülatörler arasında arayüz kuran şu API'leri uygulayın:

📄 [`protomotions/simulator/base_simulator/`](https://github.com/NVlabs/ProtoMotions/tree/main/protomotions/simulator/base_simulator/)

En küçük başvuru uygulaması olarak MuJoCo arka ucuna bakın:

📄 [`protomotions/simulator/mujoco/`](https://github.com/NVlabs/ProtoMotions/tree/main/protomotions/simulator/mujoco/)

### 🤖 Kendi robotunuzu ekleyin

Kendi robotunuzu eklemek mi istiyorsunuz? Şu adımları izleyin:

1. `.xml` MuJoCo tanım dosyanızı [`protomotions/data/assets/mjcf/`](https://github.com/NVlabs/ProtoMotions/tree/main/protomotions/data/assets/mjcf/) dizinine ekleyin
2. Doğrudan tabanı `RobotConfig` olan tam olarak bir sınıf tanımlayan `protomotions/robot_configs/<your_robot>.py` dosyasını oluşturun ([`protomotions/robot_configs/g1.py`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/robot_configs/g1.py) dosyasını kopyalayıp alanları doldurun)
3. Kayıt bundan ibarettir: [`protomotions/robot_configs/factory.py`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/robot_configs/factory.py) o dizini tarar, bu yüzden dosyanın kök adı `--robot-name` ile verdiğiniz addır. Fabrikada düzenlenecek bir şey yoktur. Robot bu depoya girecekse ve test paketini çalıştırıyorsanız, kayıtlı listeyi sabitleyen [`protomotions/tests/test_robot_facts_d1.py`](https://github.com/NVlabs/ProtoMotions/blob/main/protomotions/tests/test_robot_facts_d1.py) içindeki `_ROBOTS` listesine onun adını da ekleyin.

Hepsi bu kadar!

---

## Belgeler (orijinal)

📚 **[Tam belgeler](https://nvlabs.github.io/ProtoMotions/)**

- [Kurulum Kılavuzu](docs/KURULUM.md) (Türkçe)
- [Hızlı Başlangıç](docs/HIZLI-BASLANGIC.md) (Türkçe)
- [GPC ve PEFT](https://nvlabs.github.io/ProtoMotions/user_guide/gpc.html)
- [AMASS veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/amass_preparation.html)
- [PHUMA veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/phuma_preparation.html)
- [SEED BVH veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/seed_bvh_preparation.html)
- [SEED G1 CSV veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/seed_g1_csv_preparation.html)
- [Kimodo veri hazırlama](https://nvlabs.github.io/ProtoMotions/getting_started/kimodo_preparation.html)
- [Eğitimler](https://nvlabs.github.io/ProtoMotions/tutorials/)
- [API başvurusu](https://nvlabs.github.io/ProtoMotions/api_reference/)
- [G1 dağıtımı: veriden gerçek robota](docs/G1-GERCEK-ROBOTA-DAGITIM.md) (Türkçe)

---

## Katkı

Katkılarınızı memnuniyetle karşılıyoruz! Çekme isteği (pull request) göndermeden önce lütfen [**Katkı Kılavuzumuzu**](https://github.com/NVlabs/ProtoMotions/blob/main/CONTRIBUTING.md) okuyun.

## Lisans

ProtoMotions3 kaynak kodu [**Apache-2.0 Lisansı**](https://github.com/NVlabs/ProtoMotions/blob/main/LICENSE.md) ile yayımlanır.

NVIDIA'nın sağladığı önceden eğitilmiş modeller, model kartları açıkça başka bir lisans belirtmedikçe **NVIDIA Open Model License** ile yayımlanır. Her model dizini kendi model kartını, lisansını ve atıf bildirimini içerir. Bkz. [model lisanslama ve yeniden dağıtım](https://github.com/NVlabs/ProtoMotions/blob/main/legal/MODEL-LICENSING.md).

Üçüncü taraf yazılım ve paketli varlık bildirimleri, Unitree, BeyondMimic, Isaac Lab ve SMPL/SMPL-H atıf ve lisans bildirimleri dahil, [legal/](https://github.com/NVlabs/ProtoMotions/tree/main/legal/) klasöründe listelenir.

**Bu çevirinin lisansı:** Bu depodaki metinler, NVIDIA belgelerinin Türkçe çevirisidir ve kaynakla aynı [Apache-2.0](https://www.apache.org/licenses/LICENSE-2.0) lisansıyla yayımlanır. Özgün eser: NVIDIA, [NVlabs/ProtoMotions](https://github.com/NVlabs/ProtoMotions) belgeleri. Değişiklik: Türkçeye çeviri ve Türkiye bağlantıları. Bu depo kaynak kod, model, hareket verisi ya da medya dosyası barındırmaz. Görseller orijinal depodan gösterilir. Ayrıntı [LICENSE](LICENSE) dosyasında.

---

## Atıf

ProtoMotions3'ü araştırmanızda kullanırsanız lütfen şu atfı verin:

```bibtex
@misc{ProtoMotions,
  title = {ProtoMotions3: An Open-source Framework for Humanoid Simulation and Control},
  author = {Tessler*, Chen and Jiang*, Yifeng and Peng, Xue Bin and Coumans, Erwin and Shi, Yi and Zhang, Haotian and Rempe, Davis and Chechik†, Gal and Fidler†, Sanja},
  year = {2025},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/NVLabs/ProtoMotions/}},
}
```

---

## 📚 Belgeler

Bu depoda Türkçeye çevrilen belgeler, Unitree G1 ile kurulumdan gerçek robota dağıtıma uzanan yolu izler:

- **[Kurulum](docs/KURULUM.md):** dört simülasyon arka ucunun (IsaacGym, IsaacLab, Newton, MuJoCo) kurulumu, uv ile bağımlılık olarak kullanım ve sorun giderme.
- **[Hızlı Başlangıç](docs/HIZLI-BASLANGIC.md):** önceden eğitilmiş modelleri çalıştırma, ilk G1 taklit ajanını eğitme, simülatör ve robot seçimi, deney yönetimi, yapılandırma geçersiz kılmaları ve değerlendirme.
- **[Unitree G1 robot modeli](docs/UNITREE-G1-ROBOT-MODELI.md):** 29 DoF'lu G1 modelinin varsayılan sözleşmesi, gövdeleri ve PD varsayılanları.
- **[G1 tüm gövde izleyicisi: veriden gerçek robota](docs/G1-GERCEK-ROBOTA-DAGITIM.md):** BONES-SEED verisi, eğitim, ONNX dışa aktarımı, MuJoCo testi, RoboJuDo ile simülasyon ve gerçek G1 dağıtımı, güvenlik ve dağıtım aşamaları.
- **[Dağıtım çalışma zamanı (deneysel)](docs/DAGITIM-CALISMA-ZAMANI.md):** dışa aktarım araçları, C++20 çalışma zamanı, Unitree SDK2 arka ucu, Metatron v1 ve TensorRT doğrulama hattı.

| Türkçe belge | Kaynak dosya |
| --- | --- |
| [docs/KURULUM.md](docs/KURULUM.md) | `docs/source/getting_started/installation.rst` |
| [docs/HIZLI-BASLANGIC.md](docs/HIZLI-BASLANGIC.md) | `docs/source/getting_started/quickstart.rst` |
| [docs/UNITREE-G1-ROBOT-MODELI.md](docs/UNITREE-G1-ROBOT-MODELI.md) | `docs/source/robots/g1.rst` |
| [docs/G1-GERCEK-ROBOTA-DAGITIM.md](docs/G1-GERCEK-ROBOTA-DAGITIM.md) | `docs/source/tutorials/workflows/g1_deployment.rst` |
| [docs/DAGITIM-CALISMA-ZAMANI.md](docs/DAGITIM-CALISMA-ZAMANI.md) | `deployment/README.md` |

Öteki belgeleri orijinal adresinden okuyabilirsiniz:

- [Önceden eğitilmiş modeller](https://nvlabs.github.io/ProtoMotions/getting_started/pretrained_models.html) ve [Metatron G1 öğrenci kılavuzu](https://nvlabs.github.io/ProtoMotions/user_guide/metatron_student.html)
- [PyRoki ile yeniden hedefleme](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/retargeting_pyroki.html), [özel robot](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/custom_robot.html) ve [alan rastgeleleştirmesi](https://nvlabs.github.io/ProtoMotions/tutorials/workflows/domain_randomization.html)
- [Yapılandırma sistemi](https://nvlabs.github.io/ProtoMotions/user_guide/configuration.html), [değerlendirme](https://nvlabs.github.io/ProtoMotions/user_guide/evaluation.html) ve [geliştirici ipuçları](https://nvlabs.github.io/ProtoMotions/user_guide/developer_tips.html)
- [IsaacLab 3 geçişi](https://nvlabs.github.io/ProtoMotions/user_guide/isaaclab3_migration.html), [Slurm ile eğitim](https://nvlabs.github.io/ProtoMotions/user_guide/slurm_training.html) ve [kavramlar](https://nvlabs.github.io/ProtoMotions/concepts/index.html)
- [G1 Racer](https://nvlabs.github.io/ProtoMotions/user_guide/g1_racer.html), [G1 futbol](https://nvlabs.github.io/ProtoMotions/user_guide/g1_football.html) ve [tarayıcı demosu](https://nvlabs.github.io/ProtoMotions/user_guide/browser_demo.html)

## 🧭 Simülatör seçimi ve Türkçe kaynaklar

ProtoMotions aynı politikayı `--simulator` bayrağıyla dört fizik motorunda çalıştırır. Ayrıntılı kurulum adımları [Kurulum](docs/KURULUM.md) belgesinde.

| Simülatör | ProtoMotions'taki yeri | İlgili Türkçe kaynağımız |
| --- | --- | --- |
| Isaac Gym (Preview 4) | Eski GPU arka ucu, Python 3.8, NVIDIA'dan elle kurulum | [Unitree RL Gym rehberi](https://www.robotlar.org/tr/rl-gym): Isaac Gym'de eğitim, MuJoCo'da doğrulama · [unitree-rl-gym-turkce](https://github.com/Unitree-Turkiye/unitree-rl-gym-turkce) |
| IsaacLab (12.0.0, Isaac Sim 6.0) | Eğitim için önerilen arka uç, önerilen kontrol noktaları burada eğitildi | [Unitree G1 Isaac GR00T rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-isaac-gr00t-rehberi) |
| Newton (1.0.0) | NVIDIA Warp üzerine kurulu, GPU hızlandırmalı fizik motoru | [Unitree G1 MotionBricks rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-motionbricks-rehberi): MotionBricks da görüntüleyici ve sahne için Newton kullanır |
| MuJoCo (3.0+) | Yalnız CPU, tek ortam, hızlı doğrulama ve dağıtım öncesi test | [unitree-mujoco-turkce](https://github.com/Unitree-Turkiye/unitree-mujoco-turkce) · [Unitree RL Gym rehberi](https://www.robotlar.org/tr/rl-gym) |

## 🧬 NVIDIA İnsansı Hareket Ailesi (Türkçe)

NVIDIA'nın animasyon, robotik ve fiziksel yapay zeka için yayımladığı insansı hareket projeleri birbirini tamamlar: SOMA ortak gövde modelini, BONES-SEED veriyi, GEM videodan hareketi, Kimodo ve ARDY hareket üretimini, MotionBricks gerçek zamanlı hareket planlamayı, SOMA Retargeter insan hareketini Unitree G1'e aktarmayı, ProtoMotions fizik tabanlı eğitimi, GEAR SONIC de gerçek robotta tüm gövde kontrolünü üstlenir. Akışın bütününü [merkez rehberde](https://www.robotlar.org/tr/rehber/nvidia-insansi-hareket-ekosistemi-unitree-g1-rehberi) anlatıyoruz.

| Proje | Ne yapar | Türkçe repo | Türkçe rehber | Resmi depo |
|---|---|---|---|---|
| MotionBricks | Bağlam ve hedef pozlar arasında gerçek zamanlı hareket üretir, Unitree G1 modeliyle gelir | [unitree-g1-motionbricks-turkce](https://github.com/Unitree-Turkiye/unitree-g1-motionbricks-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/unitree-g1-motionbricks-rehberi) | [NVlabs/motionbricks](https://github.com/NVlabs/motionbricks) |
| Kimodo | Denetlenebilir çevrim dışı insan hareketi üretimi | [nvidia-kimodo-turkce](https://github.com/Unitree-Turkiye/nvidia-kimodo-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-kimodo-hareket-uretimi-rehberi) | [nv-tlabs/kimodo](https://github.com/nv-tlabs/kimodo) |
| ARDY | Kimodo üzerine gerçek zamanlı, etkileşimli hareket üretimi | [nvidia-ardy-turkce](https://github.com/Unitree-Turkiye/nvidia-ardy-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-ardy-gercek-zamanli-hareket-rehberi) | [nv-tlabs/ardy](https://github.com/nv-tlabs/ardy) |
| SOMA Body Model | Birleşik, parametrik insan gövde modeli | [nvidia-soma-govde-modeli-turkce](https://github.com/Unitree-Turkiye/nvidia-soma-govde-modeli-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-soma-govde-modeli-rehberi) | [NVlabs/SOMA-X](https://github.com/NVlabs/SOMA-X) |
| BONES-SEED | SOMA ve G1 biçiminde büyük hareket yakalama veri seti | [bones-seed-hareket-veri-seti-turkce](https://github.com/Unitree-Turkiye/bones-seed-hareket-veri-seti-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/bones-seed-hareket-veri-seti-rehberi) | [bones-studio/seed](https://huggingface.co/datasets/bones-studio/seed) |
| **ProtoMotions** | **Benzetilen insan ve insansıları eğitmek için GPU hızlandırmalı simülasyon çatısı** | **[nvidia-protomotions-turkce](https://github.com/Unitree-Turkiye/nvidia-protomotions-turkce)** | **[Rehber](https://www.robotlar.org/tr/rehber/nvidia-protomotions-insansi-simulasyon-rehberi)** | **[NVlabs/ProtoMotions](https://github.com/NVlabs/ProtoMotions)** |
| SOMA Retargeter | SOMA hareketini Unitree G1 gibi insansı robotlara aktarır | [unitree-g1-soma-retargeter-turkce](https://github.com/Unitree-Turkiye/unitree-g1-soma-retargeter-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/soma-retargeter-unitree-g1-hareket-aktarimi-rehberi) | [NVIDIA/soma-retargeter](https://github.com/NVIDIA/soma-retargeter) |
| GEM | Görüntü ve videodan tüm gövde ve el hareketini kestirir | [nvidia-gem-x-turkce](https://github.com/Unitree-Turkiye/nvidia-gem-x-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/nvidia-gem-x-videodan-hareket-rehberi) | [NVlabs/GEM-X](https://github.com/NVlabs/GEM-X) |
| GEAR SONIC | Fiziksel insansı robotlar için davranış temel modeli ve tüm gövde kontrolü | [unitree-g1-gear-sonic-turkce](https://github.com/Unitree-Turkiye/unitree-g1-gear-sonic-turkce) | [Rehber](https://www.robotlar.org/tr/rehber/gear-sonic-unitree-g1-tum-govde-kontrol-rehberi) | [NVlabs/GR00T-WholeBodyControl](https://github.com/NVlabs/GR00T-WholeBodyControl) |

İlgili Türkçe kaynaklar: [Unitree G1 + Isaac GR00T](https://github.com/Unitree-Turkiye/unitree-g1-isaac-groot-turkce) · [XR teleoperasyon](https://github.com/Unitree-Turkiye/unitree-teleoperasyon-turkce) · [LeRobot](https://github.com/Unitree-Turkiye/unitree-lerobot-turkce) · [RL Gym](https://github.com/Unitree-Turkiye/unitree-rl-gym-turkce) · [MuJoCo](https://github.com/Unitree-Turkiye/unitree-mujoco-turkce)

---

## 🚀 Unitree G1 ile ProtoMotions: Türkiye'de nereden başlanır

ProtoMotions'ta G1, `--robot-name g1` ile seçilen 29 DoF'lu bir modeldir: 33 gövde, gözlem çapası `torso_link`, fiziksel kök `pelvis`. Depoyla gelen `g1-bones-deploy` izleyicisi, BONES-SEED veri setinin yaklaşık 142 bin G1 hareketiyle 24 A100 GPU üzerinde eğitildi. Model kataloğunda alan rastgeleleştirmesinin aktarıma yönelik tam tarifiyle eğitilen politika bu izleyicidir. Katalog, uyumlu simülatörlere ve donanıma aktarılması beklenen G1 modelleri olarak bu izleyiciyi ve Metatron v1'i gösterir. Dağıtım hattı tek bir ONNX modeli dışa aktarır, MuJoCo'da doğrular ve RoboJuDo üzerinden gerçek G1'e götürür. Gerçek robotta acil durdurma Unitree kumandasındaki A düğmesindedir ve geçişler askıyla yürütülen dört aşamadan oluşur. Ayrıntılar [G1 dağıtım belgesinde](docs/G1-GERCEK-ROBOTA-DAGITIM.md).

Deneysel C++ çalışma zamanı, 29 gövde DoF'u için isteğe bağlı bir Unitree SDK2 arka ucu ve G1 başlatıcıları içerir. Başlatıcılar durmuş/sönümleme kipinde açılır. Robot güvenceye alınmışken eklem sırasını, birimleri, kazançları, sınırları ve fiziksel durdurma denetimini doğrulayın. Ayrıntılar [dağıtım çalışma zamanı belgesinde](docs/DAGITIM-CALISMA-ZAMANI.md).

**Platform seçimi**

| Ürün | ProtoMotions iş akışındaki yeri |
| --- | --- |
| [Unitree G1](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | Depodaki G1 modeli, izleyici ve Metatron politikalarının hedeflediği insansı robot, 132 cm, 35 kg, 23-43 DoF |
| [Unitree G1 EDU teklif](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) | Geliştirici ve araştırma sürümü için teklif formu |
| [Unitree G1 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-plus) | 2 DoF boyun, yüksek tork, dokunsal kafa ve 6 mikrofonlu G1 |
| [Unitree G1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-d) | İnsansı robot veri toplama ve eğitim platformu, Jetson Orin |
| [G1 Ar-Ge platformu](https://www.robotlar.org/tr/g1-arge-platformu) | Üniversite ve Ar-Ge laboratuvarları için G1 çözüm sayfası |

**Türkçe rehberler (iş akışına göre)**

- Bu deponun özeti ve kurulum yolu: [NVIDIA ProtoMotions insansı simülasyon rehberi](https://www.robotlar.org/tr/rehber/nvidia-protomotions-insansi-simulasyon-rehberi)
- Eğitilen politikaları gerçek robotta tüm gövde kontrolüne taşımak: [GEAR SONIC ile Unitree G1 tüm gövde kontrolü](https://www.robotlar.org/tr/rehber/gear-sonic-unitree-g1-tum-govde-kontrol-rehberi)
- İnsan hareketini G1'e aktarmak: [SOMA Retargeter ile Unitree G1 hareket aktarımı](https://www.robotlar.org/tr/rehber/soma-retargeter-unitree-g1-hareket-aktarimi-rehberi)
- Isaac Gym'de pekiştirmeli öğrenme ve MuJoCo'da doğrulama: [Unitree RL Gym](https://www.robotlar.org/tr/rl-gym)
- Isaac GR00T ve IsaacLab akışı: [Unitree G1 Isaac GR00T rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-isaac-gr00t-rehberi)
- Gerçek hareket verisi toplamak için XR teleoperasyon: [Unitree G1 teleoperasyon](https://www.robotlar.org/unitree/g1-teleoperasyon)
- Robotla haberleşme ve düşük seviye kontrol: [Unitree SDK rehberi](https://www.robotlar.org/unitree/sdk-rehberi)
- Robot üstü hesaplama: [Jetson Orin rehberi](https://www.robotlar.org/tr/rehber/saha-robotu-jetson-orin-dayanikli-bilgisayar)

**Türkçe GitHub depoları:** [G1](https://github.com/Unitree-Turkiye/unitree-g1-turkce) · [SDK2](https://github.com/Unitree-Turkiye/unitree-sdk2-turkce) · [RL Gym](https://github.com/Unitree-Turkiye/unitree-rl-gym-turkce) · [MuJoCo](https://github.com/Unitree-Turkiye/unitree-mujoco-turkce) · [GEAR SONIC](https://github.com/Unitree-Turkiye/unitree-g1-gear-sonic-turkce) · [SOMA Retargeter](https://github.com/Unitree-Turkiye/unitree-g1-soma-retargeter-turkce)

Üniversite ve kurumlar için platform, eğitim ve Türkçe geliştirici desteği konusunda [kurumsal teklif formundan](https://www.robotlar.org/tr/fiyat/unitree-kurumsal-teklif-al) bize ulaşabilirsiniz.

## 🤖 Unitree Ürün Ailesi (Türkiye)

Robotlar.org, Unitree Robotics'in Türkiye resmi distribütörüdür. Aşağıdaki ürünlerin Türkçe tanıtımı, teknik özellikleri ve kurumsal teklif formu sitede yer alır. Fiyatlar konfigürasyona göre değiştiği için [teklif üzerinden](https://www.robotlar.org/tr/fiyat/unitree-kurumsal-teklif-al) paylaşılır.

### İnsansı robotlar

| Ürün | Ne işe yarar |
|---|---|
| [Unitree G1](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | 132 cm, 35 kg, 23-43 DoF insansı robot |
| [Unitree G1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-d) | İnsansı robot veri toplama ve eğitim platformu, Jetson Orin |
| [Unitree G1 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-plus) | 2 DoF boyun, yüksek tork, dokunsal kafa ve 6 mikrofonlu G1 |
| [Unitree H2](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2) | 182 cm, 70 kg, 31 DoF tam boy insansı robot |
| [Unitree H2-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-d) | 7 DoF kollu, tekerlekli çift kollu insansı robot |
| [Unitree H2 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-plus) | NVIDIA Isaac GR00T referans insansı robotu, 75 DoF, Jetson Thor |
| [Unitree H2-X](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-x) | Hizmet ve koruma görevleri, 4 m platformdan iniş |
| [Unitree R1](https://www.robotlar.org/tr/insansi-robotlar/unitree-r1) | 123 cm, 29 kg, 26 DoF insansı robot (AIR, R1, EDU sürümleri) |
| [Unitree R1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-r1-d) | R1-A5 ya da R1-A7 (7-DOF) kollu çift kollu insansı platform |
| [Tüm insansı robotlar →](https://www.robotlar.org/tr/insansi-robotlar) | |

### Robot elleri

| Ürün | Ne işe yarar |
|---|---|
| [Unitree Dex3-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex3-1) | G1 için 7 serbestlik dereceli el, opsiyonel 33 dokunsal sensör |
| [Unitree Dex5-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex5-1) | 20 serbestlik dereceli (16 aktif) el, Dex5-1P sürümünde 94 dokunsal sensör |
| [Unitree Dex5-S](https://www.robotlar.org/tr/robot-elleri/unitree-dex5-s) | 22 aktif serbestlik, 620 g, çift T1s Ethernet, 1000 Hz |
| [Unitree Dex1-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex1-1) | İki çeneli tutucu: 90 mm strok, 5-120 N ayarlı kavrama, RS-485 |
| [Tüm robot elleri →](https://www.robotlar.org/tr/robot-elleri) | |

### Dört ayaklı robotlar ve bileşenler

[Go2](https://www.robotlar.org/tr/kopek-robotlar/unitree-go2) · [Go2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-go2-w) · [B2](https://www.robotlar.org/tr/kopek-robotlar/unitree-b2) · [B2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-b2-w) · [A2](https://www.robotlar.org/tr/kopek-robotlar/unitree-a2) · [AS2](https://www.robotlar.org/tr/kopek-robotlar/unitree-as2) · [Tüm dört ayaklı robotlar](https://www.robotlar.org/tr/kopek-robotlar) · [4D LiDAR L2](https://www.robotlar.org/tr/unitree-bilesenler/4d-lidar-l2) · [Tüm Unitree bileşenleri](https://www.robotlar.org/tr/unitree-bilesenler)

## 📖 Türkçe Rehberler

- [NVIDIA insansı hareket ailesi ve Unitree G1 (merkez rehber)](https://www.robotlar.org/tr/rehber/nvidia-insansi-hareket-ekosistemi-unitree-g1-rehberi)
- [Unitree G1'de hareket taklidi: BeyondMimic'ten SONIC'e hareket izleme yöntemleri](https://www.robotlar.org/tr/rehber/unitree-g1-hareket-taklidi-motion-tracking-rehberi) · [Türkçe araştırma deposu: unitree-g1-hareket-taklidi-arastirma](https://github.com/Unitree-Turkiye/unitree-g1-hareket-taklidi-arastirma)
- [NVIDIA ProtoMotions insansı simülasyon rehberi](https://www.robotlar.org/tr/rehber/nvidia-protomotions-insansi-simulasyon-rehberi)
- [GEAR SONIC ile Unitree G1 tüm gövde kontrolü](https://www.robotlar.org/tr/rehber/gear-sonic-unitree-g1-tum-govde-kontrol-rehberi)
- [SOMA Retargeter ile Unitree G1 hareket aktarımı](https://www.robotlar.org/tr/rehber/soma-retargeter-unitree-g1-hareket-aktarimi-rehberi)
- [Unitree G1 MotionBricks rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-motionbricks-rehberi)
- [Unitree G1 Isaac GR00T rehberi](https://www.robotlar.org/tr/rehber/unitree-g1-isaac-gr00t-rehberi)
- [Unitree LeRobot: veri toplama ve taklit öğrenme](https://www.robotlar.org/tr/rehber/unitree-lerobot-rehberi)
- [Unitree G1 XR teleoperasyon](https://www.robotlar.org/unitree/g1-teleoperasyon)
- [Unitree RL Gym](https://www.robotlar.org/tr/rl-gym)
- [Unitree SDK rehberi](https://www.robotlar.org/unitree/sdk-rehberi)
- [Unitree eğitim çözümleri (üniversite ve laboratuvar)](https://www.robotlar.org/tr/unitree-egitim-cozumleri)
- [Tüm açık kaynak Unitree projeleri (Türkçe)](https://www.robotlar.org/tr/acik-kaynak)

## 🔗 Unitree Türkiye GitHub Ağı

**Modeller:** [Go2](https://github.com/Unitree-Turkiye/unitree-go2-turkce) · [Go2-W](https://github.com/Unitree-Turkiye/unitree-go2-w-turkce) · [B2](https://github.com/Unitree-Turkiye/unitree-b2-turkce) · [A2](https://github.com/Unitree-Turkiye/unitree-a2-turkce) · [G1](https://github.com/Unitree-Turkiye/unitree-g1-turkce) · [H2](https://github.com/Unitree-Turkiye/unitree-h2)
**Geliştirici:** [SDK2](https://github.com/Unitree-Turkiye/unitree-sdk2-turkce) · [XR Teleoperasyon](https://github.com/Unitree-Turkiye/unitree-teleoperasyon-turkce) · [LeRobot](https://github.com/Unitree-Turkiye/unitree-lerobot-turkce) · [RL Gym](https://github.com/Unitree-Turkiye/unitree-rl-gym-turkce) · [Becerikli El](https://github.com/Unitree-Turkiye/unitree-hunerli-el-turkce) · [MuJoCo](https://github.com/Unitree-Turkiye/unitree-mujoco-turkce) · [ROS2](https://github.com/Unitree-Turkiye/unitree-ros2-turkce) · [VLA](https://github.com/Unitree-Turkiye/unitree-vla-turkce)
**G1 araştırma:** [G1 MotionBricks](https://github.com/Unitree-Turkiye/unitree-g1-motionbricks-turkce) · [G1 Isaac GR00T](https://github.com/Unitree-Turkiye/unitree-g1-isaac-groot-turkce) · [G1 GEAR SONIC](https://github.com/Unitree-Turkiye/unitree-g1-gear-sonic-turkce) · [G1 SOMA Retargeter](https://github.com/Unitree-Turkiye/unitree-g1-soma-retargeter-turkce) · [G1 dokunsal algılama giysisi](https://github.com/Unitree-Turkiye/unitree-g1-dokunsal-giysi-turkce) · [🏠 Unitree Türkiye](https://github.com/Unitree-Turkiye)

## 🙏 Teşekkür

Orijinal proje şu çalışmalarla birlikte yürür (orijinal depodan):

- Unitree G1 üzerinde dağıtım çatısı olarak [RoboJuDo](https://github.com/HansZ8/RoboJuDo)
- Yeniden hedefleme için [PyRoki](https://github.com/chungmin99/pyroki) (önceki sürümlerde [Mink](https://github.com/kevinzakka/mink))
- Hareket taklidi için kardeş depo [MimicKit](https://github.com/xbpeng/MimicKit/tree/main)
- Metinden hareket üretimi için [Kimodo](https://research.nvidia.com/labs/sil/projects/kimodo/), görseller için [NVIDIA Human Motion Modeling Research](https://research.nvidia.com/labs/sil/human_motion_modeling/)
- Hareket verisi için [AMASS](https://amass.is.tue.mpg.de/) ve [BONES-SEED](https://huggingface.co/datasets/bones-studio/seed)
- Unitree robot tanımları ve ağları (mesh), BeyondMimic, Isaac Lab ve SMPL/SMPL-H için atıf bildirimleri [legal/](https://github.com/NVlabs/ProtoMotions/tree/main/legal/) klasöründe

## ✉️ İletişim

Orijinal projeyle ilgili sorular, sorunlar ve katkılar için [orijinal depoyu](https://github.com/NVlabs/ProtoMotions/issues) ve [katkı kılavuzunu](https://github.com/NVlabs/ProtoMotions/blob/main/CONTRIBUTING.md) kullanın. Bu Türkçe çeviriyle ilgili düzeltme önerilerinizi bu depoda issue olarak iletebilirsiniz.

---

> ℹ️ **Unitree Türkiye:** Unitree Robotics'in Türkiye resmi satış ve teknik distribütörü. Unitree G1 ailesi için teklif, kurulum, eğitim ve Türkçe geliştirici desteği: **[robotlar.org](https://www.robotlar.org/tr/rehber/unitree-turkiye-distributoru)** · 📧 info@robotlar.org

© 2026 **[www.robotlar.org](https://www.robotlar.org)**
