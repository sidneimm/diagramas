# Requisitos para Prova de Conceito - IDM Data Replication DB2 para DB2

## Visão Geral
Este documento descreve os requisitos de infraestrutura e configuração necessários para executar uma Prova de Conceito (PoC) de IBM InfoSphere Data Replication (IDM) para replicação de dados entre bancos de dados DB2.

## Arquitetura da Solução

A solução requer três componentes principais:
1. **Management Console** - Interface de gerenciamento (Windows)
2. **Access Server** - Servidor de acesso e controle (Linux Red Hat)
3. **Agentes de Capture e Apply** - Componentes de captura e aplicação de dados (Linux Red Hat)

---

## 1. Requisitos da Máquina Windows (Management Console)

### 1.1 Especificações de Hardware
- **CPU**: Mínimo 4 cores (recomendado 8 cores)
- **Memória RAM**: Mínimo 8 GB (recomendado 16 GB)
- **Disco**: 
  - 100 GB de espaço livre para instalação
  - Disco adicional para logs e backups (recomendado 50 GB)
- **Rede**: Interface de rede Gigabit Ethernet

### 1.2 Sistema Operacional
- **Windows Server 2016** ou superior (64-bit)
- **Windows 10/11 Professional** ou Enterprise (64-bit) - alternativa para ambiente de teste
- Todas as atualizações críticas e service packs instalados

### 1.3 Software Pré-requisitos
- **.NET Framework**: Versão 4.7.2 ou superior
- **Java Runtime Environment (JRE)**: Versão 8 ou 11 (64-bit)
- **Navegador Web**: Chrome, Firefox ou Edge (versões atualizadas)
- **Microsoft Visual C++ Redistributable**: Versões 2015-2019

### 1.4 Configurações de Rede
- **Portas TCP/IP** que devem estar abertas:
  - Porta 10101 (comunicação com Access Server)
  - Porta 11701 (Management Console)
  - Portas customizadas conforme configuração
- **Resolução de nomes**: DNS configurado ou arquivo hosts atualizado
- **Conectividade**: Acesso de rede ao servidor Linux Red Hat

### 1.5 Permissões e Segurança
- Usuário com privilégios de **Administrador Local**
- **Firewall do Windows**: Configurado para permitir tráfego nas portas necessárias
- **Antivírus**: Exceções configuradas para diretórios de instalação do IDM
- **User Account Control (UAC)**: Configurado adequadamente

---

## 2. Requisitos da Máquina Linux Red Hat (Access Server e Agentes)

### 2.1 Especificações de Hardware
- **CPU**: Mínimo 8 cores (recomendado 16 cores)
- **Memória RAM**: Mínimo 16 GB (recomendado 32 GB)
- **Disco**:
  - Sistema operacional: 50 GB
  - Instalação IDM: 100 GB
  - Staging area: 200 GB (dependendo do volume de dados)
  - Logs: 50 GB
  - Total recomendado: 400 GB
- **Rede**: Interface de rede Gigabit Ethernet (recomendado 10 Gbps)

### 2.2 Sistema Operacional
- **Red Hat Enterprise Linux (RHEL)**: Versão 7.x ou 8.x (64-bit)
- Kernel atualizado com últimos patches de segurança
- Subscrição Red Hat ativa para acesso a repositórios

### 2.3 Pacotes e Bibliotecas Necessárias
```bash
# Pacotes essenciais
- glibc (GNU C Library)
- libstdc++
- ksh (Korn Shell)
- libaio
- numactl
- pam
- compat-libstdc++

# Ferramentas de desenvolvimento (se necessário)
- gcc
- make
- kernel-devel
```

### 2.4 Configurações do Sistema Operacional

#### 2.4.1 Limites do Sistema (ulimits)
Adicionar ao arquivo `/etc/security/limits.conf`:
```
idmuser soft nofile 65536
idmuser hard nofile 65536
idmuser soft nproc 16384
idmuser hard nproc 16384
idmuser soft stack 10240
idmuser hard stack 32768
```

#### 2.4.2 Parâmetros do Kernel
Adicionar ao arquivo `/etc/sysctl.conf`:
```
kernel.sem = 250 256000 32 1024
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
kernel.shmmni = 4096
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```

### 2.5 Usuários e Grupos
- Criar usuário dedicado para IDM (ex: `idmuser`)
- Criar grupo dedicado (ex: `idmgroup`)
- Usuário deve ter permissões sudo para instalação
- Após instalação, usuário deve ter acesso aos diretórios:
  - `/opt/ibm/InfoSphereDataReplication` (ou diretório de instalação)
  - Diretórios de staging e logs

### 2.6 Configurações de Rede
- **Portas TCP/IP** que devem estar abertas:
  - Porta 10101 (Access Server)
  - Porta 11001 (Capture Agent)
  - Porta 11101 (Apply Agent)
  - Portas customizadas conforme configuração
