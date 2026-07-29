# QuNova PT Interview — Q&A Preparation

Answers are talking points, not scripts. Say the qualification FIRST when a claim is empirical.
Items marked ★ are highest priority.

---

## Part 1 — Adiabatic CoVaR

### ★ Q1. Why not just use quantum phase estimation?
**A:** QPE gives eigenvalues with guaranteed precision, but at the cost of deep circuits — long controlled time evolutions — which near-term and even early fault-tolerant devices cannot afford. It also still needs an initial state with good overlap with the target eigenstate. Our approach uses a shallow ansatz throughout, and empirically tolerates much larger morphing steps: QPE for instantaneous eigenstates pays T ∝ 1/g, conventional adiabatic evolution 1/g², while we observed Δt scaling ~log(g_min). QPE is the right tool once deep fault-tolerant circuits are cheap; we are targeting the regime before that.

### ★ Q2. What exactly happens at a level crossing / very small gap? Walk me through the failure mechanism.
**A:** In parameter space, each eigenstate of H(t) corresponds to a root of the covariance map, with its own basin of attraction for the Newton iteration. As the gap shrinks, the roots corresponding to the two nearly-degenerate eigenstates approach each other in parameter space, and their basins shrink. If Δt is large enough that the previous root lands in the neighboring basin after the morph, CoVaR converges to the neighboring eigenstate — an effective excitation or de-excitation. It still converges to *an* eigenstate; we lose control over *which*. Mitigations we discussed: smaller Δt near small gaps, small random parameter fluctuations at each timestep.

### Q3. Why does root-finding escape local traps when minimization doesn't? What's the mechanism?
**A:** Three ingredients. (1) Different landscape: VQE's local minima are minima of one scalar surface; they are generally NOT roots of the full covariance system, so the iteration is not attracted to them. (2) Over-determination: we impose far more constraints than parameters, so the Gauss–Newton/LM step uses much more information per iteration than a single gradient. (3) Stochasticity: the operator pool is resampled every iteration, so the covariance system itself changes — a spurious near-stationary point of one sample is not stationary for the next. I'd be careful to claim a proof; this is the mechanistic picture supported by the numerics.

### Q4. The necessary conditions are logically redundant — why include them?
**A:** Correct — the sufficient set already pins down eigenstates, and every eigenstate automatically satisfies the rest. Their role is numerical, not logical: (1) over-determination improves Gauss–Newton conditioning; (2) extra rows restore Jacobian rank along directions where the sufficient set is flat; (3) more constraints average shot noise, like more data points in a regression; (4) via classical shadows they are nearly free — same dataset, O(1) classical work each. Sufficient = definition, necessary = regularization.

### Q5. How do you choose H₀ and the ansatz? What if the tracked state leaves the ansatz manifold mid-path?
**A:** H₀ needs analytically known eigenstates preparable by a shallow circuit — e.g. non-interacting / diagonal models. The ansatz must remain expressive along the whole path, not only at the endpoints — that is a real limitation I flag myself: if the instantaneous eigenstate exits the expressible manifold, tracking degrades even with perfect optimization. Ansatz choice motivated by the problem structure; details per experiment are in the paper.

### Q6. Newton needs J⁻¹ — what about singular / ill-conditioned Jacobians?
**A:** We never invert a bare square Jacobian. The update is the Levenberg–Marquardt regularized form — [JᵀJ + λId]⁻¹Jᵀ — shown on the workflow slide. Over-determination plus the λ damping handles rank deficiency; the stochastic pool resampling changes the row set each iteration, which also helps escape persistently flat directions.

### ★ Q7. The log(g_min) scaling — proven or observed? How many instances?
**A:** (Say this before they finish the question.) Empirical, in one problem setting — not a proven scaling. What we can say precisely: across the instances tested, the maximum usable Δt sat far above the conventional adiabatic requirement and grew roughly logarithmically as the gap closed. Making that rigorous — or finding where it breaks — is exactly the kind of follow-up work I want to do.

