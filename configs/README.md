# Serve configs (recipes)

Machine-tuned `serve/server.py --config` recipes. Paths are absolute for the
host they were tuned on; adjust `exe`, `--pack`, `--native`, `--ple-gguf` and
`tokenizer` for your own checkout.

## strata-navin-ad427-rx6700m.json

MSI Delta 15 (Ryzen 5800H + RX 6700M, gfx1031, 10 GB VRAM, 64 GB RAM),
decoding `Navin-Models/Qwen3.8-Flash-Next-Uncensored-AD-4.27-GGUF` through a
Strata native pack (IQ3_S gate/up + IQ2_S down experts).

Points of interest:

- `--spec 4` + `STRATA_SPEC_PROB=1`: wider verify windows with
  probability-based acceptance. Measured against `--spec 3` on identical
  prompts: 2.5-2.9 accepted tokens per verify window vs 1.7-2.1, warm decode
  23.4-23.7 tok/s vs 15.0-20.2, draft acceptance 70-78% vs 52-77% - despite a
  *lower* expert-cache hit rate (37-42% vs 47-49%). The wider window
  amortizes the fixed miss cost over more tokens.
- `--resident-experts` + `--expert-profile-save`: turns on the adaptive tier
  (#477), so the VRAM expert cache learns the routing of the *actual* traffic
  instead of trusting only the static profile, and persists what it learned
  (written every `--expert-profile-save-every` minutes, default 10).
- `--vram-reserve-mib 797`: the engine's own suggested reserve on this card at
  `--max-context 150000`. Below it the verify-window capture drives free VRAM
  to ~37 MiB and requests stall (the engine then prints the same suggestion).
- `--conversation-cache-mib 1024`: 2048 let conversation parking eat the last
  RAM headroom and starve the prefill expert-activation allocation.
- `STRATA_DECODE_TIMING=1`: one anatomy line per request (tokens per verify
  window, GPU-reach wait vs per-layer host compute) - the numbers above came
  from these lines.
