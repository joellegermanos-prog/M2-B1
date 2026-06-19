# Datasheet — German Credit (version Eckmühl, clean)

> **Modèle Gebru et al. (2018), version simplifiée 7 sections, 1 page max.**
> À transmettre avec le dataset Parquet livré à Eckmühl.
> Public cible : DPO + équipe métier — lisible par un non-data-scientist.

## 1. Motivation

> Pourquoi ce dataset existe-t-il ? Qui l'a créé et pour quel objectif initial ?

* **Pourquoi ce dataset existe-t-il ?**
  Ce jeu de données étendu a été conçu pour entraîner et auditer un modèle de tarification et d'octroi de crédits à la consommation pour le compte de la banque Eckmühl. L'objectif technique est d'identifier les profils à risque (défaut de paiement potentiel) tout en maximisant l'équité des décisions. 

* **Qui l'a créé et pour quel objectif initial ?**
  Les données de base historiques proviennent du jeu de données académique et industriel public *Statlog German Credit*, initialement compilé par le Professeur Dr. Hans Hofmann (Université de Hambourg) pour classifier les risques de crédit selon 20 attributs bancaires. 

## 2. Composition

> Combien d'observations, quelles colonnes, quels types, distribution de la
> cible, **variables sensibles signalées explicitement**.

| Aspect | Valeur |
|---|---|
| Nombre de lignes | 1000 |
| Nombre de colonnes | 22 |
| Cible | `credit_risk` : `good_credit` / `bad_credit` |
| Distribution cible | `good_credit` 70 % / `bad_credit` 30 % |
| Variables sensibles | `age`, `personal_status_sex` ⚠️ composite, `foreign_worker` | customer_segmment

**Schéma des colonnes** (types + modalités pour les catégorielles) :

| Colonne | Type | Modalités / range | Note |
|---|---|---|---|
| `duration_months` | int | 4 — 72 | Durée du crédit en mois |
| `credit_amount` | int | 250 — 18424 | Montant demandé (en DM) |
| `installment_rate` | int | 1 — 4 | Taux d'effort en % du revenu disponible |
| `residence_since` | int | 1 — 4 | Ancienneté dans le logement actuel (années) |
| `age` | int | 19 — 75 | ⚠️ **Variable sensible** (Risque d'âgisme) |
| `n_existing_credits` | int | 1 — 4 | Nombre de crédits en cours dans cette banque |
| `n_people_liable` | int | 1 — 2 | Nombre de personnes financièrement à charge |
| `account_status` | object | `< 0 DM`, `0-200 DM`, `>= 200 DM`, `no account` | État du compte courant du demandeur |
| `credit_history` | object | `no credits`, `all paid`, `existing paid`, `delay`, `critical` | Historique de remboursement du client |
| `purpose` | object | `car_new`, `car_used`, `furniture`, `radio/tv`, `appliances`, `repairs`, `education`, `retraining`, `business`, `others` | Motif de la demande d'emprunt |
| `savings` | object | `< 100 DM`, `100-500 DM`, `500-1000 DM`, `>= 1000 DM`, `unknown` | Épargne totale disponible |
| `employment_since` | object | `unemployed`, `< 1 year`, `1-4 years`, `4-7 years`, `>= 7 years` | Ancienneté chez l'employeur actuel |
| `personal_status_sex` | object | `male_divorced`, `female_divorced/married`, `male_single`, `male_married`, `female_single` | ⚠️ **Variable sensible** (Composite genre / statut marital) |
| `other_debtors` | object | `none`, `co-applicant`, `guarantor` | Présence de garants ou de co-emprunteurs |
| `property` | object | `real estate`, `building society`, `car`, `unknown` | Type de patrimoine possédé |
| `other_installment_plans` | object | `bank`, `stores`, `none` | Engagements financiers externes (autres crédits) |
| `housing` | object | `rent`, `own`, `for free` | Statut d'occupation du logement |
| `job` | object | `unemployed`, `unskilled_resident`, `skilled`, `management/highly_qualified` | Niveau de qualification de l'emploi |
| `telephone` | object | `yes`, `no` | Client joignable par téléphone ou non |
| `foreign_worker` | object | `yes`, `no` | ⚠️ **Variable sensible** (Statut de travailleur étranger) |
| `customer_segment` | object | `basic`, `plus`, `premium`, `private` | ⚠️ **Variable d'extension** — Ordinale. Proxy socio-économique majeur |
| `credit_risk` | object | `good_credit`, `bad_credit` | 🎯 **Variable Cible (Target)** (70% Good / 30% Bad) |


## 3. Processus de collecte

> Connu / inconnu ? Période de collecte ? Quelles personnes sont
> représentées ? Quel biais de sélection probable ?

* **Connu / inconnu ?**
  Le processus de collecte est **partiellement documenté**. 
  Le cœur du dataset d'origine (*Statlog German Credit*) a été collecté de manière académique auprès d'une grande banque allemande. Cependant, les modalités exactes d'échantillonnage initiales (quelles agences précises, critères d'exclusion à la saisie) restent inconnues.
  Le complément de données pour la colonne `customer_segment` est quant à lui **connu et tracé en interne** : il provient d'une extraction directe des bases du CRM marketing d'Eckmühl, réaligné de manière déterministe par position d'index avec le fichier principal.

