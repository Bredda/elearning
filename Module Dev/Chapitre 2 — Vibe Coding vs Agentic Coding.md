# Chapitre 2 — Vibe Coding vs Agentic Coding

---

## 2.1 Le spectre de délégation

L'IA dans le développement n'est pas un interrupteur on/off. C'est un spectre continu, et la bonne position dépend de la tâche, du contexte et de la maîtrise du domaine.

![Le spectre de délégation](./assets/fig.1.11.png "Le spectre de délégation")

**Autocomplétion inline**
Le modèle complète ce qui est tapé. Le contrôle reste total : chaque suggestion peut être acceptée ou refusée. Risque minimal pour valeur immédiate. C'est le mode le plus universel.

**Chat / Q&A**
Une question, une réponse. Utile pour déboguer, comprendre une API inconnue, expliquer du legacy. Le développeur intègre manuellement ce qui est pertinent.

**Génération guidée**
Une tâche est décrite, le modèle génère une fonction, un test, un composant. Le développeur lit et valide avant d'intégrer. C'est là que la qualité du prompt commence à compter vraiment.

**Edit mode / multi-fichiers**
Le modèle modifie plusieurs fichiers selon les instructions. Le développeur review les diffs avant d'accepter.

**Agent autonome**
Le modèle planifie, exécute et itère de façon autonome sur une tâche complexe. Le développeur spécifie l'objectif et valide à des checkpoints. C'est là que la démarche agentique (§2.3) devient indispensable.

Plus on avance vers la droite du spectre, plus la délégation est large et plus la validation doit être rigoureuse. Quel que soit le mode, ce qui compte est la posture adoptée face au code produit. Deux postures s'opposent :

![Vibe Coding vs Agentic Coding](./assets/fig.1.12.png "Vibe Coding vs Agentic Coding")

Les deux sections suivantes détaillent le vibe coding (§2.2), puis l'agentic coding (§2.3).

---

## 2.2 Le vibe coding

Le terme a été popularisé par Andrej Karpathy début 2025 pour décrire une pratique déjà répandue : générer du code avec l'IA en se fiant au fait que "ça a l'air de marcher", sans forcément le lire ligne à ligne, le comprendre en détail ou le valider formellement. C'est un mode de collaboration avec l'IA à part entière, pas un accident de parcours. Reste à savoir dans quel contexte il est pertinent.

### Pourquoi c'est tentant — et parfois légitime

Le vibe coding produit des résultats visibles immédiatement. Une feature qui aurait pris une journée prend une heure. Les tests passent. La démo fonctionne. Pour un prototype jetable, une exploration personnelle d'un framework inconnu, un POC de démonstration en hackathon, c'est le bon choix : la vitesse prime légitimement sur la compréhension exhaustive quand le code n'a pas vocation à survivre.

Le risque apparaît quand ce même code change de destination, lorsqu'il glisse en production, ou devient la base d'une feature livrée à un client (voir §2.5), sans repasser par une phase de compréhension et de validation. Le critère qui fait basculer le vibe coding en risque n'est donc pas le mode de génération, c'est **la destination du code**. C'est dans ce cas que les problèmes suivants deviennent réels.

### Ce que ça produit, sans review

**Des bugs silencieux.**
Le modèle génère du code syntaxiquement correct mais logiquement faux. Une condition inversée sur un cas limite, une gestion d'erreur manquante, une hypothèse implicite sur la forme des données : le code compile, les tests unitaires passent, mais le comportement est incorrect dans des situations que les tests ne couvrent pas.

**De la dette technique invisible.**
Du code généré sans compréhension est du code que personne ne sait maintenir. Dans un contexte professionnel, c'est le code que l'organisation devra faire évoluer pendant des années. Si le développeur qui l'a écrit ne comprend pas la logique, personne ne pourra l'expliquer lors des passations.

**Une perte de maîtrise progressive.**
Plus on accepte sans comprendre, moins on est capable de guider l'IA dans la bonne direction lors de l'itération suivante. L'IA s'appuie sur ce qu'on lui fournit : si le contexte est du code non maîtrisé, la qualité des suggestions se dégrade d'autant plus au fil du temps.

