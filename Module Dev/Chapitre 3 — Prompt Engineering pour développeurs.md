# Chapitre 3 — Prompt Engineering pour développeurs

---

## 3.1 Pourquoi le prompt engineering pour dev est différent

Un prompt généraliste cherche à obtenir une réponse utile. Un prompt de développeur cherche à obtenir du code correct, cohérent avec une codebase existante, dans un format précis, sur un domaine technique spécifique.

Les exigences sont plus strictes : un texte approximatif peut être acceptable, un code approximatif introduit des bugs. La tolérance à l'approximation est fondamentalement différente.

Trois différences structurelles :

**Le contexte est décisif.**
Un LLM sans contexte projet va générer du code générique. Avec le bon contexte (architecture, conventions, contraintes métier) il génère du code cohérent avec ce qui existe. La qualité du contexte fourni est souvent plus déterminante que le modèle utilisé.

**Le format de sortie compte autant que le contenu.**
En usage généraliste, le format de la réponse est secondaire. En dev, recevoir du code dans le mauvais langage, avec les mauvaises conventions de nommage, ou sans les imports nécessaires crée du travail supplémentaire. Spécifier le format attendu fait partie du prompt.

**La validation est systématique.**
En usage généraliste, on lit la réponse et on juge. En dev, chaque ligne de code produite doit être comprise et validée. Un bon prompt réduit la surface de validation en contraignant le modèle dans la bonne direction dès le départ.

---

## 3.2 Anatomie d'un prompt de dev efficace

Un prompt de dev structuré comporte cinq éléments. Tous ne sont pas toujours nécessaires mais chaque élément absent est une hypothèse que le modèle fera à votre place.

