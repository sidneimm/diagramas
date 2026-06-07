# Documentação IBM InfoSphere Data Replication (CDC)

Este repositório contém documentação técnica completa para implementação de Prova de Conceito (PoC) de IBM InfoSphere Data Replication utilizando tecnologia Change Data Capture (CDC).

## 📋 Conteúdo

### Requisitos de Infraestrutura

1. **[requisitos-poc-idm-data-replication.md](requisitos-poc-idm-data-replication.md)**
   - Requisitos completos e detalhados para PoC
   - Especificações de hardware e software
   - Configurações de rede e segurança
   - Cronograma e planejamento

2. **[requisitos-infraestrutura-cliente-idm.md](requisitos-infraestrutura-cliente-idm.md)**
   - Versão focada nos requisitos que o cliente deve providenciar
   - Checklist de validação
   - Informações de contato e aprovação

### Diagramas de Arquitetura

3. **[diagrama-arquitetura-idm-cdc.txt](diagrama-arquitetura-idm-cdc.txt)**
   - Diagrama ASCII art da arquitetura original
   - 3 máquinas: Windows (Management Console), Linux (Access Server + Agentes), DB2

4. **[diagrama-arquitetura-idm-cdc-v2.svg](diagrama-arquitetura-idm-cdc-v2.svg)**
   - Diagrama SVG profissional da arquitetura simplificada
   - 2 máquinas: Windows (Management Console + Access Server), Servidores de BD com agentes

5. **[diagrama-arquitetura-idm-cdc-simplificada.txt](diagrama-arquitetura-idm-cdc-simplificada.txt)**
   - Diagrama ASCII art detalhado da arquitetura simplificada
   - Inclui fluxo de dados, requisitos e checklist de implementação

### Documentação Técnica

6. **[portas-agentes-cdc-por-banco.txt](portas-agentes-cdc-por-banco.txt)**
   - Portas padrão dos agentes CDC por tipo de banco de dados
   - DB2, PostgreSQL e MySQL
   - Configuração de firewall e verificação de conectividade
   - Criação de usuários cdcuser para instalação dos agentes

### E-mail e Comunicação

7. **[email-requisitos-poc-idm.txt](email-requisitos-poc-idm.txt)**
   - Versão formatada para e-mail dos requisitos
   - Pronta para copiar e colar no Outlook

## 🏗️ Arquitetura da Solução

### Arquitetura Simplificada (Recomendada)

```
┌─────────────────────────────────────┐
│   Windows Server                    │
│   • Management Console (11701)      │
│   • Access Server (10101)           │
└─────────────────────────────────────┘
            │                │
            ▼                ▼
┌──────────────────┐  ┌──────────────────┐
│  Servidor SOURCE │  │  Servidor TARGET │
│  • Capture Agent │  │  • Apply Agent   │
│  • DB2/PG/MySQL  │  │  • DB2/PG/MySQL  │
└──────────────────┘  └──────────────────┘
```

**Total: 3 máquinas**

## 🔌 Portas Utilizadas

### Componentes de Gerenciamento
- **Management Console**: 11701 (HTTPS)
- **Access Server**: 10101 (TCP)

### Agentes CDC por Banco de Dados

| Banco de Dados | Capture Agent | Apply Agent | Dados CDC |
|----------------|---------------|-------------|-----------|
| DB2            | 11001         | 11101       | 10001     |
| PostgreSQL     | 11002         | 11102       | 10002     |
| MySQL          | 11003         | 11103       | 10003     |

## 👤 Usuários Necessários

### Sistema Operacional
- **cdcuser**: Usuário dedicado para executar agentes CDC
- **cdcgroup**: Grupo dedicado

### Banco de Dados
- **cdccapture**: Usuário para Capture Agent (source)
- **cdcapply**: Usuário para Apply Agent (target)

## 📦 Requisitos Mínimos

### Máquina Windows (Management Console + Access Server)
- Windows Server 2016+ (64-bit)
- 16 GB RAM (recomendado 32 GB)
- 8 cores (recomendado 16 cores)
- 200 GB disco livre
- .NET Framework 4.7.2+
- Java JRE 8 ou 11 (64-bit)

### Servidores de Banco de Dados (Source e Target)
- Linux/Windows
- 16 GB RAM (recomendado 32 GB)
- 8 cores (recomendado 16 cores)
- 300 GB disco livre
- DB2 10.5+ / PostgreSQL 9.6+ / MySQL 5.7+

## 🌐 Requisitos de Rede
- Largura de banda: Mínimo 100 Mbps (recomendado 1 Gbps)
- Latência: < 10ms entre componentes
- Resolução de nomes (DNS ou hosts)
- Portas de firewall abertas conforme documentação

## 📚 Tipos de Replicação

1. **REFRESH**: Carga inicial completa
2. **MIRROR**: Replicação contínua em tempo real (CDC)
3. **REFRESH + MIRROR**: Híbrido (mais comum em produção)

## 🚀 Início Rápido

1. Revisar requisitos de infraestrutura
2. Provisionar máquinas conforme especificações
3. Instalar e configurar bancos de dados
4. Criar usuários cdcuser nos servidores
5. Instalar Management Console e Access Server (Windows)
6. Instalar Capture Agent (servidor source)
7. Instalar Apply Agent (servidor target)
8. Configurar subscrições via Management Console
9. Executar testes de replicação

## 📖 Documentação Adicional

- [IBM Knowledge Center - InfoSphere Data Replication](https://www.ibm.com/docs/en/idr)
- [IBM Support](https://www.ibm.com/support)

## 📝 Notas

- Esta documentação foi criada para PoC (Prova de Conceito)
- Ambiente de não-produção recomendado para testes
- Realizar backup dos bancos de dados antes de iniciar
- Monitoramento contínuo durante a PoC

## 📧 Contato

Para dúvidas ou suporte, consulte a documentação IBM ou abra um caso no IBM Support Portal.

---

**Última atualização**: Junho 2026  
**Versão**: 1.0