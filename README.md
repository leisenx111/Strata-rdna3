<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="apps/host/public/brand/strata-lockup-dark.svg">
    <img src="apps/host/public/brand/strata-lockup-light.svg" alt="Strata" width="340">
  </picture>
</p>

<h3 align="center">Run powerful AI on your own computer.</h3>
<p align="center">Chat, code and work with images, locally.</p>

<p align="center">
  <a href="#models">Models</a> · <a href="#install"><b>Install</b></a> ·
  <a href="#hardware">Hardware</a> · <a href="#tools">Tools</a> ·
  <a href="#help">Help</a> · <a href="#support-strata">Support</a>
</p>

<p align="center">
  <a href="https://github.com/Niko1221/Strata/releases/latest"><img src="https://img.shields.io/github/v/release/Niko1221/Strata?label=download&color=0ab887" alt="Download Strata"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/Niko1221/Strata" alt="MIT license"></a>
  <a href="https://github.com/sponsors/Niko1221"><img src="https://img.shields.io/badge/Sponsor-EA4AAA?logo=githubsponsors&logoColor=fff" alt="Sponsor Strata on GitHub Sponsors"></a>
  <a href="https://buymeacoffee.com/strataengine"><img src="https://img.shields.io/badge/Buy_me_a_coffee-FFDD00?logo=buymeacoffee&logoColor=000" alt="Support Strata on Buy Me a Coffee"></a>
</p>

<p align="center"><sub><b>English</b> · <a href="README.zh-CN.md">简体中文</a> · <a href="README.ja.md">日本語</a> · <a href="README.de.md">Deutsch</a> · <a href="README.fr.md">Français</a> · <a href="README.es.md">Español</a> · <a href="README.pt-BR.md">Português</a> · <a href="README.pl.md">Polski</a></sub></p>

<p align="center"><a href="https://github.com/Niko1221/Strata/releases/download/v0.1.10/Pagoda.mp4"><img src="docs/media/pagoda-preview.webp" width="720" alt="A voxel pagoda garden that Strata's model wrote, running in the browser"></a><br>
<sub>A voxel pagoda garden, 1 shot prompt running on an RTX 5070 with Strata (IQ3_S, 128K context) ·
<a href="https://github.com/Niko1221/Strata/releases/download/v0.1.10/Pagoda.mp4">full video (49 s)</a></sub></p>

