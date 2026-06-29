# Complexidade Técnico-Linguística de Termos Médicos do SUS

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/L42Matheus/mpti_cd_final-projeto/blob/main/colab_passo_a_passo.ipynb)

**Disciplina:** Ciência de Dados — IFPB/PPGTI — 2026.1  
**Docentes:** Damires Souza e Alex Cunha  
**Equipe:** Lucas Matheus Santos da Silva · Pedro Ivo

📄 **Artigo (Overleaf):** [editar projeto](https://www.overleaf.com/project/6a32dec9264398e4b5715d92)

---

## Sobre o projeto

Construção de um **dataset exploratório** sobre diagnósticos (CID-10) e procedimentos (SIGTAP) do SUS, com uma **classificação heurística** de **complexidade técnico-linguística** em três níveis (baixa, média ou alta) derivada de atributos do próprio texto.

> **Importante:** este **não é um dataset rotulado** no sentido estrito — não houve anotação por especialistas em saúde ou linguística. A categoria de complexidade é uma **proxy heurística** calculada a partir de características mensuráveis das descrições (tamanho, jargão, siglas etc.). Uma rotulagem real exigiria um painel de especialistas e tempo de validação que vão além do escopo desta disciplina.

O projeto segue a metodologia **CRISP-DM** e está relacionado ao tema de mestrado [Traduz Saúde](https://github.com/L42Matheus), que visa simplificar a linguagem médica para pacientes — mas mantém escopo independente para a disciplina de Ciência de Dados.

---

## Fontes de dados

| Fonte | Descrição | Link |
|---|---|---|
| SIGTAP | Procedimentos do SUS (competência **05/2026**) | [sigtap.datasus.gov.br](https://sigtap.datasus.gov.br/) |
| CID-10 | Diagnósticos — versão V2008 (vigente no SUS até 2027) | [tabnet.datasus.gov.br](https://tabnet.datasus.gov.br/) |
| DeCS/MeSH | Vocabulário controlado de termos médicos *(em espera — ver nota)* | [decs.bvsalud.org](https://decs.bvsalud.org/) |

> Apenas dados **secundários e públicos**. Nenhum dado de paciente.

> **Nota sobre o DeCS.** O planejamento prevê o uso do DeCS/MeSH como vocabulário controlado para identificação de termos técnicos. Foi feita **solicitação formal do XML completo** à BIREME/BVS para evitar consultas termo a termo (inviável dado o volume). Como a liberação ainda não saiu até o fechamento desta entrega, a identificação de jargão médico é feita, **provisoriamente**, por análise morfológica greco-latina (sufixos como *-ite*, *-ectomia*, *-scopia*; radicais como *cardio-*, *gastro-*, *nefro-*), com base em Rezende (2004). Quando o XML do DeCS for liberado, esse atributo será refinado.

---

## Pipeline (CRISP-DM)

```
1. Coleta        → download dos arquivos oficiais (CID-10 e SIGTAP)
2. Integração    → concat das duas fontes numa só tabela
3. Limpeza       → nulos + duplicidades semânticas (descrição repetida em códigos distintos)
4. Atributos     → qtd_palavras, qtd_caracteres, qtd_termos_tecnicos, tem_sigla, tem_abreviacao
5. Classificação → score ponderado + tercis → baixa / média / alta (heurística)
6. Análise EDA   → distribuição por fonte, histograma do score, outliers (boxplot)
7. Modelagem     → árvore de decisão (max_depth=5) + validação cruzada 5-fold
```

---

## Estrutura do repositório

```
.
├── colab_passo_a_passo.ipynb   # notebook principal com todo o pipeline
├── dados/
│   ├── CID10/tb_cid.txt        # tabela de diagnósticos (CID-10 V2008)
│   ├── SIGTAP/tb_procedimento.txt   # tabela de procedimentos (SIGTAP 05/2026)
│   └── DeCS/decs_termos_pt.txt # vocabulário DeCS em português (BIREME/BVS)
├── docs/                       # planejamento do projeto
└── README.md
```

---

## Como reproduzir

1. Clique no badge **Open in Colab** no topo deste README
2. Execute as células em ordem

Os arquivos `tb_cid.txt` e `tb_procedimento.txt` ficam versionados em `dados/CID10/` e `dados/SIGTAP/`. O notebook lê direto do raw do GitHub — **não é preciso fazer upload manual**.

> **Atenção:** o SIGTAP é atualizado mensalmente. Para reproduzir exatamente os resultados deste trabalho, use a competência **05/2026** (a versão fixada no repositório).

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

## Natureza do dataset e limitações assumidas

- **Dataset exploratório, não rotulado.** A categoria `complexidade` é derivada automaticamente de atributos do texto, não anotada por especialistas. Trata-se de uma **proxy heurística** de legibilidade, não de uma rotulagem validada.
- **Rotulagem real exigiria especialistas.** Uma classificação confiável de complexidade médica demandaria um painel multidisciplinar (médicos, linguistas, pacientes) e tempo de validação incompatíveis com o escopo da disciplina. Fica como **trabalho futuro**.
- **A árvore de decisão tem papel interpretativo.** Como o `score` deriva dos próprios atributos, o modelo reaprende a heurística — a alta acurácia mostra consistência interna, não capacidade preditiva sobre rótulos humanos.
- **CID-10 V2008.** A versão em português está congelada desde 2008, mas continua vigente no SUS (Nota Técnica MS 91/2024).
