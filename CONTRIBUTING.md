# Q-SHIELD: Post-Quantum TLS Anomaly and Downgrade Detection
### IBM Fall Fest 2026 | Trinexis x IEEE | Domain: Banking / Cybersecurity | Participant Guide v2 (DRAFT FOR REVIEW)

> `[ORGANIZER TO CONFIRM]` marks event details not yet fixed (dates, submission portal, quota policy, scoring weights).

---

## 1. Problem statement

### 1.1 Context
Banks are moving TLS from classical cryptography (X25519 key exchange, RSA/ECDSA certificates) to post-quantum cryptography (ML-KEM key exchange, ML-DSA signatures) and to hybrid modes that mix both. During this migration a security operations centre (SOC) must tell apart:

1. **Normal** handshakes from many different, equally legitimate crypto configurations.
2. **Misconfigurations and downgrades** (for example post-quantum offered over TLS 1.2, or a key-exchange mismatch).
3. **Attacks and degraded networks** (classical-traffic mimicry, data exfiltration, high loss, latency or jitter).

A detector trained only on today's configurations will meet new, perfectly legitimate configurations tomorrow. If it flags them all, analysts drown in false alerts and stop trusting it.

### 1.2 Your task
Build and compare three detectors (one classical, two hybrid quantum-classical) that answer:

> Can a detector separate malicious or malformed PQC-TLS handshakes from unusual-but-legitimate connections, while keeping false alerts low on **cryptographic configurations it has never seen**?

You will run the quantum parts on a **real IBM quantum computer** through IBM Cloud, within about **10 minutes of QPU time**, using a small cohort of sessions.

### 1.3 Ground rules
- **No quantum simulation, ever.** Forbidden: `qiskit_aer`, `Aer`, `AerSimulator`, `Statevector*`, `BasicSimulator`, `Fake*Backend`, `QasmSimulator`, tensor-network or noisy simulators, mocked or hand-typed counts, synthetic TLS data, cached demo outputs.
- Every quantum number must come from measurements returned by a real IBM backend. A classical optimizer may *propose* circuit parameters; it may never *invent* a circuit's result.
- Local CPU is allowed for pandas, scikit-learn, XGBoost, feature ranking, QUBO construction, transpilation and plotting.
- If something is unavailable (Kaggle access, IBM token, quota), stop that phase, mark it `BLOCKED_<REASON>`, and say so. Never fill the gap with fake data.
- Never put API keys in files or notebooks. Use environment variables.

---

## 2. Expected outcome

### 2.1 What you must submit
1. **One executed Jupyter notebook** (`.ipynb`, outputs saved, runs top to bottom with no errors).
2. **Evidence files** (Section 8): job IDs, raw counts, metric tables, figures.
3. **A findings section** at the end of the notebook stating which method you would choose and why, backed by your own numbers, including negative results.

### 2.2 What your solution must demonstrate
| # | Outcome | How it is judged |
|---|---|---|
| 1 | A leakage-safe data pipeline with a frozen, reproducible split | Printed overlap checks, logged hashes, documented exclusions |
| 2 | One classical ML model evaluated on all three evaluations (Section 3.5) | Metrics with confidence intervals |
| 3 | Two hybrid quantum models (kernel method and QAOA+VQC) on real hardware | Job IDs, raw counts, QPU seconds |
| 4 | A fair three-way comparison on identical sessions | Same IDs, same threshold policy, same metrics |
| 5 | A circuit **before/after** comparison on the same backend, rows and shots | Transpiled depth and two-qubit gate counts, paired predictions |
| 6 | Honest interpretation of results | Insight text matches the printed numbers; no "improved" claims the numbers do not support |

### 2.3 Target to aim for
On evaluation C (unseen legitimate crypto families, Section 3.5) keep **false alerts at or below 10 per 1,000 benign sessions** (FPR <= 1 %), while recalling as many anomalies as possible on evaluations A and B. A model that scores near 100 % on familiar traffic but flags new legitimate families is **not** a good detector for this task.

---

## 3. Data