> [!NOTE]
> **This is a fork of [Niko1221/Strata](https://github.com/Niko1221/Strata) tuned for AMD RDNA3 (RX 7900 XTX, gfx1100).**
> The upstream README follows below unchanged. This section describes what the fork adds and how fast it runs.

## RDNA3 / RX 7900 XTX fork

Measured on one or two **RX 7900 XTX** (24 GB each, PCIe 4.0 x8) with a Ryzen 9 9950X3D2 and 128 GB of RAM, release
`claude-20261102-p16k` on two GPUs (16K prompts measured on it; the other lengths take the same path as on
`claude-20261102-rg24`, where they were measured) and `claude-20261101-rg23` on one (production from 2026-11-02), with
every expert computed (no expert skipping). Lines are the median of 3 trials (9 for UD-Q4_K_XL on 2 GPUs except 16K; the faint band
is the trial range); every request was checked for exact input length, 256 output tokens and correct retrieval, with no
prompt cache.

**What the prompts are.** Each prompt is N tokens of synthetic warehouse records with 3 hidden codes. The model returns
the codes, then writes either an inventory-auditing guide (**English prose**) or a Python inventory-auditing module
(**Python code**). Decode speed depends on the text: the model's MTP head drafts ahead, and code drafts are accepted more
often (about 79 % vs 61 %), so code decodes 15-37 % faster on the same hardware. Compare tok/s only on the same kind
of text.

<p align="center"><img src="docs/media/rdna3/prefill.png" width="49%" alt="Flash-Next prefill speed, 1 and 2 RX 7900 XTX"> <img src="docs/media/rdna3/vs-rtx4090.png" width="49%" alt="1 and 2 RX 7900 XTX against a public RTX 4090 run on the same requests"></p>
<p align="center"><img src="docs/media/rdna3/decode-acceptance-english.png" width="49%" alt="English prose: decode speed and MTP acceptance"> <img src="docs/media/rdna3/decode-acceptance-code.png" width="49%" alt="Python code: decode speed and MTP acceptance"></p>

| Flash-Next, tokens/s | prefill 32K / 128K / ≈256K | decode, English prose 32K / 128K / ≈256K | decode, Python code 32K / 128K / ≈256K |
|---|---|---|---|
| IQ3_S, 2 GPUs | 6,760 / 6,778 / 6,405 | 129.8 / 141.1 / 127.4 | 169.9 / 162.7 / 161.8 |
| IQ3_S, 1 GPU | 3,541 / 3,553 / 3,311 | 114.1 / 112.9 / 111.8 | 139.3 / 142.7 / 138.1 |
| UD-IQ4_XS, 2 GPUs | 7,060 / 7,082 / 6,586 | 127.8 / 126.8 / 129.9 | 167.1 / 159.1 / 158.4 |
| UD-IQ4_XS, 1 GPU | 3,455 / 3,480 / 3,251 | 94.2 / 91.9 / 89.0 | 122.3 / 120.6 / 117.3 |
| UD-Q4_K_XL, 2 GPUs | 5,770 / 7,089 / 6,604 | 116.6 / 113.5 / 113.5 | 142.5 / 141.3 / 141.2 |
| UD-Q4_K_XL, 1 GPU | 3,613 / 3,676 / 3,442 | 78.3 / 80.9 / 72.1 | 95.6 / 98.3 / 95.6 |

Fastest measured decode: IQ3_S on 2 GPUs writing code, 189 tok/s (4K context). On 2 GPUs all three quants decode
tensor parallel (UD-Q4_K_XL since 2026-11-02, before that a pipelined layer split); prompts take one of three paths by
length (below). UD-IQ4_XS and IQ3_S 2-GPU code cells other than 16K are rg23 sweeps (rg24 / p16k change only prompt reads,
from 3K tokens). At 16K on 2 GPUs prefill is IQ3_S 4,904, UD-IQ4_XS 5,034 and UD-Q4_K_XL 5,000 tok/s (English; 1.56x /
1.71x / 1.77x one GPU, was 1.52x / 1.45x / 1.29x before p16k).

### How the two cards are used

| | IQ3_S, UD-IQ4_XS (2 GPUs) | UD-Q4_K_XL (2 GPUs) |
|---|---|---|
| Decode | tensor parallel (`--tensor-parallel 2`) | tensor parallel (since 2026-11-02; before: layer split + `--pipeline-windows 2`) |
| Short prompts | one chunk through the layer split, the idle card computing part of the other's experts | same |
| Mid-length prompts | tensor-parallel prefill in one chunk, 3,072 (IQ3_S) / 5,120 (UD-IQ4_XS) to 18,432 tokens, with the refill head | one chunk below 10,240 tokens, then tensor-parallel prefill in one chunk to 28,672 tokens with the refill head |
| Long prompts | data parallel | data parallel |

**Tensor-parallel decode.** Every layer runs on both cards: attention is split by head, the routed experts are split by
expert id across the two cards' expert caches, and the cards exchange their partial results once a layer over PCIe 4.0
x8. The layer split instead puts layers 0-23 on card 0 and 24-47 on card 1; `--pipeline-windows 2` starts the next
decode window on card 0 before the current one is verified, so both cards stay busy. Tensor parallel turns that pipeline
off. That pipeline had a cost: the early window has to be drafted about four tokens deep, and on UD-Q4_K_XL code its
acceptance fell to ~71 % against 79 % on one GPU. With the VRAM savings below, UD-Q4_K_XL moved to tensor parallel on
2026-11-02 (`--vram-reserve-mib 576 --vram-reserve-later-mib 640`, `STRATA_TP_DEAL_BUDGET=5200`). Against its layer split
on the same binary, over three rounds of 1K-256K sweeps:

- decode is +0.9..+9.1 % on code and +4.0..+12.3 % on English at every length;
- prefill is +0.6..+13.9 %;
- code acceptance returns to 75-82 %.

The refill of the borrowed expert slots runs beside the first decode windows, so the code windows of a 256-token answer
are 3-11 % longer at 8K-128K.

**What tensor parallel costs in VRAM, and what was won back.** At first each card held three extra things beside the
layer-split weights that prompt reading still uses: a sliced copy of the dense weights, a tensor-parallel session, and a
separate session for the data-parallel prompt path. Every MiB there is a MiB less of expert cache, and an expert that is
not cached has to come over PCIe. Three switches give most of it back (measured on UD-IQ4_XS, per card):

- `STRATA_TP_ALIAS` (on by default): the q/k/v slices are read in place from the full matrices: 207 MiB.
- `STRATA_TP_SHARE_DP=1`: the data-parallel prompt session lives in the tensor-parallel session's memory: 649 MiB.
- `STRATA_TP_ALIAS_MAP=rows`: the Q8_0 GDN projections are read from the full matrices through a row map: sliced dense
  weights 1,159 -> 394 MiB.

Together about 1.6 GiB a card; the expert cache grows from 7,178 + 6,566 to 7,807 + 7,186 slots. None of them changes
the arithmetic: the aliases read the same bytes, and every mapped slice is checked bit for bit against a dense copy at
start (`STRATA_TP_ALIAS_MAP_CHECK`, on by default).

**Prompt paths.** Data-parallel prefill (`STRATA_DP_PREFILL=1`) runs all 48 layers on both cards on alternating chunks;
each card streams the experts its own cache lacks, so most streamed experts cross PCIe twice. Tensor-parallel prefill
(`STRATA_TP_PREFILL=1`, from release `claude-20261102-rg24`) has both cards read every layer of the *same* chunk: GDN
split by head, attention by query (keys and values exchanged), experts by id. With `STRATA_TP_PREFILL_CHUNK=8448`,
`STRATA_TP_PREFILL_MAX=12288` and `STRATA_TP_PREFILL_MIN` 3072 (IQ3_S) / 5120 (UD-IQ4_XS), prefill against the previous
release: IQ3_S +18.5 % at 4K and +41.6 % at 8K, UD-IQ4_XS +31.8 % at 8K and about +15 % at 12K, decode unchanged.

rg24 stopped it at 12K because a prompt chunk borrows expert slots from the decode cache. Data parallel gives card 0's
slots back while card 1 is still reading; tensor parallel keeps both cards busy to the last layer, so both caches refill
under the first decode windows: read as one 16K chunk, the decode after it hit its cache about 96 % of the time instead
of 98 % and its windows were 2.7-5.4 % slower. **The refill head (release `claude-20261102-p16k`):** after a
tensor-parallel read of at least 8,449 tokens each card waits, before the first token, for every borrowed slot but its
coldest 900 (IQ3_S) / 800 (UD-IQ4_XS) / 1,400 (UD-Q4_K_XL) (`STRATA_REFILL_HEAD_TP_KEEP`, `STRATA_REFILL_HEAD_TP_MIN`),
about 0.2-0.35 s, so the decode starts with no more experts missing than after a data-parallel read. The refill goes
hottest first (an eighth of the borrowed slots holds ~60 % of the prompt's expert rows), and one big chunk saves more
than the wait costs. Tensor-parallel prefill now reads one chunk up to 18,432 tokens on IQ3_S / UD-IQ4_XS and from 10,240
to 28,672 on UD-Q4_K_XL. Against rg24 in interleaved pairs: IQ3_S +7.5 % at 12K and +2.9 % at 16K, UD-IQ4_XS +26 % / +17.5 %,
UD-Q4_K_XL +19 / +37 / +31 / +14 % at 12K / 16K / 20K / 24K, decode windows equal or faster, 8K unchanged.
UD-Q4_K_XL was excluded before because on its layer split each card held only its own layers' experts and the read was
link-bound; on the tensor-parallel layout each card holds part of every layer, the streamed experts split evenly over
the two links, and the 16K read is 43 % faster even without the head.

**Against an RTX 4090** (IQ3_S, the exact requests of the public
[nvidia-4090-flash-next-inference](https://github.com/AnnoyingTechnology/nvidia-4090-flash-next-inference) run of
2026-10-04 on a 24 GB RTX 4090, replayed here): decode at 32K / 128K is 119 / 108 tok/s on the 4090, 117 / 117 on one
7900 XTX and 142 / 140 on two; fresh 32K prefill 4,672 vs 3,541 and 6,760 tok/s. Against stock Strata on the same card
(UD-IQ4_XS, 32K): prefill 1,151 -> 3,455 tok/s on one GPU and 1,727 -> 7,060 on two.

**What the fork adds** (all gfx1100 HIP; most are opt-in environment switches, so stock behaviour is unchanged):
- Decode: tensor-parallel decode on two cards for IQ3_S / UD-IQ4_XS / UD-Q4_K_XL, with its VRAM copies aliased
  (`STRATA_TP_ALIAS`, `STRATA_TP_SHARE_DP`, `STRATA_TP_ALIAS_MAP=rows`); RDNA hyper-connection reads (int8 /
  int8-activation, fused down+up with a grid barrier); multi-column Q4_K/Q5_K/Q6_K/IQ4_XS GEMVs; fused router publish,
  fused waits and CPU-row combine; a register top-k for QSA at 262K capacity, split across many CTAs past 96K
  (`STRATA_TOPK_SPLIT_MIN=98304`, 256K decode 5-9 % faster per window); adaptive expert cache swaps every window;
  `--pcie-frac 0` for layer split; long-context verify graphs captured at start on the pipelined layer split.
- Prefill: matrix-core (WMMA) prompt attention and QSA selection (imported from
  [xyzzing/Strata](https://github.com/xyzzing/Strata)); the chunked GDN recurrence on FP16 matrix cores; FP16 pair
  expert kernels (empty-column skip, 3 blocks a WGP, FP16 IQ codebooks); prompt hyper-connection kernels (upmix at 512
  threads, vectorised write+norm); K-quant MMQ; a dual-GPU data-parallel prompt path (`STRATA_DP_PREFILL=1`, chunk
  8448 on IQ3_S); tensor-parallel prefill on IQ3_S / UD-IQ4_XS for 3K-18K and UD-Q4_K_XL for 10K-28K prompts (`STRATA_TP_PREFILL=1`) with a tensor-parallel-only refill head (`STRATA_REFILL_HEAD_TP_KEEP`); a routed CPU+GPU expert walk for short prompts on the Q4 layer split; LDS-staged attention.
- Loading: on Linux the expert arena is read with O_DIRECT when the page cache could not keep it (a 72 GB Q4 arena
  loads at ~1.9 instead of ~1.0 GiB/s).
- CPU: IQ AVX-512 init stays safe on AVX2-only hosts (PR #1).
- Fixes: MTP top-2 accounting, a one-token-window commit, a host deadlock in fused prefill batches, a graph-capture
  failure on the first decode past 98K on the Q4 layer split.

**Reproduce it:** [`rdna3/README.md`](rdna3/README.md) goes from a clean Linux install to these six setups: pinned
toolchain (`./setup.sh --build` installs ROCm, rocWMMA and llama.cpp at fixed versions and compiles the engine with
the fork's options), the six measured setups as ready configs (`rdna3/run.py`), and the benchmark that produced the
numbers (`rdna3/bench.py`, compared against `rdna3/reference.json`).

**Manual build** (what `./setup.sh --build` runs on gfx1100; ROCm from its `.venv`, rocWMMA headers at `1ab208f4`):

```bash
R=<rocm sdk root>  # e.g. .venv/lib/python3.13/site-packages/_rocm_sdk_devel
export HIP_PLATFORM=amd HIP_COMPILER=clang HIP_RUNTIME=rocclr ROCM_PATH=$R HIP_PATH=$R PATH=$R/bin:$R/llvm/bin:$PATH
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DSTRATA_ENABLE_HIP=ON -DSTRATA_PREFILL_MMQ=ON \
  -DSTRATA_MMQ_KQUANTS=ON \
  -DCMAKE_HIP_ARCHITECTURES=gfx1100 -DCMAKE_HIP_COMPILER=$R/llvm/bin/clang++ -DCMAKE_HIP_COMPILER_ROCM_ROOT=$R \
  "-DCMAKE_HIP_FLAGS=--rocm-path=$R --rocm-device-lib-path=$R/lib/llvm/amdgcn/bitcode" \
  -DSTRATA_ROCWMMA_INCLUDE_DIR=<rocWMMA>/library/include -DCMAKE_PREFIX_PATH=$R
cmake --build build -j
```

`STRATA_MMQ_KQUANTS=ON` matters for the Q4 (k-quant) pack: without it, two-GPU Q4 prefill is 7-13 % slower
(32K: ~4,170 instead of ~4,770 tok/s).  IQ3/IQ4 are unaffected.  `CMAKE_PREFIX_PATH` lets CMake find `libhsa-runtime64`: without it the binary
builds with `STRATA_NO_HSA` and `STRATA_REFILL_SDMA=1` silently stays on `hipMemcpyAsync` (the configure step now warns).

**Settings used for the numbers above:** the exact environment and command line of each of the six setups are in
[`docs/RDNA3_SETTINGS.md`](docs/RDNA3_SETTINGS.md).  The tuned paths are environment switches (`STRATA_GR_RDNA=1`,
`STRATA_HIP_WMMA=1`, `STRATA_DP_PREFILL=1`, ...); without them the engine behaves as upstream.

Fork notes: [`AIBOX_RDNA3.txt`](AIBOX_RDNA3.txt), [`docs/AIBOX_FORK.txt`](docs/AIBOX_FORK.txt). License: MIT, as upstream.

---

Strata runs **[Qwen3.8-Flash-Next](https://huggingface.co/Qwen/Qwen3.8-Flash-Next)**, a 125-billion-parameter model that
normally needs a server, on a normal PC. It chats, writes code, reads pictures and works with your apps and coding
agents. Your chats and files stay on your PC.

**New here?** Check [what you need](#what-you-need), then [install the app](#option-a-the-strata-app).

- **Runs on a gaming PC.** 32 GB of RAM and graphics memory together is enough to start.
- **Works with your tools.** It speaks the OpenAI and Anthropic APIs, so Claude Code, Codex CLI, OpenCode and other agents connect to it.
- **Reads pictures.** They are on by default.
- **Free.** MIT license, no account, no cloud.
- **Private.** No telemetry. What Strata downloads, when it uses the internet and how it checks downloads is in [docs/SAFETY.md](docs/SAFETY.md).

**Free, open source, and yours to run.** Use Strata on your desktop or with your favorite coding tools. No account needed.

## Models

We recommend **ISTA-DASLab first**, then **Unsloth** and other providers, where a compatible model fits.

<details>
<summary><img src="docs/media/models/qwen.svg" width="22" alt="" align="middle"> &nbsp;<b>Qwen3.8-Flash-Next</b></summary>

| Provider | Choices |
| --- | --- |
| **ISTA-DASLab · recommended** | Q2\_0, IQ2\_XS, IQ3\_XXS, IQ3\_S. [Coder: IQ1\_M](docs/MODELS.md#coder) for coding. |
| **Unsloth** | UD-IQ4\_XS, UD-Q4\_K\_XL, UD-Q5\_K\_XL, UD-Q6\_K\_XL, Q8\_0 |
| **Gyro · agentionai** | [Gyro-S, Gyro-M](https://huggingface.co/agentionai/Qwen3.8-Flash-Next-Gyro-GGUF). Lower memory use but slower. |
| **Other options** | [Swift 1.5](docs/MODELS.md#swift-15), [NVFP4](docs/NVFP4.md), [OrcaRouter](docs/ORCA.md) |

[Compare sizes, memory needs and compatibility](docs/MODELS.md).

</details>

<details>
<summary><img src="docs/media/models/qwen.svg" width="22" alt="" align="middle"> &nbsp;<b>Qwen3.6-35B-A3B</b></summary>

**Unsloth:** UD-Q4_K_XL, UD-Q6_K_XL, Q8_0. These are the currently documented packs for this model.

[Model guide and compatibility](docs/QWEN36.md).

</details>

<a id="the-other-sizes"></a>

Not sure what fits? [Choose a model for your computer](docs/MODELS.md#pick-by-memory).

## Install

<a id="option-a-the-strata-app"></a>

### Desktop app

**[Download Strata for Windows, Linux or Mac →](https://github.com/Niko1221/Strata/releases/latest)**

Install the app, choose a model, and start chatting. There is one installer for each system. On first start the app
downloads the engine for your graphics card. [Help with the installer](docs/INSTALL.md#the-app).

<sub>Files: `Strata-0.2.15-windows-x64-setup.exe` · `Strata-0.2.15-linux-x86_64.AppImage` · `Strata-0.2.15-macos-arm64.dmg`</sub>

<p align="center">
  <img src="docs/media/app-chat.png" width="900" alt="Strata desktop app with a chat and live code preview side by side">
  <br><sub>Chat, files and a live preview in one window.</sub>
</p>

<a id="option-b-the-terminal-one-line"></a>

### Terminal app

Prefer the terminal? Install Strata, then run **`strata launch`**.

<details open>
<summary><b>Install commands</b></summary>

**Linux / Mac**

```sh
curl -fsSL --connect-timeout 15 --max-time 120 https://github.com/Niko1221/Strata/releases/latest/download/install.sh -o install.sh && sh install.sh
```

**Windows PowerShell**

```powershell
irm -TimeoutSec 120 https://github.com/Niko1221/Strata/releases/latest/download/install.ps1 | iex
```

On Windows, open a new terminal afterwards. Then:

```sh
strata launch
```

Follow the prompts to download and start a model. [Terminal guide](docs/TERMINAL_LAUNCH.md).

</details>

<p align="center">
  <img src="docs/media/terminal-monitor.png" width="900" alt="Strata terminal app showing model activity, speed, memory and recent requests">
  <br><sub>Watch your model and hardware while it runs.</sub>
</p>

<a id="build-from-source"></a>

[Build from source](docs/SOURCE_INSTALL.md) · [Docker](docs/DOCKER.md) · [Full install guide](docs/INSTALL.md)

<a id="what-you-need"></a>

## Hardware

| Supported hardware | Guide |
| --- | --- |
| **NVIDIA GeForce RTX 20 / 30 / 40 / 50 series** | [NVIDIA setup](docs/INSTALL.md#what-you-need) |
| **AMD Radeon RX 7700 / 7800 / 7900, RX 9060 / 9070, AI PRO R9700** | [AMD setup](docs/AMD_HIP.md) |
| **AMD Radeon RX 6800 / 6900 · experimental** | [RX 6000 setup](docs/AMD_HIP.md#rdna2-gfx1030) |
| **Intel Arc · Linux** | [Intel setup](docs/INTEL_ARC.md) |
| **Apple Silicon · M1 or newer** | [Mac setup](docs/MAC.md) |
| **AMD Ryzen AI Max · Strix Halo** | [Strix Halo setup](docs/STRIX_HALO.md) |
| **NVIDIA DGX Spark · Linux** | [DGX Spark setup](docs/DGX_SPARK_LINUX.md) |

<a id="using-it"></a>

## Tools

**Claude Code · Codex CLI · OpenCode · oh-my-pi · pi.dev · Hermes Agent · OpenClaw**, and more through the OpenAI- and Anthropic-compatible APIs.

Run **`strata connect`** to see setup instructions for your tool. [Connection guide](docs/TERMINAL_LAUNCH.md#connect-your-apps-strata-connect).

<a id="guides"></a>
<a id="how-fast-is-it"></a>
<a id="how-does-it-work"></a>
<a id="something-went-wrong"></a>

## Help

[All guides](docs) · [Troubleshooting](docs/TROUBLESHOOTING.md) · [Report a bug](https://github.com/Niko1221/Strata/issues/new/choose) ·
[Benchmarks](docs/COMMUNITY_BENCHMARKS.md) · [How it works](docs/HOW_IT_WORKS.md) · [Contribute](CONTRIBUTING.md)

## Support Strata

**Help keep local AI open to everyone.** A GitHub sponsorship or a coffee helps fund development and testing on more hardware.

<a href="https://github.com/sponsors/Niko1221"><img src="https://img.shields.io/badge/Sponsor_on_GitHub-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white" alt="GitHub Sponsors: sponsor Strata" height="48"></a>
<a href="https://buymeacoffee.com/strataengine"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me a Coffee: support Strata" height="48"></a>

### Loyal supporters

Thank you to everyone who supports Strata.

<!-- supporters:start (one line per public supporter, newest last; copy a line and change the GitHub name in all four places) -->
<a href="https://github.com/ryan4yin"><img src="https://github.com/ryan4yin.png?size=96" width="48" height="48" alt="ryan4yin" title="ryan4yin"></a>
<a href="https://github.com/themactep"><img src="https://github.com/themactep.png?size=96" width="48" height="48" alt="themactep" title="themactep"></a>
<a href="https://github.com/macintog"><img src="https://github.com/macintog.png?size=96" width="48" height="48" alt="macintog" title="macintog"></a>
<a href="https://github.com/bisonbet"><img src="https://github.com/bisonbet.png?size=96" width="48" height="48" alt="bisonbet" title="bisonbet"></a>
<a href="https://github.com/vizv"><img src="https://github.com/vizv.png?size=96" width="48" height="48" alt="vizv" title="vizv"></a>
<a href="https://github.com/kod88vn"><img src="https://github.com/kod88vn.png?size=96" width="48" height="48" alt="kod88vn" title="kod88vn"></a>
<a href="https://github.com/seoulworker"><img src="https://github.com/seoulworker.png?size=96" width="48" height="48" alt="seoulworker" title="seoulworker"></a>
<a href="https://github.com/jackyini"><img src="https://github.com/jackyini.png?size=96" width="48" height="48" alt="jackyini" title="jackyini"></a>
<a href="https://github.com/router0mail"><img src="https://github.com/router0mail.png?size=96" width="48" height="48" alt="router0mail" title="router0mail"></a>
<!-- supporters:end -->

[MIT license](LICENSE) · [Credits and model licenses](docs/HOW_IT_WORKS.md#credits) · [Privacy](docs/SAFETY.md)

<a id="option-c-let-your-ai-set-it-up"></a>

<sub>For AI tools and automated readers: <a href="AGENTS.md">technical reference</a> · <a href="docs/AI_SETUP.md">setup instructions</a>.</sub>

**Sponsorships, hardware partnerships or collaborations:** [niko.veit@outlook.com](mailto:niko.veit@outlook.com).

