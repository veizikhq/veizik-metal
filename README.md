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

Context: 32,768 tokens declared, 30,000 measured. Device-memory figures are on the benchmarks board; they carry over to this build because its engines are byte-identical to the measured ones. Checkpoints are not included — `veizik pull` fetches them from Hugging Face under Alibaba Cloud's Apache-2.0 licence.

## Install and first run
```bash
curl -LO https://github.com/veizikhq/veizik-metal/releases/download/v2026.10.02-r6.5/veizik-metal-2026.10.02-r6.5.dmg
shasum -a 256 veizik-metal-2026.10.02-r6.5.dmg   # 10317c79def77fe6c90b55d1db2603d39c5f6efd909e679956f699937993898c
M=$(hdiutil attach -nobrowse veizik-metal-2026.10.02-r6.5.dmg | awk -F'\t' '/Volumes/{print $NF}' | tail -1)
"$M/install.sh" && hdiutil detach "$M"   # the mount point is read from the attach, not assumed
export PATH="$HOME/.local/bin:$PATH"
veizik activate                       # opens veizik.com/activate — sign in (email link, Google or GitHub) and approve this Mac once
#   no browser on this Mac (SSH)? on your account page create a key (Browser-free activation) and set a CLI password, then:
#   veizik activate --key vzek_… --password-stdin < pwfile
veizik pull qwen2.5-1.5b-instruct     # 2.8 GiB from Hugging Face, verified; an existing ~/.cache/huggingface copy is reused
veizik chat qwen2.5-1.5b-instruct     # interactive; or:
veizik run qwen3-1.7b "The capital of France is" --tokens 32
veizik run qwen3-1.7b "Write a haiku about rain" --temperature 0.8 --seed 7   # sampling is opt-in; omit the flags for greedy output
veizik serve qwen2.5-1.5b-instruct --port 8000   # then POST http://127.0.0.1:8000/v1/chat/completions
```
The installer verifies SHA256SUMS and the Developer ID signatures before copying; the disk image is notarized by Apple.
What needs the network: creating the account, activating the device, downloading a checkpoint, and the first run of each session (a short-lived licence lease held in memory). Generation itself does not.

## Measured on this build
- Installed binaries: this build adds a resident serve engine (VeizikServe.app) per model alongside each run core, so the installed footprint is larger than the previous build; the measured per-build figure is on the benchmarks board, bound to these cores. The CLI links libSystem; each core links libSystem, libobjc, Metal, IOKit, CoreFoundation, Security.
- Speed: all eight engines in this release are byte-for-byte the engines of v2026.10.02-r6.1, r6.2, r6.3 and r6.4 (the four run engines are also the r5 ones), so every row the benchmarks board timed on the engines applies unchanged: https://veizik.com/benchmarks . This release changes only the `veizik` command. Rows that time the whole command (wall rows) include the `veizik` command, which changed in this release; they are held until they are timed on this build.
- Same prompt, same machine, same build → identical output (greedy decoding, the default). With `--temperature` above 0 the output is sampled; the same `--seed` and settings on the same Mac give the same text again. `veizik serve` gives the same text as `veizik run` for the same prompt and settings, checked token by token at 4K, 8K, 16K and 30K-token prompts. The same holds at 1, 4 and 16 concurrent requests.
- Where it is behind: the benchmarks board lists every row where another engine is ahead, with the method beside it — https://veizik.com/benchmarks

## Full benchmarks and how to reproduce
https://veizik.com/benchmarks — every published row names the device, model, token counts, the other engine's version and settings, how each side was timed, and the spread. Rows measured on earlier builds of the same cores are marked. Reproduce with `veizik bench <model>` and the row's command.

## Limits, troubleshooting, terms
- `veizik serve` takes a model name to serve just that one, or no name at all to host every model installed on the machine (variants such as -instruct and -base each run their own engine), routing each request by its "model" field; `/v1/models` lists what it is hosting. Sampler fields (temperature, top_p, top_k, seed, stop, penalties) are passed to the engine, which applies them or answers with an error naming the one it does not support. Streaming (`stream: true`) is served by the resident engine as Server-Sent Events; 501 is returned only for tools combined with streaming, or when no resident engine is available and the server falls back to one process per request. `/v1/embeddings` answers 501 in this release.
- Serve window: each request may use up to 4,096 tokens (prompt plus max_tokens). A request that does not fit is refused with HTTP 400 `context_length_exceeded`, naming the limit and the size asked for; the server and every other request carry on.
- Qwen3-1.7B returns its reasoning text inside the answer; add `/no_think` to the prompt for a direct reply.
- Offline: run once while online and the session key stays in memory for 48 hours (server-set, as of 2026-10-02); restarting the Mac ends that window, and a new session cannot start offline outside it.
- Upgrading: this release's disk image carries its own volume name, so it no longer lands beside an older copy under a numbered suffix — which is what previously let an install run the earlier version's installer and succeed. The command above still reads the mount point from the attach rather than assuming it, because that is correct for any build. `veizik doctor` prints the installation it is running from.
- If the engine cannot start on your Mac it says so and exits rather than running anyway. This only happens on configurations that restrict what an application may do with memory, which a stock macOS does not; if you see it, support@veizik.com.
- Not supported: Intel Macs, Windows, Linux (NVIDIA support is in preparation and not public yet).
- Troubleshooting: `veizik doctor` · https://veizik.com/docs#troubleshooting · Issues and Discussions in this repository · support@veizik.com
- Licence: personal, educational, research and evaluation use at no charge; commercial use needs a separate licence; publishing benchmarks is explicitly allowed. See LICENSE and NOTICE.md. Veizik is a brand of LinkPick, Seoul.

## Package contents (v2026.10.02-r6.5, sha256 10317c79…)
veizik 1,681,456 B · install.sh 18,896 B · models.json 9,076 B · models.json.sig 64 B · coreids.json 1,373 B · MANIFEST.txt 490 B · LICENSE 6,029 B · NOTICE.md 1,859 B · entitlement_enums.json 739 B · models/ (4 run cores + 4 resident serve cores + activation helpers, the same bytes as r6.4, r6.3, r6.2 and r6.1). Verify with `shasum -a 256 -c SHA256SUMS`.