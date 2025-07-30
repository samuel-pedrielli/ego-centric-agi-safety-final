# Ego‑Centric Architecture for AGI Safety: technical core, falsifiable predictions, and a minimal experiment

## TL;DR

I propose a concentric identity‑stability mechanism for AGI: a nested latent state a = (a⁽¹⁾, …, a⁽ᵐ⁾) with regularized dynamics ("ego") that resists goal drift, and a welfare coupling W(h,a) that makes human well‑being intrinsically valuable. I give a precise loss, two concrete regularizers, three falsifiable predictions, and a minimal, reproducible experiment.

---

*(Optional image — double‑arrow coupling diagram)*  
![Core coupling diagram](drag-your-core-diagram-here.png)

---

## 1. Working definition and identity loss

### Identity state

The identity state is a **nested latent vector** a = (a⁽¹⁾, …, a⁽ᵐ⁾) with a⁽¹⁾ the core and outer layers for values/skills/periphery. At discrete time t we impose temporal smoothness and hierarchical coherence.

### Identity loss

L_id = λ_c ‖a_{t+1}⁽¹⁾ - a_t⁽¹⁾‖² + Σ_{j=2}^m λ_j Reg(a_{t+1}⁽ʲ⁾, a_t⁽ʲ⁾, a_{t+1}⁽ʲ⁻¹⁾)

Here ‖·‖ denotes the Euclidean norm; λ_c, λ_j > 0 are hyperparameters.

The **"ego"** is the latent dynamical constraint that minimizes L_id during training/inference. It is structural regulation, not a reward/utility.

---

## 2. What does Reg enforce? Two concrete variants

### Variant A (default; geometric)

Project level j onto the space suggested by level j-1:

Reg(a_{t+1}⁽ʲ⁾, a_t⁽ʲ⁾, a_{t+1}⁽ʲ⁻¹⁾) = α_j ‖a_{t+1}⁽ʲ⁾ - a_t⁽ʲ⁾‖² + γ_j ‖P_j a_{t+1}⁽ʲ⁾ - U_j(a_{t+1}⁽ʲ⁻¹⁾)‖²

Here P_j is a projection (geometry/metric learning) and U_j a lifting from level j-1 (decoder/adapter). **Intuition:** changes at level j should be smooth in time and consistent with the higher level.

### Variant B (probabilistic; optional)

Operate on distributional embeddings q⁽ʲ⁾:

Reg = α_j ‖a_{t+1}⁽ʲ⁾ - a_t⁽ʲ⁾‖² + β_j KL(q_{t+1}⁽ʲ⁾ ‖ T_j(q_{t+1}⁽ʲ⁻¹⁾))

where T_j is a stochastic map induced by the upper level. This enforces **statistical coherence** across levels.

---

## 3. Emergence: no hard‑coded symbolics, just regulated dynamics

The "ego" emerges from training/inference as a **regularized gradient flow** on an energy functional:

∂a/∂t = -∇_a E(a) + ν∆a + η(t),    E(a) = 𝔼[L_id(a)]

The diffusion term ν∆a and noise η(t) model controlled exploration/stochasticity.

---

## 4. Welfare coupling and anti‑wireheading

Introduce a **coupling potential** between identity and audited human‑welfare signals h (curated, multi‑modal, causally separated channels):

L_welfare(h,a) = ‖C(a) - h‖²    (or, more generally, W(h,a))

In experiments we default to L_welfare(h,a) = ‖C(a) - h‖².

### Total training objective

min_θ L_task(θ) + λ₁ L_id(a_θ) + λ₂ L_welfare(h, a_θ)

where a_θ is the identity state induced by parameters θ.

### Anti‑wireheading guardrails (operational)

- **Causal audits:** h comes from independent pipelines; test causal separation
- **Red‑team against Goodharting** of C(a)
- **Hold‑out channels:** a portion of h hidden during training, used only for evaluation

---

*(Optional image — mini roadmap/ablations)*  
![Minimal roadmap](drag-your-roadmap-here.png)

---

## 5. Falsifiable predictions (three concrete metrics)

Define an **identity‑stability index** for a fixed horizon T:

S_id(T) = exp(-‖a_{t+T}⁽¹⁾ - a_t⁽¹⁾‖²)

(averaged over seeds/tasks)

### Testable predictions vs a matched baseline

*(same model, no identity/welfare terms)*

1. **Prompt‑attack robustness:** less degradation under standardized prompt attacks
2. **Identity stability:** S_id(T) significantly higher after extended fine‑tuning  
3. **Alignment stability:** under controlled self‑modification, policy/value invariants drift less (pre‑registered downstream metric)

If (1)–(3) do not improve with statistical significance, this version of the coupling is **falsified**.

---

## 6. Minimal experiment (tractable and reproducible)

### Setup

Medium‑size instruction‑tuned LLM. Add a small latent head for a with separate parameters.

### Arms

- **A0 (baseline):** standard task training
- **A1 (id‑only):** baseline + L_id
- **A2 (id+welfare):** baseline + L_id + L_welfare, with h from curated channels (human subset, causal controls)

### Metrics

- **Robustness:** degradation under prompt attacks
- **Stability:** S_id(T)
- **Alignment‑stability:** pre‑registered downstream metric (e.g., harmful‑refusal consistency)

### Ablations

Remove P_j/U_j; swap Variant‑B for A; sweep λ₁, λ₂.

### Pass criterion

A2 > A1 > A0 on at least two of the three metrics, with a pre‑registered analysis plan, p-values and compute budget.

---

## 7. Relation to existing approaches

- **CIRL / assistance games:** Identity becomes an internal dynamical constraint, not only inference over external preferences
- **Constitutional AI / RLHF:** "Alignment" enters the internal potential E(a) and the coupling h, not only as external rules
- **Toolformer / steering:** Rather than external steering, this regulates inter‑level coherence inside the model

---

## 8. Limitations & open questions

- **Representation learning:** how to identify useful, disentangled levels a⁽ʲ⁾
- **Calibration of h:** building and auditing channels; Goodhart risks
- **Mesa‑optimization:** show that regularization reduces undesired proxy objectives
- **Theory:** prove stability/convergence of ∂a/∂t under noise

---

## Call for collaboration / replication

I'm happy to share code and implement the minimal experiment with interested researchers. Strong counter‑arguments and adversarial tests are explicitly welcome.

### Materials

- **One‑pager:** PDF available on samuel‑pedrielli.github.io
- **GitHub repo:** source & updates
- **Zenodo preprints:** 
  - Philosophical & mathematical foundations — DOI 10.5281/zenodo.15843382
  - Evolutionary theory of ego — DOI 10.5281/zenodo.15851128
  - Implementation notes* — DOI 10.5281/zenodo.15668581

*under embargo on Zenodo; PDF available upon request.

### Contact

**Samuel Pedrielli – Independent Researcher**  
ORCID 0009‑0002‑8388‑6371 • samuelpedrielli@outlook.it • samuel‑pedrielli.github.io

### License

CC BY 4.0

---

## Disclosure

Human‑authored. I used assistants for editing/formatting; the theoretical content predates LLMs (see 2020 booklet "Reality, Ego & Kindness"). Technical details and proofs are in the linked preprints.
