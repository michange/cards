Moi: comment on fait produire llm des évaluations (docimologie) cohérentes et stables dans le temps, concernant des domaines qualitatifs, comme la lisibilité d'u texte?
perplexity :
Ta question touche un point de tension crucial dans la recherche actuelle sur l’évaluation par LLM : comment obtenir des jugements stables et fiables sur des dimensions qualitatives comme la lisibilité, la cohérence ou la fidélité. Produire une docimologie (science de l’évaluation) robuste avec des modèles probabilistes reste difficile, mais des méthodologies solides commencent à émerger. Voici les grandes pistes actuelles.

1. Encadrer la subjectivité : calibration et ancrage
   Les LLM ont une grande variabilité dans leurs jugements — même à prompt identique, une légère variation du temperature ou du contexte peut modifier la note.

Pour limiter cela :  
Définir des critères explicites et hiérarchisés, inspirés des grilles d’évaluation humaines (ex. : longueur moyenne de phrase, diversité lexicale, clarté syntaxique).  
Fournir au modèle des exemples d’ancrage (« Voici un texte de lisibilité faible : score 2/5 ; voici un texte clair : score 5/5 »). Cela oriente son espace d'interprétation.  
Employer des invariant checks : mêmes textes réévalués à intervalles différents pour mesurer la stabilité intra-modèle.
2. Utiliser des approches multi‑juges et consensus
   Comme dans la docimologie humaine, un seul évaluateur n’est pas fiable.

Ainsi, on combine les jugements de plusieurs LLM (ou plusieurs prompts) :
Échantillonnage de juges virtuels : faire évaluer chaque texte par plusieurs instances indépendantes du même LLM, puis agréger (moyenne, médiane, vote majoritaire).  
Cross‑model evaluation : comparer les jugements entre modèles (ex. GPT‑4, Claude, Gemini). La convergence entre eux est un indicateur de robustesse.  
LLM‑as‑a‑Judge avec auto‑revue : un modèle juge une réponse, puis une deuxième instance justifie et vérifie la cohérence du premier jugement — méthode qui réduit fortement le bruit selon blent.ai.
3. Structurer l’évaluation autour de métriques hybrides
   La lisibilité est difficile à capturer uniquement par apprentissage sémantique. Les approches robustes combinent :
   Indicateurs objectifs classiques : Flesch, SMOG, longueur de phrase, densité lexicale, taux de subordination (coria-taln-2025.lis-lab.fr).  
   Jugement LLM contextualisé : le modèle estime la clarté, la cohérence logique et la fluidité, en se basant sur le sens plutôt que sur la forme.  
   Évaluation agrégée : pondérer ces scores selon des poids calibrés à partir d’un panel humain.
   Des frameworks récents comme RAGAS ou LLM-as-a-Judge permettent justement d’orchestrer ces évaluations semi‑automatisées à grande échelle (blent.ai).

4. Maintenir la cohérence temporelle : contrôle de dérive
   Les modèles évoluent (nouvelles versions, ajustements internes). Pour garantir la stabilité des notations :
   Stocker et réutiliser les prompts exacts et contextes de calibration.  
   Évaluer périodiquement la dérive en ré‑évaluant un corpus de référence (même textes) à chaque mise à jour — et ajuster les seuils ou pondérations si la distribution change.  
   Externaliser la définition du standard : conserver un jeu de référence annoté humainement servant de repère ; le LLM n’évalue que par rapport à ces textes.
5. Encourager la transparence du raisonnement
   Faire expliciter par le modèle pourquoi il donne une certaine note améliore la cohérence interne.

Exemple de démarche :  
« Évalue le texte selon la clarté syntaxique (1–5). Donne ensuite trois justifications précises sur les aspects influençant ta note. »
Cette verbalisation stabilise la décision et facilite les audits humains.
En résumé
Objectif	Méthode clé	Exemple d’outil / pratique
Réduire la variabilité	Calibration + exemples d’ancrage	prompt‑template avec barème défini
Garantir la fiabilité	Multi‑LLM et consensus	LLM‑as‑a‑Judge croisé
Capturer la lisibilité réelle	Combinaison métriques symboliques + jugement sémantique	RAGAS, indices Flesch + analyse sémantique
Préserver la stabilité temporelle	Corpus témoin + ré‑évaluations	tests de dérive réguliers
Faciliter l’audit	Justifications et rationales explicites	« chain‑of‑thought visible »
Souhaites-tu que je te montre un exemple concret de protocole d’évaluation de la lisibilité via LLM, prêt à implémenter (en Python ou via API) ?
LLM-as-a-Judge : définitions et exemples
L'évaluation des systèmes d'IA générative constitue l'un des défis les plus épineux auxquels font face les équipes qui déploient des solutions basées sur les LLM. Comment mesurer objectivement si u...

J'ai commence à lire.
Capture d’écran 2026-03-26 à 06.56.23.png

J'ai lu tout ton premier rapport. Commentaires à chaud, avant lecture du deuxième. Commentaires dans to header :

