# Automated Readability Assessment of EU Project Cards

## 1. Executive Summary

This project develops a reliable, LLM-based method for evaluating the readability of project cards published by the EU's Project Results Platform (PRP). Each card consists of a title and a truncated description (200 characters), representing EU-funded initiatives (Erasmus+, Creative Europe, etc.).

Three rounds of scoring runs have been conducted, comparing Claude, ChatGPT, and Gemini, as well as human assessments. Key findings: LLMs handle binary criteria (e.g., "Is the description in English?") with perfect consistency (0% delta across runs). However, qualitative criteria (e.g., "Does the title make sense standalone?") show significant drift — similar justifications lead to different scores across runs, with deltas up to 35%.

Providing human assessment examples as reference did not improve consistency. Calculation errors (notably the /3 penalty for non-English content) persist across runs.

The project's primary objective is consistency over time — reducing drift between successive scoring runs — rather than convergence with human scores. The strategy is to attack the worst-performing criterion first (corner case), restructure the prompt into modular components, mechanize calculations outside the LLM, and automate the analysis pipeline where Claude analyst orchestrates Claude webEditor.

## 2. Scoring Runs Conducted So Far

Three rounds of analysis were conducted by Isabelle, documented in three reports:

### PRP_01 — LLM Score Analysis & Comparison
- 10 text samples scored by Claude (3 iterations), ChatGPT, and Gemini
- Binary criteria (Desc_EN, Title_EN): 0% delta across all LLMs and iterations
- Qualitative criteria: significant drift, up to 35% delta on Substantiality, 70% on Desc_funding
- Similar justifications across iterations but disparate scores
- Calculation errors on /3 penalty and subscore aggregation
- Gemini tends toward extreme scores; ChatGPT closer to Claude

### PRP_02 — Human Assessment & Comparison
- 10 new text samples scored by a human and by Claude
- 83% overall alignment, 17% average delta
- Biggest disagreement: Title_standalone (33% delta) — human is stricter, penalizes vague/generic titles
- LLM mixes criteria (Title_clear vs Title_standalone), uses description elements where not required
- Human is more rule-based; LLM more interpretive and lenient

### PRP_03 — Prompt Combined with Human Examples
- Same 10 samples from PRP_01, scored by Claude (3 iterations) with 10 human assessments provided as reference
- No improvement in consistency — average delta slightly increased (5% vs 4%)
- Human references influenced interpretation unevenly across iterations
- Calculation errors persist
- Conclusion: providing examples as reference with a generic instruction ("use as reference for your scoring") did not improve consistency. However, this was not a structured few-shot approach — examples were not targeted per criterion, not contrastive (good vs bad), and the instruction did not specify what the LLM should extract from them. The potential of properly structured few-shot prompting remains to be tested.

## 3. Objectives

1. **Consistency first** — Achieve stable, reproducible readability scores across successive scoring runs on the same cards with the same prompt. Measured by standard deviation across runs.

2. **Ranking before scoring** — For qualitative criteria, have the LLM rank (order) cards relative to each other before assigning numerical scores. Relative ranking is hypothesized to be more stable than direct absolute scoring.

3. **Modular prompt architecture** — Structure the evaluation prompt into separate, independently tuneable components (criteria definitions, reference examples, weighing, scoring rules, scale).

4. **Corner case resolution** — Identify the worst-performing criterion (currently Title_standalone, 33% average delta) and solve it first. If we can stabilize the hardest case, the rest follows.

5. **Mechanize calculations** — Remove arithmetic from the LLM (subscore aggregation, /3 penalty). The LLM produces raw subscores and justifications; code handles the math.

6. **Automate the pipeline** — Build toward a setup where Claude analyst orchestrates Claude webEditor: run scorings, compile stats, compare conditions, flag drift.

7. **Convergence with human assessment** — Once consistency is achieved, improve alignment with human evaluators. Secondary objective.

## 4. Next Steps

### 4.1 Modular Prompts

Restructure the current monolithic prompt into separate, versioned files:
- `criteria.text` — List of criteria with explicit measurement methods per criterion
- `card_examples.text` — Reference corpus: fully evaluated cards with justifications, structured as few-shot examples (targeted, contrastive, explicit)
- `weighing.text` — Weight of each criterion in the final score
- `scoring.text` — Rules for calculating the final score from weighted subscores (including /3 penalty rules)
- `scale.text` — Readability categories to match the final score against (Excellent, Good, Just readable, Difficult, Cryptic)

The main prompt references these files and defines the role (web editor), the method (ranking then scoring, one criterion per turn), and the expected output format.

This architecture allows controlled experimentation: change one component, measure the impact on consistency, keep everything else stable.

### 4.2 Asking Claude to Run Scorings and Generate CSV Results

Automate the scoring execution: Claude webEditor receives the modular prompt, a target corpus, and produces structured output — not free text but CSV-formatted results with one row per card, columns for each criterion subscore, justification, and raw total (before mechanized calculations).

This enables:
- Direct ingestion into analysis scripts (no manual copy-paste from chat)
- Consistent output format across runs
- Easy diff between runs for drift detection

The /3 penalty, subscore aggregation, and scale classification are computed by code, not by the LLM. The LLM only produces raw subscores and justifications.

### 4.3 Asking Claude to Prepare Setups

Claude analyst takes on the role of experiment designer: given the modular prompt architecture, it prepares scoring run setups — specific combinations of prompt components to test.

Examples of setups to compare:
- Single turn (all criteria at once) vs one turn per criterion
- With vs without reference corpus
- Ranking-first then scoring vs direct scoring
- Different few-shot strategies (contrastive examples, number of examples, targeted per criterion)

