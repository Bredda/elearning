# AGENTS.md — Workspace e-learning « Développeur augmenté par l'IA »

Ce workspace contient les modules de formation e-learning asynchrone destinés aux collaborateurs d'une ESN. Le contenu est rédigé en Markdown, en français.

Votre rôle : aider à **rédiger, relire et mettre à jour** les modules. Vous ne développez pas d'application.

## Périmètre

| Dossier | Statut | Contenu |
|---|---|---|
| `Module Dev/` | Actif | Module théorique « Développeur augmenté par l'IA » : 6 chapitres, glossaire, évaluation théorique |
| `Module Testing/` | À venir (dossier vide) | Module théorique et pratique pour les testeurs. Voir la section dédiée |

Hors périmètre : le module pratique GitHub Copilot, pris en charge par un collègue. Il n'est plus dans ce workspace. N'en recréez pas le contenu, et ne modifiez pas les passages des chapitres qui y renvoient sans en parler d'abord.

D'autres modules (autres profils) pourront s'ajouter. Quand l'un d'eux apparaît, ajoutez-le à ce tableau et à la section « Modules » plutôt que de créer un second fichier d'instructions.

## Sources de référence

1. `Module Dev/_programme.md` : plan validé du module Dev (public, registre, durée, contenu de chaque chapitre). C'est la référence pour savoir ce qu'un chapitre doit couvrir. Il a été rédigé en « tu » car c'est un document de travail, pas du contenu appris.
2. Les chapitres existants font foi pour le ton, la structure et la terminologie. Alignez-vous sur eux avant d'inventer une convention.

Si un chapitre s'écarte du plan, signalez-le au lieu de choisir silencieusement lequel a raison.

## Public et registre

- **Public** : collaborateurs internes d'une ESN, niveaux hétérogènes.
- **Vouvoiement** dans tout le contenu des modules (« votre codebase », « Réécrivez »). Pas de tutoiement, même dans les exercices et les quiz.
- **Registre** : analogies conservées, profondeur technique assumée. On nomme les mécaniques (tokens, context window, tool calling...) au lieu de les survoler.
- Texte sobre : pas d'emojis dans le corps (seul « ✅ » est utilisé pour les réponses de quiz), pas de ton promotionnel, pas de formules creuses.
- Les termes anglais courants du métier restent en anglais (prompt, hook, skill, context window...), avec une explication à la première occurrence.

## Structure des chapitres

Reprenez le format des chapitres existants :

- Titre `# Chapitre N — ...`, puis sections numérotées `## N.x Titre`, sous-sections en `###`.
- Séparateur `---` entre les sections principales.
- Exemples de code dans des blocs balisés avec leur langage. Un exemple « mauvais / mieux » suit la forme `// Mauvais` puis `// Mieux`.
- Corrections d'exercices dans des blocs `<details><summary>`.
- Figures dans `Module Dev/assets/`, insérées en `![texte alternatif](./assets/figN.NN.png "titre")`.
- Quiz de 5 à 10 questions en fin de chapitre : question, 4 options A à D, réponse introduite par `> ✅ **Réponse : X** —` avec une justification.
- Quiz : une seule bonne réponse, des distracteurs plausibles, et une justification qui explique aussi pourquoi les autres options sont fausses quand c'est instructif.

Après tout ajout, suppression ou déplacement de section, vérifiez la numérotation des sections suivantes **et** les renvois dans les autres chapitres (par exemple « voir Chapitre 3 », « section 3.5 »).

## Rédiger

- Respectez la durée indiquée par le plan : un chapitre de 30 minutes ne doit pas devenir un chapitre de 60.
- Chaque notion nouvelle s'appuie sur un exemple concret et crédible. Préférez un exemple de projet déjà utilisé dans le module (l'API Orders Node/TypeScript, par exemple) à un nouvel exemple isolé.
- Allez au besoin du lecteur : un exemple raccourci vaut mieux qu'un exemple exhaustif, tant qu'il reste réaliste.
- Pas de contenu spécifique à un outil sans le signaler comme tel. Les chapitres théoriques sont outil-agnostiques ; quand un exemple suit la syntaxe d'un outil (Claude Code, Copilot...), dites-le.
- Un terme défini dans un chapitre doit être cohérent avec le glossaire. S'il manque au glossaire, signalez-le.

