# Yes Baby : contexte du projet

Boutique en ligne **Yes Baby** : vêtements, jouets et accessoires pour enfants, en Côte d'Ivoire. Site publié sur **https://yesbaby.ci** (GitHub Pages, fichier `CNAME`), compte GitHub `zirignon`. Page Facebook : facebook.com/yesbaby225. Interface en **français**, prix en **FCFA**, clientèle majoritairement sur mobile.

Projet distinct d'Ivoire Électroménagers : même façon de travailler, mais **compte et projet Supabase séparés**. Ne jamais mélanger les deux.

## Stack

- **HTML/CSS/JavaScript vanilla**, sans build ni npm. Deux pages :
  - `index.html` : la boutique (catalogue, fiche produit, panier, codes promo, commande, paiement, suivi de commande) ;
  - `admin.html` : l'administration (produits, commandes, codes promo), exclue des moteurs de recherche par `robots.txt`.
- **Supabase** via CDN (`@supabase/supabase-js@2`), projet `fycphdiutywlrpctafjk`. Le client s'appelle **`supabaseClient`**. La clé dans le code est la clé **publishable** (publique) : la sécurité repose sur la **RLS**.
  - Tables : `products`, `orders`, `promo_codes`. Fonction SQL : `increment_promo_usage`.
  - Storage : bucket `product-images` (photos produits, envoyées depuis l'admin).
  - Edge Function : `create-payment` (crée le paiement PayDunya et renvoie `checkout_url`). Son code **n'est pas dans ce dépôt**.
  - Admin : connexion Supabase Auth (e-mail + mot de passe).
- **Quill.js 2** (CDN) dans l'admin pour les descriptions produits (HTML stocké en base, affiché dans la fiche produit).
- **Paiement** : paiement à la livraison avec confirmation WhatsApp (`wa.me/2250142291447`), ou paiement en ligne **PayDunya** (Mobile Money / carte). Après le retour de PayDunya, la page interroge la commande jusqu'à ce que `payment_status` vaille `payee` ou `echouee`.
- Le panier en attente de paiement est sauvegardé dans le `localStorage` sous la clé `yesbaby_pending_cart`, et vidé seulement quand le paiement est confirmé.
- SEO : `sitemap.xml`, `robots.txt`, `og-image.jpg`.
- Thème visuel rose (thème bébé), couleurs en variables CSS dans `:root`.

## ⚠️ Fichiers compactés (minifiés)

`index.html` et `admin.html` sont écrits sur très peu de lignes, dont certaines font plus de 50 000 caractères, avec des images en base64 intégrées.

- Ne **jamais** afficher, recopier ou réécrire un fichier en entier.
- Chercher avec `grep -o` ou un petit script Python qui affiche seulement le contexte utile.
- Faire les modifications avec des remplacements ciblés (script Python de préférence), puis vérifier que le remplacement a eu lieu exactement une fois.
- Ne pas reformater tout le fichier : le diff deviendrait illisible.

## Règles du projet

1. **Sécurité côté serveur** : la RLS doit protéger les données. Masquer un bouton ou filtrer en JavaScript ne protège rien, puisque tout le code du site est public.
2. **Données clients** (`orders` : nom, téléphone, adresse) : seuls les comptes admin doivent pouvoir les lire. Un visiteur ne doit jamais pouvoir lister les commandes des autres.
3. **Prix et totaux** : un client peut modifier le JavaScript. Pour le paiement en ligne, le montant facturé doit être recalculé côté serveur (Edge Function) à partir des prix de la table `products` et du code promo, jamais repris tel quel depuis le navigateur.
4. **Secrets** (clés PayDunya, `service_role`) : uniquement dans les variables d'environnement des Edge Functions, jamais dans le HTML.
5. **XSS** : tout texte venant d'un client (nom, adresse, recherche) doit être échappé avant d'être inséré avec `innerHTML`.
6. Ne jamais repartir d'une ancienne copie des fichiers : vérifier que le panier, les codes promo, le paiement PayDunya, le retour de paiement et le suivi de commande sont toujours présents après une modification.
7. Ajouter un produit ou un code promo, c'est passer par l'admin (des données), pas modifier le code.

## Points connus à corriger

- **Suivi de commande** : la recherche par téléphone charge **toutes** les commandes (`orders.select("*")`) puis filtre dans le navigateur. Si la RLS autorise cette lecture aux visiteurs, les noms, téléphones et adresses de tous les clients sont lisibles publiquement. À remplacer par une fonction côté serveur (RPC ou Edge Function) qui ne renvoie que les commandes du numéro demandé, avec des informations limitées, et en retirant la lecture publique de `orders`.
- **Codes promo** : la table `promo_codes` est lue directement depuis le navigateur pour valider un code. À vérifier : la RLS ne doit exposer que le strict nécessaire, et la réduction doit être revérifiée côté serveur au moment du paiement.

## Vérifications avant de livrer

- Extraire le JavaScript et vérifier sa syntaxe avec `node --check`.
- Vérifier : catalogue et fiche produit, variantes (taille, couleur), panier, code promo, commande à la livraison avec WhatsApp, paiement PayDunya et retour, suivi de commande, connexion admin, ajout de produit avec photo, gestion des commandes et des codes promo.

## Environnement d'Ahissé

- Windows 10, **cmd.exe** (pas PowerShell) : les commandes qui lui sont données doivent être compatibles cmd.
- Mise en ligne par **glisser-déposer sur github.com** (dépôt `yes-baby`). Livrer les fichiers complets finaux quand ils doivent être déposés.
