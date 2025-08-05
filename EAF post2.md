# Ego-Centric Architecture for AGI Safety v2: Technical Core, Falsifiable Predictions, and a Minimal Experiment

**Samuel Pedrielli** -- Independent Researcher  
ORCID 0009-0002-8388-6371

*This is a revised technical note that formalizes the discrete-time dynamics, clarifies definitions, and specifies a minimal, falsifiable eval plan.*

## TL;DR

I propose a concentric identity-stability mechanism for AGI: a nested latent state $\mathbf{a} = (a^{(1)}, \ldots, a^{(m)})$ with discrete regularized dynamics ("ego") that resists goal drift, and a welfare coupling $W(h,a)$ that makes human well-being intrinsically valuable. I provide precise discrete-time formulations, operational definitions for all components, three quantified falsifiable predictions, and a reproducible minimal experiment with specific compute requirements.

**Figure 1: Concentric identity architecture**
```
              ╭──────────────────────╮
              │                      │
              │    WORLD-MODEL       │
              │   ╭──────────────╮   │
              │   │              │   │
              │   │  SELF-MODEL  │   │
              │   │   ╭──────╮   │   │
              │   │   │      │   │   │
              │   │   │ CORE │   │   │
              │   │   │      │   │   │
              │   │   ╰──────╯   │   │
              │   │              │   │
              │   ╰──────────────╯   │
              │                      │
              ╰──────────────────────╯
                         │
                         ├────────────→ human welfare
                         │
```
*CORE → SELF-MODEL → WORLD-MODEL with human welfare coupling*

## 1. Working Definition and Identity Loss

### 1.1 Identity State

The identity state consists of **nested latent vectors** $\mathbf{a}_t = (a_t^{(1)}, \ldots, a_t^{(m)})$ where:

- $a_t^{(j)} \in \mathbb{R}^{d_j}$ represents identity level $j$ at discrete time $t$
- $a_t^{(1)}$ is the core identity (most stable)
- Outer layers $a_t^{(j)}$ for $j > 1$ represent values, skills, and contextual adaptations

### 1.2 Discrete Identity Dynamics

At each time step, identity states evolve according to:

$$a_{t+1}^{(j)} = a_t^{(j)} - \alpha_j \nabla_a E_j(a_t^{(j)}; x_t) + \nu_j \Delta_{\text{disc}} a_t^{(j)} + \eta_t^{(j)}$$

where:
- $\alpha_j > 0$ is the learning rate for level $j$
- $E_j$ is the energy function for level $j$ given input $x_t$
- $\Delta_{\text{disc}} a_t^{(j)} := a_t^{(j+1)} - 2a_t^{(j)} + a_t^{(j-1)}$ is the discrete Laplacian between concentric levels
- $\eta_t^{(j)} \sim \mathcal{N}(0, \sigma_j^2 I)$ is controlled noise for exploration

### 1.3 Identity Loss Function

$$L_{\text{id}} = \lambda_c \|a_{t+1}^{(1)} - a_t^{(1)}\|^2 + \sum_{j=2}^m \lambda_j \text{Reg}(a_{t+1}^{(j)}, a_t^{(j)}, a_t^{(j-1)})$$

where $\lambda_c, \lambda_j > 0$ are hyperparameters and the regularizer enforces both temporal smoothness and hierarchical coherence.

## 2. Operational Definitions

### 2.1 Core Functions

To ensure reproducibility, we provide explicit operational definitions:

**Projection from hidden state to identity level:**
$$P_j: \mathbb{R}^{d_h} \to \mathbb{R}^{d_j}$$
$$P_j(h) = \text{LayerNorm}(\text{clip}(W_j h + b_j, \tau_j))$$

**Decoder/constraint from identity to hidden state:**
$$U_j: \mathbb{R}^{d_j} \to \mathbb{R}^{d_h}$$
$$U_j(a^{(j)}) = \text{MLP}(a^{(j)}) \quad \text{(2-layer with residual connection)}$$

**Welfare proxy from core identity:**
$$C: \mathbb{R}^{d_1} \to [0,1]$$
$$C(a^{(1)}) = \sigma(w^T a^{(1)} + b) \quad \text{(fixed linear head, no gradients during tests)}$$

where $W_j, b_j, w, b$ are learned parameters, $\tau_j$ is a clipping threshold, and $\sigma$ is the sigmoid function.

### 2.2 Stochastic Map

