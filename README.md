## **Projeto MinGo em parceria com a ClickBus**

### **Arquitetura de MLOps com Kubernetes**

Este documento descreve a arquitetura de Machine Learning Operations (MLOps) orquestrada por Kubernetes. A solução integra várias tecnologias para criar um ambiente robusto e escalável para o ciclo de vida de modelos de Machine Learning.

#### **Visão Geral da Arquitetura**

A arquitetura utiliza o Kubernetes como orquestrador central para gerenciar diversos serviços e contêineres. Cada componente desempenha um papel específico:

* **ZenML**: Plataforma de MLOps responsável por gerenciar os pipelines e metadados. O `zenml-server` atua como o cérebro da orquestração.
* **MLflow**: Componente fundamental para o ciclo de vida dos modelos. É utilizado para rastreamento de experimentos, registro de modelos e gerenciamento de artefatos.
* **PostgreSQL**: Banco de dados relacional que serve como backend para o ZenML e o MLflow, armazenando metadados, métricas e informações de rastreamento.
* **MinIO**: Armazenamento de objetos compatível com S3, utilizado para persistir os artefatos de modelos (como arquivos `.pkl` ou `.joblib`) de forma durável e acessível.
* **Contêineres de Predição Personalizados**: Serviços de inferência de modelos, criados a partir de uma imagem Python personalizada. **Para o empacotamento desses serviços, o BentoML foi utilizado**, gerando a imagem que contém a lógica de servir a API. Cada serviço é dedicado a um tipo de predição (classificação binária, multiclasse e clustering).

#### **Tecnologias e Funções no Ecossistema**

| Tecnologia | Componente no K8s | Função na Pipeline de MLOps |
| :--- | :--- | :--- |
| **ZenML** | `zenml-server` | Orquestra a pipeline de ML, gerencia fluxos de trabalho e metadados dos modelos. |
| **MLflow** | `mlflow-server-deployment` | Rastreia métricas de treinamento (ex: acurácia de 0.75), registra modelos e gerencia artefatos no MinIO. |
| **PostgreSQL** | `postgres-deployment` | Backend de metadados do ZenML e do MLflow, garantindo persistência e confiabilidade. |
| **MinIO** | `minio-server-deployment` | Armazenamento de artefatos de modelos (bins) de forma segura e escalável, substituindo a necessidade de serviços de cloud como AWS S3. |
| **BentoML** | `*-prediction` Deployments | Servidores de inferência de modelos empacotados pelo BentoML. Cada contêiner carrega um modelo específico a partir do MinIO e o expõe via um endpoint HTTP, pronto para ser consumido por aplicações externas. |

#### **Implementação e Fluxo de Trabalho**

A pipeline de MLOps é construída em etapas lógicas:

1.  **Orquestração e Rastreamento**: O ZenML (via `zenml-server`) executa os passos de uma pipeline de treinamento.
2.  **Treinamento e Logging**: Durante o treinamento, métricas e o modelo final são logados no MLflow (`mlflow-server`). O modelo treinado é salvo no MinIO, que atua como um repositório de artefatos.
3.  **Registro de Modelo**: O modelo, junto com suas métricas, é registrado no MLflow. Isso permite que a acurácia seja rastreada e comparada com versões anteriores.
4.  **Início da Inferência**: Após a aprovação do deploy, o Kubernetes inicia um dos `*-prediction` Deployments.
5.  **Serviço de Inferência**: Cada contêiner de predição, **empacotado pelo BentoML**, carrega o modelo específico do MinIO e serve as previsões em uma porta designada.

**Arquivos de Treinamento**:

É importante notar que a base para essa arquitetura são os modelos criados. A pasta `notebooks` da sua estrutura de projeto contém os três notebooks (`.ipynb`), cada um responsável por treinar e salvar um dos modelos de predição mencionados:

* **binary_notebook**: Treina o modelo de Classificação Binária.
* **binary_notebook**: Treina o modelo de Classificação Multiclasse.
* **binary_notebook**: Treina o modelo de Clustering.

Esses notebooks são a origem dos artefatos que são armazenados no MinIO, rastreados pelo MLflow e, por fim, servidos pelos contêineres de predição no Kubernetes.
