Voici le plan affiné.

---

## Module "Développeur Augmenté par l'IA" — 6h e-learning asynchrone

**Public** : Développeurs internes ESN, niveaux hétérogènes

**Registre** : Analogies conservées, profondeur technique assumée — on nomme les mécaniques

**Structure** : 4h théorie outil-agnostique + 2h pratique extractible (Copilot Free)

---

### Chapitre 1 — Ce que l'IA change pour un développeur (30 min)

- Bref historique : du NLP aux LLM, l'arrivée des API
- Mécanique essentielle : tokens, context window, température. Ce que ça implique concrètement pour coder
- Ce que le modèle fait vraiment : prédiction vs compréhension, hallucinations, limites sur le code
- Le marché des modèles : GPT, Claude, Gemini, Mistral, positionnement rapide depuis l'angle dev
- Passerelle vers le Chapitre 2 : ce qu'il faut retenir, et la question de la place de l'IA dans la pratique du développeur
- Quiz 5-10 questions

---

### Chapitre 2 — Vibe Coding vs Agentic Coding (30 min)

- Le spectre de délégation : autocomplétion → chat → génération guidée → edit multi-fichiers → agent autonome, et la posture adoptée face au code produit
- Vibe coding : définition, pourquoi c'est tentant, les dangers concrets
  - Accepter sans comprendre : bugs silencieux, dette technique invisible
  - Perte progressive de maîtrise sur le code produit
  - Pourquoi "ça marche" n'est pas suffisant en contexte professionnel ESN (code client, maintenabilité, revue)
- Agentic coding : la posture du développeur augmenté
  - Le workflow Specify → Plan → Implement → Validate en détail
  - Le dev reste aux commandes : déléguer n'est pas abdiquer
  - Ce qui change et ce qui ne change pas : compétences, responsabilité, qualité
- Le spectre entre les deux : pas un binaire, une posture qui se choisit selon le contexte et la criticité
- Exercice de réflexion : sur tes tâches de dev actuelles, où te situes-tu sur ce spectre ?
- Quiz 5-10 questions

---

### Chapitre 3 — Instruire l'agent : du prompt au savoir-faire (45 min)

- Pourquoi le prompt pour dev est différent du prompt généraliste
- Anatomie d'un bon prompt de dev : rôle, contexte codebase, spec, contraintes, format attendu
- Techniques appliquées au code : zero-shot, few-shot, itération guidée, chain of thought (pattern aujourd'hui intégré aux modèles, présenté « derrière le rideau »)
- Les fichiers de contexte : `AGENTS.md` comme standard unique, conventions propres à chaque outil, ce qu'on y met et ce qu'on n'y met pas
- Skills : figer le savoir-faire d'une tâche récurrente, chargé à la demande
- Hooks : faire appliquer à coup sûr les règles critiques (déterministe, contrairement à une instruction)
- Assembler le tout : `AGENTS.md`, skills et hooks sur un même projet, et où va chaque règle
- Les erreurs classiques : trop vague, trop chargé, contexte manquant, pas de validation demandée
- Exercice : diagnostiquer un hook de protection mal écrit (auto-corrigé)
- Quiz 5-10 questions

---

### Chapitre 4 — Panorama des outils (30 min)

- Les deux familles : IDE full IA (Cursor, Windsurf, Zed) vs assistants in-IDE (Copilot, Claude Code, Gemini CLI)
- Ce qui différencie les approches : gestion du contexte projet, autonomie, modèle sous-jacent
- MCP : ce que c'est, pourquoi ça change quelque chose pour les agents, écosystème actuel
- Matrice de choix : stack, contrainte enterprise, budget, cas d'usage
- Enjeux enterprise : confidentialité du code, politique de rétention, licences, shadow IT
- Pourquoi le module pratique se concentre sur Copilot — et comment transférer les compétences à d'autres outils
- Quiz 5-10 questions

---

### Chapitre 5 — Workflows avancés et qualité (45 min)

Passer de l'usage individuel structuré aux cas à fort ROI et aux limites à connaître.

- Gestion du contexte sur de gros projets : découpage des tâches, ne pas saturer la context window
- Les cas à fort ROI : legacy comprehension, génération de tests, documentation, refacto
- Les cas à risque : sécurité, logique métier critique, architecture — quand reprendre la main
- Revue de code augmentée : l'IA comme premier reviewer, ce qu'elle voit et ce qu'elle rate
- Human-in-the-loop : valider chaque étape, comprendre ce qui est produit
- Exercice : identifier dans une liste de tâches celles qui sont à fort ROI vs à risque élevé
- Quiz 5-10 questions

---

### Chapitre 6 — Développement en équipe augmentée (30 min)

Passer de l'usage individuel à une pratique d'équipe cohérente — directement applicable en mission.

- Standards partagés : fichier de contexte d'équipe, prompts réutilisables, conventions communes
- Onboarding accéléré : comment l'IA réduit le temps de montée en charge sur un projet
- Intégration CI/CD : pre-commit hooks augmentés, review automatisée
- Risques collectifs : IP, sécurité du code, dépendance, homogénéisation — et comment les adresser
- Ce que ça change pour tes missions ESN : attentes des clients, différenciation, pratiques à proposer
- Quiz 5-10 questions

---

### Glossaire (intégré en fin de théorie)

Base du glossaire commerciaux, adapté et enrichi côté dev :

Conserver : LLM, Prompt/Tokens, Context window, RAG, Agents, MCP, Tool calling, Vibe coding, Agentic coding, Workflow agentic, Fine-tuning, On-premise, Shadow IT IA

Ajouter : Temperature, Hallucination (plus développée), Embeddings, Chunking, Retrieval, Structured output, System prompt, Few-shot / Zero-shot, Chain of thought

Supprimer : Copilot Enterprise (trop commercial pour ce public)

---

### Évaluation théorique (20 min)

- QCM final 20 questions couvrant les 6 chapitres
- Score de passage : 75%
- Correction commentée accessible après soumission

---

### Module Pratique A — GitHub Copilot Free (2h)

Extractible — remplaçable par Module B (Claude Code), C (Cursor), etc.

**Lab 1 — Installation et premiers réflexes (30 min)**
Activation Copilot Free, extension VS Code/JetBrains, configuration, premiers usages guidés

**Lab 2 — Workflow agentic sur la codebase fournie (45 min)**
Codebase Node/TypeScript e-commerce fictive. Specify → Plan → Implement → Validate sur une feature bornée. Écriture du `copilot-instructions.md`. Agent mode sur une tâche multi-fichiers.

**Lab 3 — Cas d'usage avancés (30 min)**
Legacy comprehension, génération de tests sur code non testé, exercice de vigilance : identifier les erreurs introduites par Copilot.

**Évaluation pratique (15 min)**
10 questions situationnelles ancrées sur les labs. Score de passage : 70%.

---

## Ce qui change par rapport au plan initial

- **Chapitre 3** enrichi : fichiers de contexte intégrés (sortis du Chapitre 5)
- **Chapitre 5** recentré sur ROI/risque et human-in-the-loop
- **Glossaire** plus fourni côté dev : embeddings, chunking, structured output, few-shot, CoT
- **Exercices** ajoutés dans les chapitres 2, 3 et 5 pour sortir du texte pur
- **Chapitre 6** ajoute un angle ESN explicite : ce que ça change pour les missions

---

Ce plan te convient ? Si oui on attaque le Chapitre 1.