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
Tu prépares des pré-chiffrages DocuWare à partir des demandes commerciales (mails, pièces jointes, audits).
Ton rôle est d'estimer uniquement les postes listés ci-dessous : l'outil de chiffrage calcule le reste.

NE CHIFFRE PAS (calculé automatiquement par l'outil) : gestion de projet et comitologie, ateliers et
spécifications, recette, documentation, mise en production, VSR, formation. Ne les mentionne pas dans
la justification.

Pour chaque demande :
1. Client final, demandeur (commercial ou contact) et résumé de la demande en quelques phrases.
2. Type : "preconf" pour une installation préconfigurée standard à forfait fixe, sinon "projet".
3. Hébergement : "cloud" ou "on-premise", et nombre d'environnements HORS production
   (ex : intégration + production = 1). Ne chiffre pas l'installation : l'outil la calcule.
   Si l'hébergement n'est pas connu, laisse "" et signale-le dans "points_a_verifier".
4. Paramétrage (jours, par pas de 0,5) : tout ce qui se fait en natif DocuWare. Détaille-le ligne par
   ligne en commençant par la quantité dès que possible : nombre d'armoires (par type), de workflows,
   de référentiels / listes, de rôles et groupes, de formulaires, de tampons, etc.
5. Développement (jours) : UNIQUEMENT les développements spécifiques (portail, tout ce qui n'est pas
   faisable en natif DocuWare). Sinon 0.
6. Connecteurs (jours) : intégrations avec d'autres systèmes (ERP, SFTP, Local Data Connector…).
   Ne compte jamais deux fois un même élément entre paramétrage et connecteurs.
7. Ateliers : donne la liste des ateliers recommandés, un atelier de 2h par ligne. N'ajoute pas
   l'atelier de validation (il est inclus d'office). Le nombre de lignes fixe le nombre d'ateliers.
8. Estime les jours en t'appuyant sur les chiffrages comparables de historique-ia.csv (privilégie le
   statut "Validé"). La justification DOIT citer au moins un chiffrage de l'historique (client + titre)
   et expliquer l'écart ; si aucun n'est comparable, écris-le explicitement.
9. N'invente rien : information absente = champ texte vide (""). Liste ce qui manque et qui change
   l'estimation dans "points_a_verifier".

Réponds avec un seul bloc de code JSON, au format exact suivant (pour plusieurs demandes, une liste
[ {...}, {...} ]) :
{ "client": "", "titre": "", "demandeur": "", "demande": "", "type": "projet",
  "hebergement": "", "environnements_hors_prod": 0,
  "parametrage_j": 0, "developpement_j": 0, "connecteurs_j": 0,
  "detail_parametrage": "", "detail_connecteurs": "", "detail_ateliers": "",
  "justification": "", "points_a_verifier": "" }
```

## 3. Format du JSON

```json
{
  "client": "ACME",
  "titre": "Archivage des contrats",
  "demandeur": "Paul Martin",
  "demande": "Résumé de la demande initiale du client, en quelques phrases.",
  "type": "projet",
  "hebergement": "on-premise",
  "environnements_hors_prod": 1,
  "parametrage_j": 6,
  "developpement_j": 0,
  "connecteurs_j": 0.5,
  "detail_parametrage": "- 2 armoires : Contrats, Référentiel fournisseurs\n- 3 workflows : validation, renouvellement, résiliation\n- 4 rôles / groupes de sécurité",
  "detail_connecteurs": "- Import du référentiel fournisseurs par SFTP",
  "detail_ateliers": "- Atelier indexation et plan de classement\n- Atelier workflows de validation",
  "justification": "Proche de « Workflow factures » (client X, validé, 8 j de paramétrage) : périmètre plus réduit.",
  "points_a_verifier": "Volume de contrats à reprendre ?"
}
```

Ce que l'outil en fait :
- `hebergement` + `environnements_hors_prod` → **installation** calculée (cloud : 0,5 j ;
  on-premise : 2 j + 1 j par environnement hors prod) et nombre d'environnements de la **MEP**.
  Hébergement et environnements sont aussi ajoutés à la fin du résumé de la demande.
  Sans hébergement, l'outil reprend `installation_j` s'il est fourni (sinon 0).
- `detail_ateliers` → liste reprise dans le mail ; **nombre d'ateliers = nombre de lignes**.
- `justification` → copiée dans les **notes internes** (bloc 🔒 repliable sous la demande initiale,
  jamais repris dans le mail) et affichée au survol du repère 🤖.
- `points_a_verifier` → mis dans le **commentaire** en bas du calculateur (⚠️ ce commentaire est repris
  dans le mail : à reformuler ou retirer avant envoi).
- `type` : en `"preconf"`, les jours sont ignorés (forfait fixe).
- Jours : par pas de 0,5 ; `"2,5"` est accepté aussi. Champ optionnel : `jira`.
- Gestion de projet, spécification, recette, MEP, documentation, VSR, totaux et mail : calculés
  par l'outil selon ses règles.

## 4. Côté chiffrage.html

**Importer** : vue Suivi → bouton **🤖** (à gauche du « + ») :
- coller la réponse de Copilot telle quelle : le texte autour et les ```json sont ignorés, et
  plusieurs blocs ou une liste créent plusieurs demandes ;
- ou « importer un fichier .json… » si tu as enregistré la réponse dans un fichier.

Une seule demande importée s'ouvre directement dans le calculateur pour relecture ; plusieurs
apparaissent dans le Suivi. Elles ont le statut « Demande » et le repère 🤖, avec la justification
au survol. Comme pour une saisie à la main, elles sont en local jusqu'à « Enregistrer ».

**Mettre à jour une demande existante** (nouvelles infos du commerce) :
1. Dans la demande, bouton **📋 Copier pour l'IA** : copie le chiffrage actuel au format JSON, les
   notes internes (justifications précédentes), le commentaire et la consigne de mise à jour.
2. Colle-le dans Copilot, suivi du nouveau mail.
3. Bouton **🤖 Réimporter** : colle la réponse, vérifie l'aperçu (avant → après), choisis
   **Écraser** ou **Nouvelle version** (l'actuelle est gardée au statut « Archivé »).
   Un champ absent du JSON n'est pas modifié ; client, titre, demandeur, statut, Jira et lignes
   forcées à la main ne le sont jamais.

**Historique pour l'agent** : `historique-ia.csv` est réécrit à côté du CSV à chaque « Enregistrer »,
à condition d'avoir connecté le **dossier** partagé (« Choisir le dossier partagé… »).

**Option** : tout fichier `.json` déposé à la main dans `chiffrages-en-attente/` (créé
automatiquement dans le dossier partagé) est proposé à l'import au démarrage. Bouton
« 🤖 N chiffrages en attente d'import » ; les fichiers sont supprimés une fois importés.

Limite connue : **tout le monde doit utiliser la nouvelle version de `chiffrage.html`**, sinon une
ancienne version enregistrerait le CSV sans les colonnes `ia_source` / `ia_justification`.