### 3.1 Source (the only allowed source)
- Kaggle: `https://www.kaggle.com/datasets/mdabrarhossainzahin/cic-pqc-oav-2025` (slug `mdabrarhossainzahin/cic-pqc-oav-2025`), MIT licence.
- Documentation only (not a data source): `https://www.unb.ca/cic/datasets/pqc-oav-dataset-2025.html`.
- Download with `kagglehub.dataset_download(...)` (you may need Kaggle credentials). Log each file's size and SHA-256.
- The data is emulated lab traffic: no personal data, no real bank traffic.

### 3.2 Files
| File | Rows | Content |
|---|---|---|
| `ml_features_and_labels.csv` | 40,010 | Engineered handshake features and label |
| `scenario_manifest.csv` | 40,010 | Scenario name, class, crypto family per session |

Discover tables by their real columns, not by filename. Join the two tables on a key you validate (one-to-one, no duplicates). If your observed row count differs from 40,010, report what you observe.

### 3.3 Label and scenarios
- `label = 1` is **anomaly**, `0` is normal. Roughly 12.5 % of sessions are anomalies, so accuracy is misleading; use the metrics in Section 6.
- 21 scenarios: 10 normal crypto configurations and 11 anomaly scenarios.

| Group | Scenario ids |
|---|---|
| Normal, classical | `normal_X25519_ecdsa_p256_cert`, `normal_X25519_ecdsa_p384_cert`, `normal_X25519_rsa3072_cert`, `normal_X25519_rsa_cert` |
| Normal, ML-KEM-768 | `normal_mlkem768_mldsa65_cert`, `normal_mlkem768_mldsa87_cert`, `normal_mlkem768_rsa_cert` |
| Normal, ML-KEM-1024 | `normal_mlkem1024_mldsa65_cert`, `normal_mlkem1024_mldsa87_cert`, `normal_mlkem1024_rsa_cert` |
| Anomaly | `anomaly_hybrid_pki`, `anomaly_kem_mismatch`, `anomaly_pqc_over_tls12`, `model_gray_rsa_mimicry`, `net_high_jitter_classic`, `net_high_latency_pqc`, `net_high_loss_pqc`, `raf_alg_mismatch`, `raf_downgrade_overt`, `raf_robustness_malformed_keyshare`, `threat_data_exfil` |

### 3.4 Features: what you may and may not use
Group every column before modelling and publish the table with reasons.

| Group | Rule |
|---|---|
| **Behaviour features** (numeric handshake measurements: sizes, entropies, timings): `e2c_total_bytes`, `e4_entropy_h`, `e5_entropy_c`, `e6_time_char`, `e6b_flow_duration_ms`, `e2_client_size`, `e2_client_record_len`, `e2_server_record_len` | **Primary feature list.** `[ORGANIZER TO CONFIRM: add official one-line definitions from the UNB page]` |
| Outcome / protocol / algorithm-suite flags (`e9_*`, `e8_*`, `e1_*`, `e1b_*`, `e2b_*`, `e3_*`) | Optional secondary comparison only. Some restate the label or the crypto configuration. |
| Label-derived or metadata columns (`label`, `taxonomy*`, `scenario_id`, `operational_class`, `subtype`, `ID`, `split`) | **Never** model inputs. Use for grouping and reporting only. |

The dataset's own `split` column is not safe to use (see 3.6). Build your own.

### 3.5 The three evaluations you must run
Hold out the following **before any modelling**. They are part of the challenge protocol so that all teams are comparable.

| ID | Evaluation | Held-out content | Primary metric |
|---|---|---|---|
| **A** | Duplicate-safe in-distribution test | A test partition of the remaining (seen) scenarios | Recall at FPR <= 1 %, PR-AUC |
| **B** | Unseen anomaly scenarios | All sessions of `anomaly_hybrid_pki`, `net_high_loss_pqc`, `raf_downgrade_overt`, scored with the same threshold as A | Recall |
| **C** | Unseen legitimate crypto families | All sessions of `normal_X25519_ecdsa_p256_cert` and `normal_mlkem768_rsa_cert` (normal only) | **False alerts per 1,000 benign sessions** |

Train, validation and the A test partition use only the remaining scenarios. B and C scenarios must not appear in training, validation or threshold selection.

