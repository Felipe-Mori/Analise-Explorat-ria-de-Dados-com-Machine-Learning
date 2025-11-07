# Health&Life Analytics — Café & Sono

Projeto end-to-end para **explorar**, **visualizar** e **prever** a **qualidade do sono (Sleep_Quality)** a partir de consumo de café, estresse e características dos clientes. Pensado para rodar em um único comando e ser fácil de adaptar.

---

## Objetivo
Apoiar a Health&Life Analytics a:
1) **Entender** como cafeína e estresse impactam sono e sua qualidade.  
2) **Prever** *Sleep_Quality* para orientar recomendações (redução de cafeína à tarde/noite, higiene do sono, manejo do estresse, rotina de horários).

---


## Pré-requisitos
- Python 3.9+  
- Pip (ou uv/poetry, se preferir)

**requirements.txt**
```
pandas
numpy
matplotlib
scikit-learn
joblib
```

---

## Instalação
```bash
# criar e ativar venv
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS/Linux
source .venv/bin/activate

# instalar dependências
pip install -r requirements.txt
```

---

## Como executar
Coloque seu CSV em `data/` (ex.: `data/synthetic_coffee_health_10000(in).csv`) e rode:

```bash
python main.py --data "data/synthetic_coffee_health_10000(in).csv"
```

Parâmetros opcionais:
```
--outputs_dir   pasta de saídas (default: outputs)
--models_dir    pasta de modelos (default: models)
```

Exemplo:
```bash
python main.py --data data/meus_dados.csv --outputs_dir reports --models_dir artefacts
```

---

## O que o pipeline faz
1) **EDA & Qualidade**
- Dimensões, tipos, nulos, duplicados.
- Estatísticas e frequências chave.

2) **Visualizações (salvas em `outputs/figs/`)**
- Boxplot: horas de sono por **gênero**.
- Dispersão com tendência: **idade × cafeína (mg)**.
- Heatmap: média de horas de sono por **faixa etária × estresse**.
- Barras: sono por **quartil de cafeína**, segmentado por **estresse**.

3) **Features derivadas**
- `Caffeine_per_SleepHour = Caffeine_mg / Sleep_Hours`  
- `Pulse_Pressure = Systolic − Diastolic` (se houver)  
- `Stress_Num` (mapeamento ordinal robusto)

4) **Modelagem (target = `Sleep_Quality`)**
- Pré-processamento: imputação (num/cat), one-hot (cat) e padronização (num).
- Split estratificado (80/20).
- Modelos: LogisticRegression e RandomForest.
- Avaliação: acurácia, classification report e matriz de confusão (PNG).
- Diagnóstico rápido de over/underfitting (gap treino vs teste).

5) **Export**
- `outputs/hla_processed_dataset.csv` (features transformadas + alvo).
- `models/hla_best_model.pkl` (pipeline completo: preprocess + modelo vencedor).

6) **Recomendações para o Negócio**
- Impressas no terminal (ex.: reduzir cafeína após 15h; rotinas antiestresse; higiene do sono).

---

## Saídas geradas
- **Gráficos** (`outputs/figs/`):
  - `sleep_by_gender_box.png`
  - `age_vs_caffeine.png`
  - `heatmap_sleep_age_stress.png`
  - `sleep_by_caffeineQ_stress_*.png`
  - `confusion_LogisticRegression.png`
  - `confusion_RandomForest.png`
- **Dataset processado**: `outputs/hla_processed_dataset.csv`
- **Melhor modelo**: `models/hla_best_model.pkl`

---

## Como usar o modelo salvo
```python
import joblib
import pandas as pd

# carrega pipeline completo (pré-processamento + modelo)
pipe = joblib.load("models/hla_best_model.pkl")

# novos clientes (mesmas colunas brutas do treino)
df_new = pd.read_csv("data/novos_clientes.csv")

pred = pipe.predict(df_new)
print(pred[:10])
```

---

## Personalização
- **Nomes de colunas**: o código detecta sinônimos comuns (`Caffeine_mg`, `Sleep_Hours`, `Stress_Level`, etc.).  
  Se estiver muito diferente, ajuste os “guessers” em `main.py`.
- **Novas features**: acrescente no bloco “Features derivadas”.
- **Mais modelos**: adicione (XGBoost/LightGBM/SVC) no dicionário de modelos.
- **Métricas extras**: inclua F1 macro, ROC-AUC (one-vs-rest), calibração, cross-val e tuning.

---

## Boas práticas
- Não versione dados sensíveis (mantenha em `data/`, ignorado no Git).
- Documente o dicionário de dados (unidades/faixas).
- Para produção: use CV, tuning e monitore drift/qualidade.

---

## FAQ
**“Coluna alvo não encontrada.”**  
Confirme o nome da *Sleep_Quality* no CSV e ajuste o guesser no `main.py`.

**“Erro no OneHotEncoder (sparse_output).”**  
Versões antigas do scikit-learn usam `sparse`. O código já tenta ambos.

**“Gráficos vazios.”**  
Alguma coluna chave pode estar ausente. Veja no terminal quais colunas foram detectadas.

---

## Licença
Sugerido **MIT**. Ajuste conforme a política da sua organização.
