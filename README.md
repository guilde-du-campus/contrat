# La Guilde · Le contrat

Le contrat d'interface entre les deux modules M1 DFS : le front (S1) le consomme, le back (S2) l'implémente. Toute évolution passe par une nouvelle version, jamais par une modification silencieuse.

Version courante : **v1.1** (13/08/2026). Changement majeur depuis la v1 : tout le code est passé en anglais (routes, champs, enums, codes de badges, événements temps réel), conformément à la règle « on ne code jamais en français ». Les textes destinés aux utilisateurs (noms de badges, titres honorifiques, messages d'erreur) et la documentation restent en français.

## Contenu du dossier

| Fichier | Rôle |
|---|---|
| `openapi.yaml` | Le contrat REST v1.1, validé (Redocly lint), documenté en français |
| `contrat-openapi.html` | Le même contrat en version navigable (Redoc), pour la relecture et pour les étudiants |
| `codex.graphql` | Le schéma du Codex : la consultation en GraphQL, lecture seule |
| `MODELE_DE_DONNEES.md` | Les entités, le cycle de vie, l'économie des points, l'XP, les badges, les invariants |
| `diagrammes/cycle-vie-quete.mmd` | Diagramme d'états-transitions (Mermaid), le schéma pivot du projet |
| `diagrammes/classes-domaine.mmd` | Diagramme de classes du domaine (Mermaid) |
| `diagrammes-uml.html` | Les deux diagrammes rendus, aux couleurs Sergent.dev, pour relecture |

## Points validés le 13/08/2026

1. Cycle de vie et gardiens : passe de cohérence faite (contrat, doc, diagrammes, code et tests racontent la même chose), re-validation complète de bout en bout.
2. Économie du séquestre : validée, simple et efficace (100 points de bienvenue, débit à la création, crédit à la validation, remboursement à l'annulation et à la modération).
3. Langue du code : anglais partout, règle définitive pour toutes les formations dev.
4. Barèmes XP, niveaux et badges : validés tels quels.

## Statut

v1.1 gelée et implémentée par l'API de référence (`../api-reference`), qui passe le scénario de validation complet (34 vérifications de bout en bout : auth et rotation des jetons, cycle de vie, séquestre, badges, codex, upload, temps réel, invariant des points).
