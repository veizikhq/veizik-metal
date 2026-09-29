# Veizik Metal runtime — Third-Party Notices

Copyright © 2026 LinkPick. All rights reserved. Veizik is a brand of LinkPick.

## 1. Redistributed with this package

Nothing. Every file here was built by LinkPick: one execution core and one
activation helper per model, the `veizik` command, the installer, the model
catalogue, and the checksums. `shasum -a 256 -c SHA256SUMS` lists the complete set.

**No model weights are included, and no model publisher's inference code.**

## 2. Required at run time, not redistributed

The runtime links only frameworks Apple ships with macOS. Their use is governed
by your macOS licence, not by this notice. No third-party runtime, interpreter,
toolchain or package manager is required, and none is installed.

## 3. Model checkpoints — yours, under their own licence

The runtime reads a checkpoint you obtained yourself, under the licence its
publisher attached to it. You accept that licence from the publisher, not from
us.

| Checkpoint | Publisher | Licence |
| --- | --- | --- |
| `Qwen/Qwen2.5-0.5B` | Alibaba Cloud | Apache-2.0 |
| `Qwen/Qwen2.5-1.5B` | Alibaba Cloud | Apache-2.0 |
| `Qwen/Qwen2.5-7B`   | Alibaba Cloud | Apache-2.0 |
| `Qwen/Qwen3-1.7B`   | Alibaba Cloud | Apache-2.0 |
| `Qwen/Qwen2.5-0.5B-Instruct` | Alibaba Cloud | Apache-2.0 |
| `Qwen/Qwen2.5-1.5B-Instruct` | Alibaba Cloud | Apache-2.0 |
| `Qwen/Qwen2.5-7B-Instruct`   | Alibaba Cloud | Apache-2.0 |
| `Qwen/Qwen3-1.7B-Base`      | Alibaba Cloud | Apache-2.0 |

All of these checkpoints are distributed by Alibaba Cloud under the Apache-2.0 licence.

## 4. Trademarks

Qwen is a trademark of its owner. Apple, Mac, macOS and Metal are trademarks of
Apple Inc. Use here is nominative — naming what this runs on and what it reads —
and implies no endorsement or affiliation.

Questions about these notices: <https://veizik.com>
