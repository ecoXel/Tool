# Agent Copilot « Pré-chiffrage »

L'agent lit une demande commerciale (mail, pièces jointes) et dépose **un fichier JSON par
demande** dans le dossier partagé. L'outil `chiffrage.html` les importe, calcule le reste avec
ses règles habituelles et les enregistre dans le CSV.

```
Dossier OneDrive partagé/
├── demandes_chiffrage.csv      ← la base, écrite uniquement par chiffrage.html
├── historique-ia.csv           ← version allégée, réécrite à chaque enregistrement (source de l'agent)
└── chiffrages-en-attente/      ← l'agent dépose ici ; vidé par chiffrage.html après import
    └── erreurs/                ← fichiers JSON illisibles, mis de côté
```

L'agent **n'écrit jamais dans le CSV principal**.

> ⚠️ Les noms de menus Microsoft ci-dessous sont indicatifs : l'interface de Copilot Studio et de
> Power Automate change souvent et n'a pas été vérifiée. Il faut aussi que ces outils soient
> ouverts dans le tenant (licence et autorisation de l'IT).

---

## 1. Format du fichier déposé

Un fichier `.json` par demande, avec un nom unique (ex : `chiffrage-20261012-143005.json`) :

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
- Textes de détail : une ligne par élément, commençant par `- ` → liste à puces dans l'outil.
- Champ optionnel : `jira`.
- Tout le reste (gestion de projet, spécification, recette, MEP, doc, VSR, totaux, mail) est
  calculé par l'outil.

## 2. Consignes de l'agent (à coller dans les instructions)

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

Produis un objet JSON strictement au format suivant, sans aucun texte autour :
{ "client": "", "titre": "", "demandeur": "", "demande": "", "type": "projet",
  "parametrage_j": 0, "installation_j": 0, "developpement_j": 0, "connecteurs_j": 0,
  "detail_parametrage": "", "detail_connecteurs": "", "detail_ateliers": "", "justification": "" }

Montre-moi le JSON et demande confirmation, puis appelle l'action "Déposer un pré-chiffrage" avec ce JSON.
```

## 3. Mise en place (Copilot Studio + Power Automate)

Un agent simple (Agent Builder) sait **lire** des sources mais pas **écrire** de fichier :
il faut un agent Copilot Studio avec une action Power Automate.

1. **Créer l'agent** dans Copilot Studio, coller les consignes ci-dessus.
2. **Connaissances** : ajouter `historique-ia.csv` (OneDrive/SharePoint). Si le format CSV n'est
   pas accepté comme source, le signaler : l'outil peut écrire l'historique dans un autre format.
3. **Action « Déposer un pré-chiffrage »** = un flux Power Automate déclenché par l'agent :
   - entrée : un texte `json` ;
   - étape **OneDrive for Business → Créer un fichier** :
     - Chemin du dossier : `/…/<dossier partagé>/chiffrages-en-attente`
     - Nom du fichier : `concat('chiffrage-', utcNow('yyyyMMdd-HHmmss'), '.json')`
     - Contenu du fichier : l'entrée `json`
   - sortie : « Pré-chiffrage déposé ».
4. **Publier** l'agent dans Microsoft 365 Copilot / Teams / Outlook.

Usage : dans Outlook, sur le mail du commercial → « Pré-chiffre ce mail ».

## 4. Côté chiffrage.html

- Connecter une fois le **dossier** partagé (bouton « Choisir le dossier partagé… »). Le navigateur
  (Edge/Chrome) demande l'autorisation de modifier les fichiers ; après un redémarrage du
  navigateur, un clic sur « Reconnecter » suffit.
- Au démarrage ou après « Charger », le bouton **« 🤖 N chiffrages en attente d'import »** apparaît
  s'il y a des fichiers dans `chiffrages-en-attente/`.
- L'import crée les demandes (statut « Demande », repère 🤖, justification au survol), enregistre
  le CSV, puis supprime les JSON traités.
- `historique-ia.csv` est réécrit à chaque « Enregistrer ».

Limites connues :
- **Synchronisation OneDrive** : un fichier déposé par Power Automate met quelques secondes à
  quelques minutes à arriver sur le PC.
- **Tout le monde doit utiliser la nouvelle version de `chiffrage.html`** : une ancienne version
  enregistrerait le CSV sans les colonnes `ia_source` / `ia_justification`.
- Si deux personnes importent exactement au même moment, une demande peut être créée en double
  (rare : il suffit de supprimer le doublon).
