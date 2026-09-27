# Analyse et visualisation des données e-commerce Olist

## 1. Présentation du projet

Ce projet a pour objectif d'analyser les données de la plateforme e-commerce brésilienne **Olist** afin de mieux comprendre l'activité commerciale, le comportement des commandes, la performance logistique et la satisfaction des clients.

Le jeu de données contient des informations relatives aux commandes, aux clients, aux produits, aux vendeurs, aux paiements, aux avis clients ainsi qu'aux données géographiques.

L'analyse est réalisée en Python à travers différentes étapes : exploration des données, contrôle de leur qualité, compréhension des relations entre les tables, préparation des données, analyses statistiques et création de visualisations.

Une attention particulière est portée à la qualité des données et au niveau de détail des différentes tables afin d'éviter les erreurs liées aux jointures et à la duplication des observations.


## 2. Objectifs du projet

L'objectif principal est de transformer les données brutes d'Olist en informations exploitables permettant de mieux comprendre l'activité de la plateforme.

Les analyses réalisées cherchent notamment à :

- analyser l'évolution du nombre de commandes dans le temps ;
- étudier la distribution des prix des articles et identifier les valeurs potentiellement aberrantes ;
- identifier les catégories de produits les plus représentées dans les ventes ;
- analyser les délais de livraison et le respect des dates de livraison estimées ;
- évaluer la satisfaction des clients à partir des notes attribuées ;
- comparer la satisfaction des clients selon le respect des délais de livraison ;
- étudier la relation entre le poids des produits et les frais de livraison.



## Source des données

Les données utilisées dans ce projet proviennent du jeu de données public **Brazilian E-Commerce Public Dataset by Olist**, disponible sur Kaggle.

🔗 Dataset : https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

Les fichiers CSV ne sont pas inclus directement dans ce dépôt en raison de leur taille.

Après téléchargement, les fichiers doivent être placés dans le dossier suivant :

