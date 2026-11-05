🦵 COTB - Termografia da Tíbia: Segmentação, Análise Longitudinal e Classificação de Patologias Ósseas

Este repositório contém os algoritmos em MATLAB, rotinas de processamento de imagens e análises estatísticas referentes ao artigo científico intitulado "COTB - Termografia da Tíbia", voltado à detecção e diferenciação de patologias ósseas e processos infecciosos por meio de matrizes termográficas infravermelhas (.csv).

📌 Visão Geral do Projeto

A termografia infravermelha é uma técnica não invasiva e não ionizante capaz de mapear a distribuição superficial de temperatura, refletindo alterações vasculares e inflamatórias profundas. O objetivo deste projeto é implementar um pipeline automatizado de visão computacional e aprendizado de máquina para isolar a região anatômica da tíbia, extrair seu perfil térmico longitudinal e classificar indivíduos em quatro grupos clínicos:

Saudável (Hígido) (em pé e deitado)

Pseudoartrose

Fratura Óssea

Osteomielite da Tíbia

⚙️ Arquitetura Metodológica do Pipeline

O algoritmo processa matrizes térmicas brutas em lote (batch processing) seguindo cinco estágios principais:

Ingestão e Filtragem Clínica:

Leitura recursiva de arquivos radiométricos .csv via imageDatastore e limpeza de artefatos (NaN).

Aplicação automática de critérios de exclusão clínica (descarte de membros com fixadores externos metálicos ou lesões já cicatrizadas).

Segmentação Morfológica e Invariância Postural:

Isolamento das duas pernas em relação ao fundo utilizando limiarização adaptativa de Otsu (graythresh), filtragem de área (bwareaopen) e preenchimento morfológico (imfill).

Detecção automática da orientação postural do paciente pela razão de aspecto da BoundingBox (em pé vs. deitado), aplicando rotação de $-90^\circ$ quando necessário para padronizar o eixo longitudinal na horizontal.

Recorte Anatômico Heurístico (ROI da Tíbia):

Remoção dos $20\%$ iniciais (articulação do joelho) e dos $25\%$ finais (tornozelo e pé), isolando os $55\%$ centrais correspondentes à diáfise tibial.

Normalização Espacial e Subdivisão Anatômica:

Extração do perfil térmico longitudinal médio (excluindo o fundo residual) e interpolação linear (interp1) para uma escala percentual fixa de 100 pontos ($0\%$ a $100\%$ da extensão da tíbia).

Subdivisão do perfil térmico em três terços anatômicos por indivíduo:

Porção Proximal ($\bar{T}_p$): $0\%$ a $30\%$

Porção Medial ($\bar{T}_m$): $30\%$ a $70\%$

Porção Distal ($\bar{T}_d$): $70\%$ a $100\%$

Análise Estatística e Aprendizado de Máquina (KNN):

Geração de curvas médias com bandas de desvio padrão ($\pm 1\text{ DP}$), boxplots comparativos por porção anatômica e gráficos de dispersão no espaço de características ($\bar{T}_{prox} \times \bar{T}_{dist}$).

Treinamento e validação de um classificador K-Vizinhos Mais Próximos (KNN, $k = 5$) com distância Euclidiana padronizada (Z-score) e particionamento estratificado Holdout (70% Treino / 30% Teste).

📊 Principais Resultados Obtidos

A execução do pipeline sobre o conjunto de dados completo processou 729 arquivos CSV, resultando na extração e validação de 1.398 tíbias:

Saudáveis (Hígidos): $1.079$ pernas

Pseudoartrose: $181$ pernas

Fratura Óssea: $97$ pernas

Osteomielite: $41$ pernas

Desempenho do Classificador KNN ($k = 5$)

Na validação com os 30% de amostras de teste, o modelo atingiu uma Acurácia Global de $75,18\%$, destacando-se na identificação do grupo controle saudável (Sensibilidade de $92,88\%$, Precisão de $83,33\%$ e F1-Score de $87,85\%$).

🛠️ Tecnologias e Requisitos

MATLAB (R2020b ou superior recomendado)

Toolboxes Necessárias:

Image Processing Toolbox (essencial para regionprops, imbinarize, graythresh, bwareaopen, imfill, imcrop, imrotate)

Statistics and Machine Learning Toolbox (essencial para fitcknn, cvpartition, confusionchart, boxplot, anova1)

📂 Estrutura do Repositório

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


🚀 Como Executar o Código

Clone este repositório em sua máquina local:

git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git


Abra o MATLAB e navegue até a pasta do projeto.

No início do script principal, ajuste a variável pasta_principal para apontar para o diretório onde os seus arquivos .csv estão armazenados:

pasta_principal = "Caminho\Para\Sua\Pasta\data";


Execute o script. O MATLAB exibirá o resumo da extração no Command Window e gerará automaticamente as 4 janelas de figuras:

Figura 1: Perfil Longitudinal com Sombra de Desvio Padrão (Hígido vs. Patológico).

Figura 2: Boxplots das distribuições térmicas nas porções Proximal, Medial e Distal.

Figura 3: Espaço de Agrupamento por Indivíduo ($\bar{T}_{prox} \times \bar{T}_{dist}$).

Figura 4: Matriz de Confusão e Fronteira de Decisão do Classificador KNN ($k = 5$).

📄 Licença

Este projeto está sob a licença especificada no arquivo LICENSE.
