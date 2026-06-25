# Complexidade Técnico-Linguística de Termos Médicos do SUS

**Disciplina:** Ciência de Dados — IFPB/PPGTI — 2026.1  
**Docentes:** Damires Souza e Alex Cunha  
**Equipe:** Lucas Matheus Santos da Silva · Pedro Ivo

---

## Sobre o projeto

Construção de um dataset rotulado que classifica diagnósticos (CID-10) e procedimentos (SIGTAP) do SUS em três níveis de **complexidade técnico-linguística**: baixa, média ou alta.

O projeto segue a metodologia **CRISP-DM** e está relacionado ao tema de mestrado [Traduz Saúde](https://github.com/L42Matheus), que visa simplificar a linguagem médica para pacientes — mas mantém escopo independente para a disciplina de Ciência de Dados.

---

## Fontes de dados

| Fonte | Descrição | Link |
|---|---|---|
| SIGTAP | Procedimentos do SUS (competência **05/2026**) | [sigtap.datasus.gov.br](https://sigtap.datasus.gov.br/) |
| CID-10 | Diagnósticos — versão V2008 (vigente no SUS até 2027) | [tabnet.datasus.gov.br](https://tabnet.datasus.gov.br/) |
| DeCS/MeSH | Vocabulário controlado de termos médicos | [decs.bvsalud.org](https://decs.bvsalud.org/) |

> Apenas dados **secundários e públicos**. Nenhum dado de paciente.

---

## Pipeline (CRISP-DM)

```
1. Coleta        → download dos arquivos oficiais (CID-10 e SIGTAP)
2. Integração    → concat das duas fontes numa só tabela
3. Limpeza       → remoção de nulos e duplicatas
4. Atributos     → qtd_palavras, qtd_caracteres, qtd_termos_tecnicos, tem_sigla, tem_abreviacao
5. Rotulagem     → score ponderado + tercis → baixa / média / alta
6. Análise EDA   → distribuição por fonte, histograma do score, outliers (boxplot)
7. Modelagem     → árvore de decisão (max_depth=5) + validação cruzada 5-fold
```

---

## Estrutura do repositório

```
.
├── colab_passo_a_passo.ipynb   # notebook principal com todo o pipeline
├── dataset_final_rotulado.csv  # dataset gerado (saída do notebook)
└── README.md
```

---

## Como reproduzir

1. Abra o notebook no Google Colab
2. Faça upload dos arquivos de dados:
   - `tb_procedimento.txt` — extraído do ZIP do SIGTAP competência 05/2026
   - `tb_cid.txt` — disponível no mesmo pacote SIGTAP
3. Execute as células em ordem

> **Atenção:** o SIGTAP é atualizado mensalmente. Para reproduzir exatamente os resultados deste trabalho, use a competência **05/2026**.

---

## Dataset gerado

| Campo | Tipo | Exemplo |
|---|---|---|
| `codigo` | texto | `J15.9` / `0301010013` |
| `descricao` | texto | `Pneumonia bacteriana não especificada` |
| `fonte` | categoria | `CID-10` / `SIGTAP` |
| `qtd_palavras` | inteiro | `4` |
| `qtd_caracteres` | inteiro | `38` |
| `qtd_termos_tecnicos` | inteiro | `2` |
| `tem_sigla` | booleano | `0` / `1` |
| `tem_abreviacao` | booleano | `0` / `1` |
| `score` | float | `0.412` |
| `complexidade` | categoria | `baixa` / `media` / `alta` |

---

## Tecnologias

- Python 3 · Google Colab
- `pandas` · `numpy` · `scikit-learn` · `matplotlib` · `seaborn`

---

## Limitações assumidas

- A variável `complexidade` é uma **proxy heurística** de legibilidade, não validada com pacientes reais.
- A árvore de decisão tem papel **interpretativo** (reaprende a heurística), não preditivo.
- A versão da CID-10 em português (V2008) está congelada desde 2008, mas continua vigente no SUS (Nota Técnica MS 91/2024).
