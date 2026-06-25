# Complexidade Técnico-Linguística de Termos Médicos do SUS

**Ciência de Dados — IFPB/PPGTI — 2026.1**  
Docentes: Damires Souza e Alex Cunha  
Responsável: Lucas Matheus Santos da Silva

---

## Sobre o projeto

Este projeto constrói um **dataset rotulado** que classifica diagnósticos (CID-10) e procedimentos (SIGTAP) do SUS em três níveis de **complexidade técnico-linguística**: baixa, média e alta.

O objetivo é identificar quais termos médicos são mais difíceis de compreender para pacientes e usuários do SUS, contribuindo com iniciativas de simplificação da linguagem em saúde.

> Todos os dados utilizados são **públicos e secundários** (DataSUS). Nenhum dado de paciente foi utilizado.

---

## Estrutura do repositório

```
├── notebooks/
│   └── colab_passo_a_passo.ipynb   # Pipeline completo: coleta → modelagem
│
├── dados/
│   ├── CID10/
│   │   └── tb_cid.txt              # Tabela de diagnósticos (CID-10 V2008, DataSUS)
│   └── SIGTAP/
│       └── tb_procedimento.txt     # Tabela de procedimentos (SIGTAP 05/2026)
│
├── docs/
│   └── Planejamento (...).pdf      # Planejamento do projeto
│
└── README.md
```

---

## Pipeline

| # | Etapa | O que faz |
|---|---|---|
| 1 | **Coleta** | Leitura das fontes oficiais (CID-10, SIGTAP) |
| 2 | **Integração** | União das fontes numa só tabela (17.182 registros) |
| 3 | **Limpeza** | Tratamento de valores ausentes e duplicidades |
| 4 | **Atributos** | Extração de 5 características textuais |
| 5 | **Rotulagem** | Classificação heurística em baixa / média / alta |
| 6 | **Análise** | Exploração, visualização e detecção de outliers |
| 7 | **Modelagem** | Árvore de decisão (acurácia 92%, CV 5-fold: 87%) |

---

## Fontes dos dados

- **CID-10 (V2008):** tabela oficial de diagnósticos vigente no SUS — [DataSUS](https://datasus.saude.gov.br)
- **SIGTAP (05/2026):** tabela de procedimentos do SUS — [SIGTAP](http://sigtap.datasus.gov.br)

---

## Como executar

1. Abra `notebooks/colab_passo_a_passo.ipynb` no [Google Colab](https://colab.research.google.com)
2. Faça upload dos arquivos `dados/CID10/tb_cid.txt` e `dados/SIGTAP/tb_procedimento.txt`
3. Execute as células em ordem

---

## Resultados

- **17.182 registros** rotulados (12.198 diagnósticos + 4.984 procedimentos)
- Procedimentos SIGTAP apresentam complexidade linguística significativamente maior que diagnósticos CID-10
- Os atributos mais relevantes para a classificação foram `qtd_termos_tecnicos` e `qtd_palavras`