### Q8. 10 qubits is tiny. What breaks first at 50–100 qubits?
**A:** Honest answer: unknown, and that is a research question. The candidates, in the order I'd worry: (1) ansatz depth/expressivity to keep tracking; (2) operator pool composition — locality of useful operators as systems grow; (3) classical per-iteration cost of the LM solve as the pool and parameter count grow. Measurement cost is the one thing that scales gracefully — shots grow only logarithmically in the number of covariances.

### Q9. Your VQE baseline failed in ALL instances — was it tuned fairly?
**A:** The comparison was deliberately controlled: same initial parameters, and in the adiabatic-VQE variant, the same morphing schedule — so the only difference is the update rule, minimization vs. root-finding. [Verify optimizer/hyperparameters used in the paper before the interview; if pressed beyond what you recall: "I'd rather check the exact settings than guess."]

### Q10. What shot budgets did you simulate for the noise robustness claims?
**A:** [Look up the exact numbers in the paper before the interview. If not recalled in the moment:] "The precision-vs-Δt experiment was run with and without shot noise at the budgets reported in the paper — I don't want to misquote the exact number; I can point to the figure."

### ★ Q11. How would adiabatic CoVaR apply to molecular Hamiltonians?
**A:** Two changes. Measurement: molecular Hamiltonians after fermion-to-qubit mapping have many non-local terms, so plain Pauli shadows lose efficiency — fermionic (matchgate) shadows are the natural replacement, at the cost of deeper scrambling circuits. Morphing path: chemistry offers natural interpolations — e.g. turning on interaction terms from a mean-field H₀. The open question I find interesting is composing this with subspace-based methods where variational tracking is weakest.

### ★ Q12. Have you read HI-VQE? How does it relate?
**A:** [Read arXiv 2503.06292 fully before the interview — then replace this placeholder with 2–3 specifics.] Frame: complementary angles on the same bottleneck — HI-VQE restructures the computation into quantum configuration sampling plus classical diagonalization; adiabatic CoVaR restructures the optimization itself. I'm interested in where they compose rather than compete.

### Q13. CoVaR needs many covariance estimates per iteration — measurement cost vs. subspace methods?
**A:** Our card: N_s = O[ν log(N_c) ε⁻²] — logarithmic in the number of covariances, linear in circuit parameters. So the pool size is essentially free; the real cost driver is ν and the target precision. A fair comparison against subspace methods depends on their sampling requirements per basis state — happy to work through it on a whiteboard.

---

## Part 2 — RL Order Execution

### ★ Q14. Does hindsight reward shaping bias the optimal policy? Is it potential-based?
**A:** [DECIDE YOUR HONEST ANSWER BEFORE THE INTERVIEW — pick the true one:]
- If potential-based: "Yes — it can be written in potential-based form, so policy invariance holds by the standard argument."
- If heuristic-but-anchored: "It is not potential-based, so I don't claim provable invariance. The anchoring is that per-step scores aggregate to the terminal objective, and we validated empirically that improving the shaped return improved the terminal metric. I'd rather claim that honestly than overclaim invariance."

### Q15. How did you validate? Protection against backtest overfitting?
**A:** Structural, not bolted on: walk-forward validation (train on past, evaluate strictly forward), sensitivity analysis over hyperparameters, and evaluation conditioned on market regime/direction. Distrust of any single in-sample number is the default posture — the same discipline I'd bring to benchmarking heuristic quantum algorithms.

### Q16. Why PPO?
**A:** Pragmatically: stable under noisy objectives, well-understood failure modes, strong baselines available. But my honest takeaway is that the optimizer mattered far less than the objective — the same PPO failed on the raw reward and worked on the shaped one. Cost-function design dominated optimizer choice.

### Q17. Sim-to-real: how is market impact modeled in your simulator?
**A:** That gap is real and I flagged it as the honest caveat — it's a core reason validation against classical baselines is part of the method, not an afterthought. On simulator internals I'll stay at pattern level for confidentiality; the principle: conservative impact assumptions, and never trusting simulator alpha that doesn't survive forward evaluation.

