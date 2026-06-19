# Audit Banque Eckmühl — Pipeline scoring crédit conso

> Document à rendre au DPO Klaus Eichmann. **Max 2 pages.**
> Public cible : DPO non-technicien — vocabulaire métier, pas de jargon
> scikit-learn.

## 1. Verdict qualité

> 3 à 5 problèmes principaux **chiffrés** + ce que tu as fait pour les
> traiter dans le pipeline.



> 1. **Distribution de la Cible :** On visualise parfaitement le déséquilibre (ratio $70/30$). 
> La classe majoritaire est good_credit (700 individus). Le modèle aura naturellement tendance à être 
> "optimiste", un élément à surveiller pour éviter qu'il n'attribue de bons scores à l'aveugle.

> 2. **Distribution de l'Âge :** La courbe montre une asymétrie positive très marquée. 
> La grande majorité des demandeurs a entre 23 et 35 ans, avec un pic autour de 26-27 ans. 
> Passé 40 ans, les effectifs s'effondrent. Le modèle va donc s'entraîner majoritairement sur des profils 
> jeunes, créant un risque de sous-représentation (et donc de moins bonnes prédictions) pour les seniors.

> 3. **Montant du Crédit :** Le boxplot est sans équivoque. La boîte principale (qui contient 50% des données) 
> est ramassée entre 1 300 et 4 000 DM. En revanche, la nuée de petites croix x à droite de la moustache montre 
> une quantité massive d'outliers (valeurs atypiques) qui s'étirent jusqu'à plus de 16 000 DM. Sans normalisation, 
> ces gros montants vont fausser les calculs.

> 4. **Crosstab Sensible :** C'est la confirmation visuelle de l'anti-pattern. Le groupe male single 
> écrase complètement toutes les autres catégories en termes de volume. Plus flagrant encore : regarde le groupe 
> female divorced/separated/married : la proportion de bad_credit (barre foncée) par rapport aux good_credit 
> (barre claire) y est visuellement beaucoup plus élevée que chez les hommes célibataires. Un biais historique 
> en défaveur des femmes semble déjà inscrit dans les données.


## 2. Verdict éthique

> 2 à 3 alertes principales — variables sensibles, disparate impact
> chiffré, intersectionnalités si pertinentes.

> **Les variables suivantes sont sensibles**
> personal_status_sex 
> age 
> foreign_worker
> customer_segment (pose un immense problème d'éthique sociale et de discrimination indirecte)

L'application de la règle des 4/5ème (frontières réglementaires fixées entre 0,80 et 1,25) 
met en lumière des biais de sélection historiques sévères :

> 1. **Discrimination flagrante sur le statut de travailleur étranger (`foreign_worker`) :** 
> Le calcul révèle un Disparate Impact de **0,777** (Proportion de profil de 69,3 % pour les étrangers 
> contre 89,2 % pour les nationaux). 
> Le score étant inférieur au seuil critique de 0,80, il existe un biais systémique historique 
> défavorable aux travailleurs étrangers. 
> Un modèle entraîné sur ces données reproduira mécaniquement cette discrimination légale.

> 2. **Signal d'alerte sur la variable genre/statut civil (`personal_status_sex`) :** 
> Le Disparate Impact entre le groupe des femmes (`female divorced/separated/married`) et le groupe 
> de référence masculin (`male single`) s'établit à **0,866**. Bien que techniquement au-dessus du seuil 
> couperet des 0,80, cette valeur proche de la limite indique une pénalisation systématique des profils féminins. 
> L'interprétation fine reste toutefois limitée par la nature composite de la variable qui fusionne indûment 
> le sexe et la situation maritale 


## 3. Recommandations

> Liste courte d'actions concrètes pour Eckmühl (3-5 items).

> Pour garantir la reproductibilité et la robustesse des transformations, j'ai structuré les caractéristiques 
> du dataset selon trois axes techniques stricts au sein de notre module de préproduction (src/preprocess.py) :

> 1. Branche Numérique (NUMERIC_FEATURES) — Imputation Médiane + Standardisation :
> Dédiée aux variables continues ou discrètes quantitatives (ex: duration_months, credit_amount). 
> L'application d'un StandardScaler recentre et réduit les échelles afin de neutraliser 
> l'impact des outliers majeurs détectés lors de l'audit qualité.

> 2. Branche Ordinale (ORDINAL_FEATURES) — Imputation Modale + Ordinal Encoding :
> Dédiée aux variables possédant une hiérarchie ou une graduation sémantique naturelle (ex: savings_account, customer_segment). 
> L'ordre des modalités est explicitement configuré en dur afin de permettre au modèle de capter la progression logique sans inflation de colonnes.

> 3. Branche Categorielle Nominale (CATEGORICAL_FEATURES) — Imputation Modale + One-Hot Encoding :
> Dédiée aux variables catégorielles sans relation d'ordre sous-jacente (ex: personal_status_sex, foreign_worker). 
> Elles sont éclatées en variables binaires denses, avec le paramètre handle_unknown="ignore" 
> activé pour immuniser le système contre l'apparition de nouvelles modalités lors des futures requêtes en production.

> *Recommandation d'exclusion post-audit :* Il est fortement préconisé d'exclure les variables sensibles `personal_status_sex` et `customer_segment` avant l'étape d'entraînement du modèle final pour se conformer aux exigences d'équité (Fairness).

## 4. Limites de cet audit

> Ce que tu n'as **pas** fait, et qu'il faudrait faire plus tard.

- Pas de mitigation des biais à ce stade (cf. modules M7-M8 du parcours).
- Pas de test de robustesse sur dataset d'évaluation séparé (déjà discuté
  avec Karim).
- Il faut spliter la colonne personal_status_sex pour la remplacer par deux features distinctes et étanches lors de la phase de collecte et de stockage :
Feature 1 : applicant_sex (Genre) : male / female / non-binary / undisclosed.
Feature 2 : marital_status (Statut civil) : single / married / divorced / separated / widowed.
Fusionner ces deux dimensions empêche d'isoler mathématiquement l'effet pur du genre de celui de la situation familiale.


---

*Audit produit par Joelle, 16/06/2026, dans le cadre du brief M2-B1 ATOS.*