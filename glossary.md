# Glossary

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