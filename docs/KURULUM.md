> Bu belge NVlabs/ProtoMotions deposundaki `docs/source/getting_started/installation.rst` dosyasının Türkçe çevirisidir (Apache-2.0, değişiklik: Türkçeleştirme).

# Kurulum

ProtoMotions dört simülasyon arka ucunu destekler: IsaacGym, IsaacLab, Newton ve MuJoCo. İstediğiniz simülatörü kurabilirsiniz. Arka uç, `train_agent.py` ya da `inference_agent.py` çalıştırılırken `--simulator` bayrağıyla seçilir.

**Test edilen sürümler:**

<p>
  <a href="https://pypi.org/project/newton/1.0.0/"><img src="https://img.shields.io/badge/Newton-1.0.0-brightgreen.svg" alt="Newton"></a>
  <a href="https://github.com/isaac-sim/IsaacLab/commit/4ecd0b036da19ff6ad2bb4d621f886b63e9f6db8"><img src="https://img.shields.io/badge/IsaacLab-3.0-blue.svg" alt="IsaacLab"></a>
  <a href="https://developer.nvidia.com/isaac-gym"><img src="https://img.shields.io/badge/IsaacGym-Preview_4-blue.svg" alt="IsaacGym"></a>
  <a href="https://github.com/google-deepmind/mujoco"><img src="https://img.shields.io/badge/MuJoCo-3.0+-orange.svg" alt="MuJoCo"></a>
</p>

> **Not:** Bağımlılık çakışmalarından kaçınmak için her simülatöre **ayrı bir sanal ortam** kurmanızı öneririz. IsaacGym ve MuJoCo için **conda** ya da **venv**, IsaacLab ve Newton için **uv** kullanmanızı öneririz.

## Hangi kurulum yolu?

| Simülatör | Desteklenen kurulum | Notlar |
| --- | --- | --- |
| MuJoCo, Newton | Kaynak klonu **ya da** uv bağımlılığı | MuJoCo yalnız CPU üzerinde ve tek ortamla çalışır. |
| IsaacLab | **Sabitlenmiş kaynak klonu** | Isaac Sim 6.0 ile IsaacLab 12.0.0, Python 3.12 ve Linux x86_64 gerektirir. |
| IsaacGym | **Yalnız kaynak klonu** | IsaacGym PyPI üzerinden dağıtılmaz: NVIDIA'dan indirip elle kurarsınız ve **Python 3.8** gerektirir. |

Önceden eğitilmiş kontrol noktalarını (checkpoint), hareket dosyalarını ya da `examples/` deneylerini istiyorsanız kaynak klonu kullanın. Bunlar paketin içinde değil, Git LFS'tedir.

## Ön koşullar

Depoyu klonladıktan sonra Git LFS'te tutulan dosyaları indirip çıkarın:

```bash
git lfs install
git lfs pull
```

Önceden eğitilmiş kontrol noktaları, hareket dosyaları, ağlar (mesh) ve USD varlıkları büyük olduğu için bu adım biraz sürebilir. Varlıkların bir kısmını elle indiriyorsanız dosyaların gerçekten çıkarıldığından, hala Git LFS işaretçisi olarak durmadığından emin olun. İşaretçi dosyalar `version https://git-lfs.github.com/spec/v1` satırıyla başlar ve IsaacLab robot varlıklarını yüklerken `is not a valid usda layer` gibi hatalara yol açabilir.

## ProtoMotions'ı bağımlılık olarak kullanmak (uv)

ProtoMotions'ı doğrudan Git'ten kurun. Robot ağları ve USD varlıkları Git LFS nesneleridir, bu yüzden kaynak LFS açık olarak indirilmelidir. `lfs = true` için uv 0.11.32 ya da üstü gerekir:

```bash
uv init --python 3.11 my-project
cd my-project
uv add --lfs "protomotions[newton] @ git+https://github.com/NVlabs/ProtoMotions.git"
```

Aynı bağımlılığı alt projedeki `pyproject.toml` içinde de tanımlayabilirsiniz:

```toml
[project]
dependencies = ["protomotions[newton]"] # or [mujoco] / [isaaclab]

[tool.uv]
required-version = ">=0.11.32"

[tool.uv.sources]
protomotions = { git = "https://github.com/NVlabs/ProtoMotions.git", lfs = true }
```

Ardından eğitimi kurulan giriş noktası üzerinden çalıştırın:

```bash
uv run protomotions train-agent \
    --robot-name g1 --simulator newton \
    --experiment-path experiments/my_experiment.py \
    --experiment-name my_run \
    --motion-file data/my_motion.pt \
    --num-envs 4096 --batch-size 16384
```

`uv run protomotions info`, çözümlenen varlık kökünü ve hangi simülatör modüllerinin içe aktarılabildiğini yazdırır.

