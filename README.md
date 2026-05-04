# Ollama Model Specs

**Live site:** https://jlaiii.github.io/ollama-model-specs/

Searchable database of every Ollama model with:
- 229 model families with parameter sizes
- Benchmark scores (MMLU, MATH, HumanEval, GPQA, Arena, etc)
- System requirements (min/rec RAM, min/rec VRAM, GPU vs CPU)
- VRAM estimates by quantization (Q2_K, Q4_K_M, Q5_K_M, Q8_0, FP16)
- Capability filters (vision, code, math, reasoning, chat, etc)
- Your system spec checker — input your RAM/VRAM and see which models run
- Side-by-side model comparison (up to 6)

## Data sources
- Model list: ollama.com/library
- Benchmarks: https://github.com/jlaiii/ollama-benchmarks
- Requirements: estimated from parameter count + quantization math