### 3.6 Splitting and preprocessing rules
1. **Fingerprint first.** Hash the canonicalised behaviour feature values (plus label for audit). Count identical-feature clusters and clusters whose rows have different labels. Quarantine contradictory clusters; do not silently drop them.
2. **Group split.** Split by fingerprint so identical rows never sit on both sides. Use a fixed seed (`42` recommended). Suggested: stratified group 5-fold; fold 0 test, fold 1 validation, folds 2-4 train.
3. **Print the checks:** rows, classes and scenarios per partition, and the fingerprint overlap between every pair of partitions (must be 0).
4. **Fit on training only:** median imputation, clipping bounds (for example 1st/99th percentile), scaling to an angle range for quantum models. Never fit on validation or test.
5. **Missing values.** Check whether missingness correlates with the label before deciding how to treat it, and say what you decided.
6. **Threshold selection:** choose the operating threshold on validation data only (target FPR <= 1 %). Never tune on test, B or C.
7. Save the split recipe, seed and row IDs so a reviewer can reproduce it.

### 3.7 Small quantum cohort (because QPU time is limited)
Quantum models cannot afford the full data. Draw a deterministic, class-balanced, fingerprint-disjoint sample from your frozen partitions:

| Cohort | Sessions | Anomaly / Normal | Use |
|---|---|---|---|
| Quantum train | 8 | 4 / 4 | Kernel fit, VQC training, QAOA feature ranking |
| Quantum validation | 4 | 2 / 2 | VQC monitoring |
| Quantum test | 8 | 4 / 4 | Matched comparison of all methods |

Sizes may go **down** if your quota requires, never up silently. Save the chosen session IDs. Every method in the comparison must be scored on the **same quantum test session IDs**. Never compare full-data numbers with 8-session numbers as if they were equivalent.

---

## 4. Methods to implement and compare

| # | Method | Type | Open-source stack |
|---|---|---|---|
| **M1** | **XGBoost** | Classical ML | `xgboost`, `scikit-learn` |
| **M2** | **Quantum-kernel SVM (QSVC)** | Hybrid, kernel | `qiskit`, `qiskit-ibm-runtime` (`SamplerV2`), `sklearn.svm.SVC(kernel="precomputed")` |
| **M3** | **QAOA feature selection, then Variational Quantum Classifier (VQC)** | Hybrid, variational | `qiskit`, `qiskit-ibm-runtime`, SPSA (or similar) written by you |

You may add extra baselines, but M1, M2 and M3 are mandatory.

### 4.1 M1: XGBoost
- Train on the behaviour features, training partition only. Tune a small, pre-declared grid on validation.
- Report evaluations A, B and C on the full data, then an **exactly matched control** trained on the *same two features and same 8 training sessions* as the quantum models and tested on the *same 8 test sessions*.
- Run a sensitivity analysis: which features does the model rely on, and what happens to evaluation C if you remove them?
- Time single-record CPU inference (p50 and p95).

### 4.2 M2: QSVC on hardware
1. Define a **measurement-free** 2-qubit feature map `U(x)` using hardware-friendly gates (`ry`, `rz`, one entangler). Features are angle-encoded in a bounded range such as [-pi, pi].
2. For every pair of sessions build the overlap circuit `U(x_j)` followed by `U(x_i)^dagger`, measure both qubits. The kernel entry is `K_ij = count("00") / shots`, **measured on the QPU**.
3. Measure the training Gram matrix (upper triangle, symmetric) and the test-vs-train matrix. Batch circuits into as few jobs as possible and save the row/column mapping.
4. Check symmetry, diagonal and eigenvalues. If the measured matrix is not positive semidefinite, apply a documented eigenvalue clip to the **training** kernel only, with a consistent out-of-sample transform. Never use test labels.
5. Fit `SVC(kernel="precomputed")`. SVM decision scores are not probabilities: rank with them, do not report Brier score for them.
6. Required comparison: a **basic** map versus an **improved** map (your design), both measured on hardware.

### 4.3 M3: QAOA, then VQC
**QAOA feature selection (4 qubits).**
1. On training data, rank behaviour features with a cheap classical score (for example mutual information) and take **4 candidates**.
2. Build a QUBO that picks exactly 2 of the 4:
   `E(z) = -sum_i r_i z_i + lambda (sum_i z_i - 2)^2 + mu sum_{i<j} |corr_ij| z_i z_j`
   with relevance `r_i` normalised on training data. Choose and log `lambda`, `mu`. Convert to Ising coefficients analytically (`z = (1 - Z)/2`) and verify the conversion.
