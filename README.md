# veizik-metal

On-device large language model inference for Apple Silicon. Models run locally on the
Apple GPU (Metal); prompts and generated text stay on the machine.

## Models

Qwen2.5-0.5B · Qwen2.5-1.5B · Qwen3-1.7B · Qwen2.5-7B

## Requirements

Apple Silicon Mac (M-series), recent macOS.

## Install & activate

1. Open the `.dmg` and run `install.sh`.
2. Activate the machine online with the `veizik` CLI (one-time, per machine — no key file).
3. Run a model with the `veizik` CLI.

## License

See `LICENSE`. Product page: https://veizik.com

## Package contents

Sizes of the top-level files in this package (verify the full set with shasum -a 256 -c SHA256SUMS):

  veizik 766736 B
  install.sh 6216 B
  models.json 9076 B
  models.json.sig 64 B
  coreids.json 1373 B
  MANIFEST.txt 490 B
  LICENSE 6029 B
  NOTICE.md 1859 B
