# RATs: Playful Agentic Robot Learning

[Project Page](https://Playful-RATs.github.io) &ensp;|&ensp; [Paper](https://playful-rats.github.io/assets/papers/rats-paper.pdf) &ensp;|&ensp; [arXiv](https://arxiv.org/abs/2606.19419)

<p align="center">
  <video src="https://github.com/user-attachments/assets/9c2edc29-e4cd-421c-ba82-2b0b8df17d7d" controls="controls" muted="muted" playsinline="playsinline" width="100%"></video>
</p>

---

**RATs** is a multi-agent Code-as-Policy system for lifelong robot skill learning. During free-form *play* a team of LLM agents invents its own tasks, writes code-as-policy, and distills successful executions into a reusable skill library; at *evaluation* those skills are reused as planner context — no gradients, no RL, all learning through structured natural-language feedback and code reuse. RATs runs in two stages:

| Stage | What it does |
| ----- | ------------ |
| **Play** | Curiosity-driven skill acquisition: a proposer → planner → policy-writer → verifier → failure-diagnoser loop invents tasks, executes code-as-policy, and grows a skill library from what works. |
| **Evaluation** | Frozen learned skills are injected as planner context and compared head-to-head with CaP-X baselines, in two complementary settings: **in-domain** (LIBERO-PRO, MolmoSpaces) and **cross-environment transfer** (reusing the same skills in Robosuite and on a real Franka Panda). |

<p align="center">
  <a href="https://playful-rats.github.io/assets/videos/pipeline-overview-v9.mp4">
    <img src="https://playful-rats.github.io/assets/images/rats-teaser.png" width="100%" alt="RATs pipeline overview — click to play the video">
  </a>
</p>

---

## Installation

RATs uses [uv](https://docs.astral.sh/uv/) for dependency management and runs on **Python 3.10** with a **CUDA-capable GPU**. The full guide — cache/quota tuning, every submodule, the MolmoSpaces bridge, and the `capx-baseline/` environments — is in **[docs/setup.md](docs/setup.md)**.

```bash
# Clone
git clone --branch main --depth 1 https://github.com/Playful-RATs/RATs rats
cd rats

# Root runtime submodules (LIBERO-PRO + MolmoSpaces)
git submodule update --init --depth 1 \
  rats/third_party/LIBERO-PRO \
  rats/third_party/libero_dependencies/robosuite \
  rats/third_party/robosuite \
  rats/third_party/contact_graspnet_pytorch \
  rats/third_party/curobo \
  rats/third_party/sam3

# Root env
uv venv .venv --python 3.10
source .venv/bin/activate
uv sync --frozen --active --extra libero --extra contactgraspnet

# LIBERO prompts for ~/.libero/config.yaml on first import in a fresh shell.
# Pre-create it to keep setup and batch runs non-interactive.
mkdir -p ~/.libero
cat > ~/.libero/config.yaml <<EOF
benchmark_root: $PWD/rats/third_party/LIBERO-PRO/libero/libero
bddl_files: $PWD/rats/third_party/LIBERO-PRO/libero/libero/./bddl_files
init_states: $PWD/rats/third_party/LIBERO-PRO/libero/libero/./init_files
datasets: $PWD/rats/third_party/LIBERO-PRO/libero/datasets
assets: $PWD/rats/third_party/LIBERO-PRO/libero/libero/./assets
EOF
```

> The root `.venv` runs both LIBERO-PRO and MolmoSpaces. MolmoSpaces additionally needs a `conda` bridge env and ~tens of GB of assets; CaP-X baselines, Robosuite transfer, and real-world transfer run from `capx-baseline/` with their own envs. See [docs/setup.md](docs/setup.md) for all of it.

## Quick Start

This section assumes the root `.venv` from Installation is active. Set a model provider and the runtime exports first:

```bash
export OPENAI_API_KEY="sk-..."       # and/or GEMINI_API_KEY / OPENROUTER_API_KEY
export PYTHONPATH="$PWD:${PYTHONPATH:-}"
export CAPX_ENV_STACK=libero
export MUJOCO_GL=egl
export PYOPENGL_PLATFORM=egl
```

### 1. Play — acquire skills

```bash
python scripts/run_rats.py \
  --config env_configs/libero/rats_libero_play.yaml \
  --explore --iterations 50 \
  --output-dir outputs/play_libero
```

MolmoSpaces play with the launcher-managed bridge is in [docs/play.md](docs/play.md).

### 2. Evaluate — in-domain (RATs vs. CaP-X)

The batch driver runs all six LIBERO-PRO suites (× 10 tasks × 5 trials) with skill reuse frozen:

```bash
RATS_VERIFIER_STRICT_BENCHMARK=1 python scripts/run_rats_libero_pro_batch.py \
  --seed-skill-library outputs/play_libero/snapshots/iter050/skills.json \
  --extra-rats-flags "--model google/gemini-3.5-flash" \
  --output-dir outputs/rats_libero_pro_iter050seed \
  --gpus 0,1,2,3,4,6,7 --workers 18 --skip-completed
```

CaP-X baselines, the MolmoSpaces eval, and result summarization are in [docs/evaluation.md](docs/evaluation.md).

### 3. Evaluate — cross-environment transfer

Reuse LIBERO-learned skills as planner context in Robosuite and on a real Franka — see [docs/cross-environment.md](docs/cross-environment.md).

---

## Documentation

| Guide | Contents |
| ----- | -------- |
| [Setup](docs/setup.md) | Full environment setup: prerequisites, cache/quota tuning, submodules, LIBERO-PRO / MolmoSpaces / CaP-X-baseline runtimes |
| [Play](docs/play.md) | Skill acquisition in LIBERO-PRO and MolmoSpaces with Contact-GraspNet |
| [Evaluation](docs/evaluation.md) | RATs vs. CaP-X in LIBERO-PRO and MolmoSpaces; output artifacts |
| [Cross-Environment Evaluation](docs/cross-environment.md) | Transferring learned skills to Robosuite and a real Franka Panda |

---

## Citation

```bibtex
@article{rats2026playful,
  title   = {Playful Agentic Robot Learning},
  author  = {Zhang, Junyi and Ge, Jiaxin and Yoo, Hanjun and Fu, Letian and Yang, Zihan and Liu, Yaowei and Saravanan, Raj and Yin, Shaofeng and Yu, Justin and Niu, Dantong and Wang, Zirui and Herzig, Roei and Goldberg, Ken and Bai, Yutong and Chan, David M. and Stoica, Ion and Kanazawa, Angjoo and Lei, Jiahui and Feng, Haiwen and Darrell, Trevor},
  journal = {arXiv preprint arXiv:2606.19419},
  year    = {2026}
}
```

## License

The original RATs code is licensed under the [Apache License 2.0](LICENSE).

Third-party code and dependencies retain their original licenses. In particular,
`capx-baseline/` remains covered by its [MIT license](capx-baseline/LICENSE),
and submodules under `rats/third_party/` retain their upstream licenses and notices.