- **Hostname**: Nome de host totalmente qualificado (FQDN) configurado
- **Resolução de nomes**: DNS configurado ou `/etc/hosts` atualizado
- **Firewall (firewalld/iptables)**: Regras configuradas para permitir tráfego

### 2.7 Segurança
- **SELinux**: Configurado em modo permissive ou com políticas adequadas
- **Firewall**: Configurado para permitir comunicação entre componentes
- **SSH**: Acesso configurado para administração remota
- **Certificados SSL/TLS**: Se necessário para comunicação segura

---

## 3. Requisitos do Banco de Dados DB2

### 3.1 Banco de Dados Origem (Source)
- **Versão DB2**: 10.5 ou superior (recomendado 11.5)
- **Licença**: DB2 Enterprise Edition ou Advanced Edition
- **Configuração**:
  - Log archiving habilitado (LOGARCHMETH1)
  - Retenção de logs adequada (LOGRETAIN = RECOVERY)
  - Espaço suficiente para logs ativos e arquivados
  - User Exits configurados (se necessário)

### 3.2 Banco de Dados Destino (Target)
- **Versão DB2**: 10.5 ou superior (recomendado 11.5)
- **Licença**: DB2 Enterprise Edition ou Advanced Edition
- **Configuração**:
  - Tablespaces criados com espaço suficiente
  - Esquema de destino preparado
  - Permissões adequadas para usuário de replicação

### 3.3 Usuários e Permissões DB2

#### Usuário para Capture (Source)
```sql
-- Permissões mínimas necessárias
GRANT CONNECT ON DATABASE TO idmcapture;
GRANT DATAACCESS ON DATABASE TO idmcapture;
GRANT DBADM ON DATABASE TO idmcapture;
-- Ou permissões específicas:
GRANT SELECT ON SCHEMA source_schema TO idmcapture;
GRANT SELECT ON SYSCAT.TABLES TO idmcapture;
GRANT SELECT ON SYSCAT.COLUMNS TO idmcapture;
```

#### Usuário para Apply (Target)
```sql
-- Permissões mínimas necessárias
GRANT CONNECT ON DATABASE TO idmapply;
GRANT DATAACCESS ON DATABASE TO idmapply;
GRANT DBADM ON DATABASE TO idmapply;
-- Ou permissões específicas:
GRANT INSERT, UPDATE, DELETE ON SCHEMA target_schema TO idmapply;
GRANT CREATE TABLE ON SCHEMA target_schema TO idmapply;
```

### 3.4 Conectividade
- **Catalogação**: Bancos de dados catalogados no servidor Linux
- **Teste de conexão**: Validar conectividade usando `db2 connect to <database>`
- **Latência de rede**: Baixa latência entre servidores DB2 e servidor IDM

---

## 4. Requisitos de Software IBM

### 4.1 Licenças Necessárias
- IBM InfoSphere Data Replication (versão 11.4 ou superior)
- Licenças para número de cores/processadores
- Entitlement para download do software

### 4.2 Mídia de Instalação
- **Management Console**: Instalador para Windows
- **Access Server**: Instalador para Linux
- **Agentes CDC**: Instaladores para DB2 (Capture e Apply)
- **Documentação**: Guias de instalação e configuração

### 4.3 IBM Fix Central
- Acesso ao IBM Fix Central para download de patches
- Fix packs mais recentes aplicados

---

## 5. Requisitos de Rede e Conectividade

### 5.1 Topologia de Rede
```
[Management Console - Windows]
         |
         | (Porta 10101)
         |
    [Access Server - Linux RHEL]
         |
         +-- [Capture Agent] --> [DB2 Source]
         |
         +-- [Apply Agent] --> [DB2 Target]
```

### 5.2 Requisitos de Largura de Banda
- **Mínimo**: 100 Mbps
- **Recomendado**: 1 Gbps ou superior
- **Latência**: < 10ms entre componentes

### 5.3 Resolução de Nomes
Todos os servidores devem resolver nomes entre si:
- Management Console deve resolver Access Server
- Access Server deve resolver ambos os DB2
- Configurar DNS ou arquivos hosts (`/etc/hosts` e `C:\Windows\System32\drivers\etc\hosts`)

---

## 6. Requisitos de Backup e Recuperação

### 6.1 Backup do Ambiente
- **Snapshots de VM**: Se ambiente virtualizado
- **Backup de configurações**: Arquivos de configuração do IDM
- **Backup de metadados**: Definições de subscrições e mapeamentos

### 6.2 Plano de Rollback
- Procedimento documentado para reverter instalação
- Backup dos bancos de dados antes de iniciar replicação
- Ponto de restauração do sistema operacional

---

## 7. Documentação e Acesso

