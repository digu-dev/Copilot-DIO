# 📚 Material de Estudo - Certificação AZ-900

> Material completo gerado a partir de sessão de estudos focada nos principais tópicos da certificação Microsoft Azure Fundamentals (AZ-900)

---

## 📋 Índice

1. [Infraestrutura Global e Alta Disponibilidade](#1-infraestrutura-global-e-alta-disponibilidade)
2. [SLAs dos Principais Serviços](#2-slas-dos-principais-servi9os)
3. [Segurança de Rede](#3-seguran9a-de-rede)
4. [VPN Gateway e ExpressRoute](#4-vpn-gateway-e-expressroute)
5. [Serviços Serverless](#5-servi9os-serverless)
6. [Azure App Service Plans](#6-azure-app-service-plans)
7. [M3quinas Virtuais e VMSS](#7-m3quinas-virtuais-e-vmss)
8. [Azure App Service e Container Apps](#8-azure-app-service-e-container-apps)
9. [Container Instances e AKS](#9-container-instances-e-aks)
10. [Azure Functions e Logic Apps](#10-azure-functions-e-logic-apps)
11. [Redes no Azure](#11-redes-no-azure)
12. [Segurança e Identidade](#12-seguran9a-e-identidade)
13. [Modelos de Serviço: IaaS, PaaS, SaaS e Serverless](#13-modelos-de-servi9o-iaas-paas-saas-e-serverless)
14. [Hierarquia de Governança e Custos](#14-hierarquia-de-governan9a-e-custos)
15. [Conceitos Básicos de Cloud Computing](#15-conceitos-b9sicos-de-cloud-computing)
16. [Ferramentas de Custo](#16-ferramentas-de-custo)
17. [Service Health vs Resource Health](#17-service-health-vs-resource-health)
18. [Tipos de Armazenamento](#18-tipos-de-armazenamento)
19. [Pegadinhas Comuns](#19-pegadinhas-comuns)

---

## 1. Infraestrutura Global e Alta Disponibilidade

### Regiões vs Zonas de Disponibilidade

| Conceito | O que é | SLA | Quando usar |
|----------|---------|-----|-------------|
| **Região** | Área geográfica (ex: Brazil South) | N/A | Base para qualquer recurso |
| **Zona de Disponibilidade (AZ)** | Datacenters físicos independentes dentro de uma região | N/A | Primeira linha contra falhas |
| **Availability Set** | Isolamento lógico de VMs no mesmo datacenter | **99,95%** | Protege contra falhas de hardware |
| **Availability Zone** | VM distribuída em datacenters diferentes | **99,99%** | Protege contra queda de datacenter |
| **VMSS** | Escala automática de VMs | Herda do tipo | **Limite: 1.000 instâncias** ⚠️ |

**Regra de Ouro:** Sets = datacenter único; Zones = múltiplos datacenters = melhor SLA

---

## 2. SLAs dos Principais Serviços

### Computação

| Configuração | SLA |
|--------------|-----|
| VM única (qualquer disco) | **99,9%** |
| 2+ VMs em Availability Set | **99,95%** |
| 2+ VMs em Availability Zones | **99,99%** |
| AKS padrão | **99,95%** |
| AKS com Uptime SLA | **99,99%** |

### Bancos de Dados

| Serviço | SLA |
|---------|-----|
| SQL Database | **99,99%** |
| Cosmos DB (Single Region) | **99,99%** |
| Cosmos DB (Multi-Region Writes) | **99,999%** ⭐ |
| MySQL/PostgreSQL Flexible Server | **99,95%** |

### Armazenamento

| Tipo | SLA |
|------|-----|
| LRS | **99,9%** |
| ZRS | **99,95%** |
| GRS | **99,9%** |
| RA-GRS | **99,99%** ⭐ |
| RA-GZRS | **99,99%** |

### Rede

| Serviço | SLA |
|---------|-----|
| Load Balancer Basic | **99,9%** |
| Load Balancer Standard | **99,99%** ⭐ |
| Application Gateway | **99,95%** |
| VPN Gateway | **99,9%** |
| ExpressRoute Premium | **99,95%** |

### Resumo em Grupos

```
99,9% (MAIS COMUM)
• VM única, App Service, LRS Storage, VPN Gateway, Entra ID

99,95% (INTERMEDIÁRIO)
• Availability Set, AKS, ZRS Storage, Application Gateway, Azure Synapse

99,99% (MELHOR)
• Availability Zone, Load Balancer Standard, RA-GRS Storage, AKS com Uptime SLA

99,999% (PREMIUM)
• Cosmos DB Multi-Region Writes
```

---

## 3. Segurança de Rede

### As 4 Camadas de Defesa

```
INTERNET
    ↓
1️⃣ AZURE FIREWALL (Perímetro - protege toda a VNet)
    ↓
2️⃣ NETWORK SECURITY GROUP - NSG (Subnets / NICs)
    ↓
3️⃣ APPLICATION GATEWAY + WAF (HTTP/HTTPS - Layer 7)
    ↓
4️⃣ VMs / APPS
```

### Comparação dos Serviços

| Ferramenta | Camada | Escopo | WAF | FQDN Filter | Custo |
|-----------|--------|--------|-----|-------------|-------|
| NSG | L4 | Subnet/NIC | ❌ | ❌ | Baixo |
| Azure Firewall | L3-4+L7 | VNet (perímetro) | ❌ | ✅ | Alto |
| App Gateway | L7 | App específica | ✅ | ❌ | Médio |
| WAF | L7 | App específica | ✅ | ❌ | Alto |

### Regras de Uso

- **NSG** → regras simples por porta/IP
- **Azure Firewall** → perímetro central, regras complexas, FQDN filtering
- **Application Gateway + WAF** → balanceamento HTTP + proteção contra SQLi/XSS
- **Azure Bastion** → acesso RDP/SSH sem expor IP público
- **DDoS Protection** → mitigação de ataques volumétricos L3/L4

### WAF - Ataques Bloqueados

- ✅ SQL Injection
- ✅ XSS (Cross-Site Scripting)
- ✅ Command Injection
- ✅ Path Traversal
- ✅ DDoS (OWASP Top 10)

---

## 4. VPN Gateway e ExpressRoute

### Comparação

| Critério | VPN Gateway | ExpressRoute |
|----------|-------------|--------------|
| Tecnologia | Internet público (IPSec) | Conexão dedicada (BGP) |
| Velocidade | Até 10 Gbps | 50 Mbps - 100 Gbps |
| Latência | ~100ms (variável) | ~10ms (estável) |
| SLA | 99,9% | 99,95% (premium) |
| Custo | Baixo ✅ | Alto |
| Setup | Minutos ✅ | Semanas |
| Usa Internet? | Sim | Não ✅ |

### Quando Usar

**VPN Gateway:**
- Orçamento limitado
- Setup rápido necessário
- Tráfego variável/ocasional
- Filial pequena ou trabalho remoto

**ExpressRoute:**
- Tráfego crítico e previsível
- SLA alto exigido (99,95%+)
- Latência baixa necessária
- Grande volume de dados
- Banco, hospital, governo

---

## 5. Serviços Serverless

### O que é Serverless?

```
SERVERLESS = Você NÃO gerencia infraestrutura
✅ Não gerencia: VMs, servidores, scaling manual
✅ Não paga por: Tempo inativo
✅ Paga por: Tempo de execução real
✅ Escala: Automática
```

### Principais Serviços

| Serviço | Tipo | Melhor Para | SLA |
|---------|------|-------------|-----|
| Azure Functions | Code-on-demand | Webhooks, processamento | 99,95% |
| Logic Apps | Workflow visual | Orquestração sem código | 99,9% |
| Container Instances | Containers | Trabalhos curtos | 99,9% |
| Azure Batch | Jobs paralelos | Processamento em massa | 99,95% |
| Service Bus | Messaging | Filas confiáveis | 99,95% |
| Event Grid | Pub/Sub | Eventos real-time | 99,99% |
| Cosmos DB Serverless | Database | Dados escaláveis | 99,99% |
| Static Web Apps | Frontend+API | SPA com backend | 99,95% |

### Árvore de Decisão

```
Precisa rodar CÓDIGO?
  ✅ SIM
    ├─ Rápido (ms)?       → Azure Functions
    ├─ Container?         → Container Instances
    └─ Paralelo massivo?  → Azure Batch

  ❌ NÃO
    ├─ Workflow visual?   → Logic Apps
    ├─ Mensagens?         → Service Bus
    ├─ Eventos real-time? → Event Grid
    └─ API pública?       → API Management
```

---

## 6. Azure App Service Plans

### Comparação dos Planos

| Plano | Tipo | SLA | Auto-Scale | Custo |
|-------|------|-----|------------|-------|
| Free | Compartilhado | ❌ Nenhum | ❌ | $0 |
| Shared | Compartilhado | ❌ Nenhum | ❌ | ~$15/mês |
| Basic | Dedicado | 99,95% | Até 3 | ~$70-280/mês |
| Standard | Dedicado | 99,95% | Até 10 | ~$100-400/mês |
| Premium | Dedicado | 99,95% | Até 100 | ~$300-1200/mês |
| Isolated | Isolado | 99,95% | Até 100 | ~$1500-6000/mês |

### Recursos por Plano

| Recurso | Free | Shared | Basic | Standard | Premium | Isolated |
|---------|------|--------|-------|----------|---------|----------|
| Always On | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Staging Slots | 0 | 0 | 1 | 5 | 20 | 20+ |
| Domínio Custom | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| VNet Integration | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |

> ⚠️ **Importante:** App Service NÃO é serverless. Você gerencia o plano/capacidade.

---

## 7. Máquinas Virtuais e VMSS

### Azure Virtual Machines (IaaS)

**Você gerencia:**
- Sistema operacional (patching, antivirus, firewall do SO)
- Runtime, aplicação e dependências
- Escala e alta disponibilidade

**Azure gerencia:**
- Datacenter físico, energia, rede
- Host/hypervisor

**Quando usar:**
- Software legado (ex: Windows Registry)
- Controle total do ambiente
- Aplicações stateful/monolíticas

### VMSS - Virtual Machine Scale Sets

**O que FAZ:**
- ✅ Cria múltiplas VMs automaticamente
- ✅ Escala baseado em regras
- ✅ Gerencia ciclo de vida

**O que NÃO FAZ:**
- ❌ Distribuir tráfego
- ❌ Load balancing
- ❌ Health checks

> ⚠️ **Pegadinha:** VMSS SEMPRE precisa de **Load Balancer** ou **Application Gateway** para distribuir tráfego!

### Arquitetura Correta com VMSS

```
INTERNET
    ↓
Load Balancer (distribui tráfego)
    ↓
VMSS (cria/remove VMs automaticamente)
[VM1] [VM2] [VM3] ... [VMn]
```

---

## 8. Azure App Service e Container Apps

### Azure App Service

- **Tipo:** PaaS
- **Uso:** Web Apps, APIs, backends
- **Você gerencia:** aplicação + plano de capacidade
- **Azure gerencia:** SO, patching, infraestrutura
- **Custo:** por hora do plano (mesmo parado)
- **Ideal para:** apps web/APIs sempre ligadas

### Azure Container Apps

- **Tipo:** PaaS serverless para containers
- **Uso:** microserviços, APIs containerizadas
- **Scale to zero:** ✅ sim
- **Kubernetes sob o capô:** sim (abstrato)
- **Ideal para:** containers sem operar Kubernetes

### Comparação

| Aspecto | App Service | Container Apps |
|---------|-------------|----------------|
| Tipo | PaaS | PaaS Serverless |
| Scale to zero | ❌ | ✅ |
| Container | Opcional | Obrigatório |
| Custo base | Por plano | Por uso |
| Ideal para | Web apps tradicionais | Microserviços em container |

---

## 9. Container Instances e AKS

### Azure Container Instances (ACI)

- **Uso:** rodar container sob demanda, sem orquestração
- **Cobrança:** por **segundo** ✅
- **Setup:** segundos
- **Ideal para:** jobs batch curtos, execuções sob demanda
- **NÃO é:** orquestrador de microserviços

### AKS - Azure Kubernetes Service

- **Uso:** orquestrar containers em escala (Kubernetes gerenciado)
- **Você gerencia:** manifestos, namespaces, RBAC, node pools
- **Azure gerencia:** control plane do Kubernetes
- **Ideal para:** muitos microserviços, controle fino, portabilidade

### Comparação

| Aspecto | ACI | AKS |
|---------|-----|-----|
| Complexidade | Baixa | Alta |
| Orquestração | ❌ | ✅ |
| Cobrança | Por segundo | Por nó |
| Ideal para | Jobs rápidos | Microserviços em escala |

---

## 10. Azure Functions e Logic Apps

### Azure Functions (FaaS)

- **Tipo:** Serverless - código sob demanda
- **Triggers:** HTTP, Timer, Blob, Queue, Service Bus, Event Grid, Cosmos DB
- **Planos:** Consumption (serverless puro), Premium, Dedicated
- **Ideal para:** webhooks, processamento de arquivos, tarefas agendadas

### Azure Logic Apps

- **Tipo:** Serverless - workflow visual (low-code/no-code)
- **Uso:** orquestração de processos com 200+ conectores
- **Ideal para:** integração entre SaaS, aprovações, automação de processos

### Comparação

| Aspecto | Azure Functions | Logic Apps |
|---------|----------------|------------|
| Código | ✅ Necessário | ❌ Visual |
| Flexibilidade | Alta | Média |
| Conectores prontos | Poucos | 200+ |
| Ideal para | Lógica customizada | Integração enterprise |

---

## 11. Redes no Azure

### VNets e Subnets

- **VNet:** rede privada do Azure (espaço de endereçamento CIDR)
- **Subnet:** fatia da VNet para segmentar workloads
- **Subnet NÃO é perímetro** - é segmentação interna

### VNet Peering

- Conecta **VNet ↔ VNet** via backbone Microsoft (não Internet pública)
- **NÃO é VPN** (VPN = on-premises ↔ Azure)
- Regional ou Global

### Serviços de Entrada

| Serviço | Camada | Escopo | WAF | Routing Avançado |
|---------|--------|--------|-----|-----------------|
| Load Balancer | L4 (TCP/UDP) | Regional | ❌ | ❌ |
| Application Gateway | L7 (HTTP/HTTPS) | Regional | ✅ | Por path/host |
| Front Door | L7 (edge global) | Global | ✅ | Por região/latência |

### Segurança de Rede

| Serviço | Função | Camada |
|---------|--------|--------|
| NSG | Regras por porta/IP | L3/L4 |
| Azure Firewall | Perímetro central | L3-L7 |
| Azure Bastion | RDP/SSH sem IP público | N/A |
| DDoS Protection | Mitigação volumétrica | L3/L4 |
| WAF | Proteção aplicação web | L7 |

### Árvore de Decisão - Qual Escolher?

```
Precisa conectar redes?
  Azure ↔ Azure  → VNet Peering
  On-prem ↔ Azure → VPN Gateway (internet) ou ExpressRoute (dedicado)

Precisa balancear tráfego?
  TCP/UDP → Load Balancer
  HTTP/HTTPS + WAF → Application Gateway
  Global + Edge → Front Door

Precisa de segurança?
  Regras simples → NSG
  Perímetro central → Azure Firewall
  Administrar VMs → Azure Bastion
  SQLi/XSS → WAF
  DDoS → DDoS Protection
```

---

## 12. Segurança e Identidade

### Microsoft Entra ID vs Entra Domain Services

| Aspecto | Entra ID | Entra Domain Services |
|---------|----------|-----------------------|
| Tipo | Identidade moderna (cloud) | AD compatível (legado) |
| Protocolos | OAuth 2.0, OIDC, SAML | LDAP, Kerberos, NTLM |
| Para quê | SSO, MFA, Conditional Access | Apps legadas, domain join |
| Gerenciamento | Você gerencia | Azure gerencia DCs |

### Autenticação

**SSO:** login único → acesso múltiplos apps  
**MFA:** segundo fator além de senha (app, SMS, FIDO2)  
**Conditional Access:** política "SE…ENTÃO…"

```
Condições válidas em Conditional Access:
✅ User risk level
✅ Sign-in risk level
✅ User location
✅ Device state/compliance
✅ Application type
✅ Client app

❌ NÃO é condição válida:
❌ Device type (iPhone/Android/Windows)
```

### RBAC - Roles Principais

| Role | Pode criar/alterar | Pode delegar acesso |
|------|-------------------|---------------------|
| **Owner** | ✅ | ✅ |
| **Contributor** | ✅ | ❌ |
| **Reader** | ❌ | ❌ |

> RBAC é herdado por escopo: Management Group → Subscription → Resource Group → Recurso

### Microsoft Defender for Cloud

**Features reais:**
- ✅ Security posture management (Secure Score)
- ✅ Threat detection e response
- ✅ Vulnerability assessment
- ✅ Compliance monitoring

**NÃO é feature:**
- ❌ Password management (isso é Entra ID)
- ❌ MFA (isso é Entra ID)
- ❌ Conditional Access (isso é Entra ID)

### B2B vs B2C

| Aspecto | B2B | B2C |
|---------|-----|-----|
| Para quem | Parceiros/fornecedores externos | Consumidores finais |
| Login | Identidade corporativa do parceiro | Email, login social (Google, Facebook) |
| Escala | Centenas/milhares | Milhões |
| Ideal para | Acesso a apps corporativas | Apps de e-commerce, banco para clientes |

---

## 13. Modelos de Serviço: IaaS, PaaS, SaaS e Serverless

### 1️⃣ IaaS – Infrastructure as a Service

> **Ideia:** Azure entrega infraestrutura virtual (servidores, rede, disco). Você é responsável pelo SO para cima.

| Você gerencia | Azure gerencia |
|---------------|----------------|
| Sistema operacional (Windows/Linux) | Datacenter físico |
| Patches, antivirus, firewall do SO | Hosts/hypervisor |
| Runtime (.NET, Java, Node, etc.) | Rede física, energia, refrigeração |
| Aplicação e dados | — |

#### Exemplos de IaaS no Azure

| Serviço | Descrição |
|---------|-----------|
| Azure Virtual Machines (VMs) | Servidor virtual Windows/Linux |
| Azure Virtual Machine Scale Sets (VMSS) | Grupo de VMs com autoscale |
| Azure Managed Disks | Discos para VMs |
| Azure Virtual Network (VNet) | Rede privada |
| Azure Load Balancer (Basic/Standard) | Balanceamento L4 para VMs |
| Azure VPN Gateway | Túnel VPN site-to-site/point-to-site |
| Azure Dedicated Host | Host físico dedicado para suas VMs |

#### Quando usar IaaS
- Migrar servidor on-prem "como está" para nuvem (lift and shift)
- Software legado que precisa de acesso ao SO/Registry/driver
- Controle fino de sistema e middleware

---

### 2️⃣ PaaS – Platform as a Service

> **Ideia:** Azure entrega uma plataforma gerenciada (runtime, SO, patches). Você foca em código e dados, não em SO.

| Você gerencia | Azure gerencia |
|---------------|----------------|
| Código/aplicação | Sistema operacional |
| Configuração da app (conn strings, settings) | Runtime da linguagem |
| Dados e schema | Patching, disponibilidade da plataforma |
| — | Escalabilidade da plataforma (dentro de limites de plano) |

#### Exemplos de PaaS no Azure

| Serviço | Descrição |
|---------|-----------|
| Azure App Service (Web Apps, API Apps) | Hospedar sites e APIs |
| Azure SQL Database | SQL Server gerenciado como serviço |
| Azure Cosmos DB (mode provisionado) | NoSQL multi-modelo |
| Azure Synapse Analytics (SQL pool) | Data warehouse gerenciado |
| Azure Service Bus | Filas e tópicos enterprise |
| Azure Event Hubs | Ingestão de eventos em grande volume |
| Azure Container Apps | PaaS serverless para containers/microserviços |
| Azure Kubernetes Service (AKS) | Orquestração de containers gerenciada |
| Azure API Management | Gateway/gerenciador de APIs |
| Azure Key Vault | Config/segredos gerenciados |

#### Quando usar PaaS
- App web/API moderna (HTTP/HTTPS) sem dependência de SO específico
- Reduzir esforço de operação (não quer patch de sistema)
- Ganhar rapidez em desenvolvimento e deploy

---

### 3️⃣ SaaS – Software as a Service

> **Ideia:** Azure/Microsoft entrega o aplicativo pronto. Você só usa; não gerencia infra, SO, nem código.

| Você gerencia | Azure/Microsoft gerencia |
|---------------|--------------------------|
| Usuários, licenças | Toda a aplicação (código) |
| Configurações de uso (políticas, templates) | Plataforma, SO, infra, segurança da aplicação |
| Dados que coloca no serviço | — |

#### Exemplos de SaaS da Microsoft

| Serviço | Descrição |
|---------|-----------|
| Microsoft 365 | Exchange Online, SharePoint Online, OneDrive, Teams |
| Dynamics 365 | CRM/ERP SaaS da Microsoft |
| Power BI (serviço) | BI na nuvem |
| Azure DevOps Services | Hospedado como serviço |
| GitHub | SaaS puro (ecossistema Microsoft) |

> 💡 **Dica AZ-900:** "Modelo em que você só consome a aplicação, sem gerenciar nada de infraestrutura" → **SaaS**

---

### 4️⃣ Serverless – Modelo de Execução (PaaS 2.0)

> **Importante:** Serverless não é uma "quinta categoria" oficial como IaaS/PaaS/SaaS — é um **estilo de execução** dentro de PaaS com características especiais.

#### Características de Serverless

```
✅ Você não provisiona nem gerencia servidores/VMs/plano
✅ Escala automática (inclui scale-to-zero)
✅ Cobra por uso/execução, não por capacidade parada
✅ Fortemente event-driven (executa em resposta a eventos)
```

#### Exemplos de Serverless no Azure

**a) Compute / Código**

| Serviço | Descrição |
|---------|-----------|
| Azure Functions | Funções disparadas por HTTP, Timer, Blob, Queue, Event Grid, Service Bus, Cosmos DB. Plano Consumption: escala automático, paga por execução/GB-segundo |

**b) Workflow / Integração**

| Serviço | Descrição |
|---------|-----------|
| Azure Logic Apps | Workflows visuais com conectores (Office 365, SAP, Dynamics, ServiceNow, etc.). Paga por ação/execução |

**c) Containers Serverless**

| Serviço | Descrição |
|---------|-----------|
| Azure Container Instances (ACI) | Executa containers sob demanda, paga por segundo. Sem cluster, sem VM |
| Azure Container Apps | Containers com autoscale (KEDA), incluindo scale-to-zero. PaaS serverless para containers/microserviços |

**d) Eventos e Mensagens**

| Serviço | Descrição |
|---------|-----------|
| Azure Event Grid | Roteamento de eventos (pub/sub) totalmente gerenciado e serverless |
| Azure Service Bus | Fila/tópico gerenciado — paga por operação/unidade, sem se preocupar com servidores |
| Azure Queue Storage | Modelo simples de fila, também gerenciado |

**e) Dados Serverless**

| Serviço | Descrição |
|---------|-----------|
| Azure Cosmos DB (serverless mode) | Em vez de provisionar RU/s fixos, paga por operação |
| Azure SQL Database serverless | Escala automática com pausa automática quando inativo |
| Azure Synapse serverless (SQL on-demand) | Consultas ad-hoc sem provisionar recursos |

---

### Resumão: Exemplos por Modelo

```
IaaS
  • Azure Virtual Machines
  • Azure Virtual Machine Scale Sets
  • Azure Managed Disks
  • Azure Virtual Network
  • Azure Load Balancer
  • Azure VPN Gateway

PaaS
  • Azure App Service
  • Azure SQL Database
  • Azure Cosmos DB (provisionado)
  • Azure Synapse Analytics
  • Azure Service Bus
  • Azure Event Hubs
  • Azure Kubernetes Service (AKS)
  • Azure API Management
  • Azure Key Vault

SaaS
  • Microsoft 365 (Exchange, SharePoint, Teams, OneDrive)
  • Dynamics 365
  • Power BI Service
  • Azure DevOps Services
  • GitHub (como serviço gerenciado)

Serverless (modelo de execução)
  • Azure Functions
  • Azure Logic Apps
  • Azure Container Instances
  • Azure Container Apps
  • Azure Event Grid
  • Cosmos DB (serverless mode)
  • Synapse SQL serverless
```

---

## 14. Hierarquia de Governança e Custos

### 1️⃣ Hierarquia de Governança

> Ordem do mais alto nível para o mais baixo:

```
Tenant (Microsoft Entra ID)
  ↓
Management Groups (Grupos de gerenciamento)  ✅ PODEM ser nested
  ↓
Subscriptions (Assinaturas)                  ❌ NÃO podem ser nested
  ↓
Resource Groups                               ❌ NÃO podem ser nested
  ↓
Resources (VM, Storage, SQL, etc.)           ❌ NÃO podem ser nested
```

#### Management Group (Grupo de Gerenciamento)

- Agrupa várias subscriptions
- **Pode ser nested ✅** (árvore de MGs):

```
MG-Root
  ├─ MG-Prod
  │    ├─ Sub-Prod-A
  │    └─ Sub-Prod-B
  └─ MG-Dev
       └─ Sub-Dev-A
```

**Usos principais:**
- Aplicar Azure Policy em larga escala (governança técnica)
- Aplicar RBAC de alto nível (permissões por unidade de negócio)

#### Subscription (Assinatura)

- Unidade de cobrança (billing) e limites (quotas)
- Contém vários Resource Groups
- **Pode ser nested? ❌ NÃO**
- Você organiza subscriptions usando Management Groups, não uma dentro da outra

#### Resource Group

- Contêiner lógico para recursos com mesmo ciclo de vida
- **Pode ser nested? ❌ NÃO**
- Um RG não contém outro RG, apenas recursos

#### Resources

- Os objetos reais: VMs, Storage, DBs, VNets, etc.
- Sempre pertencem a um Resource Group
- **Pode ser nested? ❌ NÃO**

---

### 2️⃣ Governança ↔ Custos

> Governança bem feita ajuda a organizar custos, evitar gastos indevidos e atribuir custos corretamente.

**Exemplos práticos:**

- **Separar subscriptions por ambiente:**
  - Sub-Prod / Sub-Dev / Sub-Test → cada uma com billing, budgets e alertas próprios

- **Management Groups para unidades de negócio:**
  - MG-Financeiro → Sub-Fin-Prod, Sub-Fin-Dev
  - MG-Marketing → Sub-Mkt-Prod, Sub-Mkt-Dev
  - → Permite ver custo por área no Cost Management

- **Tags:**
  ```
environment = prod/dev
department  = Financeiro/Marketing
project     = LojaOnline/CoreBanking

  → Filtra custo por Tag e vê quem está gastando quanto

- **Azure Policy:**
  - Bloquear tipos de VMs muito caros
  - Obrigar Tag `costCenter` em todo recurso

---

### 3️⃣ Microsoft Purview

> Plataforma de **governança de dados** (não de infra).

**Principais funções:**
- Catalogar fontes de dados (Azure SQL, Storage, Power BI, on-premises, etc.)
- Classificar dados (PII, dados sensíveis, conformidade LGPD/GDPR)
- Ver onde estão dados sensíveis
- Rastrear linhagem de dados (de onde vem, por onde passa, onde vai)

> 💡 **Dica AZ-900:** Se a pergunta falar em **catalogar dados**, **descobrir dados sensíveis** ou **governança de dados corporativos** → resposta tende a ser **Microsoft Purview**.

---

### 4️⃣ Azure Policy

> Ferramenta para criar e aplicar políticas de configuração e conformidade em recursos Azure.

| Efeito | O que faz | Exemplo |
|--------|-----------|---------|
| **Deny** | Bloqueia criação se não seguir a política | Negar VMs fora de Brazil South |
| **Audit** | Marca como "não conforme" sem bloquear | Marcar recursos sem tag costCenter |
| **Append** | Adiciona config automaticamente | Adicionar tag owner com valor padrão |
| **DeployIfNotExists** | Se não existir, implanta algo | Garantir que Log Analytics esteja ligado para VMs |

**Onde aplica (escopo):**
- Management Group
- Subscription
- Resource Group
- Recurso

**Relacionado a custos:**
- Impedir criação de SKUs caros
- Impedir recursos em regiões mais caras
- Exigir tags de custo
- Manter padrão de governança que impacta diretamente o gasto

---

### 5️⃣ Resource Lock e Tags

#### Resource Lock (Bloqueio)

> Impede deleção ou alteração acidental de um recurso/RG/Subscription.

| Tipo | O que impede |
|------|-------------|
| **CanNotDelete** | Não pode deletar, mas pode modificar |
| **ReadOnly** | Não pode deletar nem modificar (apenas leitura) |

**Quando usar:**
- Resource Group de produção
- Banco de dados crítico
- Componentes de rede base

#### Tags

> Metadados em forma de chave-valor anexados a recursos, RGs ou Subscriptions.

**Exemplos:**
environment = production
department  = financeiro
project     = ecommerce
costCenter  = CC123
owner       = joao.silva
```

```



**Para que servem:**
- Organização
- Governança (Policy pode exigir Tags)
- Cost Management: filtrar custo por Tag (quem gasta quanto)
- Scripts/automação baseados em Tags

> ⚠️ **Tags não fazem nada sozinhas:**
> - ❌ Não desligam VM
> - ❌ Não aplicam desconto
> - ❌ Não movem recurso de tier
> 
> Elas ajudam a **identificar e agrupar** para você tomar ações (manuais ou automatizadas via script).

---

### 6️⃣ Azure Advisor

> "Consultor" automático de boas práticas no Azure.

**Dá recomendações em 5 áreas:**

| Área | Exemplo de recomendação |
|------|------------------------|
| **Custo** | VMs superdimensionadas, usar reservas |
| **Performance** | App com pouco CPU disponível |
| **Alta Disponibilidade** | VMs fora de Availability Set/Zone |
| **Segurança** | Integrado com Defender for Cloud |
| **Operational Excellence** | Práticas operacionais |

**Como ajuda em custo:**
- Aponta recursos subutilizados (VM muito grande, banco com DTUs/RU/s sobrando)
- Sugere otimizações: downgrade de SKU, Reserved Instances, desligar recursos sem uso

> 💡 **Dica AZ-900:** "Recomendações para otimização de custo, performance, disponibilidade" → **Azure Advisor**

---

### 7️⃣ Resource Health

> Mostra o estado de saúde de **recursos específicos** (VM, App Service, DB, etc.).

**Status típicos:**

| Status | Significado |
|--------|-------------|
| Healthy | Recurso funcionando normalmente |
| Degraded | Recurso com degradação de performance |
| Unavailable | Recurso indisponível |
| Unknown | Estado não determinado |

**Para que serve:**
- Ver se o problema é seu (configuração, app) ou da plataforma Azure (incidente/datacenter)
- Ver histórico de incidentes por recurso

**Diferença importante:**

| Serviço | Escopo |
|---------|--------|
| **Azure Status** | Saúde de serviços/regiões (global, público) |
| **Resource Health** | Saúde dos **seus** recursos específicos |

---

### Resumão Rápido — Governança e Custos

```
```
Hierarquia:
  Tenant → Management Group (nested ✅) → Subscription → Resource Group → Resource

Custos:
  Controlados por: Subscriptions, Tags, Policy, Advisor, Monitor

Microsoft Purview:
  Governança de dados (catálogo, classificação, linhagem)

Azure Policy:
  Regras para manter conformidade de recursos (bloquear/monitorar/configurar)

Resource Lock:
  Protege contra delete/alteração acidental

Tags:
  Chave-valor para organizar e atribuir custos (não automatizam nada sozinhas)

Azure Advisor:
  Recomendações de custo, performance, HA, segurança

Resource Health:
  Saúde de recursos individuais (se impacto é Azure ou seu)

Azure Monitor:
  Métricas + Logs + Alertas + Application Insights + Log Analytics
```
```
---

## 15. Conceitos Básicos de Cloud Computing

### Definição (Estilo NIST)

> Cloud computing é um modelo que permite acesso **ubíquo, conveniente e sob demanda**, via rede, a um **pool compartilhado de recursos computacionais** (redes, servidores, armazenamento, aplicações e serviços), que podem ser rapidamente provisionados e liberados com mínimo esforço de gerenciamento.

### As 5 Características Essenciais

| Característica | O que significa | Exemplo prático |
|----------------|-----------------|-----------------|
| **On-demand self-service** | Você provisiona recursos sozinho, sem solicitar manualmente ao time de infra | Criar uma VM via portal/CLI/API |
| **Broad network access** | Acessível pela rede, de múltiplos dispositivos | Acessar Azure do PC, celular, tablet |
| **Resource pooling** | Infra compartilhada por vários clientes com isolamento lógico (multi-tenant) | Datacenter do Azure serve múltiplos clientes |
| **Rapid elasticity** | Escalar para cima/baixo rapidamente, muitas vezes de forma automática | VMSS aumenta VMs conforme carga |
| **Measured service** | Uso monitorado, medido e cobrado conforme consumo | Pay-as-you-go: paga pelo que usou |

### Benefícios Principais da Nuvem

| Benefício | Definição | Exemplo |
|-----------|-----------|---------|
| **Alta Disponibilidade** | Sistema projetado para ficar disponível a maior parte do tempo (SLA 99,9%+) | Availability Zones garantem 99,99% |
| **Escalabilidade** | Capacidade de aumentar ou diminuir recursos conforme demanda | Scale up: VM maior; Scale out: mais VMs |
| **Elasticidade** | Escalar automaticamente conforme carga, sem intervenção manual | Autoscale de VMSS, App Service, Functions |
| **Agilidade** | Criar e mudar ambientes rapidamente; experimentar, testar, falhar rápido | Criar ambiente de teste em minutos |
| **Resiliência / DR** | Replicar dados e workloads entre regiões, ter planos de failover | GRS, Cosmos DB Multi-Region |
| **Pay-as-you-go** | Paga só pelo que consome | Functions: paga por execução, não por hora |

### Modelos de Nuvem: Pública, Privada, Híbrida

| Modelo | Descrição | Exemplo |
|--------|-----------|---------|
| **Nuvem Pública** | Infra do provedor (Azure) compartilhada por vários clientes. Acesso via internet ou conexão privada | Empresa hospedando site no Azure |
| **Nuvem Privada** | Infra dedicada para uma única organização. Pode ser on-premises ou hospedada exclusivamente | Datacenter próprio; Azure Stack HCI |
| **Nuvem Híbrida** | Combina nuvem pública + privada/on-premises com integração | Parte da app on-prem, parte em Azure via VPN/ExpressRoute |

> 💡 **Dica AZ-900:** "Quer manter dados sensíveis on-premises, mas usar cloud para front-end web" → **Nuvem Híbrida**

---

## 16. Ferramentas de Custo

### Azure Cost Management + Billing

> Conjunto de ferramentas para **analisar, controlar e otimizar custos** dentro do portal Azure.

**Principais funções:**

| Função | Descrição |
|--------|-----------|
| **Ver custo por** | Subscription, Resource Group, Resource, Tag (department, project, costCenter) |
| **Budgets (orçamentos)** | Criar alertas: "Alertar se passar de R$ 1000 neste mês" |
| **Forecast** | Previsão de custo até o fim do mês |
| **Exportar dados** | Exportar para análise externa (Power BI, Excel, etc.) |

**Diferença para Azure Advisor:**

| Ferramenta | Foco |
|-----------|------|
| **Cost Management** | Mostra custos detalhados, relatórios, orçamentos |
| **Azure Advisor** | Dá recomendações sobre como reduzir custo/performance/HA |

---

### Calculadora de Preço (Pricing Calculator)

> **Onde fica:** site público da Microsoft — não precisa de conta Azure.  
> **Para que serve:** estimar o custo de serviços Azure **antes de criar** os recursos.

**Como usar mentalmente:**
- "Quanto custará 3 VMs D2s_v3 rodando 24/7 no Brazil South?" → Pricing Calculator
- "Quanto custa App Service S1 + SQL Database + Storage?" → Pricing Calculator

Você escolhe: serviços, região, quantidade, horas/mês → ela estima o custo mensal.

> 💡 **Dica AZ-900:** "Ferramenta para **estimar custo ANTES** de criar recursos" → **Pricing Calculator**

---

### Calculadora de TCO (Total Cost of Ownership)

> Ferramenta para comparar o **custo total de rodar workloads on-premises vs na nuvem (Azure)**.

**Você preenche:**
- Quantidade de servidores, storage, rede
- Licenças, custos de energia, espaço físico, equipe (modelo on-prem)

**A ferramenta estima economias em:**
- Hardware, energia, espaço de datacenter, manutenção, pessoal de operação

> 💡 **Dica AZ-900:** "Ferramenta para **comparar custo atual on-premises com Azure**" → **TCO Calculator**

**Comparação das 3 ferramentas:**

| Ferramenta | Quando usar | O que faz |
|-----------|-------------|-----------|
| **Pricing Calculator** | Antes de criar recursos | Estima custo de serviços Azure |
| **TCO Calculator** | Antes de migrar para nuvem | Compara on-prem vs Azure |
| **Cost Management** | Depois de criar recursos | Analisa e controla custos reais |

---

## 17. Service Health vs Resource Health

### Azure Service Health

> Foco: Status de **serviços/regiões Azure** que você usa.

**Mostra:**
- Incidentes em curso (ex: problema em "Storage – East US")
- Problemas passados e RCA (Root Cause Analysis)
- Manutenções planejadas (maintenance)
- Health Advisory (avisos de mudança de comportamento/obrigatoriedades)

**Escopo:** Serviços + Regiões (ex: "Azure SQL – Brazil South")

**Uso típico:** Saber se uma falha generalizada em um serviço/região está afetando múltiplos clientes.

---

### Resource Health

> Foco: Status de **recursos específicos** na sua subscription (uma VM, um App Service, um SQL Database específico).

**Mostra estados como:**
- Available (Healthy)
- Degraded
- Unavailable
- Unknown

**Ajuda a responder:** "Este recurso está ruim por causa de algum incidente de plataforma Azure, ou por causa de algo que EU fiz/configurei?"

---

### Comparação Rápida

| Característica | Service Health | Resource Health |
|----------------|---------------|-----------------|
| **Escopo** | Serviços/regiões Azure | Recursos individuais da sua subscription |
| **Foco** | Incidentes globais de plataforma | Saúde de recursos específicos |
| **Exemplo** | "Problema em Storage – East US" | "Minha VM X está Unavailable" |

> 💡 **Dica AZ-900:**
> - "Quer saber se um incidente em Azure SQL numa região afetou sua app?" → **Service Health**
> - "Quer saber se aquela VM específica caiu por causa da plataforma?" → **Resource Health**

---

## 18. Tipos de Armazenamento

### Blob Storage

- **Tipo:** Armazenamento de objetos (arquivos grandes ou pequenos)
- **Usos:** Imagens, vídeos, PDFs, backups, logs, data lake (com Hierarchical Namespace)
- **Acesso:** HTTP/HTTPS, SDKs, APIs REST
- **Ideal para:** Conteúdo estático (sites, mídia), Big Data
- **Palavra-chave:** objetos (**blob** = Binary Large Object)

---

### File Storage (Azure Files)

- **Tipo:** Compartilhamento de arquivos via SMB (como um file server)
- **Usos:** Compartilhamento de rede acessível por VMs Windows e Linux (e até on-prem). Migrar aplicações que esperam um `\servidorolder`
- **Acesso:** SMB (mount em Windows/Linux) e REST/API
- **Ideal para:** Migração de file servers; apps que esperam UNC path
- **Palavra-chave:** compartilhamento de arquivos tradicional (SMB)

---

### Disk Storage (Managed Disks)

- **Tipo:** Discos para VMs (C:\, D:\ da VM)
- **Usos:** Disco do SO (OS Disk), discos de dados adicionais (Data Disks)
- **Tipos:** HDD, Standard SSD, Premium SSD, Ultra Disk
- **Acesso:** Somente pela VM que monta o disco
- **Ideal para:** VMs que precisam de armazenamento persistente e de alta performance
- **Palavra-chave:** disco de VM (sempre via VM, não acesso direto por app)

---

### Queue Storage

- **Tipo:** Fila de mensagens simples (mensagens pequenas)
- **Usos:** Comunicação entre componentes de aplicação (produtor/consumidor), processamento assíncrono

**Diferença para Service Bus Queue:**

| Aspecto | Queue Storage | Service Bus Queue |
|---------|--------------|-------------------|
| Complexidade | Simples | Enterprise |
| Recursos avançados | ❌ | ✅ Dead-letter, sessions, transactions |
| Custo | Mais barato | Mais caro |
| Ideal para | Mensageria leve | Mensageria enterprise |

- **Palavra-chave:** fila simples para desacoplamento

---

### Comparação Geral dos Tipos de Storage

| Serviço | Tipo | Cenário típico | Acesso |
|---------|------|----------------|--------|
| **Blob Storage** | Objetos (arquivos) | Imagens, vídeos, backups, data lake | HTTP/HTTPS, API REST |
| **File Storage** | Compartilhamento SMB | `\servidorolder` para apps que esperam file share | SMB, REST |
| **Disk Storage** | Disco de VM | C:\ e D:\ das VMs | Somente via VM |
| **Queue Storage** | Fila de mensagens simples | Mensageria leve entre componentes | HTTP/HTTPS, SDK |

> 💡 **Dica AZ-900:**
> - "Armazenar imagens e vídeos para um site" → **Blob Storage**
> - "Migrar file server Windows para nuvem" → **File Storage (Azure Files)**
> - "Disco para uma VM" → **Disk Storage**
> - "Desacoplar componentes com fila simples" → **Queue Storage**

---

## 19. Pegadinhas Comuns

```

```
1. App Service ≠ Serverless
   App Service = você gerencia plano/capacidade
   Functions = Azure gerencia tudo

2. VMSS não distribui tráfego!
   VMSS + Load Balancer = solução completa

3. Criptografia de storage é OBRIGATÓRIA
   Não pode ser desabilitada

4. Cost Predictability = FORECAST futuro
   Não é redução, não é evitar surpresas

5. Container = empacota app + dependências
   VM = controle total do SO
   Functions = código sob evento

6. Management Groups SÃO nested (verticais)
   Subscriptions NÃO são nested (horizontais)

7. Subscriptions = isolamento COMPLETO
   Resource Groups = organização (não isolamento)

8. Container Instances paga por SEGUNDO
   VMs pagam por HORA
   App Service paga por MÊS (plano fixo)

9. RA-GRS = 99,99% (não 99,999%)
   99,999% = Cosmos DB Multi-Region Writes

10. Load Balancer Basic = 99,9%
    Load Balancer Standard = 99,99%

11. Azure Firewall ≠ apenas DDoS
    Firewall = NAT + Network + Application rules + FQDN filtering

12. Device type NÃO é condição de Conditional Access
    Use: device state/compliance

13. Management IN cloud = ferramentas (Portal, CLI, PowerShell)
    Management OF cloud = políticas, auto-scale, backups

14. Web apps por App Service Plan = ILIMITADO
    (limitado pela capacidade CPU/RAM, não por contagem)

15. Tags categorizam recursos (não automatizam ações)
    Tags não desligam VMs, não aplicam descontos

16. IaaS = você gerencia SO para cima
    PaaS = você gerencia apenas código e dados
    SaaS = você apenas consome a aplicação

17. Microsoft Purview = governança de DADOS (não de infra)
    Azure Policy = governança de RECURSOS (infra e configuração)

18. Resource Lock CanNotDelete ≠ ReadOnly
    CanNotDelete = pode modificar, não pode deletar
    ReadOnly = não pode modificar nem deletar

19. Azure Advisor = recomendações (não aplica automaticamente)
    Você precisa agir sobre as recomendações manualmente

20. Resource Health = saúde dos SEUS recursos
    Azure Status = saúde da plataforma Azure (global)

21. Nuvem Pública ≠ Nuvem Privada ≠ Nuvem Híbrida
    Pública = Azure compartilhado; Privada = exclusivo; Híbrida = combinação

22. Pricing Calculator = estimar custo ANTES de criar
    Cost Management = ver custo DEPOIS de criar

23. TCO Calculator = comparar on-prem vs Azure (economias)
    Pricing Calculator = estimar custo de serviços Azure

24. Blob = objetos (imagens, vídeos, backups)
    File = compartilhamento SMB (\servidorolder)
    Disk = disco de VM (C:\, D:\)
    Queue = fila simples (desacoplamento de componentes)

25. Service Health = incidente geral de plataforma/região
    Resource Health = saúde do SEU recurso específico
```

---

## 📊 SLAs para Memorizar

```
99,9%   → VM única, LRS, GRS, VPN Gateway, Entra ID, Load Balancer Basic
99,95%  → Availability Set, ZRS, App Service, App Gateway, AKS
99,99%  → Availability Zone, RA-GRS, Load Balancer Standard, SQL Database, Cosmos DB
99,999% → Cosmos DB Multi-Region Writes
```
```
---

## 📝 Checklist Final

- [ ] Sei a diferença entre Região, AZ e Availability Set
- [ ] Conheço os SLAs principais (99,9% / 99,95% / 99,99% / 99,999%)
- [ ] Sei quando usar Application Gateway (WAF) vs Load Balancer
- [ ] Entendo NSG vs Azure Firewall vs WAF
- [ ] Sei que VMSS precisa de Load Balancer separado
- [ ] Conditional Access = contexto/risco; PIM = just-in-time
- [ ] Log Analytics = "balde" central de logs
- [ ] CapEx = gasto inicial; OpEx = gasto mensal deduzível
- [ ] VMSS = máximo 1.000 instâncias
- [ ] Management Groups são nested (verticais)
- [ ] Subscriptions = isolamento; RGs = organização
- [ ] Container Instances paga por segundo
- [ ] RA-GRS = 99,99%; Cosmos Multi-Region = 99,999%
- [ ] App Service NÃO é serverless
- [ ] Tags categorizam, não automatizam
- [ ] Azure Firewall = NAT + Network + Application + FQDN
- [ ] B2B = parceiros; B2C = consumidores
- [ ] Defender for Cloud ≠ password management
- [ ] IaaS / PaaS / SaaS = divisão de responsabilidade (você vs Azure)
- [ ] Serverless = event-driven, scale-to-zero, paga por execução
- [ ] Microsoft Purview = governança de dados (catálogo, classificação, linhagem)
- [ ] Azure Policy = conformidade de recursos (Deny, Audit, Append, DeployIfNotExists)
- [ ] Resource Lock = protege contra deleção/modificação acidental
- [ ] Azure Advisor = recomendações de custo, performance, HA, segurança
- [ ] Resource Health = saúde dos meus recursos (vs Azure Status = global)
- [ ] Azure Monitor = Metrics + Logs + Alerts + App Insights + Log Analytics
- [ ] Sei os 5 características de cloud (on-demand, broad access, pooling, elasticity, measured)
- [ ] Conheço os 3 modelos de nuvem (Pública, Privada, Híbrida)
- [ ] Sei quando usar Pricing Calculator vs TCO Calculator vs Cost Management
- [ ] Sei a diferença entre Blob, File, Disk e Queue Storage
- [ ] Service Health = plataforma global; Resource Health = meu recurso

---

*Material gerado com auxílio do GitHub Copilot para estudo da certificação AZ-900*  
*Microsoft Azure Fundamentals*
