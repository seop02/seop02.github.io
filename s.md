# QuNova PT Interview — Presentation Script

~24–26 min spoken pace, leaving buffer for figure slides (15–18) and transitions.
**Bold** sentences are the slide-to-slide links — speak them WHILE pressing next.
Assumes the deck edits: orphan bullet removed (slide 11), slide 13 bullet merged, slide 27 dangling sentence cut, Part 3 reordered to iNAV → Allocation → C++, page numbers removed.

---

## 1 — Title
Good afternoon. My name is Wooseop Hwang — thank you for having me. Let me start with a brief introduction.

## 2 — About me
My background is Mathematical and Theoretical Physics at Oxford — the MMathPhys program. Since then I've been at Qraft Technologies as an AI and quant researcher, working on RL-based execution, HFT ETF arbitrage systems, and production ML pipelines.

On the surface, physics and trading look like two different careers. But as the slide says — one thread runs through all of it: **restructuring hard optimization problems until they become tractable.** Everything today is an instance of that. **Here's the plan.**

## 3 — Agenda
Part 1, about twenty minutes: Adiabatic CoVaR — ground- and excited-state preparation by covariance root-finding with adiabatic Hamiltonian morphing, published in New Journal of Physics last year. Then two five-minute parts from industry: RL-based order execution, and HFT ETF arbitrage systems. **Starting with Part 1 — and to motivate it, let me first show you what the standard approach struggles with.**

## 4 — Part 1 divider
This is the NJP paper — preparing ground and excited states using Adiabatic CoVaR.

## 5 — Preliminaries (VQE)
Quick setup. A variational quantum circuit is a sequence of parametrized gates — an ansatz — applied to the zero state. A common task is finding the ground state of a problem Hamiltonian, and the VQE approach minimizes the Hamiltonian's expectation value as a cost function, so the global minimum over θ approximates the ground state.

The known problems are on the slide: training is NP-hard in general, and without good initial parameters, barren plateaus make it prohibitively expensive. And even *with* good initialization, the landscape contains exponentially many local traps.

So minimizing this energy surface directly is a hard optimization problem. **CoVaR's answer was to stop minimizing it.**

## 6 — Preliminaries (CoVaR)
CoVaR — Covariance Root finding — converts eigenstate search into root-finding. Define the covariance of two operators in a state — top equation. Then the conditions below: covariances of the Hamiltonian with its own decomposition terms vanishing is *sufficient* for an eigenstate; vanishing against any pool operator is *necessary*.

Given a parametrized state, each covariance becomes a function f_k of θ — and we search for parameters where *all* covariances are simultaneously zero, for all operators in the pool.

Same target, completely different problem: not descent on one surface, but simultaneous roots of many. **And the reason we can afford "many" is the measurement scheme.**

## 7 — Preliminaries (operator pool)
The significant advantage: we can use a very large operator pool, because classical shadows estimate a large number of covariances simultaneously from the same data. And increasing the number of constraints *improves* the convergence rate — so ideally you use as many as possible, certainly more than the number of circuit parameters. **Let me show concretely how a covariance comes out of raw bitstrings.**

## 8 — Preliminaries (classical shadows)
Each shot: measure every qubit in a random X, Y, or Z basis. The bases plus the outcome bitstring reconstruct an unbiased snapshot of the state — that's the first equation.

For a k-local Pauli string, a shot contributes only if its bases match on the support, and then the estimate is just a product of bits, scaled by three-to-the-k. Median-of-means over shots gives the expectation to standard shot-noise precision.

Then the key algebraic fact: a covariance is a linear combination of Pauli expectations — because the product of two Pauli strings is again a single Pauli string. So every term comes from the *same* dataset: one shadow serves all the covariances, order-one classical work each.

**So vanilla CoVaR is measurement-cheap and it converges. But it has one blind spot.**

## 9 — Preliminaries (features & limitations)
Summarizing: robust against local traps in the energy surface, guaranteed shot-noise cost per iteration, guaranteed convergence to an eigenstate.

The limitation — and this is the launching point of our paper: the success rate of reaching the *desired* eigenstate is proportional to the initial state's overlap with it. It converges to *an* eigenstate; you don't control which. **Our fix borrows the oldest trick in quantum computing.**