3. Build a depth-1 QAOA circuit (Hadamards, `rz` and `rzz` cost layer, `rx` mixer, measurement). State your bit-order convention explicitly (Qiskit counts are little-endian).
4. Run a small grid of 4 to 6 (gamma, beta) pairs on hardware at 64 or 128 shots in **one batched job**. From measured counts compute the mean QUBO energy and the fraction of valid 2-of-4 masks. Choose (gamma, beta) on training data, then select the most frequent valid mask.
5. Brute-force the exact optimum on CPU **as a labelled reference only** (`exact_reference != QAOA`). Report the gap and the valid-mask rate honestly.

**VQC on hardware (2 qubits, at most 4 trainable angles).**
1. Encode the 2 selected features as rotation angles; add a small trainable ansatz.
2. Class probability = fraction of shots whose bitstring has odd parity. Fix and document the readout convention.
3. Train with hardware-in-the-loop SPSA (or a tiny bounded grid) for 2 to 3+ iterations. **Every loss value must come from QPU measurements.** Use cross-entropy with clipped probabilities (epsilon 1e-4). Log loss versus number of QPU calls.
4. Do not use Qiskit Machine Learning's default `VQC` or `SamplerQNN` unless you can prove its sampler is a real-hardware one (the defaults are local simulators).

### 4.4 Recommended configuration (use these settings so submissions are comparable)
Keep them in one `config` cell at the top of the notebook. Deviate only **downward** and disclose every change and why.

| Parameter | Value | Meaning |
|---|---|---|
| `SEED` | 42 | Global random seed (splits, sampling, optimizer) |
| `FPR_BUDGET` | 0.01 | Alert-threshold target on the large classical validation set |
| `QTRAIN` / `QVAL` / `QTEST` | 8 / 4 / 8 | Quantum cohort sizes |
| `SHOTS` | 128 | Shots per circuit |
| `QAOA_CANDIDATES` / `QAOA_SELECTED` | 4 / 2 | Features entering and leaving QAOA |
| QAOA grid | 4 to 6 (gamma, beta) pairs, for example gamma in {0.3, 0.6, 0.9} x beta in {0.3, 0.6} | Hardware sweep |
| `SPSA_ITERATIONS` | 3 or more | Hardware-in-the-loop VQC training |
| VQC trainable angles | at most 4 | Keeps the circuit shallow |
| Loss clipping epsilon | 1e-4 | Cross-entropy on measured probabilities |
| Qubits | 2 for QSVC and VQC, 4 for QAOA | Hardware-friendly |
| `QPU_SOFT_BUDGET_SECONDS` | 420 | Planned usage; keep the rest as retry reserve |
| `QPU_HARD_BUDGET_SECONDS` | min(remaining quota, 600) | Never exceed |
| `RUN_REAL_QPU` / `LIVE_JOB_ACK` | `False` by default | Set both to `True` deliberately before any submission |
| Preset transpilation | `generate_preset_pass_manager(optimization_level=1, backend=backend)` | ISA circuits for the chosen device |

Hardware access skeleton (adapt and test against your installed SDK; this shows the pattern, not a result):
```python
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2
service = QiskitRuntimeService()                      # token from environment or saved account
backend = service.least_busy(operational=True, simulator=False, min_num_qubits=4)
assert not getattr(backend, "simulator", False)       # simulators are prohibited
sampler = SamplerV2(mode=backend)
job = sampler.run(isa_circuits, shots=SHOTS)
save_job_id(job.job_id())                             # persist BEFORE waiting
result = job.result()                                 # counts come from result[i].data.<register>.get_counts()
```
Use a named classical register for measurement and read counts from that register name.

---

## 5. Circuit before / after on real hardware

### 5.1 What to compare
Run a **BEFORE** (baseline) and an **AFTER** (your improved) version on the **same IBM backend, same quantum test session IDs, same shots, same task.**