Paket, Python modüllerini ve robot varlık ağacının tamamını taşır. SMPL/SMPL-H varlıkları kendi lisans koşullarına bağlı olduğu için **bu ağacın dışında** kalır. Önceden eğitilmiş kontrol noktaları, hareket dosyaları ve `examples/` deneyleri pakete girmez. Bunlara ihtiyacınız varsa bir Git LFS klonu tutun ve `PROTOMOTIONS_ASSET_ROOT` değişkenini ayarlayın.

### Bağımlılık olarak IsaacLab

Desteklenen IsaacLab yığını, tek başına bir ProtoMotions bağımlılık çözümlemesi yerine sabitlenmiş bir kaynak çalışma alanıdır. Aşağıdaki IsaacLab adımlarıyla Python 3.12 `.venv` ortamını kurun, sonra ProtoMotions'ı o ortama yükleyin. Ayrı bir projede yalın bir `uv add protomotions[isaaclab]` komutu gereken IsaacLab kaynak sürümünü kurmaz.

## Simülatörünüzü seçin

### IsaacGym

IsaacGym **Python 3.8** gerektirir.

1. Bir conda ortamı oluşturun:

   ```bash
   conda create -n isaacgym python=3.8
   conda activate isaacgym
   ```

2. IsaacGym Preview 4'ü indirin:

   ```bash
   wget https://developer.nvidia.com/isaac-gym-preview-4
   tar -xvzf isaac-gym-preview-4
   ```

3. IsaacGym Python API'sini kurun:

   ```bash
   pip install -e isaacgym/python
   ```

4. ProtoMotions'ı ve bağımlılıklarını kurun:

   ```bash
   pip install -e /path/to/protomotions
   pip install -r /path/to/protomotions/requirements_isaacgym.txt
   ```

### IsaacLab

ProtoMotions, herkese açık IsaacLab commit'i `4ecd0b036da19ff6ad2bb4d621f886b63e9f6db8` üzerindeki IsaacLab 12.0.0 ve Isaac Sim 6.0 sürümlerini hedefler. Bu yığın **Python 3.12** gerektirir. Çalışma alanı paketleri ve simülatör bağımlılıkları hazır olsun diye sabitlenmiş IsaacLab kaynak klonunu ProtoMotions'tan önce kurun.

1. IsaacLab'i klonlayın ve desteklenen sürümü seçin:

   ```bash
   git clone https://github.com/isaac-sim/IsaacLab.git
   cd IsaacLab
   git checkout 4ecd0b036da19ff6ad2bb4d621f886b63e9f6db8
   ```

2. Sabitlenmiş IsaacLab ortamını oluşturun ve Isaac Sim ekini kurun:

   ```bash
   uv venv --python 3.12
   source .venv/bin/activate
   uv pip install -e ".[isaacsim]"
   ```

   Burada `uv sync` yerine `uv pip install` kullanın. `uv sync` her platform ve ek için kilidi tek seferde çözer. Bu IsaacLab commit'inde x86-64 ve arm64 işaretçilerinin `pytetwild` sabitlemeleri birlikte çözülemediği için kilit Linux x86-64 makinesinde bile başarısız olur. `uv pip install` yalnız geçerli yorumlayıcı için çözümleme yapar.

3. ProtoMotions'ı ve bağımlılıklarını kurun:

   ```bash
   uv pip install -e "/path/to/protomotions[isaaclab]" \
     --extra-index-url https://pypi.nvidia.com
   uv pip install -r /path/to/protomotions/requirements_isaaclab.txt
   ```

> **Not:** IsaacLab/IsaacSim ilk kullanımda NVIDIA EULA onayı isteyebilir. Gözetimsiz, ekransız (headless) işleri başlatmadan önce onayı etkileşimli olarak verin.

### Newton

Newton, NVIDIA Warp üzerine kurulu, GPU hızlandırmalı bir fizik simülatörüdür ve artık PyPI'da yer alır. Kurulumun tüm ayrıntıları için [Newton Kurulum Kılavuzu](https://newton-physics.github.io/newton/1.0.0/guide/installation.html) sayfasına bakın.

**Gereksinimler:** Python 3.10+ (3.11+ önerilir), NVIDIA GPU (hesaplama yeteneği >= 5.0), sürücü 545+

1. Bir sanal ortam oluşturun:

   ```bash
   python -m venv .venv_newton
   source .venv_newton/bin/activate
   ```

2. PyTorch ve Newton'u kurun:

   ```bash
   pip install torch --index-url https://download.pytorch.org/whl/cu124
   pip install "newton[examples]==1.0.0"
   ```

   Yalnız ekransız kipe (görüntüleyicisiz) ihtiyacınız varsa `newton[examples]==1.0.0` yerine `newton[sim]==1.0.0` kullanın.

3. ProtoMotions'ı ve bağımlılıklarını kurun:

   ```bash
   pip install -e /path/to/protomotions
   pip install -r /path/to/protomotions/requirements_newton.txt
   ```

