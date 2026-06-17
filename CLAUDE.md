# CLAUDE.md

Энэ файл нь Claude Code (болон бусад AI туслахад) энэ төсөл дээр ажиллах
контекстийг өгнө.

## Төслийн зорилго

ZMS/HUR харилцагчдын зээлийн эрсдэлийн (credit risk) скоринг модел. Гол зорилго
нь 30+ хоногийн хугацаа хэтрэлт (DPD) таамаглах, харилцагч бүрт 300–900муж дахь
score онооход оршино.

## Дата

- **Файл:** `data/ZMS_HUR_data_new.xlsx`, sheet `Sheet2` (~57k мөр, 78 багана)
- **Таргет:**
  - `30` — 30+ DPD (гол таргет, INTEGER баганын нэр; `config.load_data` нь string болгож хувиргана)
  - `before_first_close_30+` — анхны зээл хаагдахаас өмнөх 30+ DPD (нэмэлт)
- **Сегмент** (`hashur`, `haszms`-ээс):
  - `both` — hashur=1 & haszms=1
  - `hur_only` — hashur=1
  - `zms_only` — haszms=1
  - `neither` — бусад
- **Категори:** `city_name`
- ⚠️ Хуучин датан дахь `actor_id`, `scr_id`, огнооны баганууд энэ датанд **байхгүй**.

## Бүтэц

```
zms_credit_risk/
├── data/                    # түүхий дата (git-д орохгүй)
├── notebooks/
│   ├── 01_eda.ipynb              # EDA: null, segment, IV, корреляц
│   ├── 02_model_scorecard.ipynb  # WoE/IV + Logistic scorecard
│   ├── 03_model_advanced.ipynb   # LightGBM segment ensemble + Optuna + SMOTE
│   └── 04_model_xbooster.ipynb   # LightGBM + PDO scoring + SHAP
├── models/                  # сургасан .joblib, scorecard CSV (git-д орохгүй)
├── reports/                 # график гаралт (git-д орохгүй)
├── src/
│   ├── config.py            # зам, таргет, сегмент, load_data
│   ├── features.py          # engineer_features (бүх NB ашиглана)
│   ├── feature_selection.py # IV + корреляцийн шүүлт
│   └── metrics.py           # AUC/Gini/KS + график функцууд
├── requirements.txt
└── README.md
```

## Чухал зарчим

- **Бүх NB `src/`-ээс импортолно** — код давхардуулахгүй. Шинэ feature эсвэл
  таргет нэмэхдээ `src/`-г засна, NB-уудыг биш.
- **Leakage-аас сэргийлэх:** IV, WoE, target encoding, SMOTE — бүгд ЗӨВХӨН train
  split дээр fit хийгдэнэ.
- **Score конвенц:** өндөр оноо = бага эрсдэл, муж 350–850.
- Зам, таргет өөрчлөгдвөл `src/config.py` дотор л засна.

## Ажиллуулах

```bash
pip install -r requirements.txt
jupyter lab        # notebooks-ийг дарааллаар нь ажиллуул: 01 → 02 → 03 → 04
```

`03_model_advanced.ipynb` дотор `N_TRIALS` (Optuna) тоог багасгаснаар хурдан
жишиж болно.

## Стайл

- Тайлбар, markdown монгол хэлээр.
- Хэмжүүр: AUC, Gini (=2·AUC−1), KS. Зорилго AUC > 0.70.