## 10 — Adiabatic CoVaR (background)
Adiabatic evolution. Assume a Hamiltonian H₀ whose ground state you know; interpolate linearly toward the target — first equation. The instantaneous ground state is defined by the eigenvalue equation at each s.

And the classic condition: with s equals t over T, the total evolution time must satisfy T much greater than ε over g-min *squared* — the Landau–Zener condition. That quadratic gap dependence, plus deep circuits for time evolution, is the cost. **Our idea: keep the morphing, drop the time evolution entirely.**

## 11 — Adiabatic CoVaR (the idea)
Adiabatic CoVaR combines gradual Hamiltonian morphing with CoVaR training a *shallow* parametrized circuit that closely follows the instantaneous eigenstates. No time evolution — the parameters of a shallow ansatz are slowly varied by CoVaR instead.

And the morphing guarantees CoVaR is always supplied an initial state with high overlap with an instantaneous eigenstate — which was exactly its weakness. Each ingredient patches the other. Two morphing forms, both shown — linear interpolation, or turning on a coupling term. **Here's the full loop.**

## 12 — Workflow
*(Walk the chart: left column down, right column up.)*

One: decompose the Hamiltonian into the morphing form. Two: prepare the variational ansatz. Three: initialize in a known eigenstate of H₀. Then the loop — four: advance time by Δt. Five: sample operators from the pool. Six: measure observables on the variational circuit via classical shadows — that's the quantum hardware's whole job. Seven: compute covariances and the Jacobian. Eight: the regularized Newton update — that's the Levenberg–Marquardt form on the slide. Converged? Then check t equals one; if yes, return the solution parameters.

Notice the division of labor — the quantum device only prepares and measures; every decision is classical. **So what does this buy us in theory?**

## 13 — Theoretical advantages
Four claims. Direct state preparation, not just energies — ground *and* excited states with a short-depth ansatz; the caveat stays: no full control over which eigenstate. Contrast VQD, which needs all n−1 lower eigenstates to prepare the n-th.

Second: large Δt tolerated — conventional adiabatic evolution needs time of order g-min to the minus two; here Δt scales favorably with the gap and barely affects precision.

Third: cheap measurements — shots per iteration scale logarithmically in the number of covariances and linearly in the circuit parameters.

Fourth: robustness via stochastic Levenberg–Marquardt — the pool is randomly resampled each iteration, which buys robustness to gate noise, shot noise, and trapping. **Claims made — here's the evidence.**

## 14 — Empirical setup
All simulations in QuEST and its Mathematica interface QuESTlink. Three problem settings, stressing different things: spin problems, the lattice Schwinger model, and combinatorial optimization. **First the head-to-head with VQE.**

## 15 — Spin problems
Left panel, the spin chain. Green is standard VQE, started from the same initial parameters — every run gets stuck in local minima well above the ground state. Orange is VQE substituted into our *own* morphing loop — adiabatic VQE — same warm starts, and it still fails to track. Blue is adiabatic CoVaR: it walks down the instantaneous eigenstates — the dashed staircase — and lands on the true ground state, every instance.

Same initialization, same schedule; the only change is minimization versus root-finding.

Right panel: the same machinery preparing *highly* excited states directly — runs converging onto the forty-second and the two-hundred-fifty-eighth energy levels, for different step sizes. **Next, a physically structured model.**

## 16 — Schwinger model
The lattice Schwinger model — a high-energy-physics benchmark. The dashed green curves are the instantaneous eigenstates; the solid curves are CoVaR tracking the ground, first, second — and in the left panel even the fifth — excited state simultaneously through the morphing.

The takeaway: direct excited-state preparation is not a spin-chain accident; it holds in a structured model. **Third family: optimization problems themselves.**

## 17 — Combinatorial optimization
Weighted MAX-CUT instances. Right panel: the energy staircases down the instantaneous eigenstates and reaches the exact ground state — this happened in all instances. Left panel: the energy error along the path stays at the ten-to-the-minus-three level.

Adiabatic methods are natural here, and adiabatic CoVaR delivers exact solutions. **Now the result I find most remarkable — why the step size can be so large.**

## 18 — Further numerical analysis
This is the paper's headline for me. Left: the usable step size Δt as a function of the minimum gap. The dashed curve is conventional adiabatic evolution's requirement — quadratic in the gap. Adiabatic CoVaR sits far above it, and empirically the dependence looks *logarithmic*.