**Des hallucinations non détectées.**
Une méthode inventée qui ressemble à la bonne syntaxe, une dépendance qui n'existe pas dans la version utilisée, une API dont le comportement a changé : sans review, ces erreurs passent. Certaines n'apparaissent qu'en production.

```typescript
// Exemple typique d'hallucination non détectée en vibe coding
// Le modèle a inventé .findByIdAndPopulate() qui n'existe pas
const user = await User.findByIdAndPopulate(id, "orders");
// Erreur silencieuse jusqu'au premier appel en prod
```

### Ce que le vibe coding n'est pas

Il est important de distinguer le vibe coding de l'acceptation rapide sur des tâches simples. Accepter une suggestion d'autocomplétion d'une ligne après l'avoir lue n'est pas du vibe coding, accepter la génération d'un bloc de 200 lignes de logique métier complexe sans le lire, oui.

---

## 2.3 L'agentic coding

L'agentic coding part d'un principe simple : le développeur délègue l'exécution, pas la décision. L'IA génère, le développeur comprend, valide et reste aux commandes à chaque étape.

### Le workflow en détail

![L'agentic coding](./assets/fig2.01.png "L'agentic coding")

**SPECIFY**

Avant de demander quoi que ce soit au modèle, définir précisément ce qu'on souhaite obtenir. Pas "ajoute une feature de discount", mais :

```
Contexte : module de commande, fichiers Order.ts et OrderService.ts joints
Tâche : ajouter la gestion des codes de remise (discount codes)
Contraintes :
  - Un code s'applique une seule fois par utilisateur
  - Les codes expirés doivent retourner une erreur métier, pas une erreur 500
  - La réduction est en pourcentage ou en valeur absolue selon le type
Format attendu : modifications dans OrderService.ts + tests unitaires
```

Plus la spécification est précise, moins l'IA doit faire d'hypothèses, moins il y a de chances de divergence.

**PLAN**

Avant d'exécuter, demander au modèle de proposer un plan d'implémentation et le valider. C'est l'étape la plus sous-estimée du workflow.

```
Humain : "Propose un plan d'implémentation avant de coder."

IA : "Voici comment je compte procéder :
  1. Ajouter l'entité DiscountCode avec les champs type, value, expiresAt, usedBy[]
  2. Créer validateDiscountCode() dans OrderService qui vérifie expiration et usage
  3. Modifier createOrder() pour appeler validateDiscountCode() si un code est fourni
  4. Tests : code valide, code expiré, code déjà utilisé, code inexistant"

Humain : "Step 1 : ajouter usedBy[] directement sur DiscountCode crée un problème
        de scalabilité (la liste grossit avec chaque utilisateur). Préfère un
        enregistrement d'usage séparé (discountId, userId)."
```

Corriger le plan prend quelques secondes. Corriger l'implémentation après qu'elle a divergé dans la mauvaise direction peut prendre des heures.

**IMPLEMENT**

L'IA génère. Le développeur suit et peut intervenir à tout moment. Lorsqu'un agent exécute lui-même les modifications, cela signifie surveiller les actions effectuées, pas simplement attendre le résultat final.

Points de surveillance :

- Les fichiers modifiés correspondent-ils à ce qui était prévu ?
- Les dépendances introduites sont-elles justifiées ?
- Les patterns utilisés sont-ils cohérents avec le reste de la codebase ?

**VALIDATE**

Review du code produit avant toute intégration. Pas une lecture en diagonale, une review de la même qualité que celle que l'on ferait sur une PR d'un collègue.

Checklist minimale :

- La logique correspond à la spécification
- Les cas limites sont couverts
- Pas d'hallucinations (APIs inexistantes, types incorrects, imports manquants)
- Les tests couvrent les comportements critiques
- Le code est maintenable par quelqu'un qui ne connaît pas le contexte

---

## 2.4 Calibrer la délégation selon la tâche

Les deux postures ne s'excluent pas. Dans une même journée de travail, un développeur augmenté se positionne à des endroits différents selon le contexte :

- **Autocomplétion courte** → acceptation rapide sans review approfondie acceptable
- **Exploration d'une feature** → vibe coding rapide pour confronter l'idée au besoin
- **Génération d'une fonction critique** → review ligne par ligne
- **Tâche multi-fichiers confiée à un agent** → checkpoints réguliers, validation des diffs

Le tableau suivant généralise ce principe par type de tâche.

![Le spectre entre les deux](./assets/fig2.02.png "Le spectre entre les deux")

![Le spectre entre les deux](./assets/fig2.03.png "Le spectre entre les deux")

La règle générale : plus la tâche est critique et irréversible, plus la validation doit être rigoureuse. Un test mal généré est facile à corriger. Une faille de sécurité en production, beaucoup moins. La compétence clé : savoir calibrer le niveau de délégation et de validation selon la criticité de la tâche.

---

## 2.5 Ce que ça change concrètement

### Sur les gains de productivité

Les deux approches répondent à des besoins différents, pas au même besoin en mieux ou en moins bien. L'agentic coding vise des gains qui tiennent dans la durée, sur des tâches bien définies et destinées à durer. Le vibe coding produit un gain différent, sur un terrain différent : la vitesse d'itération sur un prototype, une exploration, un POC. Là où la structure de l'agentic coding serait un coût disproportionné par rapport à l'enjeu.

Les chiffres de productivité, eux, sont à lire avec prudence. En 2025, METR a mené un essai randomisé auprès de 16 développeurs expérimentés, travaillant sur leurs propres dépôts open source : avec des outils IA, leurs tâches ont pris 19 % de temps en plus, alors qu'ils estimaient, après coup, avoir été accélérés d'environ 20 %. Cette mesure concerne les outils de début 2025 et un contexte précis, elle ne se généralise pas telle quelle. En février 2026, METR indique que les gains sont probablement plus élevés avec les outils récents, mais juge ses nouvelles données trop biaisées pour en tirer un chiffre : des développeurs refusent désormais de travailler sans IA.

Retenez surtout que le ressenti de vitesse n'est pas une mesure, ce qui est une raison de plus de valider plutôt que de se fier à l'impression que « ça marche ». Dans le même sens, le rapport DORA 2025 (Google Cloud) conclut que l'IA agit comme un amplificateur : elle renforce les forces comme les faiblesses d'une organisation, et les gains viennent moins de l'outil que du système de travail qui l'entoure.

Le point de vigilance n'est pas de choisir l'un contre l'autre, mais de ne pas confondre les deux registres : les gains de vitesse du vibe coding ne se transposent pas tels quels sur du code destiné à durer. Ils s'effacent généralement lors de la phase de stabilisation, de debugging et de maintenance, si le code n'a jamais été conçu ni validé pour ça.

### Sur la qualité

Un développeur qui pratique l'agentic coding comprend ce qu'il intègre, avec une profondeur de revue proportionnée au risque : une lecture rapide sur du boilerplate, une review ligne par ligne sur une logique critique (§2.4). Il peut l'expliquer, le déboguer, le faire évoluer. C'est la définition de la qualité maintenable.

### Ce qui change, ce qui ne change pas

**Ce qui change** : la répartition du temps. Moins de code écrit à la main, davantage de spécification, de relecture et d'arbitrage. Les compétences les plus sollicitées deviennent la capacité à cadrer précisément une tâche, à lire du code qu'on n'a pas écrit et à repérer ce qui cloche.

**Ce qui ne change pas** : la responsabilité. Le code livré engage le développeur et l'ESN, qu'il ait été écrit à la main ou généré. Les compétences de fond (comprendre l'architecture, déboguer, tester) restent nécessaires : sans elles, on ne peut ni cadrer correctement, ni valider. Les exigences de qualité (review, tests, maintenabilité) restent les mêmes.

Déléguer l'exécution n'est pas abdiquer la décision.

### Sur les missions ESN

En contexte de mission, le code livré au client engage la responsabilité professionnelle, mais ça ne condamne pas le vibe coding en bloc pour autant, ça déplace simplement la question. 

Si la mission demande de prototyper une idée, d'explorer une piste technique ou de produire un POC pour valider une direction avant de s'engager, le vibe coding a toute sa place, au même titre qu'en usage personnel. Ce qui change avec l'expérience, c'est justement la capacité à distinguer les deux registres : quel outil, pour quelle tâche, et surtout quelle destination pour le code produit.

Le vibe coding reste en revanche inadapté à du code qui sera maintenu par des équipes client pendant des années. L'agentic coding positionne le développeur ESN comme un professionnel qui sait choisir le bon mode de collaboration avec l'IA selon l'enjeu, pas comme quelqu'un qui applique la même recette à toutes les tâches.

---

## 2.6 Exercice

### Partie 1 — Auto-évaluation

Sur les tâches listées ci-dessous, évaluez honnêtement votre comportement habituel avec un outil IA :

![Auto-évaluation](./assets/fig2.04.png "Auto-évaluation")

Identifiez la tâche où votre comportement est le plus proche du vibe coding. Rédigez en 3-4 lignes comment vous l'aborderiez différemment avec le workflow agentic : quelle spécification vous fourniriez, ce que vous valideriez dans le plan, ce que vous checkeriez dans la review.

---

### Partie 2 — Review de diff

Vous n'avez pas besoin de connaître TypeScript : l'exercice porte sur la logique, et sur la cohérence du code avec la spécification et avec les interfaces fournies. Seul repère de syntaxe : `await` signifie « attendre la réponse de la base de données ».

La demande faite à l'agent était :

> Ajoute une fonction qui applique un code de remise à une commande existante. Un code ne peut être utilisé qu'une seule fois par utilisateur. La remise est en pourcentage (`percent`) ou en valeur absolue (`amount`).

Voici l'API disponible dans le projet :

```typescript
interface Discount { id: string; type: "percent" | "amount"; value: number; expiresAt: Date }
interface Order    { id: string; userId: string; total: number }

interface DiscountRepository {
  findOne(code: string): Promise<Discount | null>;
  hasUsed(discountId: string, userId: string): Promise<boolean>;
  markUsed(discountId: string, userId: string): Promise<void>;
}
```

Et le code généré par l'agent :

```typescript
async function applyDiscount(order: Order, code: string): Promise<number> {
  const discount = await discounts.findByCode(code);

  if (discount.expiresAt < new Date()) {
    throw new Error("Code expiré");
  }

  if (await discounts.hasUsed(discount.id, order.id)) {
    throw new Error("Code déjà utilisé");
  }

  order.total = order.total - discount.value;

  await discounts.markUsed(discount.id, order.id);
  await orders.save(order);

  return order.total;
}
```

Passez-le en revue comme si c'était une PR d'un collègue. Identifiez tous les problèmes avant de lire la correction.

_Prenez le temps d'analyser avant de lire la correction ci-dessous._

---

<details>
<summary><strong>Correction — 6 problèmes identifiés</strong></summary>

**(1) Hallucination : `findByCode` n'existe pas.**
L'interface fournie propose `findOne(code)`. L'agent a inventé une méthode plausible, qui ressemble à ce qu'on attend d'un repository.

**(2) Le cas « code inconnu » n'est pas géré.**
`findOne` peut renvoyer `null` (son type de retour l'indique). Le code accède à `discount.expiresAt` sans le tester : un code inexistant provoque une erreur non maîtrisée au lieu d'une réponse métier claire.

**(3) Mauvaise contrainte métier : `order.id` à la place de l'identifiant de l'utilisateur.**
La spécification impose une utilisation par **utilisateur**, et `hasUsed` attend un `userId`. L'agent passe `order.id` : le code vérifie donc « un code par commande ». Les deux identifiants sont des chaînes de caractères, aucun compilateur ne signale l'erreur, et le code semble cohérent. Seule la comparaison avec la spécification la révèle. Il fallait utiliser `order.userId`, pour la lecture comme pour l'écriture.

**(4) Le type de remise est ignoré.**
Pour un code `percent` de valeur 10, le code retire 10 de la commande au lieu de 10 %. La spécification et l'interface (`type: "percent" | "amount"`) le disaient explicitement.

**(5) Deux écritures non atomiques.**
`markUsed` puis `save` sont deux opérations séparées. Si la seconde échoue, le code est marqué comme utilisé alors que la remise n'a pas été appliquée : l'utilisateur perd son code. Les deux écritures doivent réussir ou échouer ensemble (transaction).

**(6) Le total peut devenir négatif.**
Si la remise dépasse le montant de la commande, `order.total` passe sous zéro. Le cas limite n'est ni géré ni testé.

**Version corrigée.** `BusinessError` et `db.transaction` sont des éléments supposés du projet ; ce code n'a pas été exécuté.

```typescript
async function applyDiscount(order: Order, code: string): Promise<number> {
  const discount = await discounts.findOne(code);
  if (discount === null) throw new BusinessError("Code inconnu");
  if (discount.expiresAt < new Date()) throw new BusinessError("Code expiré");
  if (await discounts.hasUsed(discount.id, order.userId)) throw new BusinessError("Code déjà utilisé");

  const reduction = discount.type === "percent" ? order.total * discount.value / 100 : discount.value;
  order.total = Math.max(0, order.total - reduction);

  await db.transaction(async () => {   // les deux écritures réussissent ou échouent ensemble
    await discounts.markUsed(discount.id, order.userId);
    await orders.save(order);
  });

  return order.total;
}
```

</details>

---

## ✅ Quiz — Chapitre 2

_10 questions — environ 10 minutes_

---

**Question 1.** Sur le spectre de délégation, dans quel mode la validation humaine est-elle la plus critique ?

- A) Autocomplétion inline
- B) Chat / Q&A
- C) Agent autonome sur une tâche multi-fichiers
- D) Génération guidée d'une fonction

