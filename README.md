# Data Pipeline CO₂ Calculator

A single-file, dependency-free calculator that estimates the **annual operational CO₂** of a data pipeline and the **net** effect when it reduces data volume before an analytics platform (SIEM, XDR, observability, data lake).

Companion to the article *The Hidden Carbon Cost of Data Pipelines*.

## Files

```
index.html   # the calculator (HTML, CSS and JS in one file)
README.md
```

## Features

- Compares five runtime classes for the same workload: compiled with no GC, compiled with GC, managed VM, JavaScript engine and interpreted
- Shows **gross** CO₂ avoided downstream, **minus** the pipeline's own footprint, giving the **net** result
- Adjustable workload, hardware generation, utilisation, PUE and grid carbon intensity
- Shows the full formula for every result
- Plain HTML, CSS and JS with no build step and no external libraries (Google Fonts only)

## Usage

| Where | How |
|---|---|
| **Browser** | Clone the repo and open `index.html`. |
| **GitHub Pages** | **Settings → Pages →** deploy from the `main` branch, root folder. The calculator is served at `https://<user>.github.io/<repo>/`. |
| **CodePen** | Paste the whole file into the **HTML** panel only. Leave CSS and JS empty. |
| **Medium** | Paste the public CodePen pen URL on its own line and press Enter. Medium doesn't accept iframe or embed code. |
| **Website / CMS** | Paste into any HTML block that allows `<style>` and `<script>`. |

## Inputs

| Input | Default | Meaning |
|---|---|---|
| Daily ingest volume | 7 TB/day | Data entering the pipeline |
| Runtime class | Compiled, no GC | Language and runtime the pipeline is built on |
| Baseline throughput | 50 GB/day per vCPU | Throughput of the most efficient class; other classes are scaled from this |
| Volume reduced | 45% | Share of data filtered before the analytics tier |
| Analytics throughput | 11.7 GB/day per vCPU | Downstream platform sizing (for example, 600 vCPU for 7 TB/day) |
| CPU generation | Mid (c. 2016) | Sets idle and maximum watts per vCPU |
| CPU utilisation | 50% | Average load |
| PUE | 1.56 | Data-centre overhead (industry average) |
| Grid intensity | 458 gCO₂e/kWh | Global average for 2025, or choose a preset or custom value |

## Method

```
W per vCPU        = min W + utilisation × (max W − min W)
Pipeline vCPU     = daily GB ÷ (baseline ÷ runtime CPU factor)
Pipeline RAM      = vCPU × runtime RAM factor            (0.392 W per GB)
kWh per year      = (CPU W + RAM W) × 8,760 h × PUE
CO₂               = kWh × grid intensity
Downstream vCPU   = daily GB × reduction % ÷ analytics throughput   (4 GB RAM each)
Net CO₂ avoided   = downstream CO₂ avoided − pipeline CO₂
```

**Runtime factors (CPU / RAM, relative to compiled with no GC):**

| Runtime class | CPU | RAM |
|---|---|---|
| Compiled, no GC (C, C++, Rust) | 1.0× | 1.0× |
| Compiled with GC (Go) | 1.5× | 1.5× |
| Managed VM (Java, C#) | 2.0× | 3.0× |
| JavaScript engine (Node.js) | 2.5× | 2.0× |
| Interpreted (Python, Ruby) | 4.0× | 2.0× |

These factors are deliberately conservative assumptions. Published benchmarks show much larger gaps, but real pipelines spend much of their time waiting on input and output.

## Customising

All assumptions are constants at the top of the `<script>` block in `index.html`:

- `RT`: runtime classes and their CPU and RAM factors
- `HW`: CPU generations, with idle and maximum watts per vCPU
- `GRID`: grid intensity presets in gCO₂e/kWh
- `W_PER_GB_RAM`, `DS_RAM_PER_VCPU`: memory power and downstream RAM per vCPU

Replace them with your own benchmark figures for a defensible result.

## Sources

- Pereira et al. (2021), *Ranking programming languages by energy efficiency*, Science of Computer Programming
- Cloud Carbon Footprint methodology (per-vCPU watts by CPU generation, memory coefficient)
- Uptime Institute Global Data Center Survey 2024 (PUE)
- Ember, Global Electricity Review 2025/2026 (grid intensity)

## Disclaimer

Estimates for discussion only. The calculator covers **operational electricity only**: not embodied hardware emissions, network or storage. Real values vary with workload, configuration, hardware, facility and grid. It does not evaluate or endorse any product or vendor.
