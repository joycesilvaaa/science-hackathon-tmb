# hackathon-tmb-fpd

> Modelo supervisionado para previsão de **First Payment Default** (FPD) no checkout de infoprodutos da TMB, com pipeline de leakage-control, validação temporal, tuning por Optuna e política de cobrança segmentada em 4 faixas de risco.

[![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat-square&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-EC4D26?style=flat-square)](https://xgboost.readthedocs.io/)
[![LightGBM](https://img.shields.io/badge/LightGBM-3C9F3C?style=flat-square)](https://lightgbm.readthedocs.io/)
[![Optuna](https://img.shields.io/badge/Optuna-1A237E?style=flat-square)](https://optuna.org/)
[![SHAP](https://img.shields.io/badge/SHAP-FF6F00?style=flat-square)](https://shap.readthedocs.io/)

---

## Sobre o projeto

Este projeto foi desenvolvido durante o **4º Science & Business Connection — Science Hackathon TMB × PIT (maio/2026)**. O desafio: construir um modelo de machine learning capaz de prever, **no momento do checkout**, quais operações têm maior probabilidade de se tornar FPD — *First Payment Default*, a inadimplência já no primeiro vencimento.

O pipeline simula um problema real de originação de crédito: base desbalanceada (~14% de positivos), informações com missing, mudanças temporais no perfil da carteira e risco de leakage por colunas pós-evento. A entrega cobre desde a limpeza dos dados até a tradução do score em **política de cobrança operacional segmentada por faixa de risco**.

---

## Resultados

| Métrica | Resultado |
|---|---|
| Registros analisados | 99.999 (treino + teste temporal) |
| Taxa global de FPD | 13,86% |
| Janela de treino | até 25/01/2024 (79.999 registros) |
| Janela de teste | 25/01/2024 → 31/07/2025 (20.000 registros) |
| Modelo campeão | **XGBoost + `class_weight='balanced'`** (pós-tuning Optuna) |
| ROC-AUC (teste) | **0,7991** |
| KS | **0,4520** |
| PR-AUC | 0,3247 |
| Top 10% Recall | **31,58%** (lift de 3,16x) |
| Concentração de FPDs em Alto + Crítico | ~80% do total |
| Feature mais preditiva | `SCORE_HSV5` (bureau — varejo/serviços) |
| Faixas da política de cobrança | 4 (Baixo / Médio / Alto / Crítico — cortes em p25/p50/p75) |
| Submissão final | 23.354 operações pontuadas |

---


## Pipeline em 18 etapas

| Etapa | Descrição |
|---|---|
| 1 | Imports e configuração |
| 2 | Carregamento dos dados (`basetreinamento.xlsx`) |
| 3 | **Remoção de leakage** — 25 colunas pós-evento dropadas |
| 4 | Mapeamento do target FPD (Sim/Não → 1/0) |
| 5 | Feature engineering — idade, período do dia, `score_por_valor`, `produtor_freq`, `lancamento_freq` |
| 6 | **Split temporal** (80/20 por `data_efetivacao`) |
| 7 | Funções de avaliação — KS, Top-Q Recall, ROC-AUC, PR-AUC |
| 8 | Estratégias de balanceamento — `class_weight`, SMOTE, UnderSample |
| 9 | Configuração dos modelos — LogReg, XGBoost, LightGBM |
| 10 | Grid de 9 experimentos (3 modelos × 3 estratégias) |
| 11 | Visualização comparativa em heatmaps |
| 12 | Top-3 combinações por score combinado normalizado |
| 13 | **Tuning com Optuna** — `TimeSeriesSplit` em 3 folds, busca bayesiana |
| 14 | Comparativo antes × depois do tuning |
| 15 | Análise detalhada do modelo campeão (tabela de decis, lift) |
| 16 | **Definição das faixas de risco** (cortes p25/p50/p75) |
| 17 | Geração da submissão final |
| 18 | **Interpretabilidade com SHAP** — feature importance e dependence plots |

---

## Tecnologias utilizadas

| Ferramenta | Uso |
|---|---|
| `pandas` / `numpy` | Limpeza, manipulação e feature engineering |
| `scikit-learn` | Imputação, `StandardScaler`, `LogisticRegression`, `TimeSeriesSplit`, métricas |
| `xgboost` | Modelo campeão pós-tuning |
| `lightgbm` | Modelo concorrente (vice-campeão) |
| `imbalanced-learn` | SMOTE e RandomUnderSampler para classes desbalanceadas |
| `optuna` | Busca bayesiana de hiperparâmetros com validação temporal |
| `shap` | Interpretabilidade global e local do modelo |
| `matplotlib` / `seaborn` | Visualizações comparativas e heatmaps |
| `openpyxl` | Leitura de arquivos `.xlsx` |
| `jupyter` | Ambiente de análise |

---

## Como executar

### 1. Clone o repositório
```bash
git clone https://github.com/joycesilvaaa/science-hackathon-tmb.git
cd science-hackathon-tmb
```

### 2. Crie o ambiente e instale as dependências
```bash
uv venv .venv
uv activate .venv
uv install -r requirements.txt
```

Ou direto via pip:
```bash
pip install pandas numpy scikit-learn xgboost lightgbm imbalanced-learn optuna shap matplotlib seaborn scipy openpyxl jupyter
```

## Não se esqueça de:
 - Verificar o caminho dos arquivos! 

### 3. Posicione os arquivos de dados
Coloque na raiz do projeto os arquivos fornecidos pelo desafio:
- `basetreinamento.xlsx` — base histórica rotulada
- `submissao.xlsx` — base de teste para predição

> ⚠️ Os dados **não estão versionados** neste repositório por se tratarem de informações operacionais da TMB. São fornecidos exclusivamente aos participantes do hackathon.

### 4. Execute o notebook
```bash
jupyter notebook tmb.ipynb
```

> Execute as células em ordem (1 → 38). O pipeline completo leva ~10–15 minutos em uma máquina padrão; a etapa de tuning com Optuna concentra a maior parte do tempo.

### 5. Saída esperada
Ao final da execução, será gerado:
- `submissao_final.csv` com três colunas: `pedido_id`, `prob_fpd`, `faixa_risco`
- Distribuição esperada: ~1,7% Baixo · 25,3% Médio · 53,6% Alto · 19,4% Crítico

---

## Política de cobrança proposta

A política traduz o score em ações operacionais segmentadas em 4 faixas, com condições comerciais diferenciadas (entrada, prazo, juros) e canais de cobrança escalados por risco. Documento completo em `politica_cobranca.docx`.

### Faixas de risco

| Faixa | `prob_fpd` | Taxa FPD observada | Lift | Volume na base |
|---|---|---|---|---|
| **Baixo** | < 0,1266 | 0,72% | 0,06x | ~25% (4.999 pedidos) |
| **Médio** | 0,1266 – 0,3472 | 4,94% | 0,41x | ~25% (5.001 pedidos) |
| **Alto** | 0,3472 – 0,5818 | 13,33% | 1,11x | ~25% (4.998 pedidos) |
| **Crítico** | ≥ 0,5818 | 28,95% | 2,42x | ~25% (5.002 pedidos) |

> Cortes derivados dos percentis 25/50/75 da distribuição de scores no conjunto de teste — escolhidos para garantir **volume operacional previsível** (~25% da base por faixa) e **lift monotônico crescente** (0,06 → 0,41 → 1,11 → 2,42). As faixas Alto + Crítico concentram ~80% de todos os FPDs.

### Condições comerciais por faixa

| Parâmetro | Baixo | Médio | Alto | Crítico |
|---|---|---|---|---|
| **Entrada mínima** | 0% | 10% | 20% | 40% |
| **Prazo máximo** | 48x | 36x | 24x | 12x |
| **Taxa de juros** | Base | Base + 2–4 p.p. | Base + 5–8 p.p. | Base + 10–15 p.p. |
| **Documentação extra** | — | — | Comprovante de renda | Garantia / avalista |
| **Decisão** | Aprovação automática | Aprovação com lembretes | Aprovação com contato ativo | Análise manual obrigatória |

### Canal e timing de cobrança

| Faixa | Canal | Frequência | Critério de escalada |
|---|---|---|---|
| **Baixo** | E-mail / SMS automatizado | Apenas no vencimento | — |
| **Médio** | WhatsApp / SMS automatizado | 3 dias antes + no vencimento | Não pagamento da 1ª parcela |
| **Alto** | Telefone (URA ou atendente) | Semanal até quitação da 1ª parcela | Contato não atendido ou 1º atraso |
| **Crítico** | Gerente sênior (ligação + e-mail) | Diária até definição | Imediata para supervisor |

### Matriz de escalada pós-vencimento

| Situação | Ação | Prazo | Responsável |
|---|---|---|---|
| 1º atraso (qualquer faixa) | Ligação | D+1 do vencimento | Cobrança |
| 2º atraso | Negociação | D+3 do vencimento | Supervisor |
| 3º atraso | Encaminhar jurídico | D+15 do vencimento | Jurídico |

---

## Decisões técnicas relevantes

- **Anti-leakage rigoroso:** 25 colunas pós-evento foram removidas explicitamente (`status_cobranca`, `dias_em_atraso`, `pdd`, `saldo_vencido`, entre outras). O modelo usa **apenas informação disponível no momento do checkout**.
- **Split temporal, não aleatório:** treino com vendas anteriores a 25/01/2024, teste com vendas posteriores — simula uso real em produção e mede generalização sob drift temporal.
- **`TimeSeriesSplit` no tuning:** Optuna otimiza ROC-AUC em cross-validation temporal (3 folds), não em folds aleatórios, mantendo a ordem cronológica.
- **`SimpleImputer(strategy='median')`:** missing em scores de bureau é tratado por imputação pela mediana do treino — não por exclusão de registros.
- **`class_weight='balanced'` venceu SMOTE:** apesar de SMOTE ser popular para desbalanceamento, neste problema `class_weight` entregou melhor ROC-AUC e KS em todos os modelos testados.
- **Política de cobrança derivada do score, não threshold binário:** alinhada com a regra da TMB de **não rejeitar nenhum cliente** — o score modula a *intensidade da cobrança*, não a *aprovação da venda*.

---

## Fonte dos dados

- **TMB Fintech** — dados operacionais reais de checkout, fornecidos sob acordo de confidencialidade aos participantes do hackathon.
- **Período:** vendas anteriores a 31/07/2025.
- **Volume:** 99.999 registros rotulados (treino) + 23.354 registros para predição (submissão).

---

## Equipe

Desenvolvido durante o **Science Hackathon TMB × PIT** (18–20 de maio de 2026) pela equipe:

- **Pedro Lucas Almeida dos Santos** — Ciência da Computação
- **Joyce** — Desenvolvimento de Software
- **João Pedro** — Análise e Desenvolvimento de Sistemas 
- **Davi Ferreira** — Análise e Desenvolvimento de Sistemas
