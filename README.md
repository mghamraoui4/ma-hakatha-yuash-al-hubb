# ما هكذا يُعاش الحب — Landing page & back-office des commandes

Landing page (RTL, arabe, mode sombre) pour vendre le roman **« ما هكذا يُعاش الحب »** de نبيل صقر
(éditeur : Imprimerie Librairie Omnia, 8 rue Moulay Rachid, Hassan, Rabat, Maroc), avec un back-office
**Google Sheets + Apps Script** pour suivre les commandes de bout en bout.

Prix : **186 DH** · Paiement à la livraison.

---

## 1. Contenu du projet

| Fichier | Rôle |
|---|---|
| `index.html` | La landing page complète (un seul fichier, polices Google Fonts en CDN) |
| `najma.mp4` | Vidéo du personnage Najma (à placer **à côté** de `index.html`) |
| `apps-script.gs` | Code Apps Script (`Code.gs`) : réception des commandes, règles métier, connexion, espace livreur |
| `bo.html` | Interface web du back-office (conseillers + dashboard admin), à ajouter au projet Apps Script sous le nom `bo` |
| `README.md` | Ce guide |

---

## 2. Mettre la landing page en ligne (GitHub Pages)

1. Crée un dépôt public (ex. `roman-landing`) sur GitHub.
2. **Add file > Upload files** : dépose `index.html` et `najma.mp4`, puis **Commit**.
3. **Settings > Pages > Source : Deploy from a branch > `main` / `(root)` > Save**.
4. Après ~1 minute, la page est sur `https://<ton-compte>.github.io/<depot>/`.

Pour mettre à jour : remplace le fichier dans le dépôt (Add file > Upload files) et commit.

---

## 3. Installer le back-office (Google Sheets + Apps Script)

1. Crée un **Google Sheet** vide.
2. **Extensions > Apps Script**. Dans `Code.gs`, efface tout et colle `apps-script.gs`.
3. Clique **+ > HTML**, nomme le fichier exactement **`bo`**, colle le contenu de `bo.html`. Enregistre.
4. Choisis la fonction **`setup`**, clique **Exécuter**, accepte les autorisations (Sheets + envoi d'e-mails).
   Cela crée les onglets **Commandes**, **Transporteurs**, **Agents** et **Résumé**.
5. **Déployer > Nouveau déploiement > Application Web** :
   *Exécuter en tant que* : **Moi** · *Qui a accès* : **Tout le monde**. Copie l'URL `/exec`.
6. Colle cette URL à deux endroits :
   - `apps-script.gs` (`WEBAPP_URL`), puis **Déployer > Gérer les déploiements > Modifier > Nouvelle version** ;
   - `index.html` (constante `ORDERS_ENDPOINT`).
7. Onglet **Agents** : renseigne les adresses e-mail (voir §4). Onglet **Transporteurs** : saisis les sociétés de livraison,
   puis menu **Commandes > Générer les liens livreurs**.

Le back-office est à l'adresse `…/exec?page=bo` (rappelée dans l'onglet *Agents*, cellule F2).

> Après toute modification du script ou de `bo.html` : **Déployer > Gérer les déploiements > Modifier > Nouvelle version**.
> Sans cela, l'URL continue d'exécuter l'ancienne version.

---

## 4. Comptes et accès (rattachés aux boîtes mail)

Onglet **Agents** : `Email · Nom · Rôle · Actif`.

| Rôle | Droits |
|---|---|
| `agent` (conseiller CRC) | Voit toutes les commandes et agit dessus. **3 conseillers actifs maximum** (au-delà de 3, les suivants sont refusés). |
| `admin` (Nabil Sakr, propriétaire) | Tout ce que fait un conseiller **+ Tableau de bord** des ventes. Les admins ne comptent pas dans les 3. |

- **Connexion sans mot de passe** : le conseiller saisit son adresse e-mail, reçoit un **code à 6 chiffres** (valable 10 min)
  sur sa boîte mail, le saisit, et la session reste ouverte **7 jours** sur son appareil.
- Pour ajouter ou retirer quelqu'un : modifie l'e-mail ou décoche **Actif** dans l'onglet *Agents*. L'accès est coupé
  immédiatement, même pour une session ouverte.
- Chaque commande garde le nom du conseiller qui l'a prise en charge (colonne *Agent CRC*) et l'historique de ses actions.
- Les codes sont envoyés depuis le compte Google qui a déployé le script (quota Gmail : ~100 e-mails/jour).

## 5. Process de commande

```
Client commande ──► Nouvelle ──► (appels CRC, 3 tentatives) ──► Confirmée ──► BR créé ──► Livrée
                                        │
                                        └─ 3 échecs ──► Annulée - non confirmée
```

### Étape 1 — Le client passe la commande
Formulaire de la landing : nom, téléphone, adresse, ville, quantité. La commande arrive avec le statut **Nouvelle**
(le montant est recalculé côté script).

### Étape 2 — Le conseiller appelle le client (3 tentatives)
Dans le back-office, onglet **Commandes** (vue « À traiter », la plus ancienne en premier) :