| Item | BEFORE | AFTER |
|---|---|---|
| VQC | Basic 2-qubit feature encoding and ansatz | Your improved encoding and/or ansatz (for example different entangler, data re-uploading) and the QAOA-selected features |
| QSVC | Basic feature map | Your improved feature map |

If features and circuit both change, label the result a **pipeline-level** comparison. To attribute the effect, add an **ablation**: the improved circuit on the original (non-QAOA) features. If AFTER is worse, keep and explain the negative result.

### 5.2 Evidence required per comparison
Original and transpiled circuit diagrams, depth, two-qubit gate count, qubit layout, circuit hash, per-session measured probabilities, thresholded label, shot (binomial) uncertainty, per-session BEFORE-vs-AFTER change, and a confusion matrix for each.

### 5.3 Working inside about 10 minutes of QPU time
- **QPU time is not wall-clock.** The 10 minutes counts execution on the processor; queue time is extra. Plan for waiting.
- Cost is roughly a fixed amount per job plus a small amount per circuit, so **batch many circuits into one job**. Check `job.usage()` after each job and `service.usage()` before you start.
- Suggested plan: health check (optional) -> QAOA sweep -> QSVC training kernel -> QSVC test kernel -> VQC training (one job per SPSA iteration) -> BEFORE/AFTER inference. Keep at least 20 % of the budget in reserve for retries.
- Pick the backend with `service.least_busy(operational=True, simulator=False, min_num_qubits=4)` (or name one explicitly) and **assert it is not a simulator**.
- Transpile with a preset pass manager for that backend (fixed `seed_transpiler`). Every circuit must be fully bound and measured before submission.
- **Choose physical qubits deliberately.** Do not leave placement to chance: pick the qubit pair (2-qubit circuits) and qubit set (4-qubit QAOA) from the backend's calibration data and/or a short readout-health job (idle circuits should read all zeros, X-flipped circuits all ones), then pass them as `initial_layout`. Record the layout used per job. A poor qubit pair can dominate your results.
- **Save the job ID immediately** after submission, before waiting. If your notebook restarts, retrieve with `service.job(job_id)` instead of resubmitting.
- Validate each result: number of shots returned equals requested, one result per circuit, session mapping preserved. Fail closed on cancelled or timed-out jobs.
- Stop and mark `BUDGET_EXHAUSTED` rather than overspend.

---

## 6. Benchmark metrics (report all that apply)

| Metric | Definition / note |
|---|---|
| TP, FP, TN, FN | From true labels and predictions at the stated threshold |
| Recall | `TP / (TP + FN)` |
| FPR | `FP / (FP + TN)` with **Wilson 95 % interval** |
| False alerts per 1,000 benign | `1000 * FP / (FP + TN)` (benign sessions only in the denominator) |
| Precision, F1, balanced accuracy | Standard |
| PR-AUC | `average_precision_score` on the ranking score; report `not estimable` if a class is missing, never 0 |
| ROC-AUC | Same rule |
| Brier score | Only for genuinely calibrated probabilities (not SVM scores) |
| Latency | CPU p50/p95 per record for M1; QPU wall-clock per job separately for M2/M3 |
| Cost | QPU seconds per phase from IBM's usage figure; circuit depth; two-qubit gates |

With 4 benign sessions, a 1 % FPR is not measurable: write `NOT_STATISTICALLY_RESOLVABLE` and use the interval. Never claim "1 % FPR achieved" from a tiny cohort.

**Choosing the best method.** State your rule before looking at results: first eligibility (real hardware provenance, no leakage), then highest recall under an FPR ceiling on a sample large enough to test it. If the ceiling cannot be tested for the quantum models, name an exploratory winner or declare the comparison inconclusive. Never imply quantum advantage without evidence.

---

## 7. Suggested notebook structure

You build this notebook. Every **code cell** must be preceded by a Markdown cell with six fields: **Purpose | Inputs | Method (with the maths) | Expected output | Interpretation | Failure mode**. Every **output** must be followed by an **Insights** cell written from the numbers actually printed.