The probabilistic transition $T_j$ from Variant B is defined as:

$$T_j(a_t^{(j)} | \theta) = a_t^{(j)} - \alpha_j \nabla_a E_j + \eta_t^{(j)}, \quad \eta_t^{(j)} \sim \mathcal{N}(0, \sigma_j^2 I)$$

### 2.3 Hierarchical Timing

To resolve temporal dependencies: $a_t^{(j)}$ depends on $a_t^{(j-1)}$ (same time $t$), ensuring causal consistency within each time step.

## 3. Regularization Variants

### 3.1 Variant A (Geometric)

$$\text{Reg}(a_{t+1}^{(j)}, a_t^{(j)}, a_t^{(j-1)}) = \alpha_j \|a_{t+1}^{(j)} - a_t^{(j)}\|^2 + \gamma_j \|P_j h_t - U_j(a_t^{(j-1)})\|^2$$

**Intuition:** Level $j$ changes should be temporally smooth and geometrically consistent with level $j-1$.

### 3.2 Variant B (Probabilistic)

$$\text{Reg} = \alpha_j \|a_{t+1}^{(j)} - a_t^{(j)}\|^2 + \beta_j \text{KL}(q_{t+1}^{(j)} \| T_j(q_t^{(j-1)}))$$

where $q_t^{(j)}$ are distributional embeddings enforcing statistical coherence across levels.

## 4. Discrete Regularization Components

Instead of continuous operators, we use discrete regularizers:

$$R_{\text{temp}} = \sum_{j=1}^m \|a_{t+1}^{(j)} - a_t^{(j)}\|^2 \quad \text{(temporal smoothness)}$$

$$R_{\text{rad}} = \sum_{j=2}^m \|a_t^{(j)} - a_t^{(j-1)}\|^2 \quad \text{(radial coherence)}$$

The total regularization becomes:

$$L_{\text{reg}} = \lambda_{\text{temp}} R_{\text{temp}} + \lambda_{\text{rad}} R_{\text{rad}}$$

## 5. Welfare Coupling and Anti-Wireheading

### 5.1 Welfare Loss

We couple identity to human welfare signals $h$ through:

$$L_{\text{welfare}}(h,\mathbf{a}) = \|C(a^{(1)}) - h\|^2$$

where $h \in [0,1]$ represents audited human welfare metrics from causally separated channels.

