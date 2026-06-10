# EDF-Carbon-Analysis

# EDF Carbon Emissions Analysis (2019–2024)

End-to-end data analysis project exploring EDF Group's CO₂ emissions trends across countries and over time (2019–2024), using Open Data, PostgreSQL, SQL, and Python.

## Executive Summary

This project analyzes EDF Group’s CO₂ emissions over the 2019–2024 period in order to identify carbon reduction trends, the geographical distribution of emissions, and the main contributing countries.

### Key Insights
- EDF's total CO₂ emissions decreased by approximately 48% between 2019 and 2024.
- France accounted for approximately 40–45% of the Group's total emissions over the period.
- Emissions are highly concentrated in a limited number of countries.
- The decline in emissions accelerated from 2022 onwards.
- The overall trend suggests a gradual transition toward a lower-carbon energy mix.

---

## 1. Project Overview

### Project Objective

The objective of this project is to:

- Analyze EDF Group's CO₂ emissions
- Identify the highest-emitting countries
- Study emission trends over time

---

## 2. Technologies Used

| Tool | Purpose |
|------|------------|
| PostgreSQL | Data storage and querying |
| SQL | Data analysis and exploration |
| Python | Data visualization |
| GitHub | Versioning and portfolio |

---

## 3. Data import and structuring

### Data Description

Source: Open Data EDF

Scope: émissions de CO₂ consolidées par pays

Period: 2019 → 2024

Unit: ktonnes CO₂

Level: groupe EDF (périmètre international)

### Database

The data was imported into a PostgreSQL database with the following structure:

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
### Execution Evidence (PostgreSQL)

#### Table Creation
![PostgreSQL Table Creation](visuals/create_table.png)

#### CSV Data Import
![CSV Import into PostgreSQL](visuals/import_csv.png)

---

## 4. Data Cleaning

### 4.1. Missing Values Check

#### Objective : Missing Values Check
Identify missing data that could bias statistical analyses and SQL aggregations.

#### SQL Query
Do the data contain missing values that could affect the analysis?
```sql
SELECT *
FROM edf_co2
WHERE "Emissions CO2" IS NULL
   OR "Année" IS NULL
   OR "Périmètre spatial" IS NULL;
```

#### Result

![NULL Values Quality Check](visuals/null_values_check.png)

No NULL values were detected in the analyzed columns.

#### Interpretation

The dataset is complete and suitable for statistical and time-series analysis.

### 4.2. Duplicate checking

#### Objective : Detect duplicate imports
This analysis aims to identify potential duplicates in the imported data in order to avoid overestimation of emissions.

#### SQL Query
Have any rows been imported multiple times into the database?
```sql
SELECT "Année",
       "Périmètre spatial",
       COUNT(*)
FROM edf_co2
GROUP BY "Année", "Périmètre spatial"
HAVING COUNT(*) > 1;
```

#### Result

![Duplicate Records Check](visuals/duplicate_check.png)

No duplicates were identified in the data.

#### Interpretation

Each year/spatial boundary combination appears only once in the database, ensuring the consistency of the analysis.

### 4.3. Negative Values Check

#### Objective : Detect inconsistent values
Since carbon emissions are positive values, this check helps identify potential data entry errors or anomalies in the dataset.

#### SQL Query
Are there any inconsistent or negative emission values in the dataset?
```sql
SELECT *
FROM edf_co2
WHERE "Emissions CO2" < 0;
```

#### Result

![Negative Values Detection](visuals/negative_values_check.png)

No negative values were detected.

#### Interpretation

The carbon emissions data shows consistent values that are compatible with the environmental analyses performed.

### 4.4. Available Years Check

#### Objective : Verify the study period
This check confirms that the data correctly covers the study period 2019–2024.

#### SQL Query
Do the data correctly cover the entire 2019–2024 study period?
```sql
SELECT DISTINCT "Année"
FROM edf_co2
ORDER BY "Année";
```

