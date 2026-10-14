<!-- solve-problem:bebec314-9aed-4368-998e-baad3bfab5be -->
# Composing pool infiltration and private forks

A [Tangle Solve problem](https://solve.tangle.tools/problems/bebec314-9aed-4368-998e-baad3bfab5be).

## What is the problem?

In a proof-of-work blockchain, an attacker controls hash rate and may run **multiple composed strategies simultaneously**:

1. **Pool infiltration (FAW setting):** The attacker joins one or more honest victim pools as a registered miner, submitting partial proofs-of-work (shares) but withholding full blocks. Withheld blocks are either never submitted (earning share payout from work done on the victim pool's headers) or released privately to start a fork against the victim pool (the FAW attack).
2. **Private-chain withholding (selfish/stubborn setting):** The attacker mines on its own private chain and selectively releases blocks, with stubborn parameters (lead tolerance, release aggressiveness) generalizing Eyal–Sirer selfish mining.
3. **Identity changes and detection responses:** The attacker can re-register with new pseudonymous identities after being detected/ejected, and can adapt its release/withhold policy in response to pool countermeasures (e.g., penalties, monitoring).

The problem: **find the attacker's optimal joint strategy** — allocation of hash rate across own mining vs. infiltration of multiple pools, withholding policy, fork-release policy, and re-entry policy — against specified payout rules (e.g., pay-per-share, proportional, pay-per-last-N-shares), assuming honest majority behavior otherwise. The output should be a formal model (likely an MDP or stochastic game over a Markov chain of private/public lead states per pool) and characterization of the attacker's optimal revenue (in units of relative reward per block) as a function of total hash rate α ∈ (0, 1/2 or beyond), payout rule, and defense parameters.

**Ambiguity note:** the census is informal. A reasonable formalization fixes: (a) the attacker's objective is long-run expected reward; (b) the defender side is either a fixed mechanism (single-agent optimization) or responsive pools (game); (c) network propagation parameters γ (as in selfish mining) are given. State your chosen formalization explicitly.

## Why does it matter?

Separate analyses of selfish mining, stubborn mining, and FAW underestimate attacker profit because the same hash power and withheld blocks can serve multiple attacks simultaneously — shared-resource compositions were exactly what made FAW strictly dominate plain selfish mining. A solution would give pool operators and protocol designers a correct bound on attacker revenue under composed attacks, informing payout-rule design and detection/penalty mechanisms. It also feeds directly into rationality analyses: if composed attacks are profitable below the thresholds currently assumed, security margins for Bitcoin-style pools (and their share of the ecosystem's hash rate) are overstated.

## What's already known?

- **Eyal–Sirer (2013/2014):** Selfish mining yields positive relative gain for α > 1/3 (γ-dependent threshold down to 0 with perfect propagation advantage).
- **Nayak et al. (2016):** Generalize to stubborn mining (larger parameter space: lead stakes, equal-length forks); show an "optimal" stubborn strategy within their finite Markov family; compose with eclipse attacks for amplified gains.
- **Kwon et al. (2017):** FAW attacks — infiltrating a PPS/proportional pool while running selfish mining against it. Key findings: FAW strictly dominates selfish mining even when the pool's payout exactly compensates orphaned blocks; in infiltration of multiple pools, the attacker can earn rewards from *all* victim pools for a single withheld block's work; FAW profits from attacking smaller pools more.
- **Known gaps:** Each paper fixes one composition (selfish+eclipse, or selfish+one/multiple pool infiltration). No published work (to our knowledge — verify) optimizes the full joint policy including: arbitrary release policies via dynamic programming, identity churn after detection, and response to penalty mechanisms. Related census items on composable penalty mechanisms under side contracts suggest defenders' countermeasures have also not been analyzed jointly.

## What would count as solving it?

1. **Model:** A formal MDP (or stochastic game) whose state captures private lead per chain, victim-pool share accounts, and detection status; actions = {mine-on, withhold, release, submit-share, re-register}.
2. **Result (upper bound on attack cost / optimal attacker revenue):** A theorem giving the attacker's optimal long-run reward V*(α, γ, payout rules, penalty parameters), or tight characterizations for standard payout rules (PPS, PPLNS, proportional), with the optimal policy computed exactly (value iteration on a provably truncated state space) or analytically for special cases.
3. **Comparison theorem:** V* ≥ FAW revenue ≥ stubborn revenue, with strictness conditions, and identification of regimes where composition gains are non-additive.
4. **Defense side (optional variant):** Show a payout/penalty rule under which composed infiltration is unprofitable below a stated α threshold, or a lower bound showing no such rule exists within a class.

## How to start

1. Read Nayak et al. (2016) §III–IV for the Markov-chain machinery over private/public lead states — that is your baseline state space. Then Kwon et al. (2017) §IV for how share payouts enter the reward function.
2. Formalize as an MDP: states = (own private lead, per-pool fork state, cumulative unpaid shares, identity age); verify the state space can be truncated (probabilistic tail bounds on leads, as in Eyal–Sirer's finite truncation argument).
3. Compute numerically first: value iteration for small pool counts, fixed PPS payout. Plot V*(α) vs. FAW and stubborn baselines to detect where composition matters.
4. Main obstacle: identity churn and detection responses make the game dynamic (defender observes and acts), so the single-agent MDP is only a lower bound on attacker power; you must decide whether to fix defenses (MDP) or model them (stochastic game) — the MDP version is tractable and publishable first.

## Key references

- K. Nayak, S. Kumar, A. Miller, E. Shi, "Stubborn Mining: Generalizing Selfish Mining and Combining with an Eclipse Attack," IEEE EuroS&P 2016, DOI:10.1109/EuroSP.2016.32. Generalizes selfish mining to a broader stubborn strategy family via Markov chain analysis and shows further gains when composed with eclipse attacks.
- Y. Kwon, D. Kim, Y. Son, E. Vasserman, Y. Kim, "Be Selfish and Avoid Dilemmas: Fork After Withholding Attacks on Bitcoin," CCS 2017, arXiv:1708.09790. Introduces FAW: proves pool infiltration plus selfish withholding strictly dominates selfish mining, including multi-pool and small-pool cases.
- I. Eyal, E. G. Sirer, "Majority is not Enough: Bitcoin Mining is Vulnerable," FC 2014, arXiv:1311.0243. Foundational selfish-mining analysis with the Markov-chain-over-lead-states method you will extend. (Verify edition details.)
- L. Cong, Z. Li, N. Wang, W. Yang, "Eclipse Attacks on Bitcoin's Peer-to-Peer Network" (Heilman et al.), USENIX Security 2015 — relevant if eclipse composition is included; also see pool payout-rule literature (Rosenfeld, "Analysis of Bitcoin Pooled Mining Reward Systems," arXiv:1112.4980) for formal payout rules. (Verify.)

## Warnings

- The census record is machine-generated and vague; the "detection responses" and "identity changes" components have no established formal model in the cited literature — you must define observability assumptions (what the pool sees, when ejection happens) yourself and state them prominently.
- FAW's dominance result depends on the payout rule compensating orphaned work; don't assume it transfers to PPLNS or PROP without redoing the reward accounting.
- Don't conflate per-block reward with long-run reward rate; share payouts are continuous-flow, block rewards are event-based — unify carefully.
- Avoid claiming optimality within a restricted strategy family (the stubborn-mining paper's approach) and presenting it as global optimality; a referee will ask for the MDP-level argument.
- Any citation marked "(verify)" above was not confirmed from primary sources.

---
*`BCGT-000236` · flagged for review*

## Acceptance guidance

1. **Model:** A formal MDP (or stochastic game) whose state captures private lead per chain, victim-pool share accounts, and detection status; actions = {mine-on, withhold, release, submit-share, re-register}.
2. **Result (upper bound on attack cost / optimal attacker revenue):** A theorem giving the attacker's optimal long-run reward V*(α, γ, payout rules, penalty parameters), or tight characterizations for standard payout rules (PPS, PPLNS, proportional), with the optimal policy computed exactly (value iteration on a provably truncated state space) or analytically for special cases.
3. **Comparison theorem:** V* ≥ FAW revenue ≥ stubborn revenue, with strictness conditions, and identification of regimes where composition gains are non-additive.
4. **Defense side (optional variant):** Show a payout/penalty rule under which composed infiltration is unprofitable below a stated α threshold, or a lower bound showing no such rule exists within a class.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/bebec314-9aed-4368-998e-baad3bfab5be?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000236.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000236)
