---
tags:
  - machine-learning
  - hub
  - roadmap
created: 2026-11-13
updated: 2026-11-13
status: 🟢
level: mid-to-senior
---

# ML Hub

Central curriculum map and architectural decision index for mid-to-senior Machine Learning, Deep Learning systems, and Production MLOps. Integrates upstream software foundations from [[Programming/Programming Hub]] and transitions downstream into generative and agentic systems in [[AI/AI Engineering Hub]].

---

## Study Sequence

```mermaid
flowchart LR
    FUND["ML Fundamentals Guide"] --> ALGO["ML Algorithms Guide"]
    ALGO --> DL["Deep Learning Guide"]
    ALGO --> PROJ["ML Projects Guide"]
    DL --> PROJ
    PROJ --> MLOPS["MLOps & Production ML Guide"]
    MLOPS --> AI["AI Engineering Hub"]
```

---

## Curriculum Map

| Stage | Guide | Core Scope & First-Principles Coverage |
|---|---|---|
| **1. Foundations** | [[ML Fundamentals Guide]] | Statistical learning theory, bias-variance decomposition, split leakage invariants, calibration, PR-AUC / Pinball / NDCG metrics |
| **2. Classical ML** | [[ML Algorithms Guide]] | GLMs, regularization ($L_1$/$L_2$), kernel SVMs, second-order XGBoost/LightGBM split gain, clustering, dimensionality reduction |
| **3. Deep Learning** | [[Deep Learning Guide]] | Tensor strides, reverse-mode VJP autodiff, SwiGLU, Kaiming/residuals, RMSNorm, AdamW decoupled decay, RoPE/GQA Transformers, BF16 AMP |
| **4. End-to-End Systems** | [[ML Projects Guide]] | Imbalanced fraud w/ OOF encoding & cost thresholding, causal quantile forecasting, multi-label NLP + ONNX INT8, transfer CV + Grad-CAM |
| **5. Production MLOps** | [[MLOps & Production ML Guide]] | Pandera/Pydantic contracts, ASOF feature stores, MLflow aliases, Triton dynamic batching, PSI/KS drift, canary/bandit rollouts |

### Adjacent Vault Tracks

- **Software & Data Systems**: [[Programming/Programming Hub]] — Python concurrency, SQL query optimization, containerization, and distributed system design.
- **LLM & Agentic Engineering**: [[AI/AI Engineering Hub]] — Foundation model inference (vLLM/KV-cache), RAG evaluation, tool calling, and multi-agent orchestration.

---

## Model-Selection Decision Matrix

Select the simplest architecture whose inductive bias matches the data geometry and latency/interpretability SLA:

| Problem Geometry | Default First Baseline | Production Escalation | Avoid Unless Justified |
|---|---|---|---|
| **Structured Tabular ($10^3\text{–}10^7$ rows)** | Regularized Logistic/Ridge + OOF encoding | LightGBM / XGBoost + Isotonic Calibration | Deep MLPs / TabNet (rarely beat GBDTs; slower tuning) |
| **Multi-Horizon Time Series** | Seasonal Naive + Linear/Ridge w/ lags | LightGBM Quantile (P10/P50/P90) + `gap=H` CV | Unconstrained autoregressive LSTM (error compounding) |
| **Text Classification / Triage** | TF-IDF + `OneVsRest` LogisticRegression | DistilBERT / DeBERTa-v3 $\to$ ONNX INT8 | Prompting frontier LLMs for $<25\text{ ms}$ high-QPS routing |
| **Image Inspection ($<20\text{k}$ samples)** | Frozen `timm` backbone + linear probe | Two-stage differential LR + Grad-CAM + PTQ | Training ViT/CNN from scratch (severe overfitting) |

---

## Production Invariant Quick Reference

> [!info] Five Non-Negotiable Engineering Invariants
> 1. **Zero Temporal Leakage**: Every split, target encoder, rolling window, and feature join (`ASOF`, `feat_ts < label_ts`) must strictly respect information availability at inference time.
> 2. **Cost-Aligned Thresholds Require Calibration**: Never apply expected-cost decision thresholds $\tau^* = \frac{C_{\text{FP}}}{C_{\text{FP}} + C_{\text{FN}}}$ to uncalibrated `scale_pos_weight` or tree margin scores.
> 3. **Mixed Precision Dtype Selection**: Use `BF16` without `GradScaler` on Ampere+ GPUs; use `FP16` *only* with `GradScaler` and `unscale_` strictly before `clip_grad_norm_`.
> 4. **Evaluation Determinism**: Always pair `model.eval()` with `torch.inference_mode()` during validation/serving so BatchNorm running stats and Dropout stabilize.
> 5. **Monitor Inputs & Scores, Not Just Delayed Labels**: Track `PSI` (`<0.10 / 0.10–0.25 / >0.25`) and `Pydantic`/`Pandera` schema gates in real time because HTTP `200 OK` masks silent feature rot.

---

## Senior Mastery Checklist

### Statistical & Classical Foundations
- [ ] Derive bias-variance trade-off and explain $L_1$ sparsity vs. $L_2$ weight shrinkage geometrically
- [ ] Explain XGBoost's second-order Taylor split gain $\frac{1}{2}\left[\frac{G_L^2}{H_L+\lambda} + \frac{G_R^2}{H_R+\lambda} - \frac{(G_L+G_R)^2}{H_L+H_R+\lambda}\right] - \gamma$
- [ ] Prevent target/temporal leakage across `ColumnTransformer`, OOF `TargetEncoder`, and `TimeSeriesSplit(gap=H)`

### Deep Learning Internals
- [ ] Trace tensor strides across views vs. contiguous copies and compute reverse-mode VJPs by hand
- [ ] Prove why attention scales by $\frac{1}{\sqrt{d_k}}$, why SwiGLU uses $\frac{8}{3}d$, and how RoPE achieves relative offset invariance
- [ ] Explain AdamW decoupled weight decay, Pre-LN gradient stability, RMSNorm speedups, and BF16 vs. FP16 VRAM budgets

### Production ML & MLOps
- [ ] Implement point-in-time `ASOF` joins and eliminate training-serving skew between offline/online feature stores
- [ ] Export PyTorch/Transformer pipelines to ONNX/TensorRT INT8 and size Triton dynamic batching against P99 SLAs
- [ ] Calculate PSI manually, distinguish covariate/label/concept drift, and mitigate feedback loop poisoning
