# La Guilde · Modèle de données et règles du domaine

v1.1 · 13/08/2026 · accompagne `openapi.yaml` et `codex.graphql`

Ce document décrit ce que le contrat ne dit pas : les entités, leurs relations et les règles métier qui s'exécutent côté serveur. C'est la référence commune aux deux modules. Le front l'utilise pour comprendre ce qu'il affiche, le back l'implémente dans sa couche domaine (séance 3).

Convention de langue, appliquée partout : le code est en anglais (identifiants, routes, champs, enums, codes techniques), les textes affichés aux utilisateurs sont en français (noms de badges, titres honorifiques, messages d'erreur), et la documentation est en français.

## Les entités

**Adventurer (aventurier).** Un membre de la Guilde. Il possède un solde de points d'entraide (`helpPoints`, la monnaie interne), un capital d'expérience (`xp`) qui ne descend jamais, et un rôle : `MEMBER` ou `GUILD_MASTER`. Le pseudo (`username`) est unique, l'email aussi. Le mot de passe est haché avec argon2id, jamais stocké en clair, jamais renvoyé par l'API.

**Quest (quête).** Une annonce postée au tableau : une demande de coup de main (`HELP`) ou une proposition d'échange (`BARTER`). Elle porte une récompense en points (`reward`), une catégorie, une photo facultative, et surtout un statut qui suit un cycle de vie strict. Une quête connaît son auteur (`author`) et, une fois prise, son preneur (`taker`).

**Badge.** Une distinction. La plupart sont automatiques : le domaine les attribue quand leurs conditions se réalisent. Le maître de guilde peut aussi en décerner à la main (le badge `GUILD_MASTERS_PICK`, « Coup de cœur »). Un badge donné ne s'obtient qu'une fois.

## Le cycle de vie d'une quête

Le diagramme d'états-transitions (`diagrammes/cycle-vie-quete.mmd`) est le schéma pivot du projet : c'est lui qui dicte quelles actions l'API accepte, et c'est lui que le front consulte pour savoir quels boutons afficher.

Les transitions et leurs gardiens :

| Transition | Action REST | Qui | Conditions |
|---|---|---|---|
| création → OPEN | `POST /quests` | tout membre | solde suffisant pour la récompense |
| OPEN → IN_PROGRESS | `POST /quests/{id}/take` | tout membre sauf l'auteur | le preneur n'a pas déjà une quête IN_PROGRESS |
| IN_PROGRESS → OPEN | `POST /quests/{id}/abandon` | le preneur | aucune |
| IN_PROGRESS → COMPLETED | `POST /quests/{id}/complete` | le preneur | aucune |
| COMPLETED → VALIDATED | `POST /quests/{id}/validate` | l'auteur | aucune |
| OPEN → CANCELLED | `POST /quests/{id}/cancel` | l'auteur | aucune |

Toute autre demande de transition répond `409` avec le détail des transitions permises depuis l'état courant. Deux règles complètent le tableau : une quête `OPEN` est la seule modifiable (par son auteur), et le maître de guilde peut supprimer n'importe quelle quête à des fins de modération (le séquestre est alors remboursé).

## L'économie des points d'entraide

Le principe du séquestre évite les impayés et donne un vrai sujet de transaction en base :

1. À l'inscription, chaque aventurier reçoit **100 points de bienvenue**.
2. À la création d'une quête, la récompense est **débitée immédiatement** du solde de l'auteur et mise sous séquestre. Pas assez de points, pas de quête : réponse `409`.
3. À la validation, le séquestre est **crédité au preneur**. Débit et crédit se font dans la même transaction que le changement de statut : tout passe ou rien ne passe.
4. À l'annulation ou à la suppression par le maître de guilde, le séquestre **revient à l'auteur**. La modification d'une quête `OPEN` qui change la récompense ajuste le séquestre dans les deux sens.

Les points circulent donc en circuit fermé : la somme des soldes et des séquestres est constante, aux points de bienvenue près. Une propriété qui se teste, et qui fera un joli exercice d'invariant en séance 3 du back.

## L'expérience et les niveaux

L'XP récompense l'activité validée, et elle ne s'achète pas :

- quête validée : **+25 XP pour le preneur**, **+10 XP pour l'auteur** (poster et suivre une quête, c'est aussi faire vivre la Guilde) ;
- l'XP ne descend jamais, même si les points circulent.

Les niveaux suivent des paliers fixes, lisibles et faciles à afficher. Les titres sont du contenu : ils restent en français.

| Niveau | XP cumulée | Titre honorifique |
|---|---|---|
| 1 | 0 | Novice |
| 2 | 50 | Apprenti |
| 3 | 150 | Compagnon |
| 4 | 300 | Aventurier confirmé |
| 5 | 500 | Expert |
| 6 | 800 | Maître d'œuvre |
| 7 | 1 200 | Légende de la Guilde |

Le niveau (`level`) et le titre (`honoraryTitle`) sont **calculés**, jamais stockés : c'est un exemple de donnée dérivée, et un point de discussion en cours (que stocke-t-on, que calcule-t-on ?).

## Les badges automatiques (v1)

Les codes sont des identifiants techniques (anglais, stables), les noms sont affichés (français).

| Code | Nom | Condition |
|---|---|---|
| `FIRST_QUEST` | Première quête | première quête réalisée et validée |
| `FIRST_POST` | Premier appel | première quête postée et validée |
| `THREE_IN_A_ROW` | Série de trois | trois quêtes validées en tant que preneur |
| `BIG_HEART` | Grand cœur | dix quêtes validées en tant que preneur |
| `BARTERER` | Troqueur | premier troc validé |
| `VERSATILE` | Polyvalent | quêtes validées dans trois catégories différentes |
| `GUILD_MASTERS_PICK` | Coup de cœur | décerné à la main par le maître de guilde |

Les conditions s'évaluent au moment de la validation d'une quête, dans le domaine, jamais dans le contrôleur. La liste est volontairement courte : en concevoir de nouveaux est une extension toute trouvée, et la parenthèse gamification du cours interroge justement la tentation d'en ajouter trop.

## Les invariants à tester (aide-mémoire pour la séance 3 du back)

1. La somme points en circulation + séquestres est constante, aux 100 points de bienvenue par inscription près.
2. Une quête `VALIDATED` a forcément un preneur, une date de prise, une date de fin et une date de validation, dans cet ordre.
3. Un aventurier n'a jamais deux quêtes `IN_PROGRESS` en tant que preneur.
4. Un auteur n'est jamais preneur de sa propre quête.
5. Un badge donné n'apparaît qu'une fois par aventurier.
6. L'XP d'un aventurier ne diminue jamais.

## Notes d'implémentation (pour l'API de référence et le S2)

- Identifiants en UUID v4, générés par le serveur.
- Dates en ISO 8601 UTC ; l'affichage en heure locale est l'affaire du front.
- Le mot de passe accepte 12 caractères minimum, sans règle de composition : une phrase de passe vaut mieux qu'un `P@ssw0rd!` (recommandations CNIL).
- La recherche `q` du tableau des quêtes est un `contains` insensible à la casse en v1 ; la recherche plein texte PostgreSQL est une extension.
- Les événements temps réel sont émis après le commit de la transaction, jamais avant : le WebSocket notifie, l'API REST fait foi.
