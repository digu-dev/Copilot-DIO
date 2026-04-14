## Índice

1. Introdução
2. Principais Características da Nuvem
3. Serviços de Nuvem
4. Considerações para Migrar
5. Cenários de Uso
6. Tipos de Nuvem
7. Azure
8. Segurança na Nuvem
9. Compliance
10. Estratégias de Custo
11. TCO e ROI
12. VPN e Conectividade
13. Arquitetura de Aplicações
14. DevOps na Nuvem
15. Futuro da Nuvem
16. Conceitos Básicos de Cloud Computing
17. Ferramentas de Custo
18. Tipos de Armazenamento

## Pegadinhas Comuns

1. Nuvem Pública ≠ Nuvem Privada ≠ Nuvem Híbrida
   Pública = Azure compartilhado; Privada = exclusivo; Híbrida = combinação
2. Pricing Calculator = estimar custo ANTES de criar
   Cost Management = ver custo DEPOIS de criar
3. TCO Calculator = comparar on-prem vs Azure (economias)
   Pricing Calculator = estimar custo de serviços Azure
4. Blob = objetos (imagens, vídeos, backups)
   File = compartilhamento SMB (\\servidor\pasta)
   Disk = disco de VM (C:\, D:\)
   Queue = fila simples (desacoplamento de componentes)
5. Service Health = incidente geral de plataforma/região
   Resource Health = saúde do SEU recurso específico

## Checklist Final

- [ ] Sei os 5 características de cloud (on-demand, broad access, pooling, elasticity, measured)
- [ ] Conheço os 3 modelos de nuvem (Pública, Privada, Híbrida)
- [ ] Sei quando usar Pricing Calculator vs TCO Calculator vs Cost Management
- [ ] Sei a diferença entre Blob, File, Disk e Queue Storage
- [ ] Service Health = plataforma global; Resource Health = meu recurso

---

## 16. Conceitos Básicos de Cloud Computing

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

## 17. Ferramentas de Custo

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
- **Usos:** Compartilhamento de rede acessível por VMs Windows e Linux (e até on-prem). Migrar aplicações que esperam um `\\servidor\pasta`  
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
- **Diferença para Service Bus Queue:**  

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
| **File Storage** | Compartilhamento SMB | `\\servidor\pasta` para apps que esperam file share | SMB, REST |
| **Disk Storage** | Disco de VM | C:\ e D:\ das VMs | Somente via VM |
| **Queue Storage** | Fila de mensagens simples | Mensageria leve entre componentes | HTTP/HTTPS, SDK |

> 💡 **Dica AZ-900:**
> - "Armazenar imagens e vídeos para um site" → **Blob Storage**
> - "Migrar file server Windows para nuvem" → **File Storage (Azure Files)**
> - "Disco para uma VM" → **Disk Storage**
> - "Desacoplar componentes com fila simples" → **Queue Storage**





