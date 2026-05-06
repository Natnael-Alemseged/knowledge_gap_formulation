# Colab Notebook Instructions for the LoRA Inference Benchmark

Compares three conditions on the same GPU:

1. Base model only
2. Base model + LoRA adapter loaded separately (unmerged)
3. Base model with adapter merged via `merge_and_unload()`

---

## Before you begin

You need:
- A Colab GPU runtime (T4 is fine)
- `ADAPTER_PATH` — your Week 11 LoRA adapter as a HuggingFace repo ID (e.g. `your-username/qwen3-lora-judge`) or local path
- A HuggingFace token if the adapter repo is private

---

## Cell 1 — Install

Run this, then click **Runtime → Restart session** when it finishes.

```python
%pip uninstall -y torchvision torchaudio
%pip install -q transformers==4.51.3 peft==0.14.0 accelerate==1.4.0 sentencepiece huggingface_hub
```

---

## Cell 2 — Verify (run after restart)

```python
import torch, transformers, peft, accelerate

print("transformers:", transformers.__version__)
print("peft:        ", peft.__version__)
print("accelerate:  ", accelerate.__version__)
print("CUDA:        ", torch.cuda.is_available())
print("GPU:         ", torch.cuda.get_device_name(0) if torch.cuda.is_available() else "MISSING — change runtime to GPU")
```

You want no import errors and `CUDA: True`.

---

## Cell 3 — HuggingFace login (only if adapter is private/gated)

```python
from huggingface_hub import login
login()
```

---

## Cell 4 — Config

Edit the two paths before running anything else.

```python
BASE_MODEL   = "Qwen/Qwen3-0.6B"
ADAPTER_PATH = "REPLACE_WITH_YOUR_ADAPTER_PATH"   # <-- fill this in

PROMPT = (
    "Classify the intent of this sales email: "
    "Hi team, I want to know whether your platform supports "
    "bulk onboarding for new employees next month."
)
MAX_NEW_TOKENS = 64
RUNS = 11  # first run is warmup, 10 are measured
```

---

## Cell 5 — Benchmark helpers

```python
import gc, math, time
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

assert torch.cuda.is_available(), "Switch runtime to GPU first"
DEVICE = "cuda"

tokenizer = AutoTokenizer.from_pretrained(BASE_MODEL)
inputs = tokenizer(PROMPT, return_tensors="pt").to(DEVICE)

def cleanup():
    gc.collect()
    torch.cuda.empty_cache()
    torch.cuda.synchronize()

def timed_generate(model):
    model.eval()
    torch.cuda.synchronize()
    t0 = time.perf_counter()
    with torch.inference_mode():
        model.generate(**inputs, max_new_tokens=MAX_NEW_TOKENS, do_sample=False)
    torch.cuda.synchronize()
    return time.perf_counter() - t0

def load_base():
    return AutoModelForCausalLM.from_pretrained(
        BASE_MODEL, torch_dtype=torch.float16
    ).to(DEVICE)

def load_unmerged():
    return PeftModel.from_pretrained(load_base(), ADAPTER_PATH).to(DEVICE)

def load_merged():
    m = PeftModel.from_pretrained(load_base(), ADAPTER_PATH)
    return m.merge_and_unload().to(DEVICE)

def collect(label, loader):
    cleanup()
    model = loader()
    times = [timed_generate(model) for _ in range(RUNS)][1:]  # drop warmup
    mean = sum(times) / len(times)
    std  = math.sqrt(sum((t - mean)**2 for t in times) / len(times))
    print(f"{label}: mean={mean:.3f}s  std={std:.3f}s  (n={len(times)})")
    del model
    cleanup()
```

---

## Cell 6 — Run

Takes ~5 minutes on T4.

```python
collect("base",     load_base)
collect("unmerged", load_unmerged)
collect("merged",   load_merged)
```

---

## What to send back

Copy the three printed lines, e.g.:

```
base:     mean=X.XXXs  std=X.XXXs  (n=10)
unmerged: mean=X.XXXs  std=X.XXXs  (n=10)
merged:   mean=X.XXXs  std=X.XXXs  (n=10)
```

Paste them back and the result table will be added to the explainer.

---

## Expected pattern

```
merged  ≈ base       (same tensor shapes, same memory traffic)
unmerged > base      (extra low-rank path still runs at inference time)
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `CUDA: False` | Runtime → Change runtime type → GPU |
| Import errors after install | You forgot to restart the session after Cell 1 |
| Out of memory | Set `MAX_NEW_TOKENS = 32` |
| Adapter not found | Check the exact repo ID or local path in `ADAPTER_PATH` |
| Private repo error | Run Cell 3 (HuggingFace login) |
| `torchvision::nms does not exist` | Add `%pip uninstall -y torchvision torchaudio` to the top of Cell 1, restart session, rerun |
