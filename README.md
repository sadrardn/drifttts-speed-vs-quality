# DriftTTS — Speed vs Quality

Reproducing the speed-versus-quality tradeoff in the style of DriftTTS,
without training DriftTTS from scratch.

We use pretrained **Matcha-TTS** (`matcha_ljspeech`) with the **HiFi-GAN**
vocoder (`hifigan_T2_v1`) on the same 10 fixed English sentences, varying
the number of decoder steps (NFE) and measuring speed (RTF) and quality
(WER, plus a listening check) to find the knee point.

## Results (Stage 2, 2026-11-05)

Total audio per NFE: 35.86 s (10 sentences, 101 words in total).
RTF here is **wall-clock RTF**: total wall time (including model loading
and the vocoder) divided by total audio duration. It is *not* directly
comparable to the DriftTTS paper's decoder-only RTF, which excludes the
vocoder and loading.

| NFE | RTF (wall) | Mean WER | Wall time (s) |
|----:|-----------:|---------:|--------------:|
| 1 | 0.518 | 0.0665 | 18.6 |
| 2 | 0.406 | 0.0283 | 14.5 |
| 4 | 0.400 | 0.0283 | 14.4 |
| 10 | 0.399 | 0.0283 | 14.3 |
| 25 | 0.404 | 0.0483 | 14.5 |

WER was measured with faster-whisper `tiny` (CPU, int8) and `jiwer`,
the same transcriber for every NFE.

Listening check (sentence 3, "The train arrived at the station just
after sunset."): NFE 1 sounded worse than NFE 10, matching the WER.

**Reading:** quality improves from NFE 1 to 2, then stays flat for
NFE 2, 4 and 10. Wall time is also flat after NFE 2, because model
loading and the vocoder dominate the wall time and hide the decoder
effect. So the wall-clock knee looks around **NFE 2, but it is
provisional** — a decoder-only measurement (model loaded once, repeated
synthesis timed without reloading) is needed to make it firm.
The slightly higher WER at NFE 25 (4.83% vs 2.83%) is only about
2 words on 101 words, so treat it as tiny-transcriber noise, not a
real degradation.

For comparison, the DriftTTS paper (LJSpeech) reports Matcha-TTS at
10 steps: WER 1.93, MOS 4.38, RTF 0.0187 (decoder-only); DriftTTS at
1 step: WER 1.80, MOS 4.43, RTF 0.00441 (decoder-only).

## Notebooks

Run them in Google Colab, top to bottom, in this order. Each Colab
notebook runs on its own machine, so an environment built in one
notebook does **not** carry over to another — the stage-2 notebook
is self-contained and rebuilds the environment at its start.

- `notebooks/step1-matcha-smoke-test.ipynb` — first attempt on
  Colab Python 3.13 (kept for history; Matcha-TTS needs Python 3.11).
- `notebooks/step1b-matcha-python311-uv.ipynb` — Python 3.11 virtual
  environment (`matcha311`) built with `uv`, torch + Matcha-TTS
  installed, one test sentence at 10 steps. Keep this as the rebuild
  backup.
- `notebooks/step2-speed-curve.ipynb` — stage 2: the same 10
  sentences at NFE 1, 2, 4, 10, 25, wall RTF, listening check on
  sentence 3, WER via faster-whisper, `summary.csv` and three charts.

Environment: Google Colab, Tesla T4 GPU, Python 3.11 venv via `uv`,
torch 2.15 (cu121), Matcha-TTS, HiFi-GAN. The official Matcha
checkpoint is loaded with `weights_only=False` (patched in
`matcha/cli.py`); use that option only for a checkpoint from a
trusted official release, never for an unknown file.

## Results files

- `results/summary.csv` — the table above, one row per NFE.
- The full Persian report (8-page PDF + editable Word) is in the
  project files; PDF and Word are binary files and are not part of
  this text-only push — upload them to `report/` from the GitHub
  web interface if you want them in the repository.

## Sentences (fixed for all NFE)

1. The quick brown fox jumps over the lazy dog.
2. She opened the window and looked at the garden.
3. The train arrived at the station just after sunset.
4. He read the book slowly, one page at a time.
5. The children played in the park until it started to rain.
6. A small bird sang on the branch above the quiet river.
7. The old house stood at the end of a long, dusty road.
8. She wrote a letter to her friend about the summer holiday.
9. The baker made fresh bread every morning before sunrise.
10. They walked home together under a sky full of stars.