> ✅ **Réponse : C** — Plus la délégation est large et la tâche complexe, plus la validation est critique. Un agent autonome peut modifier de nombreux fichiers en chaîne. Chaque diff mérite une review attentive.

---

**Question 2.** Quelle est la définition précise du vibe coding ?

- A) Générer du code sans le lire, le comprendre ni le valider, en se fiant au fait que "ça semble marcher"
- B) Utiliser l'IA pour générer du code rapidement sur des tâches simples
- C) Utiliser plusieurs outils IA en parallèle pour comparer les résultats
- D) Générer du code en dehors des heures de travail pour éviter les interruptions

> ✅ **Réponse : A** — Le vibe coding se définit par l'absence de lecture, de compréhension et de validation, pas par la vitesse d'exécution — ce qui le distingue de l'acceptation rapide sur des tâches simples et bien comprises. Ce n'est pas une définition condamnatoire en soi : c'est un mode de collaboration légitime dans certains contextes (prototype, exploration) et risqué dans d'autres, selon la destination du code (§2.2).

---

**Question 3.** Un développeur accepte une suggestion d'autocomplétion d'une ligne après l'avoir lue et comprise. Est-ce du vibe coding ?

- A) Oui, toute acceptation rapide est du vibe coding
- B) Oui, si la ligne fait partie d'un bloc plus large non validé
- C) Non, le critère est la compréhension, pas la vitesse d'acceptation
- D) Cela dépend du type de fichier modifié