## Relire

Lors d'une relecture, vérifiez dans cet ordre :

1. **Exactitude technique** : les affirmations sur les outils, les formats de fichier, les noms d'événements, les chemins, les limites. Ce sont les plus risquées et elles vieillissent vite.
2. **Cohérence interne** : définitions, numérotation, renvois, noms d'exemples, cohérence avec le plan et avec les autres chapitres.
3. **Pédagogie** : la progression, le niveau de détail pour un public hétérogène, les exemples, la pertinence du quiz par rapport au contenu réellement enseigné.
4. **Langue** : orthographe, tournures lourdes, vouvoiement, anglicismes inutiles.

Présentez les retours triés par gravité (erreur factuelle, incohérence, amélioration). Ne mélangez pas une remarque de fond et une coquille.

## Exactitude et sources

Les informations sur les outils d'IA changent vite. Les règles :

- Ne présentez jamais comme établie une information sur un outil, un format ou un nom d'événement que vous n'avez pas vérifiée. Dites ce qui est vérifié et ce qui ne l'est pas.
- Pour toute information datée (fonctionnalités, versions, tarifs, gouvernance d'un standard), vérifiez dans la documentation officielle quand c'est possible. Sinon, signalez-la comme à vérifier dans votre réponse.
- Citez la source de ce qui a été vérifié, hors du texte du module sauf demande contraire.
- N'inventez ni chiffres, ni statistiques, ni citations.

## Mettre à jour

- Une mise à jour part d'un besoin identifié (information périmée, outil qui change, retour de relecture). Indiquez ce qui change, où, et pourquoi.
- Cherchez les autres occurrences de ce qui change (autres chapitres, glossaire, quiz, évaluation finale) avant de conclure.
- Si une mise à jour rend une question de quiz ou une correction fausse, corrigez-la ou signalez-la dans la même passe.

## Mode d'intervention

- **Corrections mineures** (coquille, numérotation, lien cassé, mise en forme) : appliquez-les directement et mentionnez-les.
- **Tout le reste** (réécriture de fond, ajout ou suppression de section, changement de plan, nouvelle question de quiz) : proposez d'abord, appliquez après accord.
- Quand une demande est ambiguë, posez la question au lieu de supposer.
- Après modification, résumez ce qui a changé, ce qui n'a pas été vérifié, et ce qui reste à faire. Si vous n'avez pas relu le fichier ensuite, dites-le.
- Ne supprimez pas un contenu existant sans le signaler. Le dépôt est git, mais les modifications sont rarement commitées au fil de l'eau : mentionnez ce que vous retirez.
- Ne committez pas sauf demande explicite.

## Modules

### Module Dev (actif)

Voir `Module Dev/_programme.md`. Six chapitres, un glossaire en fin de théorie et une évaluation théorique de 20 questions (score de passage 75 %). Ce workspace couvre la théorie. Le module pratique Copilot est hors périmètre.

### Module Testing (à venir)

Le dossier est vide. Il visera les testeurs, avec une partie théorique et une partie pratique. Avant de rédiger quoi que ce soit, il faudra définir et consigner ici :

- le public (profils de testeurs, niveau attendu) ;
- le plan, la durée et la structure, sur le modèle de `_programme.md` ;
- ce qui est commun avec le module Dev (chapitres de base, glossaire) et ce qui est propre ;
- le format de la pratique (outils, jeu de données ou application support).

Tant que ces points ne sont pas actés, ne rédigez pas de contenu pour ce module et ne supposez pas qu'il reprend la structure du module Dev.