### Q18. What was actual performance vs. TWAP/VWAP?
**A:** (Practice this refusal so it sounds professional:) "Specific performance figures are Qraft proprietary information, so I'll decline the numbers. What I can describe is the evaluation protocol — benchmark-relative scoring, walk-forward, direction-conditioned reporting — and I'm happy to go as deep as you like on that."

---

## Part 3 — HFT / Allocation

### Q19. Why CRTP over virtual — can't the compiler devirtualize?
**A:** Sometimes, but it's not guaranteed — devirtualization depends on visible concrete types, whole-program/LTO settings, and breaks across translation-unit boundaries. CRTP makes the dispatch structural: the concrete type is a template parameter, so inlining is a property of the code, not an optimizer's mood. On a hot path, I want guarantees, not heuristics.

### Q20. When does SoA beat AoS? Why SPSC queues?
**A:** SoA wins when you iterate one field across many instruments — full cache lines of useful data, vectorizable. AoS wins when you touch many fields of one object. SPSC: with exactly one producer and one consumer you need no CAS loops, and head/tail live on separate cache lines — no contention, no ping-pong; it's the cheapest correct queue when the topology allows it.

### Q21. How do you decide allocation weights across opportunities?
**A:** Problem-shape level: joint allocation across live opportunities under shared inventory, weighing edge per unit of capital-time consumed — the lock-up priced into the decision. The specific objective and constraints are Qraft proprietary, so I'll decline those details.

### Q22. What latency did you achieve?
**A:** I'll describe methodology rather than quote figures: hardware timestamps at the wire, percentile-based reporting — tails, not means, because p99.9 during bursts is what loses money. Specific numbers are proprietary.

---

## Cross-cutting / Theme

### ★ Q23. "All three are the same move" — isn't that a stretch?
**A:** The precise common structure: in each case the optimizer stayed standard, and what changed was the problem specification — the objective itself (CoVaR: minimization→root-finding), the observation and reward the optimizer sees (RL: state encoding + hindsight shaping), the decision granularity (arbitrage: per-opportunity→joint allocation). If you push me: it's a lens, not a theorem — but it's the lens that actually produced each solution, not one applied after the fact.

### Q24. Barren plateaus are partly a cost-function property — so is VQE's problem the optimizer or the cost function?
**A:** Exactly the point — it's the cost function, which is why the fix changes the cost structure rather than the optimizer. CoVaR replaces one scalar landscape (whose geometry causes plateaus and traps) with a system of covariance constraints with different geometry. Agreeing with the premise IS my thesis.

---

## Strengths & Weaknesses

### ★ Q25-a. What is your greatest strength?
**A (fast adapter/learner — journey as proof):**
"Speed of adaptation — and I mean it in a specific, verifiable way. In the last few years I moved from theoretical quantum physics into production ML and then into low-latency systems engineering, and in each domain I reached the level of shipping real, load-bearing work: a published quantum algorithms paper, an RL execution system, HFT infrastructure running in live markets. The reason it transfers is that I don't learn domains as collections of facts — I look for the underlying problem structure, which is usually optimization, and map what I already know onto it. That's also why I'm confident about ramping up on quantum chemistry here: the mathematical core is eigenstate computation, which is exactly where I've been living."
- One-line version: "I adapt fast, and my career is the evidence — three domains, shipping-level depth in each, because I learn structures, not facts."

### ★ Q25-b. What is your greatest weakness?
**PRIMARY (recommended — relevant, honest, mitigable). VERIFY THIS IS TRUE FOR YOU BEFORE USING:**
"Domain depth in quantum chemistry specifically. My quantum background is algorithms-side — variational methods, measurement schemes, state preparation — but the chemistry side, electronic-structure theory, basis sets, what accuracy actually matters for which molecules, is knowledge I have breadth in but not yet working depth. I'm addressing it the way I've closed every gap: reading the primary literature (I've started with QuNova's own papers), and I ramp fastest when the learning is attached to a real problem — which is part of why I want to be here rather than studying it in the abstract."
- Why this works: it is the TRUE gap given your trajectory, it is directly relevant (not a fake weakness), the mitigation is already in progress, and it converts into a reason to hire you.

