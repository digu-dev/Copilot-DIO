# 📚 Material de Estudo - Certificação AZ-900

> Material completo gerado a partir de sessão de estudos focada nos principais tópicos da certificação Microsoft Azure Fundamentals (AZ-900)

---

## 📋 Índice

1. [Infraestrutura Global e Alta Disponibilidade](#1-infraestrutura-global-e-alta-disponibilidade)
2. [SLAs dos Principais Serviços](#2-slas-dos-principais-serviços)
3. [Segurança de Rede](#3-segurança-de-rede)
4. [VPN Gateway e ExpressRoute](#4-vpn-gateway-e-expressroute)
5. [Serviços Serverless](#5-serviços-serverless)
6. [Azure App Service Plans](#6-azure-app-service-plans)
7. [Máquinas Virtuais e VMSS](#7-máquinas-virtuais-e-vmss)
8. [Azure App Service e Container Apps](#8-azure-app-service-e-container-apps)
9. [Container Instances e AKS](#9-container-instances-e-aks)
10. [Azure Functions e Logic Apps](#10-azure-functions-e-logic-apps)
11. [Redes no Azure](#11-redes-no-azure)
12. [Segurança e Identidade](#12-segurança-e-identidade)
13. [Pegadinhas Comuns](#13-pegadinhas-comuns)

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

- �� SQL Injection
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

## 13. Pegadinhas Comuns

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
```

---

## 📊 SLAs para Memorizar

```
99,9%   → VM única, LRS, GRS, VPN Gateway, Entra ID, Load Balancer Basic
99,95%  → Availability Set, ZRS, App Service, App Gateway, AKS
99,99%  → Availability Zone, RA-GRS, Load Balancer Standard, SQL Database, Cosmos DB
99,999% → Cosmos DB Multi-Region Writes
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

---

*Material gerado com auxílio do GitHub Copilot para estudo da certificação AZ-900*  
*Microsoft Azure Fundamentals*