For context: phase estimation for instantaneous eigenstates pays one over the gap; conventional adiabatic evolution one over the gap squared. I'll be careful — this is an empirical observation in our setting, not a proven scaling — but it's striking.

Right: the final energy error is essentially flat across step sizes, with and without shot noise. **Let me pull Part 1 together.**

## 19 — Results & Discussion
*(If using the gain/price version of this slide:)*

What we gained by changing the problem: gradient descent on a rugged surface became joint root-finding — freeing us from barren plateaus and local traps; VQE, standard and adiabatic, failed in every spin-chain instance while adiabatic CoVaR converged in every one. Direct preparation of ground and excited states — like phase estimation, unlike VQD — demonstrated on Heisenberg, Schwinger, and exact MAX-CUT. And empirically, Δt scaling like log of the gap, against one-over-g for QPE and one-over-g-squared for adiabatic evolution.

The price we pay: tracking is gap-sensitive — near small gaps a large step can hop to a neighboring eigenstate, so we're guaranteed *an* eigenstate, not *the* eigenstate. Staying on the intended track is the failure mode to manage.

Robustness: shot noise confirmed; gate-noise perturbations average out over the large Jacobian. Extensions: fermionic shadows toward molecules and materials, and early fault-tolerant implementations.

One sentence to close Part 1: **we traded an optimization problem for a tracking problem — and empirically, the tracking problem is much cheaper. Now the same lens, on a trading floor.**

## 20 — Part 2 divider
At Qraft: reinforcement learning for optimal trade execution — sequential decision-making under noise.

## 21 — What order execution is
The problem first, since it's less familiar here. Order execution converts a decision — buy five hundred thousand shares — into fills: slicing a parent order into child orders over time, as in the diagram.

And the obligation is structural: the decision is made upstream, so executing seventy percent isn't partial success — it leaves the portfolio off-target. Deadlines come from mechanics — index effective dates, settlement dates, decaying signals, risk limits.

The bind: quantity fixed, deadline fixed — the only free variable is the trajectory in between. That trajectory is the whole optimization problem. **And the trajectory is squeezed between two costs.**

## 22 — VWAP & impact
Quality is measured against VWAP — the market's own volume-weighted average price; beating it means you traded better than the average participant.

The cost is market impact: your order moves the price against you. The mid price reflects the current balance of supply and demand — and a large order *is* new demand, so trading shifts the very balance you're priced against. Impact isn't friction on top of the market; it's the market updating on you.

Why institutions care: parent orders are large relative to displayed liquidity, and at institutional AUM this shortfall rivals management fees — alpha that survives on paper dies in implementation. **Sequential decisions, delayed cost — that structure is why RL fits.**

## 23 — Why RL
Five points on the slide. Genuinely sequential — each child order changes the state the next decision faces. Delayed, path-dependent objective — textbook credit assignment. Stochastic, partially observable — Almgren–Chriss-style closed forms need impact assumptions real microstructure violates; they're the baseline to beat, not a straw man. Static schedules are open-loop; a policy is closed-loop by construction — the diagram on the right.

And the honest caveat: the case rests on structure, not fashion — sample inefficiency and sim-to-real are real, so validation against classical baselines is part of the method.

**But the algorithm was never the hard part. The hard part was making the problem learnable — two design decisions.**

## 24 — My work (1): state encoding
Decision one: what the policy sees. The principle on top — encode what an execution trader actually watches, in stationary units.

Order-book state: multi-level depth, imbalance, spread, microprice distance. Flow and activity: signed trade flow, short-horizon volatility — how hostile is the tape right now. Own-progress: remaining inventory, remaining time, cost versus benchmark so far — this is what makes it deadline- and inventory-aware instead of a signal chaser.

Normalization is half the design: spread-relative prices, volatility-scaled returns, volume-fraction sizes — features must mean the same thing across names and regimes. And deliberately excluded: raw price levels and absolute volumes — non-stationary, and an open invitation to memorize the training period.

**Decision two is what it's rewarded for — and that was the crux.**

## 25 — My work (2): hindsight reward
The problem, top half: the natural reward — final cost versus benchmark — is one number after hundreds of decisions; sparse, delayed, nearly impossible to credit through. And naive per-step rewards are worse: dominated by market noise the agent doesn't control, so it learns to chase noise.

