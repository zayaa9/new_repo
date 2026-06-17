# ZMS/HUR Credit Risk Scoring

ZMS/HUR харилцагчдын зээлийн эрсдэлийн скоринг модел. 30+ хоногийн хугацаа
хэтрэлт (DPD) таамаглаж, харилцагч бүрт **350–850** муж дахь оноо гаргана
(өндөр оноо = бага эрсдэл).

## Тойм

Гурван өөр аргаар модел сургаж, харьцуулна:

| NB | Арга | Тайлбар |
|----|------|---------|
| `01_eda` | EDA | Null, сегмент, таргет, IV, WoE, корреляц |
| `02_model_scorecard` | WoE/IV + Logistic | optbinning Scorecard, ил тод оноо |
| `03_model_advanced` | LightGBM segment ensemble | Optuna HPO + SMOTE + 3 сегмент |
| `04_model_xbooster` | LightGBM + PDO | xBooster-style PDO scaling + SHAP |

**Таргет:** `30` (30+ DPD, гол) ба `before_first_close_30+` (нэмэлт).

## Суулгах

```bash
git clone <repo-url>
cd zms_credit_risk
pip install -r requirements.txt
```

## Ашиглах

1. Датаг `data/ZMS_HUR_data_new.xlsx` гэж байрлуул (`Sheet2`).
2. Notebook-уудыг дарааллаар нь ажиллуул:

```bash
jupyter lab
# 01_eda → 02_model_scorecard → 03_model_advanced → 04_model_xbooster
```

Сургасан модел `models/`, график `reports/`-д хадгалагдана.

## Бүтэц

```
zms_credit_risk/
├── data/            # түүхий дата (.gitignore)
├── notebooks/       # 01–04 NB
├── models/          # сургасан .joblib + scorecard CSV
├── reports/         # график гаралт
├── src/             # config, features, feature_selection, metrics
├── requirements.txt
├── CLAUDE.md
└── README.md
```

## Гол санаа

- **DRY:** бүх NB `src/` модулиудаас feature engineering, метрик, feature
  сонголтыг импортолно. Логик нэг газар.
- **Leakage-гүй:** IV, WoE, target encoding, SMOTE бүгд train split дээр л fit
  хийгдэнэ.
- **Тохиргоо нэг газар:** зам, таргет, scaling param бүгд `src/config.py`-д.

## Хэмжүүр

- **AUC** — random=0.5, perfect=1.0. Зорилго > 0.70.
- **Gini** = 2·AUC − 1. Салбарын босго > 0.30.
- **KS** — Good/Bad тархалтын хамгийн их зөрүү.

## Анхаарах

Хуучин датан дахь зарим багана (`actor_id`, `scr_id`, огноонууд) энэ шинэ датанд
байхгүй тул `src/config.py` доторх drop жагсаалт автоматаар тааруулна. `30` таргет
INTEGER баганын нэрээр уншигдах тул `config.load_data()` нь түүнийг string болгож
хувиргана.
