# 🛒 DIO — Armazenando Dados de um E-Commerce na Cloud

<p align="center">
  <img src="https://img.shields.io/badge/Microsoft%20Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white" alt="Azure"/>
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white" alt="Streamlit"/>
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/SQL%20Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white" alt="SQL Server"/>
</p>

Projeto de entrega do bootcamp **Microsoft Application Platform** da [DIO](https://www.dio.me/), com foco em armazenamento e gerenciamento de dados de um E-Commerce utilizando serviços da nuvem Microsoft Azure.

---

## 📋 Sobre o Projeto

O objetivo deste projeto é construir uma aplicação web de **Cadastro e Listagem de Produtos** para um E-Commerce, onde:

- As **imagens dos produtos** são armazenadas no **Azure Blob Storage**.
- Os **dados estruturados** (nome, descrição, preço, URL da imagem) são salvos no **Azure SQL Database**.
- A interface web é construída com **Streamlit** e pode ser executada localmente ou containerizada via **Docker** e implantada no **Azure Container Apps** ou **Azure Kubernetes Service (AKS)**.

---

## 🏗️ Arquitetura da Solução

```
Usuário (Browser)
       │
       ▼
 ┌─────────────┐       ┌─────────────────────────┐
 │  Streamlit  │──────▶│  Azure Blob Storage      │
 │  (Frontend) │       │  (Imagens dos Produtos)  │
 └──────┬──────┘       └─────────────────────────┘
        │
        ▼
 ┌────────────────────────────┐
 │  Azure SQL Database         │
 │  (nome, descrição, preço,  │
 │   URL da imagem)           │
 └────────────────────────────┘
```

**Serviços Azure utilizados:**

| Serviço | Função |
|---|---|
| Azure Blob Storage | Armazenamento de imagens dos produtos |
| Azure SQL Database | Armazenamento de dados relacionais dos produtos |
| Azure Container Apps | Hospedagem da aplicação Streamlit containerizada |
| Azure Kubernetes Service (AKS) | Orquestração de contêineres em escala |
| Azure Container Registry (ACR) | Repositório privado de imagens Docker |
| Application Insights | Monitoramento e observabilidade da aplicação |

---

## 🚀 Processo de Desenvolvimento

### Lab 01 — Aplicação Streamlit + Azure Blob Storage + Azure SQL

A aplicação principal foi desenvolvida em Python com Streamlit. Ela permite:

1. **Cadastrar produtos**: preenche nome, descrição, preço e faz upload de uma imagem.
2. **Upload da imagem**: a imagem é enviada ao Azure Blob Storage e a URL pública é retornada.
3. **Inserção no banco**: os dados do produto (incluindo a URL da imagem) são inseridos na tabela `dbo.Produtos` no Azure SQL Database.
4. **Listagem de produtos**: exibe todos os produtos cadastrados em um layout de cards responsivos.

**Schema da tabela SQL:**

```sql
CREATE TABLE Produtos (
    id          INT IDENTITY(1,1) PRIMARY KEY,
    nome        NVARCHAR(255),
    descricao   NVARCHAR(MAX),
    preco       DECIMAL(18,2),
    imagem_url  NVARCHAR(2083)
);
```

**Dependências principais:**

```
streamlit
azure-storage-blob
pymssql
pandas
```

### Lab 02 — Containerização e Deploy no AKS

A aplicação foi containerizada com Docker e implantada no Azure Kubernetes Service (AKS):

1. **Build da imagem Docker** e push para o Azure Container Registry (ACR).
2. **Criação do cluster AKS** via Azure CLI.
3. **Deploy via manifesto `deployment.yaml`** com réplicas configuradas.
4. **Exposição via `service.yaml`** com LoadBalancer público.

**Comandos principais (Azure CLI):**

```bash
# Build e push da imagem
az acr build --registry <acr-name> --image ecommerce-app:latest .

# Criar cluster AKS
az aks create --resource-group <rg> --name <aks-name> --node-count 1 --generate-ssh-keys

# Obter credenciais do cluster
az aks get-credentials --resource-group <rg> --name <aks-name>

# Deploy da aplicação
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Lab 03 — Azure Container Apps

A aplicação também foi implantada no **Azure Container Apps**, que oferece uma abstração de maior nível sobre Kubernetes, com escalonamento automático baseado em eventos (KEDA).

**Vantagens do Container Apps vs AKS:**
- Sem necessidade de gerenciar o plano de controle do Kubernetes.
- Escalonamento automático a zero (zero-scale), reduzindo custos.
- Integração nativa com Dapr para microsserviços.

---

## 💡 Insights e Aprendizados

### 🔵 Azure Blob Storage
- O Blob Storage é ideal para armazenar dados não estruturados como imagens, vídeos e arquivos.
- A geração de **URLs públicas** (ou com SAS tokens para acesso temporário seguro) torna fácil referenciar os arquivos em outras aplicações.
- O custo é extremamente baixo comparado a manter um servidor de arquivos próprio.

### 🟣 Azure SQL Database
- O serviço PaaS de SQL elimina a necessidade de gerenciar o servidor: patches, backups e alta disponibilidade são providos automaticamente.
- A integração com o `pymssql` (Python) é direta, facilitando o desenvolvimento.
- É importante usar **variáveis de ambiente** para armazenar connection strings e credenciais (nunca hardcoded em código-fonte).

### 🟠 Docker + Kubernetes (AKS)
- Containerizar a aplicação garante **portabilidade** e **reprodutibilidade** do ambiente.
- O AKS permite escalar horizontalmente a aplicação com poucos comandos.
- O Azure Container Registry (ACR) integra-se nativamente ao AKS, simplificando o pull de imagens privadas.

### 🟢 Azure Container Apps
- Para aplicações que não exigem controle fino do Kubernetes, o Container Apps oferece uma experiência mais simples com custo mais previsível.
- O modelo de cobrança por uso (consumption-based) é vantajoso para cargas de trabalho variáveis.

---

## 🛠️ Como Executar Localmente

### Pré-requisitos
- Python 3.9+
- Conta no Azure com os serviços provisionados (Blob Storage + SQL Database)

### Passos

```bash
# Clone o repositório
git clone https://github.com/Ishinaru/DIO_E-CommerceCloud.git
cd DIO_E-CommerceCloud

# Instale as dependências
pip install -r requirements.txt

# Configure as variáveis de ambiente com seus dados do Azure
export AZURE_STORAGE_CONNECTION_STRING="<sua-connection-string>"
export AZURE_SQL_SERVER="<seu-servidor>.database.windows.net"
export AZURE_SQL_DATABASE="<seu-banco>"
export AZURE_SQL_USER="<usuario>"
export AZURE_SQL_PASSWORD="<senha>"

# Execute a aplicação
streamlit run main.py
```

### Via Docker

```bash
# Build da imagem
docker build -t ecommerce-app .

# Execute o contêiner
docker run -p 8501:8501 ecommerce-app
```

---

## 📸 Screenshots

### Tela de Cadastro de Produto
> Interface desenvolvida com Streamlit para cadastro de produtos com upload de imagem para o Azure Blob Storage.

<!-- Adicione um screenshot da tela de cadastro aqui: ![Cadastro de Produto](assets/cadastro-produto.png) -->

### Listagem de Produtos
> Exibição dos produtos cadastrados em layout de cards, com imagem, nome, descrição e preço.

### Azure Portal — Blob Storage
> Contêiner de blobs com as imagens dos produtos armazenadas no Azure Storage Account.

### Azure Portal — SQL Database
> Tabela `dbo.Produtos` com os registros inseridos pela aplicação.

---

## 🔮 Possibilidades e Melhorias Futuras

- **Autenticação**: Adicionar autenticação via Azure Active Directory (Entra ID) para proteger o acesso à aplicação.
- **CI/CD**: Implementar pipeline de deploy contínuo com GitHub Actions para build e push automático da imagem Docker ao ACR.
- **Soft Delete / Edição de Produtos**: Adicionar operações de update e delete lógico.
- **Cache**: Utilizar Azure Cache for Redis para cachear a listagem de produtos e reduzir chamadas ao banco.
- **CDN**: Integrar Azure CDN para servir as imagens do Blob Storage com menor latência globalmente.
- **Monitoramento**: Configurar Application Insights para rastrear exceções e métricas de uso em tempo real.
- **Infraestrutura como Código**: Provisionar toda a infraestrutura com Bicep ou Terraform.

---

## 📚 Recursos Utilizados

- [Repositório oficial do bootcamp — DIO Microsoft Application Platform](https://github.com/digitalinnovationone/Microsoft_Application_Platform)
- [Documentação Azure Blob Storage](https://learn.microsoft.com/pt-br/azure/storage/blobs/)
- [Documentação Azure SQL Database](https://learn.microsoft.com/pt-br/azure/azure-sql/database/)
- [Documentação Azure Kubernetes Service](https://learn.microsoft.com/pt-br/azure/aks/)
- [Documentação Azure Container Apps](https://learn.microsoft.com/pt-br/azure/container-apps/)
- [Streamlit Docs](https://docs.streamlit.io/)

---

## 👤 Autor

Feito com 💙 como parte do bootcamp **Microsoft Application Platform** da [DIO](https://www.dio.me/).