![Anatomie d'un prompt de dev efficace](./assets/fig3.01.png "Anatomie d'un prompt de dev efficace")

### Rôle

Définir le rôle du modèle ancre le registre technique de la réponse.

```
Tu es un développeur senior TypeScript spécialisé en architecture Node.js.
```

L'effet est réel : un modèle à qui on donne un rôle technique précis produit un code plus proche des conventions du domaine que le même modèle sans rôle.

### Contexte codebase

Ce que le modèle ne sait pas, il va l'inventer. Lui fournir le contexte pertinent évite les hallucinations sur l'architecture et les conventions existantes.

```
Contexte :
- Framework : Express 4.18, TypeScript strict
- ORM : Prisma 5.x
- Pattern : controllers séparés des services, pas de logique métier dans les routes
- Tests : Vitest
- Les erreurs HTTP sont gérées via un middleware centralisé, pas dans chaque route
```

### Spécification de la tâche

La tâche elle-même, formulée précisément. Pas "ajoute une feature de paiement", la description complète de ce qui doit être produit, incluant les cas limites et les contraintes métier.

```
Tâche : implémenter la méthode processRefund(orderId, amount) dans OrderService
Contraintes :
- amount ne peut pas dépasser le total de la commande originale
- Un remboursement partiel est possible
- Si la commande est déjà remboursée intégralement, retourner une erreur métier (pas une erreur 500)
- Loguer l'opération avec winston (instance importée depuis @/lib/logger)
```

### Contraintes de format

Ce que vous attendez comme output, pas seulement comme contenu.

```
Format attendu :
- Uniquement le code de la méthode, pas la classe entière
- Pas d'imports (ils sont déjà en place)
- Commentaires uniquement sur les parties non évidentes
- TypeScript strict, pas de any
```

### Exemples (si nécessaire)

Un exemple concret de ce qu'on attend. Particulièrement utile pour des patterns inhabituels ou des conventions spécifiques au projet.

```
Exemple de pattern d'erreur métier dans ce projet :
throw new BusinessError('REFUND_EXCEEDS_TOTAL', {
  orderId,
  requestedAmount: amount,
  maxRefundable: order.total
});
```

---

## 3.3 Les techniques classiques

### Zero-shot prompting

Pas d'exemple, uniquement l'instruction. Fonctionne bien pour les tâches simples et bien définies où le modèle a suffisamment de contexte.

```
Écris une fonction TypeScript qui valide qu'un email a un format correct.
Retourne un boolean. Pas de dépendances externes.
```

### Few-shot prompting

Deux à cinq exemples dans le prompt pour guider le format ou le style. Particulièrement efficace pour les tâches répétitives où le pattern doit être cohérent.

```
Génère des messages d'erreur selon ce pattern :

Exemple 1 :
Code : ORDER_NOT_FOUND
Message : "La commande {orderId} est introuvable."

Exemple 2 :
Code : PAYMENT_DECLINED
Message : "Le paiement a été refusé. Vérifiez vos informations bancaires."

Génère maintenant :
Code : DISCOUNT_EXPIRED
Code : INSUFFICIENT_STOCK
Code : ADDRESS_INVALID
```

### Chain of thought

Demander au modèle de raisonner étape par étape avant de produire le code. Améliore significativement la qualité sur les algorithmes complexes et les problèmes de logique non triviale.

```
Avant d'écrire le code, explique ta stratégie pour implémenter
un rate limiter en mémoire qui supporte plusieurs fenêtres temporelles
(par minute ET par heure). Identifie les cas limites à gérer.
Ensuite seulement, écris l'implémentation.
```

Sans chain of thought, le modèle génère souvent une implémentation qui gère l'un des cas mais pas l'autre. Avec, il identifie la complexité avant de coder.

### Itération guidée

Le premier résultat n'est pas toujours le bon. Plutôt que de regénérer depuis zéro, guider l'itération est généralement plus efficace.

```
// Premier échange
"Génère la fonction de calcul de TVA"

// Après review
"La fonction ne gère pas le cas des produits exonérés de TVA (taux = 0).
Modifie uniquement la logique de sélection du taux, ne touche pas au reste."
```

La précision de la correction ("modifie uniquement X, ne touche pas à Y") évite que le modèle ne réécrive tout ce qui fonctionnait déjà.

---

## 3.4 Les fichiers de contexte projet

Un fichier de contexte projet est un prompt permanent qui vit à la racine de votre codebase qui sera envoyée au modèle avant chaque échange dans le cadre de ce projet.

C'est la façon la plus efficace de contextualiser un assistant IA à une codebase sans répéter le contexte à chaque prompt.

Chaque outil a sa convention historique :

- GitHub Copilot : `.github/copilot-instructions.md`
- Claude Code : `CLAUDE.md`
- Cursor : `.cursorrules`

Le contenu est identique dans les trois cas : c'est du texte libre structuré. Mais depuis 2025, cette fragmentation a un remplaçant.

### Vers un standard unique : AGENTS.md

`AGENTS.md` est un fichier unique à la racine du projet, lu nativement par la majorité des outils actuels (Copilot, Cursor, Codex, Gemini CLI, Windsurf, Zed...). Il répond directement au problème posé par la liste ci-dessus : maintenir plusieurs fichiers au contenu identique, avec le risque qu'ils divergent dès que l'un est mis à jour et pas les autres.

Sa gouvernance suit celle de MCP (Chapitre 1) : porté par l'Agentic AI Foundation, sous l'égide de la Linux Foundation.

En pratique : écrire `AGENTS.md` comme fichier de référence. Les outils qui exigent encore leur propre convention (Claude Code avec `CLAUDE.md`, par exemple) peuvent pointer dessus via un fichier minimal :

```markdown
# CLAUDE.md
Voir AGENTS.md à la racine du projet — ce fichier n'est qu'un pointeur pour Claude Code.
```

Ça évite la duplication et la divergence, sans attendre que tous les outils convergent complètement.

### Ce qu'on y met

```markdown
# Contexte projet — API Orders

## Stack

- Node.js 20, TypeScript 5.x strict
- Express 4.18
- Prisma 5.x (PostgreSQL). Pas de requêtes SQL directes
- Vitest pour les tests, pas Jest
- Winston pour les logs

## Architecture

- Structure : controllers / services / repositories
- Pas de logique métier dans les controllers ni les routes
- Les erreurs métier sont des instances de BusinessError (voir src/errors/BusinessError.ts)
- Toutes les routes sont authentifiées sauf /health et /auth/\*

## Conventions

- Nommage : camelCase pour les variables et fonctions, PascalCase pour les classes
- Les fonctions async doivent toujours avoir un try/catch ou être wrappées dans asyncHandler
- Pas de any, utiliser unknown et type guards si nécessaire
- Les imports de types : import type { ... }

## Ce qu'il ne faut pas faire

- Ne pas modifier les migrations Prisma existantes
- Ne pas utiliser console.log — utiliser logger.info / logger.error
- Ne pas installer de nouvelles dépendances sans validation explicite
```

### Ce qu'on n'y met pas

Le fichier de contexte n'est pas un cahier des charges complet. Il doit rester lisible et ciblé sur ce qui impacte la génération de code au quotidien. Un fichier trop long dilue les instructions importantes.

À éviter : les descriptions fonctionnelles de features, les conventions évidentes, les informations qui ne changent pas la façon dont le code est généré.

Pour les procédures ponctuelles (générer des tests selon un pattern précis, faire une revue de PR selon une checklist), une skill (Chapitre 1) est plus adaptée que d'alourdir ce fichier : c'est exactement la distinction entre contexte permanent et savoir-faire à la demande.

---

## 3.5 Les erreurs classiques

### Trop vague

```
// Mauvais
"Améliore cette fonction"

// Mieux
"Refactorise cette fonction pour extraire la logique de validation dans
une fonction séparée validateOrderInput(). Garde la même signature externe."
```

Sans critère de succès explicite, le modèle optimise selon ses propres heuristiques qui ne correspondent pas forcément aux vôtres.

### Trop long

Un prompt de 2 000 tokens sur un contexte de 128k semble raisonnable. Mais plus le prompt est long, plus le modèle doit allouer de l'attention pour l'analyser, et plus il risque de perdre de vue certaines contraintes. Un prompt dense et bien structuré vaut mieux qu'un prompt long et exhaustif.

### Contexte manquant sur les dépendances

```
// Mauvais — le modèle va inventer une implementation Redis
"Ajoute du caching sur cette route"
```

```
// Mieux
"Ajoute du caching sur cette route en utilisant l'instance Redis
exportée depuis src/lib/redis.ts. TTL de 5 minutes. Invalide le cache
quand l'entité est mise à jour."
```

### Absence de validation demandée

```
// Mauvais
"Génère les tests pour cette fonction"

// Mieux
"Génère les tests Vitest pour cette fonction. Couvre au minimum :
le cas nominal, les cas limites (valeur nulle, valeur négative),
et les cas d'erreur (entité non trouvée, violation de contrainte).
Après génération, liste les cas que tu n'as PAS couverts."
```

La dernière ligne, "liste les cas que tu n'as pas couverts", est particulièrement efficace : elle force le modèle à évaluer lui-même la complétude de sa réponse.

---

## 3.6 Exercice guidé

Voici trois prompts naïfs réels. Réécrivez chacun en appliquant les principes du chapitre, puis comparez avec la version proposée.

---

**Prompt 1 — naïf**

```
Génère une API REST pour la gestion utilisateur
```

_Problèmes : pas de stack, pas de conventions, pas de scope (quelles routes ?), pas de format de réponse attendu._

<details>
<summary><strong>Version améliorée</strong></summary>

```
Tu es un développeur senior Node.js/TypeScript.

Contexte :
- Express 4.18, TypeScript strict
- Pattern RESTful standard
- Authentification JWT déjà en place (middleware auth disponible)
- Erreurs gérées via un middleware centralisé

Tâche : génère les routes CRUD pour la ressource User (GET /users, GET /users/:id,
POST /users, PUT /users/:id, DELETE /users/:id).

Contraintes :
- Pas de logique métier dans les routes — appels au UserService uniquement
- DELETE est un soft delete (champ deletedAt, pas de suppression physique)
- Valider req.body avec zod sur POST et PUT

Format attendu : uniquement le fichier userRouter.ts, sans le UserService.
```

</details>

---

**Prompt 2 — naïf**

```
Trouve le bug dans mon code
[colle 300 lignes de code]
```

_Problèmes : pas de description du symptôme, pas de contexte sur quand le bug se produit, le modèle va scanner 300 lignes sans savoir quoi chercher._

<details>
<summary><strong>Version améliorée</strong></summary>

```
Contexte : fonction de calcul de prix dans un panier e-commerce.

Symptôme observé : quand un utilisateur applique un code de remise en pourcentage
après avoir déjà appliqué une remise en valeur absolue, le total devient négatif
sur les petites commandes.

Reproduire : order.total = 5€, remise absolue de 3€ → total = 2€,
puis remise de 50% → total = -0.5€ (attendu : 1€).

Analyse uniquement la logique de calcul des remises dans le code suivant.
Ne propose pas de refacto, juste la correction du bug.

[colle uniquement la fonction de calcul, pas les 300 lignes]
```

</details>

---

**Prompt 3 — naïf**

```
Écris des tests pour cette fonction
```

_Problèmes : pas de framework de test, pas de cas à couvrir, pas de format de mock attendu._

<details>
<summary><strong>Version améliorée</strong></summary>

```
Écris les tests Vitest pour la fonction processRefund() ci-dessous.

Framework : Vitest avec vi.mock() pour les dépendances
Mocking : mocker le repository (OrderRepository) et le logger (winston)

Cas à couvrir obligatoirement :
1. Remboursement partiel valide
2. Remboursement total valide
3. Montant supérieur au total de la commande → BusinessError
4. Commande déjà intégralement remboursée → BusinessError
5. Commande non trouvée → BusinessError

Après avoir écrit les tests, liste les cas limites que tu n'as pas couverts.

[colle la fonction processRefund()]
```

</details>

---

## ✅ Quiz — Chapitre 3

---

**Question 1.** Parmi les cinq éléments d'un prompt de dev structuré, lequel est souvent le plus déterminant pour la qualité du code généré sur une codebase existante ?

- A) Le rôle donné au modèle
- B) Le contexte codebase (stack, conventions, patterns existants)
- C) La longueur de la spécification de la tâche
- D) Le nombre d'exemples few-shot fournis

