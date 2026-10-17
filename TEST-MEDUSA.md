# Tester la landing page avec Medusa

`index.medusa-test.html` est une copie de `index.html` dont le formulaire envoie la commande à **Medusa**
(`POST /store/landing-orders`) au lieu du Google Sheet. `index.html` (la page en ligne) n'est pas modifié, et l'URL
Apps Script a été retirée de la copie : elle ne peut pas écrire dans le Sheet de production.

## Lancer le test (Windows, PowerShell)

1. Démarrer la pile : dans le dossier du projet Medusa (`nabil_sakr`), `docker compose --profile backoffice up -d`.
2. Récupérer la **clé publiable** Medusa (`pk_...`) : Admin Medusa → Paramètres → Clés API publiables
   (ou la variable `PUBLIC_MEDUSA_PUBLISHABLE_KEY` du `.env`).
3. Servir cette page **sur le port 4321** (c'est l'origine autorisée par `STORE_CORS`) :
   ```powershell
   cd <dossier de ce dépôt>
   py -m http.server 4321
   ```
4. Ouvrir : `http://localhost:4321/index.medusa-test.html?key=pk_VOTRE_CLE`
   (options : `&medusa=http://localhost:9100` · `&book=livre-1` · `&utm_source=instagram&utm_campaign=lancement`).
5. Remplir le formulaire **en plus de 3 secondes** (sinon la commande est refusée comme robot), choisir une ville, envoyer.
6. Vérifier : la commande apparaît dans l'admin Medusa (`http://localhost:9100/app`, Commandes), avec les UTM dans les métadonnées.

## À savoir

- Les **villes** viennent de la table `delivery_rate` (import CSV). Avec les données d'exemple, seules les villes `[EXEMPLE]` apparaissent.
- Le **prix** est celui de Medusa, pas celui de la page (`PRICE_DH = 186`) : le livre `livre-1` du seed vaut 120 MAD tant que
  `backoffice/data/books.json` n'a pas le vrai titre et le vrai prix. La page affiche un total estimé ; le montant réel est
  celui renvoyé par Medusa après la commande.
- Le **transport** est ajouté au total (la page d'origine n'affichait que le prix du livre).
- Pas de Chatwoot : il est en veille, aucun ticket n'est créé.
