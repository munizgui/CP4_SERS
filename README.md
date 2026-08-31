## Análise de Dados de Carga Elétrica no Sistema Interligado Nacional (SIN) com API Pública do ONS

**Curso:** Ciência da Computação  
**Disciplina:** Soluções em Energias Renováveis e Sustentáveis  
**Projeto:** Desafio Final — Processamento, Modelagem e Análise de Séries Temporais de Energia  
**Fonte dos Dados:** Operador Nacional do Sistema Elétrico (ONS) — Portal de Dados Abertos  

---

##  1. Visão Geral do Projeto

Este repositório contém a solução completa do **Desafio Final** focado na extração, tratamento, modelagem e análise do comportamento da demanda de energia elétrica no estado de **São Paulo (SP)**. O projeto utiliza dados reais e atualizados consumidos diretamente da API pública RESTful do **Operador Nacional do Sistema Elétrico (ONS)**.

O objetivo principal é simular o fluxo de trabalho real de um **Engenheiro/Analista de Dados no setor elétrico**, executando o pipeline de dados fim a fim (*Data Ingestion*, *Data Cleaning*, *Feature Engineering*, *EDA* e *Data Visualization*), extraindo *insights* cruciais para o planejamento energético e a integração de fontes renováveis não despacháveis na rede elétrica.

---

##  2. Situação-Problema e Objetivos

Uma equipe de planejamento e operação energética necessita analisar detalhadamente o perfil de consumo e a composição da carga elétrica da região de **São Paulo (SP)** em uma janela temporal contínua de 7 dias (**01/08/2025 a 07/08/2025**).

Com amostragens semi-horárias (intervalos regulares de 30 minutos), o conjunto de dados engloba **336 registros temporais**, permitindo responder a questões fundamentais:
- **Padrão de Consumo:** Qual é o comportamento da curva de carga ao longo de um ciclo diário e semanal?
- **Variação Útil vs. Fim de Semana:** Como a atividade industrial e comercial impacta a demanda em relação aos dias de descanso?
- **Efeito da Geração Distribuída (MMGD):** Qual a penetração da geração solar fotovoltaica no meio da tarde e como ela reduz a carga observada pelo operador (efeito *Duck Curve* / Curva do Pato)?
- **Aderência e Qualidade:** Como validar a consistência e a integridade das medições recebidas via transmissão de dados?

---

##  3. Dicionário de Dados da API ONS

A integração com o endpoint `/prd/cargaverificada` do ONS retorna registros no formato JSON. Cada objeto possui os seguintes atributos estruturados:

| Campo / Parâmetro | Tipo | Descrição | Unidade |
| :--- | :--- | :--- | :--- |
| `cod_areacarga` | `String` | Código de identificação da área de carga (ex: `'SP'`) | N/A |
| `dat_referencia` | `String` | Data da medição no formato `YYYY-MM-DD` | Data |
| `din_referenciautc` | `String` | Timestamp UTC do ponto de medição (`ISO 8601`) | Data/Hora |
| `val_cargaglobal` | `Float` | Demanda total de energia da região (Carga Real) | MW mediar |
| `val_cargasupervisionada` | `Float` | Carga atendida pela rede de alta/extra alta tensão (Supervisionada pelo ONS) | MW mediar |
| `val_carganaosupervisionada` | `Float` | Estimativa da carga em baixa/média tensão não supervisionada direto | MW mediar |
| `val_cargammgd` | `Float` | Carga atendida por Micro e Mini Geração Distribuída (ex: painéis solares) | MW mediar |
| `val_consistencia` | `Integer` | Flag de consistência e auditoria do dado (`0` = Dado Validado) | Código |

---

##  4. Stack Tecnológico e Arquitetura

A solução foi construída utilizando a linguagem **Python 3** e bibliotecas consolidadas da ecologia de Data Science e Analytics:

