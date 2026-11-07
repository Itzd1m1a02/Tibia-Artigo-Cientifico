# 🦵 COTB - Termografia da Tíbia: Segmentação, Análise Longitudinal e Classificação de Patologias Ósseas

![MATLAB](https://img.shields.io/badge/MATLAB-R2020b%2B-orange.svg)
![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-brightgreen.svg)

Este repositório contém os algoritmos em MATLAB, rotinas de processamento de imagens e análises estatísticas referentes ao artigo científico intitulado **"COTB - Termografia da Tíbia"**, voltado à detecção e diferenciação de patologias ósseas e processos infecciosos por meio de matrizes termográficas infravermelhas (`.csv`).

---

## 📌 Visão Geral do Projeto

A termografia infravermelha é uma técnica não invasiva e não ionizante capaz de mapear a distribuição superficial de temperatura, refletindo alterações vasculares e inflamatórias profundas. O objetivo deste projeto é implementar um **pipeline automatizado de visão computacional e aprendizado de máquina** para isolar a região anatômica da tíbia, extrair seu perfil térmico longitudinal e classificar indivíduos em quatro grupos clínicos:

1. **Saudável (Hígido)** *(em pé e deitado)*
2. **Pseudoartrose**
3. **Fratura Óssea**
4. **Osteomielite da Tíbia**

---

## ⚙️ Arquitetura Metodológica do Pipeline

O algoritmo processa matrizes térmicas brutas em lote (*batch processing*) seguindo cinco estágios principais:

1. **Ingestão e Filtragem Clínica:**
   - Leitura recursiva de arquivos radiométricos `.csv` via `imageDatastore` e limpeza de artefatos (`NaN`).
   - Aplicação automática de critérios de exclusão clínica (descarte de membros com fixadores externos metálicos ou lesões já cicatrizadas).
2. **Segmentação Morfológica e Invariância Postural:**
   - Isolamento das duas pernas em relação ao fundo utilizando limiarização adaptativa de Otsu (`graythresh`), filtragem de área (`bwareaopen`) e preenchimento morfológico (`imfill`).
   - Detecção automática da orientação postural do paciente pela razão de aspecto da *BoundingBox* (em pé vs. deitado), aplicando rotação de $-90^\circ$ quando necessário para padronizar o eixo longitudinal na horizontal.
3. **Recorte Anatômico Heurístico (ROI da Tíbia):**
   - Remoção dos $20\%$ iniciais (articulação do joelho) e dos $25\%$ finais (tornozelo e pé), isolando os **$55\%$ centrais** correspondentes à diáfise tibial.
4. **Normalização Espacial e Subdivisão Anatômica:**
   - Extração do perfil térmico longitudinal médio (excluindo o fundo residual) e interpolação linear (`interp1`) para uma escala percentual fixa de **100 pontos ($0\%$ a $100\%$ da extensão da tíbia)**.
   - Subdivisão do perfil térmico em três terços anatômicos por indivíduo:
     - **Porção Proximal ($\bar{T}_p$):** $0\%$ a $30\%$
     - **Porção Medial ($\bar{T}_m$):** $30\%$ a $70\%$
     - **Porção Distal ($\bar{T}_d$):** $70\%$ a $100\%$
5. **Análise Estatística e Aprendizado de Máquina (KNN):**
   - Geração de curvas médias com bandas de desvio padrão ($\pm 1\text{ DP}$), *boxplots* comparativos por porção anatômica e gráficos de dispersão no espaço de características ($\bar{T}_{prox} \times \bar{T}_{dist}$).
   - Treinamento e validação de um classificador **K-Vizinhos Mais Próximos (KNN, $k = 5$)** com distância Euclidiana padronizada (*Z-score*) e particionamento estratificado *Holdout* (**70% Treino / 30% Teste**).

---

## 🛠️ Tecnologias e Requisitos

- **MATLAB** (R2020b ou superior recomendado)
- **Toolboxes Necessárias:**
  - *Image Processing Toolbox* (`regionprops`, `graythresh`, `bwareaopen`, `imfill`, `imcrop`, `imrotate`)
  - *Statistics and Machine Learning Toolbox* (`fitcknn`, `cvpartition`, `confusionchart`, `boxplot`, `anova1`)

---

## 📂 Estrutura do Repositório

```text
.
├── data/                       # Diretório raiz do dataset termográfico (arquivos .csv)
│   ├── diagnosticos/
│   │   ├── fraturas_osseas/
│   │   ├── osteomielite_tibia/
│   │   └── pseudoartrose/
│   └── saudaveis/
│       ├── tibia_deitado/
│       └── tibia_em_pe/
├── matlab/
│   ├── pipeline_termografia.m  # Script principal (Segmentação, Estatística e KNN)
│   ├── document.xml            # Código/Script empacotado do MATLAB
│   └── output.xml              # Resultados e saídas geradas pelas simulações
├── figuras/                    # Gráficos exportados (Perfis, Boxplots, Matriz de Confusão)
├── README.md                   # Documentação do repositório
└── LICENSE                     # Licença do projeto
```

---

## 🚀 Como Executar o Código

1. Clone este repositório em sua máquina local:
   ```bash
   git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
   ```
2. Abra o **MATLAB** e navegue até a pasta do projeto.
3. No início do script principal (`pipeline_termografia.m`), ajuste a variável `pasta_principal` para apontar para o diretório onde os seus arquivos `.csv` estão armazenados:
   ```matlab
   pasta_principal = "Caminho\Para\Sua\Pasta\data";
   ```
4. Execute o script. O MATLAB exibirá o resumo da extração no *Command Window* e gerará automaticamente as 4 figuras de análise:
   - **Figura 1:** Perfil Longitudinal com Sombra de Desvio Padrão (Hígido vs. Patológico).
   - **Figura 2:** *Boxplots* das distribuições térmicas nas porções Proximal, Medial e Distal.
   - **Figura 3:** Espaço de Agrupamento por Indivíduo ($\bar{T}_{prox} \times \bar{T}_{dist}$).
   - **Figura 4:** Matriz de Confusão e Fronteira de Decisão do Classificador KNN ($k = 5$).

---

## 📄 Licença

Este projeto está sob a licença especificada no arquivo [LICENSE](LICENSE).