#### Result

![Available Years Validation](visuals/years_check.png)

The dataset covers the entire period from 2019 to 2024.

#### Interpretation

The continuity of the dataset allows reliable trend analysis over time.

### 4.5. Units Consistency Check

#### Objective : Ensure that all emissions are reported using the same unit of measurement.
This check ensures that all variables are expressed in consistent and appropriate units in order to guarantee the reliability and comparability of the results.

##### SQL Query
Are all emissions expressed using the same unit?
```sql
SELECT DISTINCT "Unité"
FROM edf_co2;
```
#### Result

![Measurement Units Consistency Check](visuals/units_check.png)

All data are reported in kilotonnes (ktCO₂).

#### Interpretation

The consistency of measurement units ensures reliable comparisons across countries and years.

### 4.6. Verification of extreme values

#### Objective : Identify unusually high emission values.
This analysis helps identify the spatial perimeter with the highest emissions, as well as any potential statistical anomalies.

#### SQL Query
Which spatial perimeter have the highest emissions?
```sql
SELECT *
FROM edf_co2
ORDER BY "Emissions CO2" DESC
LIMIT 10;
```

#### Result

The highest values mainly correspond to the global scope ("World"), followed by France.
Global emissions gradually declined between 2019 and 2024.

#### Interpretation

![Extreme Emissions Values Analysis](visuals/extreme_values_check.png)

The observed values are consistent with the scope of the study and highlight the significant contribution of the global perimeter and the overall downward trend in EDF's carbon emissions.

### 4.7. Spatial Perimeter Consistency Check

#### Objective : Identify naming inconsistencies.
The different values in the "Spatial Perimeter" field were reviewed to identify any potential naming inconsistencies.

#### SQL Query
Are there any naming inconsistencies across countries or geographical scopes?
```sql
SELECT DISTINCT "Périmètre spatial"
FROM edf_co2
ORDER BY "Périmètre spatial";
```

#### Detected Inconsistencies

![Spatial Perimeter Inconsistencies](visuals/data_inconsistencies_detected.png)

A naming inconsistency was identified for China.

The values "People's Republic of China" and "People's republic of China" differed only because of capitalization and were therefore treated as separate categories by PostgreSQL.

This issue could lead to duplicate analyses, incorrect averages and misleading country rankings.

#### Correction Applied

![Data Normalization Correction Applied](visuals/data_correction_applied.png)

A data normalization procedure was performed to standardize country labels.

```sql
Requête SQL
UPDATE edf_co2
SET "Périmètre spatial" = 'République populaire de Chine'
WHERE "Périmètre spatial" = 'République Populaire de Chine';
```

#### Verification After Correction

A new validation was performed to confirm data consistency.
```sql
Requête SQL
SELECT DISTINCT "Périmètre spatial"
FROM edf_co2
ORDER BY "Périmètre spatial";
```

![Post-Correction Verification](visuals/post_correction_verification.png)

#### Conclusion

This cleaning process improved the quality and reliability of the analysis and illustrates the importance of data preparation in ESG and environmental analytics projects.

---

## 5. Analyses Performed

### 5.1. Global Emissions Trend
Analysis of EDF Group's global CO₂ emissions between 2019 and 2024.

#### Main SQL Query
How did EDF Group's global CO₂ emissions evolve between 2019 and 2024?
```sql
SELECT "Année",
       SUM("Emissions CO2") AS total_emissions
FROM edf_co2
WHERE "Périmètre spatial" = 'Monde'
GROUP BY "Année"
ORDER BY "Année";
```
#### Preuves d’exécution (PostgreSQL)
![Global CO2 Emissions Trend 2019-2024](visuals/evolution_des_emissions_mondiales.png)

#### Interpretation

On observe une réduction progressive et continue des émissions de CO₂ sur l’ensemble de la période.

Cette évolution peut s’expliquer par des politiques de réduction des émissions et le renforcement des obligations de reporting carbone.