> ✅ **Réponse : C** — Le critère distinctif est la compréhension, pas la vitesse. Lire et comprendre une suggestion avant de l'accepter, ce n'est pas du vibe coding, même si ça prend deux secondes.

---

**Question 4.** Dans le workflow agentique, pourquoi l'étape PLAN est-elle souvent la plus sous-estimée ?

- A) Parce que le modèle refuse souvent de planifier avant d'implémenter
- B) Parce que corriger un plan divergent avant l'exécution prend quelques secondes, alors que corriger une implémentation divergente peut prendre des heures
- C) Parce que le plan généré par l'IA est toujours correct et ne nécessite pas de validation
- D) Parce que l'étape SPECIFY suffit à éviter toute divergence

> ✅ **Réponse : B** — Valider le plan avant d'exécuter est le meilleur investissement de temps dans le workflow. Une correction en amont de l'exécution coûte infiniment moins qu'une correction après que l'IA a implémenté dans la mauvaise direction.

---

**Question 5.** Votre agent propose un plan dont l'étape 1 vous paraît mal conçue. Que faites-vous ?

- A) Vous laissez faire et vous corrigerez à la review
- B) Vous lui demandez de tout recommencer sans autre précision
- C) Vous corrigez explicitement l'étape concernée et faites ajuster le plan avant de lancer l'implémentation
- D) Vous lancez l'implémentation et vous surveillez si elle dérive

