Análise de Dados PNAD-COVID-19 (IBGE)

## 📋 Descrição do Problema
Este projeto simula um cenário real onde fomos contratados como **Experts em Data Analytics** por um grande hospital corporativo. O objetivo principal é analisar o comportamento da população brasileira durante a pandemia da COVID-19 utilizando os microdados da **PNAD-COVID19 do IBGE**, identificando insights e indicadores críticos para o planejamento estratégico e operacional da instituição de saúde caso ocorra um novo surto epidemiológico.

A análise foca em três pilares principais recomendados pela liderança de dados:
* **Características clínicas dos sintomas**
* **Características demográficas da população**
* **Características econômicas da sociedade**

---

## 🛠️ Premissas do Projeto & Governança de Dados
Para garantir a viabilidade técnica e o foco no problema do hospital, o pipeline foi desenhado sob as seguintes regras de negócio estabelecidas:
1.  **Seleção Estratégica**: Utilização de variáveis mapeadas a partir de no máximo 20 questionamentos estruturais da pesquisa original do IBGE.
2.  **Delimitação Temporal**: Recorte temporal focado no comportamento consolidado da base.
3.  **Arquitetura em Nuvem**: Processamento distribuído utilizando serviços de Big Data na nuvem.

---

## 🏗️ Arquitetura de Dados (Medallion Architecture)
O projeto utiliza o conceito de **Lakehouse** estruturado na nuvem **AWS**, dividindo os dados em camadas de maturação e qualidade (Bronze, Silver e Gold) processadas via **AWS Glue** e persistidas no **Amazon S3**:

* **Camada Bronze:** Carga bruta dos dados originais do IBGE catalogados via AWS Athena/Glue Data Catalog. Nesta etapa, a base contava com **1.149.197 registros**.
* **Camada Silver:** Limpeza e Padronização. Identificação e remoção de **2.033 registros duplicados** com base nas chaves de identificação residencial (`UPA`, `V1008`, `Estrato`, etc.). Seleção das colunas de interesse e tradução dos códigos alfanuméricos originais do IBGE para nomes de colunas de negócio inteligíveis (ex: `B0011` vira `sintoma_febre`).
* **Camada Gold:** Camada final de consumo. Criação de variáveis agregadas (flags de grupos de risco, status consolidados de testagem, tipo de estabelecimento buscado) e tratamento final de nulidades (`Não aplicável`), otimizada em formato colunar Parquet para consumo analítico.

---

## 🔬 Mapeamento e Engenharia de Atributos (Camada Gold)
Para responder com precisão às necessidades hospitalares, criamos métricas agregadas que consolidam o comportamento do paciente:

| Variável Criada | Lógica de Negócio Aplicada (Engenharia de Atributos) | Objetivo Hospitalar |
| :--- | :--- | :--- |
| **`grupo_risco`** | Consolidação de comorbidades relatadas (Diabetes, Hipertensão, Asma, Doença Cardíaca, Depressão e Câncer). | Dimensionar o volume de pacientes críticos crônicos na região. |
| **`status_testagem`**| Cruzamento dos resultados de testes rápidos, RT-PCR (Swab) e exames de sangue para gerar um status único (*Positivo*, *Negativo*, *Aguardando*). | Avaliar a taxa de positividade e eficiência diagnóstica. |
| **`buscou_atendimento`** | Agrupamento de respostas sobre busca por postos de saúde, UPAs ou hospitais. | Medir o índice de evasão ou busca por socorro médico. |
| **`tipo_estabelecimento`**| Identificação se o paciente utilizou a rede de atendimento **SUS**, **Privada** ou **Ambas**. | Entender a pressão de demanda em cada subsetor de saúde. |
| **`cuidado_domiciliar`**| Identificação de pacientes com sintomas que optaram por isolamento domiciliar ou automedicação sem busca hospitalar imediata. | Monitorar a parcela de infectados subnotificados no ecossistema. |

---

## 🛠️ Tecnologias e Configuração do Cluster (AWS Glue)
O pipeline foi desenvolvido utilizando **PySpark** e otimizado para execução em larga escala dentro de sessões interativas do AWS Glue.

* **Kernel:** Glue PySpark (Python 3)
* **Glue Version:** 5.0 (Mais recente e otimizada para performance de execução)
* **Worker Type:** `G.1X`
* **Número de Workers:** 5 Workers em processamento distribuído parallelizado
* **Timeout Definido:** `%idle_timeout 2880`

---

## 📂 Estrutura do Repositório
* `techchallenge3.ipynb`: Notebook contendo o pipeline fim-a-fim de engenharia de dados (Extração da Bronze, Transformação na Silver, Enriquecimento e Agregação na Gold).
* `README.md`: Documentação atual do projeto.

---

## 🚀 Próximos Passos & Insights para o Hospital
Com os dados limpos, consolidados e estruturados em formato Parquet na **Camada Gold**, a base está pronta para alimentar ferramentas de Business Intelligence (como **Power BI** ou **Amazon QuickSight**) e modelos preditivos para responder perguntas como:
1.  **Dimensionamento de Leitos:** Qual a correlação histórica entre os sintomas de insuficiência respiratória (`sintoma_dificuldade_respirar`) e a necessidade real de intubação (`suporte_ventilatorio`) na rede privada vs. SUS?
2.  **Alocação de Recursos:** Mapear a distribuição de pacientes com convênio médico (`plano_saude`) que acionaram o hospital, otimizando estoques de insumos e testes rápidos.
3.  **Estratégia Preventiva:** Identificar o impacto do trabalho remoto (`trabalho_remoto`) e do nível de isolamento na contenção da curva de contágio de trabalhadores ativos.