#### Conclusion

Sur la période 2019–2024, le périmètre mondial analysé montre une tendance claire à la baisse des émissions de CO₂, suggérant une amélioration progressive de la performance environnementale.

---

### 5.2. Top Emitting Countries (2024)
Identification des pays les plus émetteurs de CO₂ en 2024 (hors périmètre global).

### Main SQL Query
Quels sont les pays les plus émetteurs de CO₂ en 2024 au sein du groupe EDF ?
```sql
SELECT "Périmètre spatial",
       "Emissions CO2"
FROM edf_co2
WHERE "Année" = 2024
AND "Périmètre spatial" != 'Monde'
ORDER BY "Emissions CO2" DESC;

```

#### Preuves d’exécution (PostgreSQL)
![Top CO2 Emitting Countries in 2024](visuals/top_emitters_2024.png)

#### Interpretation

L’analyse des émissions de CO₂ du groupe EDF en 2024 met en évidence une forte concentration des émissions sur quelques pays clés.

La France apparaît comme le principal contributeur avec plus de 7 293 unités d’émissions, suivie par l’Italie et la Chine. Cette répartition reflète l’importance historique des activités du groupe EDF en Europe ainsi que sa présence internationale sur plusieurs marchés énergétiques.

Les émissions élevées observées en Italie et en Chine peuvent s’expliquer par une présence industrielle importante, des infrastructures énergétiques plus carbonées ou des mix énergétiques nationaux davantage dépendants des énergies fossiles.

À l’inverse, certains pays comme le Royaume-Uni, le Canada ou l’Inde présentent des niveaux d’émissions très faibles dans le périmètre EDF, ce qui peut traduire une présence plus limitée du groupe, des activités moins intensives en carbone ou un portefeuille énergétique davantage orienté vers des énergies bas carbone.

#### Conclusion

Globalement, les résultats montrent que les émissions carbone du groupe EDF ne sont pas réparties uniformément entre les différents pays d’implantation. Quelques zones géographiques concentrent une part majeure des émissions totales du groupe.

### 5.3. France vs Monde
Comparaison des émissions de la France par rapport au total mondial afin d’analyser son poids relatif.

#### Requêtes SQL principales
Quelle part des émissions mondiales du groupe EDF est représentée par la France ?
```sql
WITH emissions AS (
    SELECT 
        "Année",
        MAX(CASE WHEN "Périmètre spatial" = 'France' 
            THEN "Emissions CO2" END) AS france,
        MAX(CASE WHEN "Périmètre spatial" = 'Monde' 
            THEN "Emissions CO2" END) AS monde
    FROM edf_co2
    GROUP BY "Année")

SELECT 
    "Année",
    ROUND(france::numeric,2) AS emissions_france,
    ROUND(monde::numeric,2) AS emissions_monde,
    ROUND((france / monde * 100)::numeric,2) AS poids_france_pct
FROM emissions
ORDER BY "Année";
```
#### Preuves d’exécution (PostgreSQL)
![France vs Global Emissions Comparison](visuals/france_vs_monde.png)

#### Interpretation

L’analyse comparative entre les émissions françaises et les émissions mondiales du groupe EDF met en évidence une diminution progressive des émissions de CO₂ sur l’ensemble de la période 2019–2024.

Les émissions mondiales du groupe passent d’environ 32 249 en 2019 à 16 096 en 2024, traduisant une réduction significative de l’empreinte carbone globale d’EDF. La France suit une tendance similaire avec une baisse des émissions passant de 14 094 à 7 294 sur la même période.
Malgré cette diminution, la France conserve un poids important dans les émissions totales du groupe. En moyenne, elle représente entre 40 % et 45 % des émissions mondiales d’EDF sur la période étudiée.

Cette concentration peut s’expliquer par l’importance historique du marché français pour EDF, la taille des infrastructures énergétiques nationales et la centralisation d’une partie importante des activités du groupe en France.

