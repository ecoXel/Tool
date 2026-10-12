# Agent Copilot « Pré-chiffrage »

L'agent lit une demande commerciale (mail, pièces jointes) et répond par un **bloc JSON**.
Tu importes ce JSON dans `chiffrage.html`, qui calcule le reste avec ses règles habituelles.

```
Mail du commercial ──► Agent Copilot ──► JSON
                                          │  copier-coller  (bouton 🤖 du Suivi)
                                          │  ou fichier .json
                                          ▼
                                   chiffrage.html ──► CSV
```

L'agent **n'écrit rien nulle part** : il propose, tu importes et tu relis.

> ⚠️ Les noms de menus Microsoft ci-dessous sont indicatifs : l'interface de Copilot change souvent
> et n'a pas été vérifiée.

---

## 1. Créer l'agent

Dans Microsoft 365 Copilot → **Créer un agent** (Agent Builder) :

1. **Nom** : Pré-chiffrage DocuWare.
2. **Instructions** : coller le bloc de la section 2.
3. **Connaissances** : ajouter `historique-ia.csv`, qui se trouve dans le dossier partagé à côté du
   CSV (voir section 4). Si le format CSV n'est pas accepté comme source, le signaler : l'outil peut
   écrire l'historique dans un autre format.
4. Créer, puis l'utiliser dans Copilot ou Outlook : sur le mail du commercial → « Pré-chiffre ce mail ».

## 2. Instructions de l'agent (à coller)

```
Tu prépares des pré-chiffrages DocuWare à partir des demandes commerciales (mails, pièces jointes).

Pour chaque demande :
1. Identifie le client final, le demandeur (commercial ou contact), et résume la demande en quelques phrases.
2. Choisis le type : "preconf" pour une installation préconfigurée standard à forfait fixe, sinon "projet".
3. Pour un projet, estime les jours de paramétrage, d'installation, de développement et de connecteurs,
   par pas de 0,5 j, en t'appuyant sur les chiffrages comparables du fichier historique-ia.csv
   (privilégie ceux au statut "Validé"). Mets 0 quand un poste ne s'applique pas.
4. Rédige les détails (paramétrage, connecteurs, ateliers) en lignes commençant par "- ".
5. N'invente rien : si une information est absente, laisse le champ texte vide ("").
6. Explique en une ou deux phrases sur quels chiffrages passés tu t'appuies (champ "justification").

Réponds avec un seul bloc de code JSON, au format exact suivant (pour plusieurs demandes,
une liste [ {...}, {...} ]) :
{ "client": "", "titre": "", "demandeur": "", "demande": "", "type": "projet",
  "parametrage_j": 0, "installation_j": 0, "developpement_j": 0, "connecteurs_j": 0,
  "detail_parametrage": "", "detail_connecteurs": "", "detail_ateliers": "", "justification": "" }
Après le bloc, tu peux ajouter tes remarques ou points à vérifier en une ou deux lignes.
```

## 3. Format du JSON

```json
{
  "client": "ACME",
  "titre": "Archivage des contrats",
  "demandeur": "Paul Martin",
  "demande": "Résumé de la demande initiale du client, en quelques phrases.",
  "type": "projet",
  "parametrage_j": 6,
  "installation_j": 1,
  "developpement_j": 0,
  "connecteurs_j": 0.5,
  "detail_parametrage": "- Armoire Contrats\n- Workflow de validation",
  "detail_connecteurs": "- Connecteur ERP",
  "detail_ateliers": "- Atelier indexation\n- Atelier workflow",
  "justification": "Proche de « Workflow factures » (client X, validé) : même périmètre."
}
```

- `type` : `"projet"` ou `"preconf"`. En préconf, les jours sont ignorés (forfait fixe).
- Jours : nombres par pas de 0,5. `"2,5"` est accepté aussi.
- Textes de détail : une ligne par élément commençant par `- ` → liste à puces dans l'outil.
- Champ optionnel : `jira`.
- Tout le reste (gestion de projet, spécification, recette, MEP, doc, VSR, totaux, mail) est
  calculé par l'outil.

## 4. Côté chiffrage.html

**Importer** : vue Suivi → bouton **🤖** (à gauche du « + ») :
- coller la réponse de Copilot telle quelle : le texte autour et les ```json sont ignorés, et
  plusieurs blocs ou une liste créent plusieurs demandes ;
- ou « importer un fichier .json… » si tu as enregistré la réponse dans un fichier.

Une seule demande importée s'ouvre directement dans le calculateur pour relecture ; plusieurs
apparaissent dans le Suivi. Elles ont le statut « Demande » et le repère 🤖, avec la justification
au survol. Comme pour une saisie à la main, elles sont en local jusqu'à « Enregistrer ».

**Historique pour l'agent** : `historique-ia.csv` est réécrit à côté du CSV à chaque « Enregistrer »,
à condition d'avoir connecté le **dossier** partagé (« Choisir le dossier partagé… »).

**Option** : tout fichier `.json` déposé à la main dans `chiffrages-en-attente/` (créé
automatiquement dans le dossier partagé) est proposé à l'import au démarrage. Bouton
« 🤖 N chiffrages en attente d'import » ; les fichiers sont supprimés une fois importés.

Limite connue : **tout le monde doit utiliser la nouvelle version de `chiffrage.html`**, sinon une
ancienne version enregistrerait le CSV sans les colonnes `ia_source` / `ia_justification`.