> ✅ **Réponse : C** — Le plan est le moment le moins coûteux pour corriger : quelques secondes, contre des heures après divergence. A et D reportent la correction à un stade plus cher. B ne dit pas à l'agent ce qui ne va pas.

---

**Question 6.** Pour quelle tâche la délégation à l'IA doit-elle être la plus faible ?

- A) Génération de tests unitaires
- B) Refacto d'une fonction sur un domaine maîtrisé
- C) Génération de boilerplate répétitif
- D) Implémentation d'un mécanisme d'authentification

> ✅ **Réponse : D** — La sécurité et l'authentification sont dans la catégorie "très faible délégation" du tableau. Les erreurs dans ces domaines ont des conséquences graves et irréversibles. L'IA peut assister, mais chaque ligne doit être comprise et validée.

---

**Question 7.** Un développeur pratique l'agentic coding sur une feature de gestion de paiements. Il valide le plan, review les diffs, et vérifie que les tests couvrent les cas critiques. Il n'a pas relu la section de gestion des erreurs HTTP car "elle ressemble aux autres". Est-ce suffisant ?

- A) Non, la review doit couvrir l'intégralité du code produit, y compris les sections qui "ressemblent" aux autres
- B) Oui, si le plan était bon et les tests passent, la review partielle est acceptable
- C) Oui, les sections de gestion d'erreurs sont généralement correctement générées
- D) Cela dépend de la taille totale du diff