| Bouton | Action |
|---|---|
| **📞 Appeler** | Lance l'appel (sur téléphone) |
| **Injoignable n/3** | Compte une tentative. À la **3e**, la commande est annulée (non confirmée) après confirmation. |
| **💬 WhatsApp** | Ouvre WhatsApp avec le message prêt (motif de l'appel + **lien de confirmation**). À envoyer après une tentative injoignable. |
| **✔ Confirmer** | Le client a confirmé par téléphone → **Confirmée** |
| **Client annule** | Annulation à la demande du client |

Le **lien de confirmation** ouvre une page (arabe, mobile) avec le récapitulatif et deux boutons *تأكيد طلبي* / *إلغاء الطلب*.
Si le client confirme, la commande passe seule à **Confirmée**. Le lien est signé : il ne marche que pour sa commande.

### Étape 3 — Préparation, BR et affectation au livreur
Sur une commande *Confirmée* : choisir le **transporteur**, cliquer **📦 Créer BR**. Un **N° de BR** est généré (`BR-AAMMJJ-001`),
la commande passe à **BR créé**. Le bouton **🖨 Bon de livraison** affiche et imprime le bon (client, adresse, montant à encaisser, signature).

### Étape 4 — Livraison et clôture
Chaque transporteur ouvre **son lien** (onglet *Transporteurs*, colonne C) sur son téléphone : il ne voit que ses BR
(téléphone cliquable, adresse, **montant à encaisser**) et clique **✔ Livrée** (double appui pour confirmer).
Le statut passe à **Livrée**, la date de livraison est remplie, la commande est clôturée.

### Tableau de bord (admin)
Période 7 / 30 / 90 jours ou tout : chiffre d'affaires encaissé, en cours et perdu, commandes et exemplaires, taux de confirmation
et de livraison, panier moyen, délais moyens, ventes par jour (graphique), entonnoir des statuts, ventes par ville,
performance par conseiller, situation par transporteur (à encaisser / encaissé), dernières livraisons.

> Le Sheet reste utilisable à la main (cases à cocher, statuts) : les deux modes appliquent les mêmes règles.

---

## 6. Colonnes de l'onglet « Commandes »

`Date commande` · `N° commande` · `Nom` · `Téléphone` · `Adresse` · `Ville` · `Qté` · `Montant (DH)` · `Statut` ·
`Tentatives` · `Dernière tentative` · `☎ Injoignable (+1)` · `💬 WhatsApp` · `✔ Confirmer` · `Date confirmation` ·
`Transporteur` · `📦 Créer BR` · `N° BR` · `Date livraison` · `Notes` (historique automatique) · `Agent CRC`

**Statuts** : Nouvelle · En appel · Confirmée · BR créé · Livrée · Annulée - non confirmée · Annulée - client
(le statut reste modifiable à la main via la liste déroulante, ex. annulation demandée par le client au téléphone).

L'onglet **Résumé** calcule : commandes reçues, à traiter, confirmées, en livraison, livrées, annulées, exemplaires
livrés, CA encaissé, CA en cours, taux de confirmation.

---

## 7. Configuration rapide

| Où | Variable | Rôle |
|---|---|---|
| `index.html` | `ORDERS_ENDPOINT` | URL `/exec` du script (vide = mode démo, rien n'est enregistré) |
| `index.html` | `PRICE_DH` | Prix affiché sur la page |
| `apps-script.gs` | `WEBAPP_URL` | URL `/exec` (liens de confirmation et liens livreurs) |
| `apps-script.gs` | `PRICE_DH` | Prix utilisé pour calculer les montants (source de vérité) |
| `apps-script.gs` | `MAX_TRIES` | Nombre de tentatives avant annulation (3) |
| `apps-script.gs` | `MAX_AGENTS` / `SESSION_DAYS` | Conseillers actifs max (3) / durée de session (7 jours) |
| `apps-script.gs` | `STORE_NAME` | Nom cité dans le message WhatsApp |

Penser à changer le prix **aux deux endroits** (page et script).

---

## 8. Sécurité et limites

- Le formulaire est protégé par un champ piège anti-robots, une validation côté script et une neutralisation des
  formules (`=`, `+`, `-`, `@`) dans les champs texte. Le montant n'est jamais lu depuis la page.
- Le lien client est signé (HMAC) ; le lien livreur contient une clé secrète : ne l'envoie qu'à son transporteur
  (supprime la ligne du transporteur pour révoquer l'accès).
- Le Sheet contient des données clients : ne le partage qu'avec les agents CRC.
- **WhatsApp** : le message est préparé automatiquement, mais l'agent clique sur *Envoyer* (pas d'envoi automatique).
  Un envoi 100 % automatique demande l'API WhatsApp Business (payante).
- Si deux agents travaillent en même temps, chacun voit les changements de l'autre en direct.
- Les anciennes commandes d'une version précédente du Sheet sont conservées dans un onglet `Commandes_ancien_…`
  quand `setup` est relancé.

---

## 9. Tester

1. Ouvre la landing publiée, passe une commande test : une ligne **Nouvelle** apparaît dans le Sheet et dans le back-office.
2. Ouvre `…/exec?page=bo`, connecte-toi avec ton e-mail (code reçu par mail).
3. Clique **Injoignable** trois fois : la commande passe **Annulée - non confirmée**.
4. Nouvelle commande : **💬 WhatsApp**, ouvre le lien de confirmation, confirme : **Confirmée**.
5. Choisis un transporteur, **📦 Créer BR**, ouvre le lien du livreur, **✔ Livrée** : **Livrée**.
6. Connecte-toi avec le compte admin : le **Tableau de bord** montre la vente.

Supprime les commandes de test du Sheet avant la mise en production.
