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
| SIGTAP | Procedimentos do SUS (competência **05/2026**) — 4.984 registros | [sigtap.datasus.gov.br](https://sigtap.datasus.gov.br/) |
| CID-10 | Diagnósticos — versão V2008 (vigente no SUS até 2027) — 12.198 registros | [tabnet.datasus.gov.br](https://tabnet.datasus.gov.br/) |
| DeCS | Vocabulário controlado de termos médicos em português — **96.498 termos** | [decs.bvsalud.org](https://decs.bvsalud.org/) |

> Apenas dados **secundários e públicos**. Nenhum dado de paciente.

> **Sobre o DeCS.** O DeCS (Descritores em Ciências da Saúde — BIREME/OPAS/OMS) é o vocabulário controlado oficial da área de saúde em português. Foi solicitado formalmente o **XML completo** à BIREME/BVS, pois a consulta via API termo a termo seria inviável para o volume do projeto (17 mil descrições × 96 mil termos). Com o XML em mãos, a identificação de jargão médico é feita por **correspondência de *longest-match*** sobre n-gramas normalizados — ancorando a definição de "termo técnico" no padrão oficial da área, em vez de uma heurística textual.

---

## Pipeline (CRISP-DM)

```
1. Coleta        → leitura dos arquivos oficiais (CID-10, SIGTAP, DeCS) via raw GitHub
2. Integração    → concat das duas fontes em uma só tabela
3. Limpeza       → nulos + duplicidades semânticas (descrição repetida em códigos distintos)
4. Atributos     → qtd_palavras, qtd_caracteres, qtd_palavras_longas,
                   qtd_sinais_morfologicos, tem_sigla, tem_abreviacao
5. DeCS          → qtd_termos_decs via longest-match (n-gramas normalizados)
6. Classificação → score = 0,50·T_DeCS + 0,15·M + 0,20·P + 0,10·S + 0,05·A
                   tercis → baixa / média / alta (heurística)
7. Análise EDA   → distribuição por fonte, histograma do score, outliers (boxplot)
8. Modelagem     → árvore de decisão (max_depth=5) + validação cruzada 5-fold
```

---

## Estrutura do repositório

```
.
├── colab_passo_a_passo.ipynb   # notebook principal com todo o pipeline
├── dataset_final.csv           # ⭐ dataset exploratório gerado (17.161 registros, 12 colunas)
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

Os arquivos `tb_cid.txt`, `tb_procedimento.txt` e `decs_termos_pt.txt` ficam versionados em `dados/`. O notebook lê direto do raw do GitHub — **não é preciso fazer upload manual**.

> **Atenção:** o SIGTAP é atualizado mensalmente. Para reproduzir exatamente os resultados deste trabalho, use a competência **05/2026** (a versão fixada no repositório).

---

## Dataset gerado

**17.161 registros × 12 colunas:**

| Campo | Tipo | Descrição | Exemplo |
|---|---|---|---|
| `codigo` | texto | código oficial do registro | `J159` / `0301010013` |
| `descricao` | texto | descrição oficial (texto analisado) | `Pneumonia bacteriana não especificada` |
| `fonte` | categoria | origem do registro | `CID-10` / `SIGTAP` |
| `qtd_palavras` | inteiro | palavras na descrição | `4` |
| `qtd_caracteres` | inteiro | caracteres na descrição | `38` |
| `qtd_palavras_longas` | inteiro | palavras com ≥12 caracteres | `1` |
| `qtd_sinais_morfologicos` | inteiro | morfemas greco-latinos (Rezende, 2004) | `1` |
| `qtd_termos_decs` | inteiro | expressões DeCS encontradas (longest-match) | `2` |
| `tem_sigla` | booleano | tem sigla conhecida do SUS | `0` / `1` |
| `tem_abreviacao` | booleano | tem abreviação | `0` / `1` |
| `score` | float | nota ponderada normalizada [0, 1] | `0.412` |
| `complexidade` | categoria | faixa por tercis | `baixa` / `media` / `alta` |

---

## Tecnologias

- Python 3 · Google Colab
- `pandas` · `numpy` · `scikit-learn` · `matplotlib` · `seaborn`

---

## Natureza do dataset e limitações assumidas

- **Dataset exploratório, não rotulado.** A categoria `complexidade` é derivada automaticamente de atributos do texto, não anotada por especialistas. Trata-se de uma **proxy heurística** de legibilidade, não de uma rotulagem validada.
- **Rotulagem real exigiria especialistas.** Uma classificação confiável de complexidade médica demandaria um painel multidisciplinar (médicos, linguistas, pacientes) e tempo de validação incompatíveis com o escopo da disciplina. Fica como **trabalho futuro**.
- **A árvore de decisão tem papel interpretativo.** Como o `score` deriva dos próprios atributos, o modelo reaprende a heurística — a alta acurácia mostra consistência interna, não capacidade preditiva sobre rótulos humanos.
- **Presença no DeCS ≠ dificuldade garantida.** O DeCS lista terminologia oficial de saúde, mas inclui termos de uso corrente (ex.: "febre"). Por isso o atributo `qtd_termos_decs` é interpretado como **indicador de jargão**, não como medida direta de incompreensibilidade.
- **CID-10 V2008.** A versão em português está congelada desde 2008, mas continua vigente no SUS (Nota Técnica MS 91/2024).

---

## Resultados principais

- **Cobertura DeCS:** 79,0% das descrições têm pelo menos um termo do vocabulário oficial
- **Distribuição da complexidade por fonte:** CID-10 concentra na média (34,3%); SIGTAP distribui mais para os extremos (34,6% alta)
- **Atributo dominante na árvore:** `qtd_termos_decs` (importância 0,521) — confirma empiricamente o peso atribuído a priori ao DeCS no score