The design: hindsight shaping — after the episode, score each child order against what was actually *achievable* in that market state, using information unavailable at decision time, used only for the training signal, never by the policy. The diagram: one terminal signal per episode, versus a score for every decision — aggregating to the same terminal objective.

The effect: the signal became learnable — the structure of the objective mattered more than the choice of optimizer. **Same move as Part 1, different clothes. One more domain — where the scarce resource is capital.**

## 26 — Part 3 divider
Third project: high-frequency ETF arbitrage — low-latency C++ trading infrastructure.

## 27 — iNAV
An ETF is a claim on a basket, and its real-time fair value is the intraday NAV — the weighted sum of live constituent values plus cash, per creation-unit share, as in the diagram: constituents, weighted sum, compared against the exchange quote.

The officially disseminated iNAV updates on the order of seconds — far too slow — so a live system computes its own from raw market data, on every relevant tick. That fair value is what every downstream decision runs on. **And the most interesting decision it feeds is not when to enter — it's how to allocate.**

## 28 — Allocation *(now precedes C++)*
Here's the subtlety this system taught me. An arbitrage position is not a round trip you control end-to-end: once you buy, that inventory is committed until a sell opportunity *materializes* — and its arrival time is uncertain; forcing an early exit means paying the mispricing back.

So the true cost of entering is not the entry — it's the locked capital: capacity unavailable for every better opportunity that appears during the lock-up, as the timeline shows. Greedy per-opportunity execution prices the entry edge but not the lock-up — capital piles into whatever fired first, and you watch stronger opportunities pass with an empty wallet. Locally sensible, globally wrong — the signature of a mis-specified problem.

The restructuring: lift it into one joint allocation across live opportunities, weighing edge per unit of capital-time consumed — the lock-up priced into the decision, not discovered after it.

**Standard optimizer, reshaped problem — third time today. Last piece: making all of this run at market speed.**

## 29 — C++ hot path
The guiding rule on top: anything that can be resolved before the tick arrives, must be.

Static dispatch via CRTP — no virtual calls, full inlining. Zero-copy parsing in the receive buffer — no allocation, no memcpy. The pricing math — schedules, grids, stencils — prebuilt at startup, so each tick is O(affected), never O(all). Cache-conscious layout, preallocated memory, lock-free queues, no syscalls in steady state.

The strip at the bottom is the whole philosophy: runtime decisions moved to compile time and startup — what's left on the hot path is arithmetic and one publish.

And if you're wondering what this has to do with quantum software: circuit compilation and measurement grouping are exactly the same move — precompute everything before shots are spent. **So — three domains. Side by side.**

## 30 — Closing
Adiabatic CoVaR redefined the objective — energy minimization became covariance root-finding, and ordinary Newton sufficed. RL execution redesigned what the optimizer sees — an unlearnable reward became a learnable objective, and ordinary PPO sufficed. Arbitrage lifted local decisions to a global formulation — greedy entries became one joint allocation that prices the lock-up, and a standard constrained optimizer sufficed.

*(half-beat pause)*

Variational quantum algorithms are exactly where this approach has the most leverage today — the field's bottlenecks: NP-hard training, barren plateaus, exponentially many traps — are optimization problems of exactly this kind. And QuNova's direction is this same philosophy applied to chemistry — HI-VQE restructures eigenstate computation into what the quantum sampler does best and what classical diagonalization does best.

That is the work I want to do — and this is where I want to do it.

## 31 — Thank you
Thank you. Questions welcome — on the paper, the systems, or anything in between.

---

## Delivery notes
- The bold link sentences ARE the transitions: speak them while advancing the slide. If you cut a slide under time pressure, keep its last (bold) sentence and graft it onto the previous slide.
- Three theme callbacks carry the closing: end of 19 ("traded optimization for tracking"), end of 25 ("same move, different clothes"), end of 28 ("third time today"). Never cut these.
- Slide 18: say "empirical, not proven" BEFORE anyone can ask — it reads as rigor.
- Slides 15–18: ad-lib while pointing at the figures; the script is a floor, not a ceiling.
- Closing: slow down on the last two paragraphs, eyes on the interviewers for the final sentence, then stop talking.
- Rewrite the About me theme line and the closing's final sentence in your own spoken voice before rehearsing — they must not sound read.