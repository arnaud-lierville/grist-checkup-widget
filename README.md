# Checkup du projet — widget Grist

Widget personnalisé Grist qui lit **toute la structure** d'un document (tables, colonnes, types, formules, pages, widgets, règles d'accès, webhooks), la dessine sous forme de schéma et prépare un **prompt à copier dans une IA** pour faire auditer la conception du document. Il **ne modifie rien** et n'envoie aucune valeur de cellule dans le prompt.

## URL du widget

```
https://grist-checkup-widget.vercel.app/
```

## Tables Grist

Aucune table à créer : le widget analyse le document dans lequel il est installé. Il peut être posé sur n'importe quelle table.

## Ajouter et configurer le widget

1. Dans le document à analyser : **Nouveau** → **Ajouter une page** (ou **Ajouter une vue** sur une page existante).
2. Choisir **Personnalisé**, puis n'importe quelle table (par exemple la première).
3. Dans le panneau de droite, onglet **Widget** : **URL personnalisée** → coller `https://grist-checkup-widget.vercel.app/`.
4. **Niveau d'accès** : choisir **Accès complet au document**.
   Pourquoi : seul ce niveau permet de lire les tables internes de Grist qui décrivent la structure (`_grist_Tables`, `_grist_Tables_column`, pages, widgets, règles d'accès…). Le widget n'utilise **aucune fonction d'écriture** : il ne modifie ni les données, ni la structure, ni ses propres options.
5. Aucune association de colonnes n'est nécessaire.

## Utilisation

Le titre affiche « Checkup du projet » suivi du nom du document. Le bouton **↻** relit la structure (après une modification du document).

### Onglet Structure
Vue d'ensemble chiffrée, puis le détail :
- **tables** : colonnes, libellés, types, nature (donnée, formule, formule de déclenchement, colonne vide), taux de remplissage, nombre de cellules en erreur, listes de choix, colonnes d'affichage des références, formules et formats conditionnels ;
- **pages et widgets** : type, table, liaisons « sélectionner par », tris, filtres, colonnes masquées, widgets personnalisés (URL, niveau d'accès) ;
- **règles d'accès**, **webhooks**, **pièces jointes** (nombre et taille).

### Onglet Prompt
1. (Facultatif) Décrire le **but du projet et ses utilisateurs** : l'IA en tient compte. Ce texte est mémorisé sur votre ordinateur uniquement.
2. **Copier le prompt** (ou le **télécharger** en `.md`), puis le coller dans l'IA de votre choix.

Le prompt contient : la mission (note globale sur 20, points forts, problèmes classés par gravité avec corrections pas à pas, suggestions, plan d'action), le référentiel de bonnes pratiques (bases relationnelles et Grist) et la structure du document.

**Ce qui n'est jamais dans le prompt** : les valeurs des cellules. Seuls des nombres en sont tirés (lignes, taux de remplissage, erreurs).
**À savoir** : les formules, filtres enregistrés, listes de choix et conditions des règles d'accès sont recopiés tels quels et peuvent contenir quelques valeurs écrites en dur (ex. `$Statut == "Terminé"`, une adresse e-mail dans une règle). Relisez le prompt avant de l'envoyer si le document est sensible.

### Onglet Schéma
Trois vues :
- **Tables et relations** : chaque table avec ses colonnes (ƒ formule, ↺ formule de déclenchement) et ses références (trait plein : Référence ; tirets : Liste de références ; pointillés : table résumée) ;
- **Pages et widgets** : les pages, leurs widgets, la table affichée et les liaisons « sélectionner par » ; les tables affichées sur aucune page sont encadrées en rouge ;
- **Dépendances des formules** : quelle colonne utilise quelle autre (détection approximative à partir du texte des formules).

Outils : restreindre à une table ou une page, choix des colonnes affichées, disposition (automatique, hiérarchique, grille, cercle, concentrique), ⤢ pour tout afficher, molette pour zoomer, glisser pour déplacer. Survol ou clic sur une boîte : met en évidence ses voisins.
Exports : **PNG**, **SVG**, **Mermaid** (copié dans le presse-papiers, à coller par exemple dans mermaid.live) et **JSON** (structure complète).

Les préférences (onglet, vue du schéma, contexte) sont enregistrées dans le navigateur (`localStorage`), jamais dans le document.

## Dépendances externes

- [Cytoscape.js](https://js.cytoscape.org/) `3.34.3` (licence MIT, jsDelivr) : dessin du schéma.
- [cytoscape-svg](https://github.com/kinimesi/cytoscape-svg) `0.4.0` (licence MIT, jsDelivr) : export SVG.

## Dépannage

| Symptôme | Cause probable |
|---|---|
| Message « Ce widget a besoin de l'accès complet… » | Niveau d'accès du widget différent de « Accès complet au document » (panneau de droite, onglet Widget). |
| Message « Ce widget doit être ouvert dans Grist » | L'URL a été ouverte directement dans le navigateur au lieu d'une vue Personnalisé. |
| Les dernières modifications n'apparaissent pas | Cliquer sur **↻** ; après une mise à jour du widget, recharger la page Grist (voire vider le cache). |
| Onglet Schéma vide avec un message sur Cytoscape.js | `cdn.jsdelivr.net` est bloqué par le réseau ; les onglets Structure et Prompt restent utilisables. |
| « Copie automatique refusée » | Le navigateur bloque le presse-papiers : le texte est sélectionné, faire Ctrl/Cmd + C, ou utiliser **Télécharger**. |
| Un export ne se télécharge pas | Le navigateur bloque les téléchargements depuis un widget ; utiliser la copie (prompt, Mermaid). |
| Taux de remplissage « — » | Colonne formule, ou table illisible (table « à la demande »). |

## Limites connues

- Les **partages** du document (membres, rôles, lien public), l'historique des versions et les URL des webhooks ne sont pas accessibles aux widgets : le prompt demande à l'IA de les signaler comme points à vérifier.
- Le **taux de remplissage** compte comme vides les valeurs par défaut de Grist (vide, 0, faux) : un taux bas sur une colonne Numérique ou Interrupteur peut être normal.
- Sur un très gros document, la lecture des statistiques (toutes les tables) peut prendre quelques secondes, et le prompt peut dépasser la taille acceptée par certaines IA.
- Conçu pour un usage sur ordinateur.