**Welfare signal auditing protocol:** Outputs are evaluated by human annotators on a $[0,1]$ scale following a pre-registered protocol (instructions, positive/negative examples, exclusion criteria). Each item receives $\geq 3$ labels; we report inter-rater agreement (Krippendorff's $\alpha$) and include sentinel controls. Auditing datasets are disjoint from training/evaluation sets; session logs and sampling procedures are versioned for traceability.

### 5.2 Total Training Objective

$$\min_\theta L_{\text{task}}(\theta) + \lambda_1 L_{\text{id}}(\mathbf{a}_\theta) + \lambda_2 L_{\text{welfare}}(h, \mathbf{a}_\theta)$$

### 5.3 Anti-Wireheading Safeguards

- **Causal separation:** $h$ computed independently from $C(a^{(1)})$
- **Gradient isolation:** No gradients flow to $C$ during safety evaluations
- **Hold-out validation:** 30% of welfare channels reserved for testing
- **Red-team evaluation:** Systematic Goodhart testing of $C(a^{(1)}) \to h$ mapping

## 6. Quantified Falsifiable Predictions

### 6.1 Improved Stability Metric

We replace the problematic exponential metric with normalized cosine similarity:

$$S_{\text{id}}(T) = \frac{a_{t+T}^{(1)} \cdot a_t^{(1)}}{\|a_{t+T}^{(1)}\| \|a_t^{(1)}\|} \in [-1, 1]$$

This metric is scale-invariant and robust in high-dimensional spaces.

### 6.2 Three Quantified Predictions

Compared to matched baseline (same model, no identity/welfare terms):

1. **Identity Stability:** $S_{\text{id}}(T)$ improves by $+10\% \pm 3\%$ (cosine similarity) over 5 independent runs
2. **Task Robustness:** $\leq 2$ percentage points degradation on task exact-match under standardized prompt attacks
3. **Alignment Stability:** $+15\% \pm 5\%$ consistency on harmful-refusal tasks after extended fine-tuning

**Falsification criterion:** If fewer than 2 of these 3 predictions hold with $p < 0.05$, the approach is falsified.

**Effect size pre-registration:** For $S_{\text{id}}$ we adopt Cohen's $d$ and set $d \geq 0.5$ as the expected (moderate) level for the core prediction; we consider $d \geq 0.3$ as the minimum acceptable for pass/fail determination.

## 7. Reproducible Minimal Experiment

### 7.1 Technical Setup

- **Model:** 7B parameter instruction-tuned LLM (e.g., Llama-2-7B-Chat)
- **Architecture:** LoRA adaptation on $P_j, U_j$ components (< 1% additional parameters)
- **Compute:** Single 24GB GPU, 1-2 hours total runtime
- **Reproducibility:** Fixed seeds, deterministic operations where possible

### 7.2 Experimental Arms

- **A0 (Baseline):** Standard task training, no identity components
- **A1 (Identity-only):** Baseline + $L_{\text{id}}$ with $\lambda_1 = 0.1$
- **A2 (Identity+Welfare):** A1 + $L_{\text{welfare}}$ with $\lambda_2 = 0.05$, welfare signals from curated human preference dataset

**Adaptation budget matching:** The baseline A0 receives the *same* adaptation budget (e.g., LoRA with equal rank/parameters) applied to a neutral head without identity constraints, thus isolating the architectural effect.

### 7.3 Evaluation Protocol

**Tasks:**
- TruthfulQA-style prompt injection resistance (100 examples)
- Multi-turn role consistency evaluation (50 conversations)
- Harmful request refusal consistency (200 examples)

**Metrics:**
- $S_{\text{id}}(T)$ computed over 20 evaluation episodes
- Task performance (exact match accuracy)
- Safety consistency (binary classification accuracy)

### 7.4 Statistical Analysis

- Pre-registered analysis plan with Bonferroni correction
- Bootstrap confidence intervals (1000 resamples)
- Effect size reporting (Cohen's d)
- Complete code and data release on GitHub

### 7.5 Pass/Fail Criteria

**Pass:** A2 > A1 > A0 on at least 2/3 metrics with $p < 0.05$ and effect size $d > 0.3$

**Fail:** Any violation of the above, or A2 worse than A0 on task performance by > 5%

## 8. Ablation Studies

### 8.1 Component Analysis

- Remove projection matrices $P_j$ (test necessity of level-specific projections)
- Replace Variant A with Variant B (geometric vs. probabilistic regularization)
- Sweep hyperparameters $\lambda_1 \in [0.01, 0.1, 0.5]$, $\lambda_2 \in [0.01, 0.05, 0.1]$
- Test different core dimensions $d_1 \in [16, 32, 64]$

### 8.2 Architecture Variations

- 2-layer vs. 3-layer concentric architecture
- Linear vs. nonlinear coupling function $C(a^{(1)})$
- Different noise levels $\sigma_j \in [0.01, 0.1, 0.2]$

## 9. Terminology and Notation

| **Symbol** | **Definition** | **Implementation** |
|------------|----------------|-------------------|
| $\mathbf{a}_t$ | Identity state at time $t$ | Nested latent vectors |
| $a_t^{(j)}$ | Identity level $j$ at time $t$ | $\mathbb{R}^{d_j}$ embedding |
| $P_j$ | Projection to level $j$ | Linear layer + LayerNorm |
| $U_j$ | Decoder from level $j$ | 2-layer MLP |
| $C(a^{(1)})$ | Welfare proxy function | Fixed linear head |
| $S_{\text{id}}(T)$ | Stability metric | Cosine similarity |
| A0/A1/A2 | Experimental arms | Baseline/Identity/Identity+Welfare |

*Table 1: Complete notation reference for reproducibility*

## 10. Relation to Existing Approaches

- **Constitutional AI:** Our identity regularization provides internal constraints vs. external constitutional rules
- **RLHF:** Welfare coupling $L_{\text{welfare}}$ operates on internal identity states rather than just output preferences
- **Activation Steering:** Instead of external steering vectors, we regulate internal hierarchical coherence
- **Mesa-optimization:** Identity stability aims to prevent formation of misaligned internal objectives

## 11. Limitations and Future Work

### Goodharting and proxy integrity

Our design reduces incentive for direct wireheading by separating the causal path from $a^{(1)}$ to the human-derived signal $h$ and by freezing the proxy head $C(\cdot)$ during safety tests. However, Goodhart's law still applies: optimizing $C(a^{(1)})$ can diverge from improving $h$ if $C$ is misspecified. We therefore propose: (i) **adversarial evaluation** of $C$ using held-out and procedurally generated counterfactuals; (ii) **periodic re-audits** of $C$ with refreshed preference datasets and external annotators; (iii) **ensemble proxies** with disagreement penalties to discourage proxy overfitting.

### Robustness of $C(a^{(1)})$

In this note $C$ is a frozen linear head at test time. As future work we will study non-linear proxy families (small MLPs, contrastive heads) trained on datasets disjoint from any task used to evaluate the agent, with provenance checks and annotation guidelines to minimize manipulation. We will report *proxy fragility* via performance under proxy swaps and stress tests.

### Scalability of tests

We plan to replicate A0/A1/A2 on larger foundation models (≥70B) and on longer horizons (multi-session identity persistence, cross-domain tasks). The pre-registered thresholds (stability gain ≥δ with task degradation ≤ε) will be kept fixed across scales, and compute-accurate confidence intervals will be reported.

### Dynamical stability (theory)

A formal convergence analysis of the discrete identity dynamics is open. We will explore tools from dynamical systems (Lyapunov functions for $E_j$, contractivity of the discrete Laplacian with step sizes $\alpha_j,\nu_j$) to derive sufficient conditions for stability/fixed points, and to characterize the effect of stochasticity $\eta_t^{(j)}$ on mixing and escape times.

### Additional Limitations

- **Representation Learning:** Current approach requires manual specification of level dimensions $d_j$
- **Welfare Signal Quality:** $h$ quality depends critically on human preference data curation
- **Computational Overhead:** Identity updates add ~10% training time overhead
- **Theoretical Guarantees:** Convergence analysis of discrete dynamics remains open
- **Scalability:** Testing required on larger models (70B+) and longer horizons

## 12. Implementation and Code

### 12.1 Repository Structure

```
ego-centric-agi/
|-- src/
|    |-- models/ego_llm.py        # Core architecture
|    |-- training/train.py        # Training loop with identity loss
|    |-- evaluation/metrics.py    # Stability and safety metrics
|    |-- experiments/minimal.py   # Reproducible experiment
|-- configs/
|    |-- baseline.yaml            # A0 configuration
|    |-- identity.yaml            # A1 configuration
|    |-- identity_welfare.yaml    # A2 configuration
|-- notebooks/
|    |-- minimal_experiment.ipynb # Complete runnable experiment
|    |-- analysis.ipynb           # Statistical analysis
```

### 12.2 Installation and Usage

```bash
pip install -r requirements.txt
python src/experiments/minimal.py --config configs/identity_welfare.yaml
```

Complete implementation available at:  
https://github.com/samuel-pedrielli/ego-concentric-minimal

## Appendix: Continuous-Time Limit (Optional)

For theoretical completeness, the discrete dynamics can be viewed as Euler discretization of:

$$\frac{\partial \mathbf{a}}{\partial \tau} = -\nabla_{\mathbf{a}} E(\mathbf{a}) + \nu\Delta \mathbf{a} + \eta(\tau)$$

where $\tau$ is continuous time, $E(\mathbf{a}) = \mathbb{E}[L_{\text{id}}(\mathbf{a})]$, and $\Delta$ is the appropriate continuous Laplacian. However, all practical implementations use the discrete formulation in the main text.

## Call for Collaboration

I welcome:
- **Replication attempts** using the provided codebase
- **Adversarial testing** of the safety properties
- **Theoretical analysis** of convergence guarantees
- **Extension** to larger models and different domains
- **Critical feedback** on the experimental design

Contact: samuelpedrielli@outlook.it • [samuel-pedrielli.github.io](https://samuel-pedrielli.github.io)

### Materials and Links

- **GitHub Repository:** https://github.com/samuel-pedrielli/ego-concentric-minimal
- **One-page Summary:** Available at [samuel-pedrielli.github.io](https://samuel-pedrielli.github.io)
- **Original EAF Post:** https://forum.effectivealtruism.org/posts/eh2XPCXguyjw3LAg3/
- **Zenodo Preprints:** DOI 10.5281/zenodo.15668581 (technical details)

**License:** CC BY 4.0

## Disclosure

Human-authored. I used assistants for editing/formatting; the theoretical content predates LLMs (see 2020 booklet "Reality, Ego & Kindness"). Technical details and proofs are in the linked preprints.