High overall alignment (83%) between human and LLM assessments, with an average delta of 17%, indicating generally consistent evaluation patterns.

=> Moyenne n'est pas le bon outil. On a besoin de la meilleure et de la moins bonne évaluation, et, quand on aura plu de mesures, d'un truc du genre écrat-type
LLM occasionally deviates from the prompt, notably by:
Mixing criteria (Title_clear vs Title_standalone)
=> on peut demander un LLM turn par critère.
Using title/description elements when not required
=> améliorer le prompt
Misapplying scoring rules (/3 penalty)
=>  améliorer le prompt, donner plus d'exemples, mécaniser le calcul
In several cases, similar reasoning leads to very different scores, indicating calibration differences rather than analytical gaps.
=> explique.
The human assessment tends to be stricter and more rule-based while the LLM applies more contextual and interpretive judgment  
=> on peut demander un LLM turn par critère.
=> améliorer le prompt
=> donner des exemples


Le résultat de prompt2 sont décevant et étonnants.
Je pense qu'on devrait essayer de formuler autrement l'instruction "Important: You should use the attached human assessment examples as a reference for your scoring." Et peut-être introduire le prompt différemment.

QQch du genre :

You are a web editor, and you have to rank project cards according to their readability.
You are provided with the following tools :
criteria.text : the list of criteria you have to use to evaluate an individual card, where the measurement method for each criterion is explained
card_examples.text : examples of fully evaluated cards, containing, for each criterion, a full text justification of how its specific measurement method was applied.
weighing.text : the weighs to give to each criterion to balance their relative importance
scoring.text : the way to calculate a final readability score based on weigted criteria
scale.text : readability categories to match the score against
What you have to do : ...




Une autres solution consisterait à lui faire écrire chaque évaluation de chaque critère séparément.

Une autre amelioration consisterait à lui demander de classer par ordre les cartes, sur chaque critère, avant de les évaluer..

Et voici de quoi améliorer les critères :
https://twitter.com/jamescoder12/status/2036462995887497477?ref_src=twsrc%5Etfw

Deux points à discuter :
la divergence avec nos mesures humaines est moins gênante que l'inconstance des mesures successives dans le temps.
on peut lui demander d'embrasser le truncated text comme un fait qui ne doit pas affecter son évaluation

Et merci beaucoup, Isabelle,  pour la precision, la clarté et l'exhaustivité de tes rapports.

Salut Isabelle, ready to chat when you feel like it.

Je suis dispo. Tu veux qu'on se fasse un call?

Non je suis on the move heading for coffee. On peut se caler un rev de 10h30 à 11h00 avant le Unit??

parfait!

on peut chat jusque notre rev pm aussi

tu as des q? sr mon feedback ?

Le feedback est clair, merci beaucoup!
C'est juste que je voudrais verifier avec toi quelles sont les prochaines étapes maintenant.

Est-ce que la priorité maintenant est de retravailler le prompt en vue d'obtenir des résultats plus consistants?

Et on avait écarté la piste de lister des éléments objectifs pour chaque critère, mais je vois que c'est (en partie) ce que Perplexity recommande. On reste sur notre approche ou on adapte?

Je propose :
OBJECTIf PRIORITAIRE : On essaie d'augmenter la consiostance (da,s le temps) c'est plus important que la convergence (ave l'humain, avec les LLM)

Est-ce que je peux partir sur une réécriture du prompt alors?

2. ON fait un prompt modulaire, qui ref des .text dans le remote prj, pour pouvoir modifier des params de manière plus contrôlée :

You are provided with the following tools :
criteria.text : the list of criteria you have to use to evaluate an individual card, where the measurement method for each criterion is explained


3. Oui, tu avais propose de lister des éléments objectifs pour chaque critère, et jc'est bien que c'est (en partie) ce que Perplexity recommande. C'est pour ça que je t'ai envoyé le tweet de X, et tu peix faire des recherches sur le sujet.

Ca c'est pour criteria.text

4. Mais il faut aussi s'assurer qu'il compare avec les exemples fournis : le ordering/raking doit avoir lieu durant l'application des critères.
   Ca c'est pour le prompt principal
# Instructions for the ranking of project cards :

## YOUR ROLE : Web Editor

You are a web editor, and you have to rank project cards according to their readability.

## TOOLS : You are provided with the following tools :
You are provided with the following tools :
criteria.text : the list of criteria you have to use to evaluate an individual card, where the measurement method for each criterion is explained

## METHOD :

What you have to do : ...


Essaie d'employer le format md

ok merci

5. Concentrate on failures

Commence par lui demander une seul évaluation, et celle ou il foire le plus (e.g. self-contained titles)

Voilà.

Tu nous trouves le corner Case où il a le plus difficile et si on parvient à plier celui-là on  pliera tous les autres.

Ah oui :


6. On va automatiser ton travail : compilation des stats, comparaison des conditions, rédaction des conclusions
yy
On va accumuler les tests de manière à ce que Claude analyst pilote Claude webEditor
 