# 📊 Contoso Retail : Analyse de la Performance Commerciale (Power BI)

> Rapport Power BI interactif en 3 pages (**Boutiques, Produits, Ventes**) pour analyser le réseau de magasins, le catalogue produits et la performance commerciale de Contoso sur 2020-2021.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-Mesures-blue)
![Power Query](https://img.shields.io/badge/Power%20Query-ETL-green)

---

## 🎯 Objectif du projet

Transformer des fichiers CSV bruts en un **tableau de bord qui « fait parler » les données** et répond à des questions métier :

- Où sont nos boutiques, de quel type, combien sont fermées ?
- Quels produits, marques et catégories se vendent le mieux ? Quel est le taux de retour ?
- Comment évolue le chiffre d'affaires par rapport à l'année précédente ?

---

## 🗂️ Données utilisées

| Table | Colonnes principales |
|---|---|
| **Boutiques** | IdBoutique, IdGeographie, Boutique, Status, Type, Surface, Employee, CloseReason |
| **Geographie** | IdGeographie, Continent, Type, Pays |
| **Produits** | IdProduit, Produit, Marque, Class, Fabricant, Description, Coût unitaire, Prix unitaire, IdSousCategorieProduit |
| **Sous Catégorie Produits** | IdSousCategorieProduit, SousCategorieProduit, IdCategorieProduit |
| **Catégorie Produits** | IdCategorieProduit, CategorieProduit |
| **Sales** (6 fichiers CSV, 2020 et 2021) | IdDate, IdVente, IdBoutique, IdProduit, Quantité de vente, Quantité de retour, Montant_vente, Remise, Montant_vente_après_remise |

---

## 🛠️ Étapes de réalisation

### 1. Exploration des données
Description de chaque table et de ses colonnes pour comprendre la structure du jeu de données et repérer les clés de liaison (`IdBoutique`, `IdProduit`, `IdGeographie`...).

### 2. Nettoyage et transformation (Power Query)
- Import des fichiers : **Obtenir les données → Transformer les données**.
- Combinaison des 6 fichiers CSV de ventes (2020-2021) dans une seule table `Sales`.
- Dans la table **Boutiques**, remplacement des valeurs `null` de la colonne `CloseReason` par **"Open"**.

### 3. Colonnes calculées (DAX), table `Sales`

```DAX
Montant vente =
( Sales[Quantité de vente] - Sales[Quantité de retour] )
    * RELATED( Produits[Prix unitaire] )

Remise =
IF( Sales[Montant vente] < 2000, 0,
    IF( Sales[Montant vente] < 5000, 0.05, 0.1 ) )

Montant vente après remise =
Sales[Montant vente] * ( 1 - Sales[Remise] )
```

**Règle de remise :** 0 % en dessous de 2 000 €, 5 % de 2 000 à 5 000 €, 10 % au-delà.

### 4. Mesures (DAX)

```DAX
CA = SUM( Sales[Montant vente après remise] )

CA-1 = CALCULATE( [CA], SAMEPERIODLASTYEAR( Calendrier[Date] ) )

Ecart ventes = DIVIDE( [CA] - [CA-1], [CA-1] )
```

> **Taux d'évolution = (valeur actuelle − valeur de départ) / valeur de départ.**
> On divise par l'année de référence (N-1). `DIVIDE` évite l'erreur de division par zéro.

### 5. Modélisation
Modèle en **étoile** : la table de faits `Sales` est reliée aux dimensions `Boutiques` (→ `Geographie`), `Produits` (→ `Sous Catégorie` → `Catégorie`) et à une table `Calendrier` (indispensable pour les calculs N-1).

### 6. Visualisation
Création de 3 pages de rapport, avec segments (filtres), bouton **« Effacer tous les segments »** et **menu de navigation** entre les pages.

---

## 📈 Lecture et analyse des dashboards

### 🏬 Page 1 : Boutiques

![Dashboard Boutiques](ScreenShots/Boutique.png)

**Contenu :** segments Pays et Continent, indicateurs (pays sélectionné, nombre de boutiques, nombre d'employés), répartition par type et par status, carte géographique, tableau détaillé (Pays, Boutique, Employee, Surface, Type, Status) et nombre de ventes par boutique.

**Analyse :**
- Le réseau compte **306 boutiques** et environ **11 000 employés**.
- Le réseau est dominé par les magasins physiques : **299 Store**, contre seulement 3 Online, 3 Reseller et 1 Catalog.
- **294 boutiques sont ouvertes (96 %)** et **12 sont fermées (3,9 %)**. Un clic sur le status « Off » filtre le tableau pour identifier les boutiques concernées.
- Les ventes sont **très concentrées** : 2 boutiques dépassent 30 000 ventes (44K et 31K), alors que la grande majorité des autres tourne autour de 1K. Cela suggère une forte dépendance à quelques points de vente, probablement les canaux en ligne.

### 📦 Page 2 : Produits

![Dashboard Produits](ScreenShots/Produits.png)

**Contenu :** filtres par Class et Marque, indicateurs clés, nombre de produits par catégorie, comparaison ventes vs retours, arbre de décomposition (catégorie → sous-catégorie), tableau détaillé des produits et top produits par nombre de ventes.

**Analyse :**
- Le catalogue contient **1 689 produits** pour environ **3 M de ventes** et **22 000 retours**, soit un taux de retour inférieur à 1 %.
- Les catégories les plus riches en références : **Computers (606)**, **Cameras and camcorders (371)** et **Cell phones (285)** ; **Audio** est la plus restreinte (115).
- Le graphique ventes vs retours montre des retours quasi négligeables face au volume de ventes, ce qui est un bon signe de qualité et de satisfaction.
- Le **top des produits** est dominé par les ordinateurs portables (*Adventure Works Laptop16 M1601* en versions White et Silver, puis *Fabrikam Laptop*).

### 💰 Page 3 : Ventes

![Dashboard Ventes](ScreenShots/Ventes.png)

**Contenu :** 3 graphiques en secteurs (CA par catégorie, classe et marque), segments Année / Mois / Pays / Boutique, jauge CA vs CA-1, courbe CA et nombre de ventes par date, tableau d'analyse mensuelle avec écart et **sparklines** (CA par jour, avec points max et min), et tableau détaillé des ventes.

**Analyse (année 2021) :**
- Le **chiffre d'affaires atteint environ 258 M€** pour **68 000 ventes**.
- Par rapport à 2020 (≈ 242,7 M€), la croissance globale est de **+6,14 %**.
- Les **mois en hausse** : janvier (+15,1 %), juin (+13,2 %), mai (+12,3 %), juillet (+10,5 %), août (+9,4 %), septembre (+8,3 %) et avril (+7,8 %).
- Les **mois en baisse** : février (−10,9 %) et mars (−5,6 %), repérables grâce aux flèches rouges ↓. Ce début d'année en recul est ensuite largement compensé.
- La catégorie **Computers** représente la plus grande part du CA, et la classe **Regular** pèse plus de la moitié du chiffre d'affaires.
- Les sparklines permettent de repérer d'un coup d'œil, mois par mois, les jours de pic et de creux de CA.

---

## 💡 Principales conclusions

1. Le CA progresse de **+6 %** malgré un début d'année difficile (février-mars).
2. L'activité repose fortement sur la catégorie **Computers** et sur la classe **Regular**.
3. Quelques boutiques génèrent l'essentiel des ventes : un levier de croissance potentiel en développant les autres points de vente.
4. Le faible taux de retour confirme la solidité de l'offre produits.

---

## 🚀 Pistes d'amélioration

- Ajouter une page d'analyse de la **marge** (coût unitaire vs prix unitaire).
- Mettre en place un suivi des **objectifs de vente** par boutique.
- Ajouter des **prévisions** (forecast) sur le CA.
- Analyser les causes de fermeture des boutiques (`CloseReason`).

---

## 📁 Structure du dépôt

```
├── README.md
├── ContosoRetail.pbix        # Fichier Power BI
├── data/                     # Fichiers CSV sources
└── ScreenShots/                   # Captures d'écran des dashboards
    ├── boutiques.png
    ├── produits.png
    └── ventes.png
```

## 🧰 Outils

Power BI Desktop · Power Query · DAX · Modélisation en étoile · Data visualisation

---
