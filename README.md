# databricks-pyspark-taxi

# 🚖 Yellow Taxi – Databricks Project (2024–2025)

## 🎯 Objectif
Ce projet a pour but d’analyser les données publiques **NYC Yellow Taxi** afin d’identifier les **tendances de mobilité urbaine** et de produire des **indicateurs mensuels** pour les années **2024 et 2025**.  

Les données brutes (format **Parquet**) sont traitées sur **Azure Databricks**, nettoyées, agrégées, puis exportées vers une **base Azure SQL** pour permettre le reporting et la visualisation des résultats.

---

## ⚙️ Pipeline général

### 1️⃣ Ingestion
- Lecture des fichiers Parquet : `yellow_tripdata_2024-*` et `yellow_tripdata_2025-*`
- Union des datasets 2024 & 2025
- Contrôle des années (`pickup` / `dropoff`)

### 2️⃣ Nettoyage
- Conversion des timestamps au format Spark `timestamp`
- Suppression des doublons
- Filtrage via la table de référence `taxi_universal_ref` :
  - `VendorID`, `RateCodeID`, `PaymentType`, `store_and_fwd_flag`
- Calcul de la durée du trajet (`trip_duration_min`)
- Filtrage des valeurs aberrantes :
  - distance ∈ (0, 100)
  - durée ∈ [1, 180] minutes
  - passagers ∈ [1, 6]
  - montants ≥ 0

### 3️⃣ Enrichissement
- Ajout des **labels descriptifs** via `taxi_universal_ref`
- Jointure avec `taxi_zone_lookup` pour les noms de zones (`PU_Zone`, `PU_Borough`)

### 4️⃣ Calcul des indicateurs (KPI)

| KPI | Description |
|------|-------------|
| **Top 10 zones de départ / mois** | Classement des zones les plus fréquentées |
| **Durée moyenne / mois** | Temps moyen d’un trajet (minutes) |
| **Distance moyenne par type de paiement** | Distance moyenne selon le mode de paiement |
| **Montant moyen par nombre de passagers** | Tarif moyen et total selon le nombre de passagers |
| **Somme totale des pourboires / mois** | Total des tips collectés par mois |

### 5️⃣ Export vers Azure SQL
Les DataFrames finaux sont exportés via JDBC vers **Azure SQL**, en utilisant des **secrets Databricks** stockés dans un `local-scope`.

---

## 💾 Tables créées sur Azure SQL

| Objectif | Table Azure SQL |
|-----------|-----------------|
| Top 10 zones de départ par mois | `dbo.top10_pickup_zones_monthly_dcatry` |
| Durée moyenne des trajets par mois | `dbo.avg_duration_by_month_dcatry` |
| Distance moyenne par type de paiement | `dbo.avg_distance_by_payment_dcatry` |
| Montant moyen par nombre de passagers | `dbo.avg_fare_by_passenger_count_dcatry` |
| Somme totale des pourboires par mois | `dbo.total_tips_by_month_dcatry` |
| Données nettoyées complètes | `dbo.yellow_taxi_clean_2024_2025_dcatry` |

---

## 🔐 Gestion des secrets

Les identifiants de connexion à Azure SQL sont sécurisés dans un **Secret Scope local Databricks** :

```python
user = dbutils.secrets.get("local-scope", "sql-user")
password = dbutils.secrets.get("local-scope", "sql-pass")

jdbc_url = (
    f"jdbc:sqlserver://monserveur.database.windows.net:1433;"
    f"database=mobility_analytics;"
    "encrypt=true;trustServerCertificate=false;"
)

```
## 🧰 Stack technique
- Azure Databricks (PySpark)
- Azure SQL Database
- Python / PySpark SQL
- JDBC Connector
- Databricks Secret Scope (local-scope)

## 👩‍💻 Auteur
Dorothée Catry – “dcatry”
Data Engineer (Azure Databricks / Azure SQL)
📅 2025