> ✅ **Réponse : B** — Sans contexte codebase, le modèle génère du code générique qui peut être correct en isolation mais incohérent avec ce qui existe. La qualité du contexte fourni est souvent plus déterminante que le modèle utilisé.

---

**Question 2.** Un développeur génère des messages d'erreur standardisés pour une dizaine de cas différents. Quelle technique de prompting est la plus adaptée ?

- A) Zero-shot. La tâche est simple et bien définie
- B) Chain of thought. Le modèle doit raisonner sur chaque cas
- C) Few-shot. Deux ou trois exemples du pattern attendu suffisent à cadrer tous les cas
- D) Itération guidée. Générer un par un et corriger au fil de l'eau

> ✅ **Réponse : C** — Le few-shot est conçu pour les tâches répétitives où le pattern doit être cohérent. Deux ou trois exemples cadrent le format, le ton et la structure de tous les cas suivants.

---

**Question 3.** Pour quelle raison demande-t-on au modèle d'exposer son raisonnement étape par étape (chain of thought) avant d'écrire le code ?

- A) Pour que la réponse soit plus longue et donc plus détaillée
- B) Pour que le modèle identifie la complexité et les cas limites avant de commencer à coder, ce qui améliore la qualité sur les problèmes non triviaux
- C) Pour pouvoir copier l'explication dans la documentation
- D) Parce que certains modèles refusent de coder sans expliquer leur approche