- **`requests`**: Realização de requisições HTTP GET parametrizadas à API pública do ONS com tratamento de status e exceções.
- **`pandas`**: Manipulação do JSON, criação e estruturação de DataFrames, tratamento de fuso horário (`UTC` para `America/Sao_Paulo`), indexação temporal e cálculos estatísticos agregados.
- **`numpy`**: Operações vetoriais para cálculos de indicadores energéticos.
- **`matplotlib`**: Renderização de gráficos de curva de carga, gráficos comparativos de áreas sobrepostas e histogramas de frequência.
- **`seaborn`**: Aprimoramento visual e mapas de calor para análise temporal (opcional/complementar).

---

##  5. Roteiro Passo a Passo do Desafio (Exercícios)

O notebook do projeto está dividido nas seguintes etapas encadeadas:

### **Etapa 1: Consumo da API e Ingestão (Data Ingestion)**
* Configuração da URL base (`apicarga.ons.org.br`) e passagem de parâmetros (`dat_inicio`, `dat_fim`, `cod_areacarga`).
* Validação de respostas HTTP (Status 200, tratamento de *timeouts* e retentativas).

### **Etapa 2: Limpeza e Normalização (Data Cleaning & Wrangling)**
* Parsing do arquivo JSON bruto para `pandas.DataFrame`.
* Conversão da coluna `din_referenciautc` para o tipo `datetime` nativo.
* Ajuste de fuso horário UTC para o horário local (BRT - Brasília/São Paulo UTC-3).
* Verificação de valores nulos, duplicados e validação da coluna `val_consistencia`.

### **Etapa 3: Engenharia de Recursos (Feature Engineering)**
* Extração de colunas temporais: `Dia da Semana`, `Hora`, `Minuto`, `É_Fim_de_Semana` (booleano).
* Criação de colunas de relação: Cálculo do percentual de participação da MMGD em relação à Carga Global (`val_cargammgd / val_cargaglobal * 100`).

### **Etapa 4: Análise Exploratória (EDA) e Indicadores Chave (KPIs)**
* **Métricas Principais:** Cálculo da Carga Média, Carga de Pico (Demanda Máxima) e Carga Mínima (Vale) no período de 7 dias.
* **Fator de Carga (FC):** Relação entre a Carga Média e a Carga Máxima ($\text{FC} = \frac{\text{Carga Média}}{\text{Carga Máxima}}$).
* **Análise Comparativa:** Média de consumo em Dias Úteis vs. Fins de Semana.

### **Etapa 5: Visualização e Comunicação de Dados (Data Visualization)**
1. **Gráfico 1: Curva de Carga Contínua (MW vs. Tempo)** — Identificação da oscilação diária do consumo ao longo dos 7 dias.
2. **Gráfico 2: Perfil Diário Médio (Pico vs. Vale)** — Sobreposição do perfil médio de um dia útil comparado ao fim de semana.
3. **Gráfico 3: Impacto da Geração Distribuída (MMGD)** — Gráfico de área/linha demonstrando o "abate" da carga durante o período diurno devido ao avanço da energia solar.

---

##  6. Principais Insights e Descobertas

1. **Horários de Pico (Horário de Ponta):** A demanda máxima do estado ocorre costumeiramente entre **18:00 e 21:00**, impulsionada pela combinação do consumo residencial (iluminação, chuveiros elétricos) e transição do comércio.
2. **Efeito da MMGD (Curva do Pato):** Durante o período de pico de radiação solar (entre **10:00 e 15:00**), a Micro e Mini Geração Distribuída atende uma parcela significativa do consumo regional, reduzindo substancialmente a carga demandada da rede supervisionada do ONS.
3. **Redução nos Fins de Semana:** A carga global média aos sábados e domingos apresenta uma redução estimada de **15% a 25%** em comparação aos dias úteis, refletindo a desaceleração da produção industrial e comercial.

---

Integrantes:
Guilherme Lopes Muniz RM: 569521
Gustavo Russo Balizardo RM: 569283
Fernando Lembo RM: 570228
Ryan Barreto RM: 574126