For each setup, Claude analyst generates the full prompt package ready for Claude webEditor to execute. After multiple runs, Claude analyst compiles the stats (standard deviation per criterion, delta tables), compares conditions, and flags which setup reduced drift.

This creates a controlled experiment loop: design setup → run scoring → measure consistency → iterate.

### 4.4 Preparing Reference Corpus with Claude

Before running experiments, we need a solid reference corpus — a set of cards evaluated by a human, serving as few-shot material.

Isabelle's existing human assessments (10 cards from PRP_02) are a starting point. To build a more effective reference corpus, Claude analyst assists the human evaluator:
- Propose candidate cards that avoid the extremes (not obviously Excellent or Cryptic) and focus on the mid-range where qualitative judgement is hardest and drift is highest
- For each criterion, build a progressive range of examples showing gradual differences in scoring (e.g., 3/10, 5/10, 7/10) rather than just good vs bad
- Draft preliminary evaluations that the human reviewer validates, corrects, or adjusts
- Flag edge cases that would stress-test the LLM (ambiguous titles, truncation-heavy descriptions, mixed-language content)

The human remains the authority — Claude accelerates the preparation, the human signs off. The resulting reference corpus is stored in `card_examples.text` and versioned alongside the rest of the modular prompt.

## 5. Glossary

**Card** (carte projet) — Un enregistrement composé d'un titre et d'une description (tronquée à 200 caractères) décrivant un projet financé par l'UE (Erasmus+, Creative Europe, etc.). C'est l'unité qu'on évalue.

**Scoring run** — Une exécution unique du processus d'évaluation : un LLM applique le prompt sur un ensemble de cards et produit, pour chacune, des subscores par critère + justifications. Chaque run est indépendant (nouvelle conversation).

**Consistency** (consistance) — La stabilité des scores entre plusieurs scoring runs successifs sur les mêmes cards, avec le même prompt. Se mesure par l'écart-type entre runs. C'est l'objectif prioritaire du projet.

**Convergence** — Le degré d'alignement entre les scores du LLM et ceux d'un évaluateur humain (ou d'un autre LLM). Important mais secondaire par rapport à la consistency.

**Criterion** (critère) — Un axe d'évaluation de la lisibilité d'une card. Chaque critère a un poids, une méthode de mesure, et peut être binaire (oui/non) ou qualitatif (score nuancé entre min et max).

**Binary criterion** (critère binaire) — Un critère dont la réponse est oui ou non, sans nuance. Ex : "La description est-elle en anglais ?" Les rapports d'Isabelle montrent que les LLMs les gèrent avec 0% de delta entre runs.

**Qualitative criterion** (critère qualitatif) — Un critère qui admet des scores intermédiaires entre min et max, basé sur un jugement subjectif. Ex : "Le titre fait-il sens sans lire la description ?" C'est là que la consistency pose problème.

**Delta** — L'écart entre deux scores pour un même critère sur une même card. Peut être intra-LLM (entre runs) ou inter-LLM (entre modèles différents), ou LLM-humain.

**Corner case** — Le critère ou la card sur lequel le LLM performe le moins bien (plus grand delta). La stratégie du projet est d'attaquer le corner case en premier : si on plie celui-là, on plie les autres.

**Modular prompt** (prompt modulaire) — Architecture de prompt découpée en fichiers séparés : criteria.text, card_examples.text, weighing.text, scoring.text, scale.text. Permet de modifier un paramètre sans toucher au reste.

**Calibration** — Le processus d'ancrage du LLM via des exemples de référence (cards déjà évaluées par un humain) pour orienter son échelle de notation. Reste à déterminer la meilleure méthode (few-shot structuré, ranking préalable, décomposition en questions binaires...).

**Few-shot prompting** — Technique de prompting où l'on fournit au LLM quelques exemples concrets (input + output attendu) avant de lui demander le travail. Variantes : zero-shot (aucun exemple), one-shot (1 exemple), few-shot (2-5 exemples).
<!--
Un few-shot efficace est :
- Ciblé : un critère à la fois, pas tout d'un coup
- Contrasté : montrer un bon ET un mauvais exemple pour caler l'échelle
- Explicite : dire au modèle ce qu'il doit retirer des exemples

Ce qu'Isabelle a fait dans PRP_03 (10 exemples humains + "use as reference") n'est pas du
few-shot structuré : trop d'exemples d'un coup, instruction vague, le modèle ne sait pas
quoi en extraire exactement. Résultat : pas d'amélioration de la consistency.

Un few-shot bien fait ressemblerait à :
"Voici 3 exemples d'évaluation du critère Title_standalone. Note comment un titre vague
comme 'Capture The Future' reçoit 2/10 alors qu'un titre descriptif comme 'Accredited
projects for mobility of learners' reçoit 9/10. Applique la même logique à cette card."
-->

**Drift** (dérive) — L'opposé de la consistency : la variation des scores d'un LLM entre scoring runs successifs sur les mêmes cards avec le même prompt. C'est ce qu'on cherche à réduire.

**Target corpus** — L'ensemble de cards à évaluer lors d'un scoring run.

**Reference corpus** — L'ensemble de cards déjà évaluées (par un humain) fournies en exemple au LLM pour calibrer son jugement. C'est le matériau du few-shot.

**Turn** — Un appel LLM isolé. Peut être single card (une card, tous les critères), single criterion (toutes les cards, un critère), ou single card & single criterion (une card, un critère).

**Ranking** (classement) — Ordonner les cards entre elles sur un critère donné, avant de leur attribuer un score numérique. Le ranking relatif est potentiellement plus stable que le scoring absolu. Le ranking d'une card peut être relatif au target corpus déjà ranké, relatif au reference corpus, ou relatif à l'union des deux.