> ✅ **Réponse : B** — Le chain of thought force le modèle à structurer sa réflexion avant d'exécuter. Sur les algorithmes complexes, cette étape identifie des cas limites que le modèle aurait ignorés en générant directement.

---

**Question 4.** Quelle est la différence entre `.github/copilot-instructions.md` et un prompt envoyé dans le chat Copilot ?

- A) Il n'y a pas de différence, les deux sont traités de la même façon
- B) Le fichier d'instructions est un contexte permanent injecté avant chaque échange dans le projet, le prompt chat, lui, est ponctuel
- C) Le fichier d'instructions est réservé aux administrateurs, le prompt chat est pour les développeurs
- D) Le fichier d'instructions supporte uniquement des règles de linting, pas du contexte architectural

> ✅ **Réponse : B** — Le fichier de contexte projet est un prompt permanent : il est injecté automatiquement avant chaque échange. C'est la façon la plus efficace de contextualiser l'assistant à la codebase sans répéter le contexte à chaque prompt.

---

**Question 5.** Un développeur écrit un prompt de 1 500 tokens très détaillé pour une tâche complexe. Il obtient une réponse qui ignore plusieurs de ses contraintes. Quelle est la cause la plus probable ?

- A) Le modèle a atteint sa limite de context window
- B) La température était trop élevée pour une tâche aussi précise
- C) Un prompt trop long dilue l'attention du modèle sur les instructions importantes
- D) Le modèle ne supporte pas les prompts de plus de 1 000 tokens