> **Not:** Python 3.10'da `imgui-bundle` (`newton[examples]` bağımlılığı) için hazır derlenmiş wheel bulunmaz ve paket kaynaktan derlenir. Bu 10-20 dakika sürebilir. Python 3.11+ için hazır wheel'ler vardır ve kurulum anında tamamlanır.

### MuJoCo (yalnız CPU)

MuJoCo, GPU olmadan hızlı test ve hata ayıklama için yalnız CPU üzerinde çalışan bir arka uçtur. Tek ortamı (`num_envs=1`) destekler.

**Gereksinimler:** Python 3.10+, GPU gerekmez

1. Bir conda ortamı oluşturun:

   ```bash
   conda create -n protomotions_mujoco python=3.10
   conda activate protomotions_mujoco
   ```

2. PyTorch'un CPU sürümünü kurun (daha hafiftir, CUDA gerektirmez):

   ```bash
   pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
   ```

3. ProtoMotions'ı ve bağımlılıklarını kurun:

   ```bash
   pip install -e /path/to/protomotions
   pip install -r /path/to/protomotions/requirements_mujoco.txt
   ```

4. Çıkarımı MuJoCo ile çalıştırın:

   ```bash
   python protomotions/inference_agent.py \
     --checkpoint data/pretrained_models/motion_tracker/g1-bones-deploy/inference_last.ckpt \
     --motion-file data/motion_for_trackers/g1_bones_seed_mini.pt \
     --simulator mujoco \
     --num-envs 1
   ```

   Bu örnek, depoyla gelen G1 hareket izleyicisini (motion tracker) ve ona uygun hareket verisini kullanır. Kontrol noktası dizini hem `resolved_configs.json` yapılandırma paketini hem de `experiment_config.py` tarif kopyasını taşır. Çıkarım, yükleme sırasında görünümünü paketten türetir ve tarifin çıkarım kancasını kopyadan çalıştırır.

> **Not:** MuJoCo arka ucu hızlı politika doğrulaması ve hata ayıklama içindir. Eğitim ya da büyük ölçekli değerlendirme için GPU hızlandırmalı arka uçları (IsaacGym, IsaacLab, Newton) kullanın.

## Sorun giderme

### IsaacLab sorunları

**Torch Inductor uyarısı**

Küçük GPU'larda IsaacLab değerlendirmesi şuna benzer bir uyarı yazdırabilir:

```text
Not enough SMs to use max_autotune_gemm mode
```

Bu, ölümcül olmayan bir PyTorch performans uyarısıdır. Ardından gerçek bir traceback gelmedikçe değerlendirme sürer.

### IsaacGym sorunları

**libpython hatası**

`libpython` ile ilgili hatalar görürseniz `LD_LIBRARY_PATH` değişkenini conda ortamınıza ayarlayın:

```bash
# First, check your conda environment path
conda info -e

# Then set LD_LIBRARY_PATH (replace with your actual conda env path)
export LD_LIBRARY_PATH=/path/to/conda/envs/your_env/lib:$LD_LIBRARY_PATH

# For example:
export LD_LIBRARY_PATH=${CONDA_PREFIX}/lib:$LD_LIBRARY_PATH
```

Bu ayarı yalnız bu conda ortamı için kalıcı yapmak üzere etkinleştirme kancaları ekleyin:

```bash
mkdir -p "${CONDA_PREFIX}/etc/conda/activate.d" "${CONDA_PREFIX}/etc/conda/deactivate.d"
cat > "${CONDA_PREFIX}/etc/conda/activate.d/isaacgym-libpython.sh" <<'EOF'
export _OLD_LD_LIBRARY_PATH="${LD_LIBRARY_PATH:-}"
export LD_LIBRARY_PATH="${CONDA_PREFIX}/lib:${LD_LIBRARY_PATH:-}"
EOF
cat > "${CONDA_PREFIX}/etc/conda/deactivate.d/isaacgym-libpython.sh" <<'EOF'
export LD_LIBRARY_PATH="${_OLD_LD_LIBRARY_PATH:-}"
unset _OLD_LD_LIBRARY_PATH
EOF
```

**Bellek sorunları**

Eğitim sırasında bellek sorunu yaşarsanız:

```bash
# Reduce number of environments in your training command
--num-envs 1024
```

## Sonraki adımlar

Kurulumdan sonra ilk ajanınızı eğitmek ya da önceden eğitilmiş modelleri çalıştırmak için [Hızlı Başlangıç](HIZLI-BASLANGIC.md) kılavuzuna geçin.

---

**Unitree G1 Türkiye:** [Unitree G1 insansı robot ürün sayfası](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) · [G1 EDU için teklif alın](https://www.robotlar.org/tr/fiyat/unitree-g1-edu-fiyat) · Unitree Robotics Türkiye Resmi Distribütörü Robotlar.org