* **Période de collecte ?**
  * Données historiques (Statlog) : Collectées à la fin des années 1990 (publiées en 1994).
  * Extension (`customer_segment`): Extrait et consolidé lors de la mise à jour de l'infrastructure de données en 2026.
  

* **Quelles personnes sont représentées ?**
  Le dataset représente **1 000 demandeurs de crédit à la consommation** (particuliers). Ce sont tous des clients ou prospects ayant déposé un dossier de financement auprès de l'institution financière. L'échantillon comprend des profils variés en termes d'âge (19 à 75 ans), de situation professionnelle (employés, cadres, chômeurs) et de statut de résidence (nationaux et travailleurs étrangers).

* **Quel biais de sélection probable ?**
  Le jeu de données souffre de deux biais de sélection majeurs  :
  1. Biais de survie (Survival Bias) Le dataset ne contient que des individus dont la demande de crédit a été initialement analysée ou acceptée par la banque pour l'ouverture d'un dossier.
  2. Biais de performance historique

## 4. Préprocessing appliqué (TOI)

> Ce que tu as fait dans ton pipeline. Concis, listé.

* **Pipeline**
Branche Numérique (7 features)
- Imputation : Remplacement des valeurs manquantes éventuelles par la médiane (SimpleImputer), une approche robuste pour limiter l'impact des anomalies.
- Normalisation : Application d'un StandardScaler pour écraser les écarts d'échelle massifs et neutraliser la variance disproportionnée induite par les outliers massifs de credit_amount.

Branche Ordinale (4 features)
- Imputation : Remplacement des valeurs manquantes par la modalité la plus fréquente (strategy="most_frequent").
- Codage : Transformation via OrdinalEncoder respectant une hiérarchie sémantique stricte allant du risque le plus élevé vers le profil le plus sûr (ex: pour customer_segment : basic , plus , premium , private).

Branche Catégorielle Nominale (10 features)
- Imputation : Remplacement des valeurs manquantes par la modalité la plus fréquente (le mode).
- Codage : Transformation en variables binaires via OneHotEncoder.


## 5. Usages prévus / à éviter

> Pour quoi ce dataset doit-il être utilisé (et pour quoi il ne **doit pas**
> l'être).

### Usages prévus
* **Entraînement de modèles de Score de Crédit (Modélisation de risques) :** Idéal pour concevoir, tester et benchmarker des algorithmes de classification binaire visant à évaluer la probabilité de défaut de paiement d'un client.
* **Recherche sur l'Équité des Algorithmes (Fairness Audit) :** Ce dataset standardisé est un excellent support pour mesurer l'impact des biais discriminatoires (genre, âge, statut socio-économique) et tester des techniques d'atténuation des biais (*reweighing*, *disparate impact removal*).
* **Optimisation de pipelines** Validation de la robustesse d'architectures de pré-traitement (validation de schémas, tests de contrats, persistance au format Parquet compressé).

### Usages à éviter
* **Octroi automatique de crédit en production sans filtrage éthique :** Ce dataset ne doit pas être utilisé tel quel pour industrialiser un système de décision automatique sans avoir préalablement retiré ou masqué les variables sensibles (`personal_status_sex`, `age`).
* **Utilisation brute de la variable `customer_segment` :** utiliser la colonne de segmentation marketing comme critère de décision d'octroi de prêt. L'analyse exploratoire prouvant qu'elle sert de proxy de richesse, son utilisation directe entraînerait une discrimination socio-économique systématique, pénalisant arbitrairement les clients les plus modestes (segment Basic).


## 6. Distribution

> À qui ce dataset est-il transmis ? Sous quelle forme ?

* Transmis à : aux équipes internes du projet
* Format : Parquet (compressé)
* Conditions :
  * Confidentialité Restreinte (Usage Interne) :** Ce fichier contient des informations bancaires et des variables sensibles simulées. Il ne doit faire l'objet d'aucune diffusion publique ou externe à l'infrastructure sécurisée du projet.
  * Accès via contrôle de version (Data Version Control) 

## 7. Maintenance

> Qui maintient ce dataset ? Comment signaler un problème ? Quelle version ?

Mainteneur·euse : équipe projet
Version : v1.0.0 — 16 juin 2026.
Contact issue : Ouvrir un ticket


---

*Datasheet produite par Joelle, 16 juin 2026, dans le cadre du brief M2-B1 ATOS.*