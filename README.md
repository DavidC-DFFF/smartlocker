# Casiers connectés - Bac Pro CIEL

Explorateur statique des fonctions, modules et composants d’une maquette de casiers connectés.

## Publier sur GitHub Pages

Placer le contenu de ce dossier à la racine du dépôt dédié, avec `index.html` directement à la racine.

Dans **Settings > Pages > Build and deployment** :

1. Source : **Deploy from a branch**.
2. Branch : sélectionner la branche contenant les fichiers, généralement **main**.
3. Folder : **/(root)**.
4. Cliquer sur **Save** et attendre la fin du déploiement.

Aucune compilation, dépendance ou clé d’accès n’est nécessaire. Le fichier `.nojekyll` active la publication des fichiers statiques tels quels.

## Contenu

- Navigation entre fonctions et sept vues techniques.
- 45 fiches de composants et recherche par repère.
- 16 captures historiques et trois PDF.
- Sept schémas haute définition : Block Diagram, BUS, LOCK, POWER, CONTROL, CASIER_A et CASIER_B.
- Visionneuse avec zoom à la molette, déplacement, commandes clavier et téléchargement PNG.
- Notices techniques en texte, sans transcriptions du compte ni de la conversation Speed-up.

Les hypothèses de conception restent à valider sur prototype. Les PDF et captures sont des documents consultables ; aucun fichier KiCad natif ni firmware validé n’est inclus.

## Schémas haute définition - 21 octobre 2026

Ouvrir `haute-definition/index.html`, ou le lien **Schémas HD** du site. Chaque page de module propose aussi son schéma agrandi. Les sept images ont été assemblées à partir de 142 captures chevauchantes ; les captures brutes sont conservées dans le dossier documentaire local et ne sont pas publiées ici.

Les PNG complets, leurs aperçus et le manifeste des dimensions et sommes SHA-256 sont dans `haute-definition/`. L'atlas de huit pages est disponible dans `pdf/Atlas-schemas-haute-definition.pdf`.

Les schémas restent ceux du projet source, avec ses six départs de bus. La réalisation prévue concerne deux maquettes de deux casiers, les Raspberry Pi étant déjà disponibles. Aucun agrandissement artificiel des captures historiques n'a été utilisé.

Documentation GitHub : https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