```text
data/
├── olist_customers_dataset.csv
├── olist_geolocation_dataset.csv
├── olist_order_items_dataset.csv
├── olist_order_payments_dataset.csv
├── olist_order_reviews_dataset.csv
├── olist_orders_dataset.csv
├── olist_products_dataset.csv
├── olist_sellers_dataset.csv
└── product_category_name_translation.csv

## 3. Présentation du jeu de données

Le projet utilise le jeu de données **Brazilian E-Commerce Public Dataset by Olist**, composé de plusieurs fichiers CSV représentant différentes dimensions de l'activité e-commerce.

Les données sont réparties dans **9 tables** :

| Table | Description |
|---|---|
| `customers` | Informations sur les clients et leur localisation |
| `orders` | Informations générales sur les commandes et les différentes dates du processus de livraison |
| `order_items` | Détail des articles présents dans chaque commande, avec le prix et les frais de livraison |
| `payments` | Informations relatives aux paiements des commandes |
| `reviews` | Notes et avis laissés par les clients |
| `products` | Caractéristiques des produits : catégorie, poids, dimensions, etc. |
| `sellers` | Informations sur les vendeurs |
| `geolocation` | Informations géographiques associées aux codes postaux |
| `translation` | Traduction des catégories de produits du portugais vers l'anglais |



### Principales relations entre les tables

Les tables sont reliées principalement par des identifiants communs :

- `orders.customer_id` → `customers.customer_id`
- `order_items.order_id` → `orders.order_id`
- `order_items.product_id` → `products.product_id`
- `order_items.seller_id` → `sellers.seller_id`
- `payments.order_id` → `orders.order_id`
- `reviews.order_id` → `orders.order_id`

Les contrôles d'intégrité réalisés sur ces relations n'ont révélé aucun identifiant orphelin.

### Niveau de détail des principales tables

Les tables ne possèdent pas toutes le même niveau de détail :

- une ligne de `orders` représente une commande ;
- une ligne de `order_items` représente un article d'une commande ;
- une ligne de `payments` représente un enregistrement de paiement ;
- une ligne de `reviews` représente un enregistrement d'avis ;
- une ligne de `products` représente un produit ;
- une ligne de `sellers` représente un vendeur.

Cette différence de niveau de détail est importante lors des jointures. Une commande pouvant contenir plusieurs articles, paiements ou avis, certaines jointures peuvent multiplier le nombre de lignes et fausser les indicateurs si elles ne sont pas contrôlées.


## 4. Méthodologie

L'analyse a été organisée en plusieurs étapes afin de garantir la fiabilité des résultats.

### 4.1 Exploration initiale

Une première exploration a permis de :

- vérifier les dimensions des différentes tables ;
- examiner les colonnes et leurs types ;
- observer les premières lignes des données ;
- identifier les variables utiles aux différentes analyses.

### 4.2 Contrôle de la qualité des données

Plusieurs contrôles ont été réalisés :

- recherche des valeurs manquantes ;
- recherche des doublons ;
- vérification des types de données ;
- conversion des colonnes temporelles au format `datetime` ;
- contrôle de l'unicité des identifiants ;
- vérification de l'intégrité des relations entre les tables.

La table `geolocation` contient notamment un nombre important de lignes dupliquées. Ces observations n'ont pas été supprimées automatiquement, car cette table possède un niveau de détail particulier et nécessite une analyse adaptée avant toute utilisation dans une jointure.

Certaines dates de livraison sont également manquantes. Leur absence est en grande partie associée à des commandes qui ne sont pas indiquées comme livrées. Quelques commandes ayant le statut `delivered` présentent néanmoins une date de livraison manquante ; ces cas ont été conservés et documentés.

### 4.3 Préparation et jointure des données

Une table principale d'analyse a été construite à partir de la table `orders`.

Des jointures de type `LEFT JOIN` ont été utilisées afin de conserver l'ensemble des commandes, même lorsqu'une information complémentaire est absente.

La table principale a été enrichie successivement avec :

`orders` → `customers` → `order_items` → `products` → `translation` → `sellers`

La jointure avec `order_items` augmente naturellement le nombre de lignes, car une commande peut contenir plusieurs articles. Les jointures suivantes n'introduisent pas de multiplication supplémentaire des observations.

Les tables `payments`, `reviews` et `geolocation` ne sont pas ajoutées directement à cette table lorsque leur niveau de détail risque de provoquer une multiplication des lignes. Lorsque nécessaire, les données sont d'abord agrégées au niveau approprié avant la jointure.


## 5. Analyses et visualisations réalisées

Plusieurs analyses ont été réalisées afin d'explorer les dimensions commerciales, logistiques et clients du jeu de données.

### 5.1 Évolution du nombre de commandes

Le nombre de commandes a été analysé par mois à partir de la date d'achat (`order_purchase_timestamp`).

Une courbe d'évolution temporelle permet de visualiser les variations de l'activité sur la période couverte par les données.

### 5.2 Distribution des prix des articles

La distribution de la variable `price` a été étudiée à l'aide :

- des statistiques descriptives ;
- d'un histogramme ;
- de la méthode de l'IQR ;
- d'un boxplot.

La méthode de l'IQR a permis d'identifier les valeurs potentiellement aberrantes sans les supprimer automatiquement, car des prix élevés peuvent correspondre à des produits réellement commercialisés.

### 5.3 Catégories de produits les plus représentées

Les catégories de produits ont été analysées à partir des lignes de commande.

Un classement des dix catégories comportant le plus grand nombre d'articles vendus a été réalisé afin d'identifier les familles de produits les plus représentées dans les ventes.

### 5.4 Performance logistique

Les délais de livraison ont été calculés à partir de la différence entre la date d'achat et la date de livraison effective.

L'écart entre la date de livraison réelle et la date de livraison estimée a également été calculé afin de distinguer :

- les commandes livrées à temps ou en avance ;
- les commandes livrées en retard.

### 5.5 Satisfaction client

Les notes attribuées par les clients ont été analysées afin d'évaluer leur niveau global de satisfaction.

L'analyse repose notamment sur :

- la note moyenne ;
- la note médiane ;
- la répartition des notes de 1 à 5 ;
- le nombre d'avis associé à chaque note.

### 5.6 Respect des délais et satisfaction client

La satisfaction moyenne a été comparée selon le respect ou non de la date de livraison estimée.

Les avis ont d'abord été agrégés au niveau de la commande avant la jointure avec les données de livraison. Cette étape permet d'éviter qu'une commande comportant plusieurs enregistrements d'avis soit surreprésentée dans l'analyse.

Cette analyse permet d'étudier l'association entre la performance logistique et la satisfaction des clients, sans conclure à elle seule à une relation de causalité.

### 5.7 Poids des produits et frais de livraison

La relation entre le poids des produits et les frais de livraison a été étudiée à l'aide :

- d'un nuage de points ;
- du coefficient de corrélation de Pearson ;
- du coefficient de corrélation de Spearman.

L'utilisation des deux coefficients permet de compléter l'analyse, notamment en raison de la présence de distributions asymétriques et de valeurs extrêmes dans les données.


## 6. Principaux résultats

### 6.1 Activité des commandes

Le jeu de données contient **99 441 commandes**.

L'analyse mensuelle montre une progression importante de l'activité à partir de 2017, malgré plusieurs fluctuations au cours de la période étudiée.

Une forte diminution du nombre de commandes apparaît à partir de septembre 2018. Cette baisse doit cependant être interprétée avec prudence, notamment parce que la période finale du jeu de données est partielle et que les dernières observations disponibles datent du 17 octobre 2018.

### 6.2 Prix des articles

L'analyse de **112 650 articles** met en évidence :

- un prix moyen de **120,65** ;
- un prix médian de **74,99** ;
- un premier quartile (Q1) de **39,90** ;
- un troisième quartile (Q3) de **134,90** ;
- un prix maximum de **6 735,00**.

La moyenne étant supérieure à la médiane, la distribution des prix présente une asymétrie vers les valeurs élevées.

Avec la méthode de l'IQR, la borne supérieure est de **277,40**. Au total, **8 427 observations**, soit environ **7,48 %** des articles, sont identifiées comme potentiellement aberrantes.

Ces observations n'ont pas été supprimées automatiquement, car une valeur statistiquement atypique ne constitue pas nécessairement une erreur.

### 6.3 Performance logistique

L'analyse des commandes disposant d'une date de livraison exploitable porte sur **96 476 commandes**.

Le délai entre l'achat et la livraison est en moyenne de **12,56 jours**, avec une médiane de **10,22 jours**.

Concernant le respect de la date estimée :

- **88 649 commandes** ont été livrées à temps ou en avance ;
- **7 827 commandes** ont été livrées après la date estimée.

La majorité des commandes analysées respecte donc la date de livraison estimée.

### 6.4 Satisfaction client

L'analyse porte sur **99 224 avis clients**.

Les résultats montrent :

- une note moyenne de **4,09 / 5** ;
- une note médiane de **5 / 5** ;
- **57 328 avis** avec la note maximale de 5 ;
- **19 142 avis** avec la note de 4 ;
- **8 179 avis** avec la note de 3 ;
- **3 151 avis** avec la note de 2 ;
- **11 424 avis** avec la note de 1.

Les notes sont donc globalement élevées, même si une partie des clients attribue une note faible.

### 6.5 Satisfaction et respect des délais

Une différence importante apparaît lorsque la satisfaction est comparée selon le respect de la date de livraison estimée :

- note moyenne des commandes **en retard : 2,57 / 5** ;
- note moyenne des commandes **à temps ou en avance : 4,29 / 5**.

Ces résultats mettent en évidence une association entre le respect du délai estimé et la satisfaction client. Ils ne permettent cependant pas, à eux seuls, d'établir une relation de causalité.

### 6.6 Poids des produits et frais de livraison

L'analyse de la relation entre le poids des produits et les frais de livraison repose sur **112 632 observations complètes**.

Les coefficients obtenus sont :

- corrélation de Pearson : **0,610** ;
- corrélation de Spearman : **0,447**.

Les deux coefficients étant positifs, les produits plus lourds ont globalement tendance à être associés à des frais de livraison plus élevés.

La relation n'est toutefois pas parfaite : d'autres variables peuvent également intervenir dans la détermination des frais de livraison. De plus, une corrélation ne constitue pas une preuve de causalité.


## 7. Technologies utilisées

Le projet a été réalisé avec :

- **Python** pour le traitement et l'analyse des données ;
- **Pandas** pour la manipulation des DataFrames ;
- **NumPy** pour certains calculs numériques ;
- **Matplotlib** pour la création et la personnalisation des graphiques ;
- **Seaborn** pour les visualisations statistiques ;
- **Jupyter Notebook** pour organiser le code, les résultats et les analyses ;
- **Visual Studio Code** comme environnement de développement.


## 8. Structure du projet

Le projet est organisé de la manière suivante :

PROJET_VISUALISATION_OLIST/
│
├── data/
│   ├── olist_customers_dataset.csv
│   ├── olist_geolocation_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_order_payments_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   ├── olist_orders_dataset.csv
│   ├── olist_products_dataset.csv
│   ├── olist_sellers_dataset.csv
│   └── product_category_name_translation.csv
│
├── projet_olist.ipynb
└── README.md


## 9. Exécution du projet

### Prérequis

Le projet nécessite Python ainsi que les bibliothèques suivantes :

- pandas
- numpy
- matplotlib
- seaborn

### Lancement

1. Télécharger ou cloner le projet.
2. Vérifier que les fichiers CSV sont présents dans le dossier `data/`.
3. Ouvrir le fichier `projet_olist.ipynb` dans Jupyter Notebook ou Visual Studio Code.
4. Sélectionner un environnement Python contenant les bibliothèques nécessaires.
5. Exécuter les cellules du notebook dans l'ordre.


## 10. Conclusion

Ce projet a permis d'explorer plusieurs dimensions de l'activité e-commerce d'Olist à travers une démarche allant du contrôle de la qualité des données jusqu'à l'analyse statistique et à la visualisation.

L'analyse met notamment en évidence une activité importante sur la période étudiée, une distribution des prix fortement asymétrique, ainsi qu'une satisfaction client globalement élevée.

La performance logistique apparaît également associée à la satisfaction : les commandes livrées à temps ou en avance obtiennent une note moyenne de 4,29/5, contre 2,57/5 pour les commandes livrées en retard.

Enfin, l'analyse du poids des produits et des frais de livraison met en évidence une association positive entre ces deux variables, avec une corrélation de Pearson de 0,610 et une corrélation de Spearman de 0,447.

Ces résultats montrent l'intérêt de combiner analyse descriptive, visualisation et méthodes statistiques pour transformer des données transactionnelles en informations utiles à la compréhension de l'activité.

Les résultats doivent toutefois être interprétés dans le contexte du jeu de données étudié. Une association statistique ne permet notamment pas, à elle seule, d'établir une relation de causalité.


## Auteur

**Adjaratou DIARRASSOUBA**

Projet réalisé dans le cadre d'un travail d'analyse et de visualisation de données.