**ALTERNATIVE (if you feel the chemistry one is too close to the role's core):**
"I over-invest in rigor at the front of a project — adversarially auditing my own claims, refusing to write down numbers I can't defend. It costs speed early. I've learned to time-box it: full rigor for anything that will be published, claimed, or traded on; explicit 'draft quality' labels for exploration so the audit instinct doesn't slow iteration."
- Riskier: can sound like a humble-brag. Use only with the concrete time-boxing mitigation.

**DO NOT USE:** generic fake weaknesses ("perfectionist", "work too hard") — this audience will discount you for it.

---

## General / Career

### ★ Q26. Why QuNova? Why not stay in finance — the pay is better?
**A:** "Because the question I've been working on my whole career — how hard optimization problems become tractable when you reshape them — has its highest-leverage version in variational quantum algorithms right now, and QuNova is attacking it in the way I believe it gets solved: restructuring the computation, as HI-VQE does, rather than waiting for better optimizers or better hardware. Finance was where I built the ML and systems half of the stack; this is where the whole stack points. The pay difference buys less than doing the work I came back for."

### ★ Q27. Why did you go into finance in the first place?
**A:** (Must match your 자소서.) "Two reasons that aligned. My military service situation meant spending these years in industry as a 전문연구요원. Given that constraint, I chose deliberately: quantitative finance is the fastest, most ruthless environment for exactly the fundamentals frontier quantum research increasingly demands — production ML, RL, low-latency systems, and honest statistical validation. I went in knowing what I was there to build."

### Q28. If you get a PhD offer (e.g. Oxford) in two years, do you leave?
**A:** (Think this through honestly beforehand; suggested honest frame:) "My constant is the research direction, not the institution type. A PhD is one path to doing this research; doing it at QuNova is another, and right now it's the one I'm choosing. I won't pretend to control what I'll want in several years — but I can say the work I'd want to do in a PhD is the work I'm asking to do here."

### Q29. What do you want to work on in your first six months?
**A:** [Sharpen after reading HI-VQE. Safe concrete candidates:] "Three candidates, in the order I'd propose: (1) measurement-cost reduction in the existing pipeline — it's the binding constraint of every variational method and where my shadows/CoVaR background applies immediately; (2) excited-state capabilities, since direct excited-state preparation is my paper's strength and chemistry applications need spectra; (3) benchmarking discipline — walk-forward-style honest evaluation for heuristic quantum algorithms, which I bring from finance."

### Q30. Tell me about a project that failed.
**A:** "An index-futures RL project at Qraft. I built the pipeline, ran honest evaluation, and the signal did not survive — walk-forward performance was indistinguishable from noise. I recommended killing it early rather than tuning until something looked good, and reallocated the time to the execution project that did work. The lesson I keep: a fast, clean negative result is a success of process — the failure mode would have been six more months of curve-fitting."

### Q31. 처우/전직 관련 질문 (군 특례 이직 절차 등)
**A:** 사실관계만 간결하게, 발표 논리와 분리해서. 필요한 서류/시점은 인사 절차에서 확인하겠다고 하고 기술 면접 흐름으로 복귀. (전직 가능 횟수는 지원 전에 본인이 별도 확인해 둘 것.)

---

## Delivery reminders
- Say empirical qualifications FIRST (Q2, Q7) — rigor, not concession.
- Q14: pick your honest answer tonight, not in the room.
- Q18/Q21/Q22 refusals: practice aloud once so they sound professional.
- Q12/Q29: finish arXiv 2503.06292, then replace the placeholders with specifics.
- Weakness: verify the chemistry-gap framing is true for you; never use a fake weakness with this audience.