> ✅ **Réponse : C** — Plus un prompt est long, plus le modèle doit allouer d'attention pour l'analyser, et plus il risque de perdre de vue certaines contraintes. Un prompt dense et bien structuré est préférable à un prompt long et exhaustif.

---

**Question 6.** Quelle instruction dans un prompt de génération de tests améliore la complétude de façon particulièrement efficace ?

- A) "Génère le maximum de tests possible"
- B) "Après génération, liste les cas que tu n'as pas couverts"
- C) "Assure-toi que la couverture de code est supérieure à 80%"
- D) "Génère des tests pour tous les chemins d'exécution"

> ✅ **Réponse : B** — Demander au modèle de lister ce qu'il n'a pas couvert force une auto-évaluation de sa propre réponse. C'est une technique simple qui révèle quasi systématiquement des cas limites manquants.

---

**Question 7.** Dans un contexte d'itération guidée, un développeur reçoit une implémentation correcte à 80%. Quelle approche est la plus efficace pour corriger le 20% restant ?

- A) Regénérer depuis zéro avec un prompt amélioré
- B) Spécifier précisément ce qui doit changer et ce qui ne doit pas être touché
- C) Demander au modèle d'améliorer sa réponse sans plus de détail
- D) Diviser la tâche en deux prompts séparés

> ✅ **Réponse : B** — La précision de la correction ("modifie uniquement X, ne touche pas à Y") évite que le modèle ne réécrive les parties qui fonctionnaient déjà. Regénérer depuis zéro perd l'acquis du premier échange.

---

**Question 8.** Un développeur donne ce prompt : "Ajoute du caching sur cette route." Quel est le risque principal de cette formulation ?

- A) Le modèle va refuser de répondre sans plus de contexte
- B) Le modèle va générer une implémentation de caching qui ne correspond pas à la bibliothèque utilisée dans le projet
- C) Le modèle va générer trop de code et dépasser la context window
- D) Le modèle va demander des clarifications avant de répondre

> ✅ **Réponse : B** — Sans spécifier la bibliothèque de caching utilisée (Redis, Memcached, cache mémoire custom), le modèle va choisir selon ses heuristiques ou inventer une implémentation. Le contexte manquant sur les dépendances est une des erreurs les plus fréquentes.