Les résultats suggèrent également qu’EDF a engagé une trajectoire globale de réduction carbone entre 2019 et 2024, potentiellement liée aux politiques de transition énergétique, à l’évolution du mix énergétique, à la fermeture progressive de certaines activités fortement émettrices ou à l’amélioration des performances environnementales.

### 5.4. Émissions moyennes par pays

#### Requêtes SQL principales
Quels pays présentent les émissions moyennes les plus élevées sur la période étudiée ?
```sql
SELECT "Périmètre spatial", AVG("Emissions CO2") 
FROM edf_co2
GROUP BY "Périmètre spatial"
ORDER BY AVG("Emissions CO2") DESC;

```
#### Preuves d’exécution (PostgreSQL)
![Average CO2 Emissions by Country](visuals/emissions_moyennes_par_pays.png)

### 5.5. Variations annuelles des émissions

#### Requêtes SQL principales
Quelles sont les variations annuelles des émissions de CO₂ du groupe EDF ?
```sql
SELECT 
    "Année",
    ROUND(SUM("Emissions CO2")::numeric,2) AS emissions_totales,

    ROUND(((SUM("Emissions CO2")
	        - LAG(SUM("Emissions CO2"))OVER (ORDER BY "Année"))
			/LAG(SUM("Emissions CO2"))OVER (ORDER BY "Année"))::numeric * 100,2) AS variation_pct
FROM edf_co2
GROUP BY "Année"
ORDER BY "Année";
```

#### Preuves d’exécution (PostgreSQL)
![Annual CO2 Emissions Variations](visuals/variations_annuelles_des_emissions.png)

#### Interpretation

L’analyse de l’évolution des émissions de CO₂ du groupe EDF entre 2019 et 2024 met en évidence une tendance globale fortement baissière, avec une réduction d’environ 48 % sur la période étudiée.
Cette diminution n’est pas linéaire. Après une forte baisse entre 2019 et 2020, les émissions se stabilisent légèrement en 2021 avant de reprendre une trajectoire descendante plus marquée à partir de 2022.
L’année 2023 constitue le point le plus significatif de cette dynamique avec une baisse de près de 21 %, traduisant une accélération des efforts de réduction des émissions.
En 2024, la baisse se poursuit mais à un rythme plus modéré, suggérant une phase de consolidation des gains environnementaux.

#### Conclusion
Sur la période étudiée, EDF présente une trajectoire de réduction carbone nette et structurée, avec une accélération des efforts à partir de 2022. Cette évolution peut être interprétée comme le résultat combiné de politiques de transition énergétique, d’optimisation des opérations et de transformation progressive du mix énergétique du groupe.

### 5.6. Top 10 Highest-Emitting Countries (2019–2024)

#### Main SQL Query
Which countries accounted for the highest cumulative CO₂ emissions over the 2019–2024 period?
```sql
SELECT "Périmètre spatial", SUM("Emissions CO2")
FROM edf_co2
GROUP BY "Périmètre spatial"
ORDER BY SUM("Emissions CO2") DESC
LIMIT 10;
```

#### Preuves d’exécution (PostgreSQL)
![Top 10 Cumulative Emitters 2019-2024](visuals/top_10_emitters_2024.png)

---

## 6. Python Visualizations

---

## 7. Overall Conclusion

This project analyzing the EDF Group’s carbon emissions highlights an overall downward trend in CO₂ emissions between 2019 and 2024.

The analysis shows that emissions are heavily concentrated in a few key countries. France accounts for a significant share of the Group’s total emissions, while global emissions follow a gradual downward trajectory over the period studied.

This project also demonstrates the combined use of PostgreSQL, SQL, and Python in a comprehensive data analysis approach applied to ESG and climate-related challenges.

Beyond the technical results, this study provides a better understanding of the geographical distribution of carbon emissions and the dynamics of the energy transition within a large international group.
