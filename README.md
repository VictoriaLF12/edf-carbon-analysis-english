# edf-carbon-analysis

# EDF Carbon Emissions Analysis (2019–2024)

Projet d’analyse des émissions de CO₂ du groupe EDF par pays et par année, basé sur des données Open Data.

---

## Objectif du projet

Ce projet a pour objectif de :

- Analyser les émissions de CO₂ du groupe EDF
- Identifier les pays les plus émetteurs
- Étudier l’évolution des émissions dans le temps
- Illustrer une démarche complète d’analyse de données (SQL + Python + PostgreSQL)

---

## Technologies utilisées

| Outil | Utilisation |
|------|------------|
| PostgreSQL | Stockage et requêtes sur les données |
| SQL | Analyse et exploration des données |
| Python | Visualisation des données |
| GitHub | Versioning et portfolio |

---

## Architecture du projet

---

## Base de données

Les données ont été importées dans une base PostgreSQL avec la structure suivante :

```sql
CREATE TABLE edf_co2 (
    "Année" INT,
    "Périmètre juridique" TEXT,
    "Legal perimeter" TEXT,
    "Périmètre spatial" TEXT,
    "Spatial perimeter" TEXT,
    "Emissions CO2" FLOAT,
    "Unité" TEXT,
    "Méthode de consolidation" TEXT,
    "Consolidation method" TEXT
);

```
---

# Analyses réalisées

### 🌍 Évolution des émissions mondiales
Analyse de l’évolution des émissions de CO₂ du périmètre mondial sur la période 2019–2024.

---

### 🏭 Top pays émetteurs (2024)
Identification des pays les plus émetteurs de CO₂ en 2024 (hors périmètre global).

---

### 🇫🇷 France vs Monde
Comparaison des émissions de la France par rapport au total mondial afin d’analyser son poids relatif.

---

## 🧠 Requêtes SQL principales

### 📈 Évolution des émissions mondiales

```sql
SELECT "Année",
       SUM("Emissions CO2") AS total_emissions
FROM edf_co2
WHERE "Périmètre spatial" = 'Monde'
GROUP BY "Année"
ORDER BY "Année";

```

### 🏭 Top pays émetteurs (2024)

```sql
SELECT "Périmètre spatial",
       "Emissions CO2"
FROM edf_co2
WHERE "Année" = 2024
AND "Périmètre spatial" != 'Monde'
ORDER BY "Emissions CO2" DESC;

```

### 🇫🇷 France vs Monde

```sql
SELECT "Année",
       "Périmètre spatial",
       "Emissions CO2"
FROM edf_co2
WHERE "Périmètre spatial" IN ('France', 'Monde')
ORDER BY "Année";

```

## 📈 Résultats clés

- Les émissions sont concentrées sur quelques pays majeurs
- Tendance mondiale globalement baissière
- France et Italie figurent parmi les principaux contributeurs
- Variabilité importante selon les zones géographiques
