# M2-B1 — Squelette repo (Banque Eckmühl — audit + pipeline)

> **Repo template GitHub.** Clique sur **« Use this template »** en haut à
> droite de cette page → **Create a new repository** → nomme-le
> `M2-B1-pipe-eckmuhl-<prénom>` sur **ton** compte GitHub personnel.
> C'est ce nouveau repo que tu cloneras pour travailler.

---

## 🚀 Démarrage (4 commandes)

```bash
# 0. Clone ton repo perso fraîchement créé
git clone git@github.com:<ton-user>/M2-B1-pipe-eckmuhl-<prenom>.git
cd M2-B1-pipe-eckmuhl-<prenom>

# 1. Environnement virtuel
python -m venv .venv && source .venv/bin/activate     # Linux/macOS
# .venv\Scripts\activate                              # Windows

# 2. Dépendances
pip install -r requirements.txt

# 3. Vérification
jupyter notebook notebooks/M2-B1_template.ipynb       # → s'ouvre dans le navigateur
```

Si ces 4 commandes marchent, ton poste est prêt.

> ⚠️ Le dataset `german_credit_raw.csv` te sera fourni mardi 9h par la
> formatrice (Discord). Place-le dans `data/`. Le `.gitignore` exclut
> `data/*.csv` du commit — ne te casse pas la tête.

---

## 📁 Structure du repo

```
M2-B1-pipe-eckmuhl-<prenom>/
├── data/
│   ├── german_credit_raw.csv          # fourni mardi (gitignored)
│   └── german_credit_clean.parquet    # produit en tâche 6 (gitignored)
├── notebooks/
│   └── M2-B1_template.ipynb           # à dupliquer en M2-B1_<prenom>_audit.ipynb
├── src/
│   ├── preprocess.py                  # squelette Pipeline avec TODO
│   └── pipeline.joblib                # produit en tâche 5 (gitignored)
├── ressources/                        # 📚 mini-cours d'appui (lecture juste-à-temps)
│   ├── README.md                      # ordre de mobilisation + objectifs
│   ├── 01_Audit_qualite_pandas_essentiel.md
│   ├── 02_Disparate_impact_essentiel.md
│   ├── 03_ColumnTransformer_Pipeline_essentiel.md
│   ├── 04_Parquet_pyarrow_essentiel.md
│   ├── 05_Datasheet_Gebru_essentiel.md
│   └── liens_officiels.md
├── audit.md                           # template à remplir (max 2 pages)
├── datasheet.md                       # template Gebru à remplir (1 page)
├── requirements.txt
├── .gitignore
└── README.md (ce fichier — à compléter)
```

---

## 📚 Mini-cours d'appui

Les **5 mini-cours pédagogiques** du brief sont fournis dans
[`./ressources/`](./ressources/). Lecture juste-à-temps, ~15-20 min chacun :

| Tâche | Mini-cours |
|---|---|
| Audit qualité d'un dataset tabulaire | [`01_Audit_qualite_pandas_essentiel.md`](./ressources/01_Audit_qualite_pandas_essentiel.md) |
| Disparate impact (règle des 4/5) | [`02_Disparate_impact_essentiel.md`](./ressources/02_Disparate_impact_essentiel.md) |
| Industrialisation Pipeline + ColumnTransformer | [`03_ColumnTransformer_Pipeline_essentiel.md`](./ressources/03_ColumnTransformer_Pipeline_essentiel.md) |
| Persistance Parquet (pyarrow) | [`04_Parquet_pyarrow_essentiel.md`](./ressources/04_Parquet_pyarrow_essentiel.md) |
| Datasheet Gebru (7 sections) | [`05_Datasheet_Gebru_essentiel.md`](./ressources/05_Datasheet_Gebru_essentiel.md) |

Cf. [`./ressources/README.md`](./ressources/README.md) pour l'ordre de mobilisation détaillé.

---

## 🧭 Démarche attendue

1. **Découverte** (30 min) — charge le CSV, repère cible + variables sensibles
2. **Audit qualité** (1 h) — manquants, outliers, 4 visualisations
3. **Audit éthique** (1 h) — disparate impact sur 2+ variables sensibles
4. **Choix prétraitement** (30 min) — remplis les listes dans `preprocess.py`
5. **Industrialisation** (1 h 15) — `Pipeline` + `ColumnTransformer`, persisté
6. **Parquet + datasheet** (45 min) — sauve propre, complète `datasheet.md`
7. **Synthèse** (30 min) — complète `audit.md` (verdict + recommandations)

→ Compétences visées : **C2 — imiter** + **C3 — imiter puis adapter**.

---

## ✅ Conventions de code

- Python 3.11+
- Type hints sur toutes les signatures publiques
- Pas de `print` (utiliser `display()` ou `logging`)
- `random_state=42` partout où il y a de l'aléa
- `pathlib.Path` pour les chemins (pas de `os.path`)

---

## 🆘 Bloqué·e ?

1. Relis le mini-cours correspondant à ta tâche actuelle (cf.
   [`./ressources/README.md`](./ressources/README.md)).
2. Vérifie que `german_credit_raw.csv` est bien dans `data/` (sinon le
   notebook plante au chargement).
3. Si tu butes sur **`ColumnTransformer`** : prends 10 min pour faire un
   exemple ultra-minimal en console (1 numérique + 1 catégorielle), puis
   remonte vers ton vrai dataset.
4. Demande en direct mardi sur Discord — `fil-M2-B1`. N'attends pas
   30 min sur le même point.