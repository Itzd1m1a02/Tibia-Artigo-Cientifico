# 🦵 COTB — Termografia da Tíbia: Segmentação, Análise Longitudinal e Classificação de Patologias Ósseas

![MATLAB](https://img.shields.io/badge/MATLAB-R2020b%2B-orange.svg)
![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-brightgreen.svg)

Este repositório reúne os algoritmos em MATLAB, as rotinas de processamento de imagens e as análises estatísticas do projeto **COTB — Termografia da Tíbia**. O sistema processa matrizes radiométricas infravermelhas (`.csv`) para segmentar a região das pernas, extrair o perfil térmico longitudinal da tíbia e realizar a classificação supervisionada de quatro grupos clínicos.

O pipeline foi desenvolvido para lidar com desafios comuns nos exames termográficos ortopédicos, como artefatos térmicos associados a hastes metálicas, pernas que aparecem encostadas, diferenças de orientação postural e desbalanceamento entre as classes.

---

## 📌 Visão geral do projeto

A proposta é implementar um **pipeline automatizado de visão computacional e aprendizado de máquina** que transforme matrizes térmicas brutas em descritores anatômicos de temperatura. O sistema considera as seguintes classes:

1. **Saudável (hígido)**
2. **Osteomielite**
3. **Pseudoartrose**
4. **Fratura óssea**

A unidade de análise final é o **paciente/exame**, e não cada perna isoladamente. Quando as duas pernas são segmentadas com sucesso, seus perfis térmicos são combinados por média ponto a ponto, reduzindo o risco de atribuir à perna saudável contralateral o rótulo da patologia do paciente.

---

## ⚙️ Arquitetura metodológica do pipeline

O processamento é executado em lote (*batch processing*) e está organizado nas etapas a seguir.

### 1. Ingestão, limpeza e filtragem clínica

- Varredura recursiva das pastas em busca de arquivos `.csv`, utilizando `imageDatastore` e `readmatrix`.
- Remoção de linhas e colunas compostas integralmente por `NaN`.
- Exclusão automática de arquivos cujos caminhos contenham os termos `fixador`, `cicatrizada` ou `cicatrizado`, conforme os critérios definidos no protocolo experimental.

### 2. Normalização radiométrica e segmentação das pernas

- Recorte estatístico dos valores térmicos entre os percentis **P5 e P99**, reduzindo a influência de pixels extremos e reflexos.
- Aplicação do limiar de Otsu (`graythresh`) para separar o corpo do fundo.
- Refinamento da máscara com fechamento morfológico (`imclose`, elemento disco de raio 3), preenchimento de regiões internas (`imfill`) e remoção de pequenos componentes (`bwareaopen`).
- Quando as pernas estão encostadas e formam um único objeto, o algoritmo procura o vale de menor densidade em uma projeção central e aplica um corte de 5 pixels para separá-las (*valley splitting*).

### 3. Alinhamento postural e recorte anatômico da tíbia

- A orientação é estimada pelas dimensões da caixa delimitadora (*BoundingBox*) de cada perna.
- Quando a caixa é mais alta do que larga, a matriz e sua máscara são rotacionadas em **−90°**, padronizando o eixo longitudinal na horizontal.
- A região de interesse (ROI) da tíbia corresponde ao intervalo entre **20% e 75%** do comprimento do segmento: são descartados os 20% iniciais, próximos ao joelho, e os 25% finais, próximos ao tornozelo e ao pé.

### 4. Extração do perfil térmico longitudinal

- Dentro da ROI, a máscara é refinada com limiarização local de Otsu, fechamento morfológico com disco de raio 4 e seleção do maior componente conectado.
- Para cada coluna com pelo menos três pixels válidos, a posição do eixo central é estimada pela mediana das coordenadas dos pixels de pele e suavizada por média móvel (`smoothdata`, janela 7).
- A temperatura representativa de cada seção transversal é o **percentil 75 (P75)** dos pixels válidos daquela coluna. Essa escolha reduz a influência de hastes ou pinos metálicos frios que cruzem a região analisada.

### 5. Interpolação, fusão bilateral e normalização espacial

- Cada perfil individual é reamostrado para **100 pontos**, de 0% a 100% do comprimento da tíbia, por interpolação cúbica de Hermite por partes (**PCHIP**) com `interp1(..., 'pchip', 'extrap')`.
- Quando as duas pernas do exame são válidas, os perfis são combinados pela média ponto a ponto, produzindo um único vetor de 100 posições por paciente.

### 6. Balanceamento e classificação KNN

- O sistema permite definir interativamente a quantidade de exames saudáveis a selecionar.
- A seleção é feita sem reposição (`randperm`) e com semente fixa (`rng(42)`), para facilitar a reprodução do experimento.
- A curva de 100 pontos é resumida em três descritores térmicos, calculados pela média de cada terço anatômico:

| Descritor | Intervalo do perfil | Cálculo |
|---|---:|---|
| Proximal ($T_{\text{prox}}$) | 0%–30% | Média dos pontos 1–30 |
| Medial ($T_{\text{med}}$) | 30%–70% | Média dos pontos 31–70 |
| Distal ($T_{\text{dist}}$) | 70%–100% | Média dos pontos 71–100 |

- As amostras são divididas de forma estratificada em **70% para treino e 30% para teste**, por meio de `cvpartition` com `HoldOut = 0.30`.
- O classificador utiliza **K-vizinhos mais próximos (KNN), com `k = 5`**, distância Euclidiana e padronização Z-score (`Standardize = true`).

---

## 🧪 Tecnologias e requisitos

O projeto utiliza recursos nativos do ecossistema MATLAB, sem dependência de bibliotecas externas descritas no relatório técnico.

- **MATLAB:** R2020b ou superior é a versão recomendada no README do projeto.
- **Image Processing Toolbox:** processamento e segmentação de imagens.
- **Statistics and Machine Learning Toolbox:** particionamento dos dados, treinamento do KNN e avaliação estatística.

### Principais funções MATLAB utilizadas

| Módulo | Funções e recursos | Aplicação |
|---|---|---|
| MATLAB Base | `imageDatastore`, `readmatrix`, `interp1`, `smoothdata`, `prctile`, `randperm`, `rng` | Leitura dos dados, interpolação, estatísticas por percentil e seleção de amostras. |
| Image Processing Toolbox | `graythresh`, `imclose`, `imfill`, `bwareaopen`, `bwconncomp`, `regionprops`, `imcrop`, `imrotate`, `strel` | Segmentação, operações morfológicas, análise de componentes, recorte e alinhamento. |
| Statistics and Machine Learning Toolbox | `fitcknn`, `cvpartition`, `predict`, `confusionmat`, `confusionchart`, `boxplot`, `gscatter` | Treinamento e avaliação do classificador e análise das distribuições. |
| MATLAB Graphics | `imagesc`, `exportgraphics`, `subplot`, `sgtitle`, `fill`, `xline` | Geração e exportação dos mapas, curvas, gráficos e figuras do relatório. |

---

## 📂 Estrutura do repositório

A estrutura sugerida para organizar os dados e os arquivos do projeto é:

```text
.
├── data/                              # Matrizes termográficas (.csv)
│   ├── diagnosticos/
│   │   ├── fraturas_osseas/
│   │   ├── osteomielite_tibia/
│   │   └── pseudoartrose/
│   └── saudaveis/
│       ├── tibia_deitado/
│       └── tibia_em_pe/
├── matlab/
│   ├── pipeline_termografia.m         # Script principal
│   ├── document.xml                   # Arquivo auxiliar do projeto, se aplicável
│   └── output.xml                     # Arquivo auxiliar do projeto, se aplicável
├── figuras/                           # Figuras e gráficos exportados
├── README.md                          # Documentação do repositório
└── LICENSE                            # Licença do projeto
```

> **Observação:** o pipeline faz a busca recursiva a partir do diretório indicado em `pasta_principal`. Mantenha a organização das pastas e os nomes utilizados pelo seu conjunto de dados compatíveis com as regras de identificação das classes e com os critérios de exclusão do script.

---

## 🚀 Como executar

1. Clone o repositório:

   ```bash
   git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
   ```

2. Abra o MATLAB e defina a pasta do projeto como diretório de trabalho.

3. No script `pipeline_termografia.m`, ajuste `pasta_principal` para apontar para a pasta que contém os diretórios com os arquivos `.csv`. Por exemplo:

   ```matlab
   pasta_principal = "C:\COTB\data";
   ```

4. Verifique se as toolboxes necessárias estão instaladas e execute o script:

   ```matlab
   pipeline_termografia
   ```

5. Quando solicitado no *Command Window*, informe a quantidade de exames saudáveis que deseja selecionar para o balanceamento da base.

O script processará os exames encontrados e gerará os resultados gráficos e estatísticos previstos no pipeline.

---

## 📊 Resultados gerados

De acordo com o relatório técnico, o pipeline produz quatro figuras principais, uma tabela de métricas e pranchas de auditoria do processamento.

| Saída | Conteúdo | Objetivo da análise |
|---|---|---|
| **Figura 1 — Perfil longitudinal** | Curvas médias de 0% a 100% da tíbia e bandas de ±1 desvio padrão para os grupos saudável e patológico, com marcações em 30% e 70%. | Comparar o gradiente térmico longitudinal entre os grupos. |
| **Figura 2 — Boxplots anatômicos** | Distribuições de $T_{\text{prox}}$, $T_{\text{med}}$ e $T_{\text{dist}}$ nas quatro classes. | Comparar as temperaturas nos terços proximal, medial e distal. |
| **Figura 3 — Dispersão** | Projeção dos pacientes no espaço $T_{\text{prox}} \times T_{\text{dist}}$. | Visualizar agrupamentos e sobreposição entre as classes. |
| **Figura 4 — Avaliação KNN** | Matriz de confusão e visualização da fronteira de decisão do classificador. | Examinar os acertos e as confusões entre as classes no conjunto de teste. |
| **Pranchas de auditoria** | Sequência de seis etapas por exame: matriz bruta, caixa delimitadora/separação, recorte anatômico, eixo central, isolamento da pele e curvas individuais com média. | Inspecionar visualmente as etapas de segmentação e extração. |

### Métricas de classificação

Para cada uma das quatro classes, são calculadas as seguintes métricas no conjunto de teste:

- **Sensibilidade (Recall):** proporção de casos reais da classe identificados corretamente.
- **Precisão (valor preditivo positivo):** proporção das previsões daquela classe que estão corretas.
- **F1-score:** média harmônica entre sensibilidade e precisão.

Essas métricas devem ser analisadas em conjunto com a matriz de confusão, considerando a quantidade de exames disponível em cada classe.

---

## 🔎 Considerações metodológicas

- **Amostra por paciente:** quando as duas pernas são válidas, o método utiliza a média bilateral para gerar uma assinatura térmica por exame, evitando tratar os membros como observações independentes com o mesmo rótulo clínico.
- **Artefatos metálicos:** o recorte entre P5 e P99, as operações morfológicas e o uso do P75 procuram reduzir a influência de extremos térmicos e de hastes metálicas sobre o perfil extraído.
- **Reprodutibilidade:** a seleção aleatória da classe saudável utiliza `rng(42)`.
- **Balanceamento:** a quantidade de exames saudáveis é definida pelo usuário durante a execução; portanto, os resultados dependem também dessa escolha e da composição da base disponível.

---

## 📄 Licença

Este projeto está distribuído sob a licença especificada no arquivo [LICENSE](LICENSE). Consulte esse arquivo para conhecer os termos de uso, modificação e distribuição.