### 7.1 Informações a Fornecer
- **Endereços IP** de todos os servidores
- **Hostnames** e FQDNs
- **Credenciais**:
  - Usuários administrativos (Windows e Linux)
  - Usuários DB2 (source e target)
  - Usuários IDM
- **Detalhes dos bancos de dados**:
  - Nomes dos databases
  - Schemas a serem replicados
  - Tabelas específicas (se aplicável)
  - Portas de conexão DB2

### 7.2 Acesso Remoto
- **VPN**: Acesso VPN configurado (se necessário)
- **RDP**: Acesso Remote Desktop ao Windows
- **SSH**: Acesso SSH ao Linux
- **Credenciais**: Fornecidas com antecedência

---

## 8. Cronograma e Planejamento

### 8.1 Fases da PoC
1. **Preparação do ambiente** (2-3 dias)
   - Provisionamento de servidores
   - Instalação de pré-requisitos
   - Configuração de rede

2. **Instalação do IDM** (1-2 dias)
   - Instalação do Management Console
   - Instalação do Access Server
   - Instalação dos agentes CDC

3. **Configuração e testes** (2-3 dias)
   - Configuração de subscrições
   - Mapeamento de tabelas
   - Testes de replicação inicial
   - Testes de replicação contínua

4. **Validação e documentação** (1 dia)
   - Validação de dados
   - Testes de performance
   - Documentação de resultados

### 8.2 Janela de Manutenção
- Definir janela de manutenção para instalação e testes
- Coordenar com equipes de banco de dados e infraestrutura

---

## 9. Contatos e Responsabilidades

### 9.1 Equipe do Cliente
- **Gerente de Projeto**: [Nome e contato]
- **Administrador Windows**: [Nome e contato]
- **Administrador Linux**: [Nome e contato]
- **DBA DB2**: [Nome e contato]
- **Administrador de Rede**: [Nome e contato]

### 9.2 Equipe de Implementação
- **Consultor IDM**: [Nome e contato]
- **Suporte IBM**: [Caso necessário]

---

## 10. Checklist de Pré-requisitos

### Máquina Windows
- [ ] Hardware conforme especificações
- [ ] Windows Server instalado e atualizado
- [ ] .NET Framework instalado
- [ ] Java JRE instalado
- [ ] Portas de firewall abertas
- [ ] Usuário administrativo criado
- [ ] Conectividade de rede validada

### Máquina Linux Red Hat
- [ ] Hardware conforme especificações
- [ ] RHEL instalado e atualizado
- [ ] Pacotes necessários instalados
- [ ] Limites do sistema configurados
- [ ] Parâmetros do kernel ajustados
- [ ] Usuário IDM criado
- [ ] Firewall configurado
- [ ] SELinux configurado
- [ ] Conectividade de rede validada

### Banco de Dados DB2
- [ ] DB2 instalado (source e target)
- [ ] Log archiving habilitado (source)
- [ ] Usuários de replicação criados
- [ ] Permissões concedidas
- [ ] Bancos catalogados
- [ ] Conectividade testada
- [ ] Espaço em disco suficiente

### Rede e Conectividade
- [ ] Resolução de nomes configurada
- [ ] Portas abertas entre componentes
- [ ] Latência de rede aceitável
- [ ] Largura de banda adequada

### Software e Licenças
- [ ] Licenças IBM disponíveis
- [ ] Mídia de instalação obtida
- [ ] Acesso ao Fix Central
- [ ] Documentação disponível

### Acesso e Documentação
- [ ] Credenciais fornecidas
- [ ] Acesso remoto configurado
- [ ] Documentação de ambiente entregue
- [ ] Contatos definidos

---

## 11. Observações Importantes

1. **Ambiente de Produção vs. Teste**: Esta PoC deve ser executada em ambiente de **não-produção** ou com dados de teste.

2. **Impacto no Source**: A captura de dados pode ter impacto mínimo no desempenho do banco de dados origem. Monitorar durante a PoC.

3. **Espaço em Disco**: O espaço para staging deve ser dimensionado com base no volume de transações esperado.

4. **Sincronização Inicial**: A carga inicial (refresh) pode levar tempo dependendo do volume de dados.

5. **Monitoramento**: Preparar ferramentas de monitoramento para acompanhar a replicação (Management Console fornece dashboards).

6. **Suporte IBM**: Ter contrato de suporte ativo para abertura de casos se necessário.

---

## 12. Próximos Passos

Após confirmação de que todos os requisitos foram atendidos:

1. Agendar kickoff da PoC
2. Validar acesso a todos os ambientes
3. Iniciar instalação conforme cronograma
4. Executar testes planejados
5. Documentar resultados e lições aprendidas

---

**Documento preparado em**: 05/06/2026  
**Versão**: 1.0  
**Contato para dúvidas**: [Seu nome e contato]