| Section | Content | Done when |
|---|---|---|
| 00 Cover | Problem, data source, allowed compute, status flags | Contract stated |
| 01 Environment | Package versions, seed, no-simulator scan | Versions printed; scan passes |
| 02 Download | Kaggle download, file hashes, licence | Hashes logged |
| 03 Schema | Tables, columns, types, join key | One-to-one join verified |
| 04 Label audit | Class balance, nulls, infinities, scenarios | Counts printed |
| 05-06 EDA | Class balance, scenarios, distributions by label and family, null map, correlation, box plots | Figures saved |
| 07 Leakage | Feature register with reasons | Excluded columns listed |
| 08 Duplicates | Fingerprint clusters and label collisions | Quarantine decision logged |
| 09 Splits | Frozen train/val/test, B and C hold-outs | Overlaps all 0 |
| 10 Preprocessing | Train-only imputation, clipping, scaling | Out-of-range shares reported |
| 11 Metrics | Utilities with unit checks | Safe handling of missing classes |
| 12 M1 XGBoost | Full-data A/B/C, ROC/PR, confusion, calibration, sensitivity | CIs and figures |
| 13 Quantum cohort | Freeze 8/4/8 session IDs | IDs saved |
| 14 Hardware access | Authenticate, real backend, quota | Backend is not a simulator |
| 15-16 QAOA | QUBO, hardware sweep, selected features | Valid-mask rate and gap reported |
| 17 Matched control | XGBoost on the same 2 features and 8 sessions | Metrics saved |
| 18-19 M2 QSVC | Hardware kernels, PSD check, SVC fit | Heatmaps, eigenvalues, predictions |
| 20-23 M3 VQC | BEFORE and AFTER circuits, hardware SPSA, matched inference, ablation | Loss curves, paired predictions |
| 24 Job recovery | Re-query every job ID | All completed; counts match shots |
| 25-27 Benchmark | Side-by-side table and charts, unseen-family chart | Same IDs, same metrics |
| 28 Latency | CPU vs QPU response time | p50/p95 shown |
| 29 Findings | Chosen method, claims, threats to validity, next steps | Every claim has a number |

Required figures: class prevalence, feature distributions, null map, correlation heatmap, split sizes, ROC and PR curves, confusion matrices, QUBO and QAOA energy sweep, selected masks, kernel heatmaps, VQC loss versus QPU calls, before/after circuit diagrams (logical and transpiled), measured count distributions, per-session probabilities with intervals, unseen-family false-alert chart, latency versus cost.

---

## 8. Submission checklist

**Files:** executed notebook | `qpu_jobs.json` (tag, job ID, backend, shots, circuit hash, QPU seconds, timestamps) | raw counts | split manifest (seed, IDs) | metrics CSVs | figures | `requirements-lock.txt` | your findings summary. No tokens, CRNs or credentials.

**Before you submit, confirm:**
- [ ] Data only from the Kaggle slug; hashes logged
- [ ] Fingerprint overlap between all partitions is 0; B and C scenarios absent from training and thresholding
- [ ] Threshold chosen on validation only
- [ ] A, B, C reported for M1, with intervals
- [ ] M2 and M3 ran on a real backend; every job ID retrievable; no simulator imports
- [ ] QPU use within budget, from IBM's usage figures
- [ ] Before/after on identical session IDs, shots and backend, with an ablation
- [ ] All methods compared on the same quantum test sessions
- [ ] Notebook has zero error outputs and saved outputs
- [ ] Every code cell has a six-field explanation above and an insight below
- [ ] Negative and inconclusive results reported as measured

**Submission route:** `[ORGANIZER TO CONFIRM: fork-and-pull-request flow, repository URL, deadline, portal]`.

---

## 9. Code quality standard
- Python 3.10+ (3.12 recommended); `qiskit` 2.x and a current `qiskit-ibm-runtime`; pin versions in `requirements-lock.txt` after verifying they work together. Do not install `qiskit-aer`.
- Format with `black -l 120`; lint with `ruff`; no unused imports, bare `except`, `TODO`s or placeholder cells.
- Type hints and NumPy-style docstrings on functions; comments explain **why**, not what.
- Fixed seeds; functions for repeated logic; no hard-coded secrets or absolute paths.
- A cell that fails must fail loudly. Never swallow exceptions to keep the notebook "green".
- Unit tests may cover pure classical arithmetic (for example metric formulas). They must never fabricate quantum output.

---

## 10. How submissions are judged

