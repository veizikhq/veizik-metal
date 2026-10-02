# veizik-metal

**Run a local LLM on your Mac with one small native engine.**
Veizik opens the safetensors checkpoint you already downloaded — Qwen2.5 and Qwen3 today — and runs it on the Apple GPU. No Python environment, no framework, no converted copy of the model. Prompts and generated text stay on the machine.

## Who this is for
Developers and teams who run open models on Apple Silicon and want one signed binary instead of a framework stack: a quick `chat`, a one-shot `run`, or a local HTTP server with OpenAI-style request and response shapes for your own tools.

## What it looks like
```text
$ veizik chat qwen2.5-1.5b-instruct
vz > What is a safetensors checkpoint? Answer in two sentences.
A Safetensors checkpoint is a pre-optimized model checkpoint that is designed to be compatible with …
```

## Supported environment and models
- Apple Silicon Mac (M1 or newer), macOS 13 or later, arm64 only. Tested on macOS 15.5 (M2 Pro), 15.7.7 (M4 Pro), 26.5 (M1 Max), 26.5.1 (M1 Ultra). Intel Macs are not supported.
- Models in this release (8 catalogue ids, 4 execution cores):

| Model | Hugging Face checkpoint | Chat |
|---|---|---|
| qwen2.5-0.5b / -instruct | Qwen/Qwen2.5-0.5B / -Instruct | instruct only |
| qwen2.5-1.5b / -instruct | Qwen/Qwen2.5-1.5B / -Instruct | instruct only |
| qwen2.5-7b / -instruct | Qwen/Qwen2.5-7B / -Instruct | instruct only |
| qwen3-1.7b / -base | Qwen/Qwen3-1.7B / -Base | qwen3-1.7b |

Context: 32,768 tokens declared, 30,000 measured. Device-memory figures are on the benchmarks board once re-measured on this build. Checkpoints are not included — `veizik pull` fetches them from Hugging Face under Alibaba Cloud's Apache-2.0 licence.

## Install and first run
```bash
curl -LO https://github.com/veizikhq/veizik-metal/releases/download/v2026.10.02-r2/veizik-metal-2026.10.02-r2.dmg
shasum -a 256 veizik-metal-2026.10.02-r2.dmg   # d4860f6a34f8f95f879293f50986faaf1994b49457f924d0c45f5c79af0b3f6a
open veizik-metal-2026.10.02-r2.dmg && "/Volumes/veizik-metal 2026.10.02/install.sh"
export PATH="$HOME/.local/bin:$PATH"
veizik activate                       # opens veizik.com/activate — sign in (email link, Google or GitHub) and approve this Mac once
veizik pull qwen2.5-1.5b-instruct     # 2.8 GiB from Hugging Face, verified; an existing ~/.cache/huggingface copy is reused
veizik chat qwen2.5-1.5b-instruct     # interactive; or:
veizik run qwen3-1.7b "The capital of France is" --tokens 32
veizik serve qwen2.5-1.5b-instruct --port 8000   # then POST http://127.0.0.1:8000/v1/chat/completions
```
The installer verifies SHA256SUMS and the Developer ID signatures before copying; the disk image is notarized by Apple.
What needs the network: creating the account, activating the device, downloading a checkpoint, and the first run of each session (a short-lived licence lease held in memory). Generation itself does not.

## Measured on this build
- Installed binaries: 4,777,641 bytes in 43 files; the installer aliases the instruct variants rather than copying them. The CLI links libSystem; each core links libSystem, libobjc, Metal, IOKit, CoreFoundation, Security.
- Speed figures are not printed here. Every one this project has measured was taken on the previous build's cores, which this release replaces, and a rate belongs to the binaries it was measured on. The board at https://veizik.com/benchmarks carries them as soon as this package has been measured, bound to its cores.
- Same prompt, same machine, same build → identical output (greedy decoding). Verified on the previous build across 60 benchmark cells on three Macs, 60 of 60; being re-checked on this one.
- Where it is behind: the benchmarks board lists every row where another engine is ahead, with the method beside it — https://veizik.com/benchmarks

## Full benchmarks and how to reproduce
https://veizik.com/benchmarks — every published row names the device, model, token counts, the other engine's version and settings, how each side was timed, and the spread. Rows measured on earlier builds of the same cores are marked. Reproduce with `veizik bench <model>` and the row's command.

## Limits, troubleshooting, terms
- `veizik serve` takes a model name to serve just that one, or no name at all to host every model installed on the machine, routing each request by its "model" field; `/v1/models` lists what it is hosting. Streaming and request fields such as temperature, top_p, seed, tools and response_format are rejected with an explicit error.
- Qwen3-1.7B returns its reasoning text inside the answer; add `/no_think` to the prompt for a direct reply.
- Offline: run once while online and the session key stays in memory for 48 hours (server-set, as of 2026-10-02); restarting the Mac ends that window, and a new session cannot start offline outside it.
- If the engine cannot start on your Mac it says so and exits rather than running anyway. This only happens on configurations that restrict what an application may do with memory, which a stock macOS does not; if you see it, support@veizik.com.
- Not supported: Intel Macs, Windows, Linux. Headless activation is disabled.
- Troubleshooting: `veizik doctor` · https://veizik.com/docs#troubleshooting · Issues and Discussions in this repository · support@veizik.com
- Licence: personal, educational, research and evaluation use at no charge; commercial use needs a separate licence; publishing benchmarks is explicitly allowed. See LICENSE and NOTICE.md. Veizik is a brand of LinkPick, Seoul.

## Package contents (v2026.10.02-r2, sha256 d4860f6a…)
veizik 832,992 B · install.sh 7,577 B · models.json 9,076 B · models.json.sig 64 B · coreids.json 1,373 B · MANIFEST.txt 490 B · LICENSE 6,029 B · NOTICE.md 1,859 B · models/ (4 cores + activation helpers). Verify with `shasum -a 256 -c SHA256SUMS`.