> ✅ **Réponse : A** — Sur du code de paiement, "ressemble aux autres" n'est pas suffisant. Les hallucinations produisent précisément du code qui ressemble à du bon code. La review doit être complète sur les domaines critiques.

---

**Question 8.** Un essai randomisé de METR (2025) a mesuré le temps de développeurs expérimentés avec et sans outils IA. Quel enseignement le chapitre en tire-t-il ?

- A) L'IA ralentit systématiquement les développeurs
- B) Les chiffres de productivité annoncés par les éditeurs sont tous faux
- C) Les développeurs expérimentés ne devraient pas utiliser d'outils IA
- D) Le ressenti de vitesse n'est pas une mesure : les développeurs se croyaient accélérés alors que leurs tâches avaient pris plus de temps

> ✅ **Réponse : D** — A, B et C généralisent à tort : METR précise que la mesure concerne les outils de début 2025 et un contexte précis, et que les outils récents donnent probablement de meilleurs résultats. Ce qui reste, c'est l'écart entre perception et mesure, qui justifie de valider plutôt que de se fier à l'impression que « ça marche ».

---

**Question 9.** Pourquoi le même usage du vibe coding est-il acceptable sur un prototype mais problématique sur du code de production ?

- A) Les outils IA génèrent un code de moins bonne qualité en environnement de production
- B) Sur un prototype, le code n'a pas vocation à être maintenu ; en production, il sera fait évoluer par des équipes qui n'ont pas le contexte de génération, sans que la compréhension ait jamais été vérifiée
- C) Les licences des outils IA ne couvrent pas l'usage en environnement de production
- D) Les prototypes ne contiennent jamais de logique métier critique

> ✅ **Réponse : B** — Ce qui change, ce n'est pas l'outil ni la qualité de génération, c'est la destination du code (§2.2). Un prototype jetable n'a pas besoin d'être compris, puisqu'il ne sera tout simplement pas maintenu. Du code de production le sera, par des personnes qui n'ont pas le contexte de sa génération, d'où la nécessité de la compréhension et de la validation vues en agentic coding.

---

**Question 10.** Dans l'exercice de revue de diff, l'agent passe `order.id` à `hasUsed` alors que la spécification impose une contrainte par utilisateur. Pourquoi un compilateur ne signale-t-il pas cette erreur ?

- A) Les deux identifiants sont des chaînes de caractères : le type est correct, seule la comparaison avec la spécification révèle l'erreur
- B) Le compilateur n'analyse pas les appels de méthodes
- C) Le compilateur ne vérifie que les fonctions asynchrones
- D) Le nom du paramètre fait foi, et l'agent l'a respecté

> ✅ **Réponse : A** — Un compilateur vérifie les types, pas le sens métier : `order.id` et `userId` sont tous deux des `string`, l'appel est valide et le code s'exécute sans erreur. B et C sont faux : un compilateur TypeScript signalerait par exemple `findByCode`, absente de l'interface. D est faux : le paramètre s'appelle bien `userId`, et c'est justement ce que l'agent n'a pas respecté. D'où la nécessité de confronter le code à la spécification pendant la review, y compris quand il compile et que rien ne plante.
