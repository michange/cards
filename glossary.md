# Glossary

**Card** (carte projet) — Un enregistrement composé d'un titre et d'une description (tronquée à 200 caractères) décrivant un projet financé par l'UE (Erasmus+, Creative Europe, etc.). C'est l'unité qu'on évalue.

**Scoring run** — Une exécution unique du processus d'évaluation : un LLM applique le prompt sur un ensemble de cards et produit, pour chacune, des subscores par critère + justifications. Chaque run est indépendant (nouvelle conversation).

**Consistency** (consistance) — La stabilité des scores entre plusieurs scoring runs successifs sur les mêmes cards, avec le même prompt. Se mesure par l'écart-type entre runs. C'est l'objectif prioritaire du projet.

**Convergence** — Le degré d'alignement entre les scores du LLM et ceux d'un évaluateur humain (ou d'un autre LLM). Important mais secondaire par rapport à la consistency.