Judges score against the standards of the organizers' reference implementation, in two passes. Pass 1 is binary gates; Pass 2 is the point rubric. Every gate must be evidenced inside your notebook or evidence files.

### 10.1 Pass 1: acceptance gates (pass / fail)
| # | Gate | Evidence the judge looks for |
|---|---|---|
| 1 | `DATA_REAL_KAGGLE` | Kaggle slug, file hashes, licence logged |
| 2 | `SCHEMA_AUDITED` | Tables, columns, validated one-to-one join |
| 3 | `EDA_COMPLETE` | Required figures and data-quality tables saved |
| 4 | `LEAKAGE_SAFE` | Feature register; zero fingerprint overlap across partitions |
| 5 | `UNSEEN_FAMILY_EVALUATION` | Evaluation C built from the named held-out families |
| 6 | `XGB_FULL_REAL` | XGBoost fitted on frozen train, evaluated on A, B, C |
| 7 | `XGB_MATCHED_REAL` | XGBoost control on the same 8-session cohort |
| 8 | `QAOA_HARDWARE_REAL` | Real job(s); selection derived from measured counts |
| 9 | `QSVC_KERNEL_HARDWARE_REAL` | Train and test kernels traced to real job IDs |
| 10 | `VQC_TRAIN_HARDWARE_REAL` | Every optimizer loss from QPU measurements |
| 11 | `BEFORE_QPU_REAL` | Baseline circuit measured on hardware, circuit saved |
| 12 | `AFTER_QPU_REAL` | Improved circuit measured on hardware, circuit saved |
| 13 | `NO_QUANTUM_SIMULATION` | Import and dependency scan clean; no simulator anywhere |
| 14 | `QPU_USAGE_VALIDATED` | Quota preflight and IBM usage figures within budget |
| 15 | `PAIRED_RESULTS` | Identical session IDs and labels across all methods |
| 16 | `METRICS_SOUND` | Correct denominators, intervals, PR-AUC applicability |
| 17 | `FIGURES_COMPLETE` | All required labelled figures saved |
| 18 | `LIVE_HARDWARE_JOB_RECOVERY` | Every job ID retrievable from IBM by the judges |
| 19 | `PRODUCT_DEMO` | Callable offline scoring function with stated limitations |
| 20 | `NO_SECRETS` | No tokens, CRNs or credentials in files or history |
| 21 | `EXECUTED_NOTEBOOK` | Execution counts, saved outputs, zero error outputs |
| 22-23 | Submission route (fork and pull request) | `[ORGANIZER TO CONFIRM]` |
| 24 | `OVERALL_COMPLETE` | All applicable gates pass; otherwise state `PARTIAL_BLOCKED_<reason>` honestly |

A gate that cannot be met because of a genuine external block must be reported as `BLOCKED_<REASON>`; judges treat an honest block better than an unsupported pass. Judges re-query job IDs, so fabricated or borrowed IDs are detected.

### 10.2 Pass 2: scoring rubric (PROPOSED, 100 points) `[ORGANIZER TO CONFIRM weights]`

| Area | Points |
|---|---|
| Data integrity, leakage control, frozen splits | 15 |
| Classical baseline quality (A/B/C, intervals, validation-only threshold) | 15 |
| Quantum implementation on real hardware (job IDs, counts, before/after, budget) | 25 |
| Method comparison and benchmarking rigour (controls, all metrics, honest small-n handling) | 15 |
| Insight and technical explanation quality | 15 |
| Reproducibility and code quality | 10 |
| Bonus: strong performance on unseen anomaly scenarios (B) while meeting the C target | 5 |

**Disqualification:** any simulator or fabricated quantum output, committed credentials, or tuning on held-out data.

---

## 11. Honest-reporting reminders
- Small quantum cohorts cannot support statistical claims; report intervals, not just point estimates.
- High scores on familiar traffic are not evidence of generalisation. Show evaluations B and C.
- Published results on this dataset that use random splits are not comparable to a group and scenario hold-out protocol.
- This is an offline research exercise on emulated data, not a production detector.

## 12. Support
`[ORGANIZER TO CONFIRM: help channel, mentor hours, team size, IBM access and quota instructions]`. MIT licence.


On Mon, Oct 19, 2026 at 11:05 AM Sidharth <sidharthe38943@gmail.com> wrote:
Trinexis Qiskit Fall Fest 2026 — Submission Guidelines
Welcome to Trinexis Qiskit Fall Fest 2026!

This repository is used to collect individual project submissions for the hackathon. Please follow the instructions below carefully to ensure your submission is accepted for evaluation.

1. Choose Your Domain
Choose the repository corresponding to your selected hackathon domain:

Quantum Error Correction

Quantum Healthcare

Quantum Cybersecurity

You must submit your project to the repository for your chosen domain.

2. Fork the Repository
Open the official GitHub repository for your selected domain.

Click the Fork button in the top-right corner.

Create a fork under your personal GitHub account.

Clone your fork or use the GitHub web interface to make changes.

Important: Do not create your submission directly in the original repository.

3. Create Your Submission Folder
Inside the submissions/ directory, create a folder using the following naming convention:

GITHUB_USERNAME_NAME_FALL_FEST_2026

Example:

quantumdev99_Sidharth_FALL_FEST_2026

Replace quantumdev99 with your GitHub username and Sidharth with your name.

Naming Rules
Use your own GitHub username.

Include your name for easy identification.

Use underscores instead of spaces.

Do not include email addresses in the folder name.

Use the suffix FALL_FEST_2026 exactly as specified.

Create only one submission folder per participant in the selected repository.

4. Include All Required Deliverables
Your submission folder must contain the following:

submissions/
└── GITHUB_USERNAME_NAME_FALL_FEST_2026/
    ├── README.md
    ├── src/
    ├── report.pdf
    ├── presentation.pdf
    └── demo-video-link.md
Required Files
File	Requirement
README.md	Project overview, problem statement, approach, setup and execution instructions
src/	Source code, notebooks and other relevant implementation files
report.pdf	Project report explaining the solution, methodology, results and conclusions
presentation.pdf	Final presentation slides
demo-video-link.md	Link to a demonstration video showing your project
You may include additional files or folders if they are necessary for your project.

Demo video: Upload your video to an accessible hosting platform and include the URL in demo-video-link.md. Ensure the judges have permission to view it.

5. Prepare Your Project README
Your submission's README.md should include:

Participant name and GitHub username

Selected hackathon domain

Project title

Problem statement

Proposed solution and quantum computing approach

Technologies, libraries and frameworks used

Instructions to install dependencies and run the project

Results, findings and limitations

Presentation and demo video links

References and acknowledgements, where applicable

6. Commit and Push Your Changes
After adding all required files to your submission folder:

Commit your changes to your fork.

Push the changes to your fork, if working locally.

Verify that all files and links are accessible.

Ensure that your code is organised, documented and ready for evaluation.

7. Submit a Pull Request
Once your submission is complete:

Open your fork on GitHub.

Select Contribute → Open pull request, or use the option to create a pull request against the original repository.

Set the original repository as the base repository.

Select the appropriate base branch.

Use the following pull request title format:

Submission: GITHUB_USERNAME_NAME_FALL_FEST_2026

In the pull request description, provide your project title, selected domain, a brief summary and links to your presentation and demo video.

Submit the pull request before the official deadline.

Your pull request is your official submission for evaluation.

8. Submission Checklist
Before opening your pull request, confirm that:

You forked the correct domain repository.

Your folder follows the required naming convention.

Your source code and notebooks are included.

Your project README is complete.

Your report and presentation are included.

Your demo video link works.

Your code includes the necessary setup and execution instructions.

Your pull request targets the original repository.

Your submission follows the hackathon rules and deadline.

9. Evaluation and Review
The organisers will review submissions for completeness, technical implementation, originality, relevance to the selected domain, and the applicable judging criteria.

Opening a pull request does not automatically mean your submission has been accepted. The organisers may request corrections or additional information before completing the review.

Please keep your pull request and all supporting links accessible until the evaluation process is complete.

10. Need Help?
If you encounter any difficulties with forking the repository, preparing your submission, or opening a pull request, contact the hackathon organisers through the official Trinexis communication channels.

Good luck, and happy quantum computing!

— Trinexis Qiskit Fall Fest 2026 Organising Team

