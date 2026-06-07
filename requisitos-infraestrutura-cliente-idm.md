# Requisitos de Infraestrutura - PoC IDM Data Replication DB2

## Objetivo
Este documento especifica os requisitos de infraestrutura que o **cliente deve providenciar** para execução da Prova de Conceito (PoC) de IBM InfoSphere Data Replication entre bancos de dados DB2.

---

## 1. MÁQUINA WINDOWS (Management Console)

### Especificações Mínimas
| Item | Requisito |
|------|-----------|
| **Sistema Operacional** | Windows Server 2016 ou superior (64-bit) |
| **CPU** | 4 cores (recomendado 8 cores) |
| **Memória RAM** | 8 GB (recomendado 16 GB) |
| **Disco** | 150 GB livres |
| **Rede** | Gigabit Ethernet |

### Software Pré-instalado
- ✅ .NET Framework 4.7.2 ou superior
- ✅ Java Runtime Environment (JRE) 8 ou 11 (64-bit)
- ✅ Microsoft Visual C++ Redistributable 2015-2019
- ✅ Navegador web atualizado (Chrome, Firefox ou Edge)

### Configurações Necessárias
- ✅ Usuário com privilégios de **Administrador Local**
- ✅ **Firewall configurado** para permitir:
  - Porta TCP 10101 (comunicação com Access Server)
  - Porta TCP 11701 (Management Console)
- ✅ **Antivírus**: Exceções para diretórios de instalação do IDM
- ✅ **Conectividade de rede** com o servidor Linux Red Hat

---

## 2. MÁQUINA LINUX RED HAT (Access Server e Agentes CDC)

### Especificações Mínimas
| Item | Requisito |
|------|-----------|
| **Sistema Operacional** | Red Hat Enterprise Linux 7.x ou 8.x (64-bit) |
| **CPU** | 8 cores (recomendado 16 cores) |
| **Memória RAM** | 16 GB (recomendado 32 GB) |
| **Disco** | 400 GB livres (distribuídos conforme abaixo) |
| **Rede** | Gigabit Ethernet (recomendado 10 Gbps) |

### Distribuição de Disco
- Sistema operacional: 50 GB
- Instalação IDM: 100 GB
- Staging area: 200 GB
- Logs: 50 GB

### Pacotes Linux Necessários
```bash
glibc
libstdc++
ksh (Korn Shell)
libaio
numactl
pam
compat-libstdc++
```

### Configurações do Sistema Operacional

#### Limites do Sistema (ulimits)
Configurar no arquivo `/etc/security/limits.conf`:
```
cdcuser soft nofile 65536
cdcuser hard nofile 65536
cdcuser soft nproc 16384
cdcuser hard nproc 16384
cdcuser soft stack 10240
cdcuser hard stack 32768
```

#### Parâmetros do Kernel
Configurar no arquivo `/etc/sysctl.conf`:
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

### Usuário e Permissões
- ✅ Criar usuário dedicado: **cdcuser**
- ✅ Criar grupo dedicado: **cdcgroup**
- ✅ Usuário deve ter permissões **sudo** para instalação
- ✅ Após instalação, usuário deve ter acesso total aos diretórios de instalação

### Configurações de Rede e Segurança
- ✅ **Hostname** totalmente qualificado (FQDN) configurado
- ✅ **Firewall (firewalld/iptables)** configurado para permitir:
  - Porta TCP 10101 (Access Server)
  - Porta TCP 11001 (Capture Agent)
  - Porta TCP 11101 (Apply Agent)
- ✅ **SELinux**: Configurado em modo permissive ou com políticas adequadas
- ✅ **SSH**: Acesso habilitado para administração remota

---

## 3. BANCO DE DADOS DB2 (Origem e Destino)

### Versão e Licenciamento
- ✅ **DB2 versão 10.5 ou superior** (recomendado 11.5)
- ✅ **Licença**: DB2 Enterprise Edition ou Advanced Edition
- ✅ Ambos os bancos (origem e destino) devem estar **instalados e operacionais**

### Configurações do DB2 Origem (Source)
- ✅ **Log archiving habilitado**:
  ```sql
  UPDATE DB CFG FOR <dbname> USING LOGARCHMETH1 DISK:/path/to/archive
  UPDATE DB CFG FOR <dbname> USING LOGRETAIN RECOVERY
  ```
- ✅ **Espaço suficiente** para logs ativos e arquivados
- ✅ **Banco de dados ativo** e acessível

### Configurações do DB2 Destino (Target)
- ✅ **Tablespaces criados** com espaço suficiente
- ✅ **Esquema de destino** preparado (ou permissão para criar)
- ✅ **Banco de dados ativo** e acessível

### Usuários DB2 com Permissões

#### Usuário para Capture (Source Database)
```sql
-- Criar usuário
CREATE USER cdccapture;

-- Conceder permissões
GRANT CONNECT ON DATABASE TO cdccapture;
GRANT DATAACCESS ON DATABASE TO cdccapture;
GRANT DBADM ON DATABASE TO cdccapture;
```

#### Usuário para Apply (Target Database)
```sql
-- Criar usuário
CREATE USER cdcapply;

-- Conceder permissões
GRANT CONNECT ON DATABASE TO cdcapply;
GRANT DATAACCESS ON DATABASE TO cdcapply;
GRANT DBADM ON DATABASE TO cdcapply;
```

### Catalogação de Bancos
- ✅ Bancos de dados devem estar **catalogados** no servidor Linux Red Hat
- ✅ Teste de conectividade validado: `db2 connect to <database>`

---

## 4. REDE E CONECTIVIDADE

### Topologia Requerida
```
[Windows - Management Console]
         |
         | Porta 10101
         |
    [Linux RHEL - Access Server]
         |
         +-- [Capture Agent] --> [DB2 Source]
         |
         +-- [Apply Agent] --> [DB2 Target]
```

### Requisitos de Rede
- ✅ **Largura de banda**: Mínimo 100 Mbps (recomendado 1 Gbps)
- ✅ **Latência**: Menor que 10ms entre componentes
- ✅ **Conectividade**: Todos os servidores devem se comunicar entre si

### Resolução de Nomes (DNS ou Hosts)
Todos os servidores devem resolver nomes entre si:
- ✅ Management Console → Access Server
- ✅ Access Server → DB2 Source
- ✅ Access Server → DB2 Target

**Configurar DNS ou arquivos hosts:**
- Windows: `C:\Windows\System32\drivers\etc\hosts`
- Linux: `/etc/hosts`

### Portas que Devem Estar Abertas

| Origem | Destino | Porta | Protocolo | Finalidade |
|--------|---------|-------|-----------|------------|
| Management Console | Access Server | 10101 | TCP | Gerenciamento |
| Management Console | Access Server | 11701 | TCP | Console Web |
| Access Server | DB2 Source | 50000* | TCP | Conexão DB2 |
| Access Server | DB2 Target | 50000* | TCP | Conexão DB2 |
| Access Server | Capture Agent | 11001 | TCP | Controle Capture |
| Access Server | Apply Agent | 11101 | TCP | Controle Apply |

*Porta padrão DB2, pode variar conforme instalação

---

## 5. ACESSOS NECESSÁRIOS

### Credenciais a Fornecer

#### Máquina Windows
- Usuário e senha com privilégios de Administrador
- Acesso via Remote Desktop (RDP)

#### Máquina Linux Red Hat
- Usuário e senha com privilégios sudo
- Acesso via SSH

#### Banco de Dados DB2 (Source)
- Nome do database
- Hostname/IP e porta
- Usuário: cdccapture
- Senha do usuário
- Schema(s) a serem replicados

#### Banco de Dados DB2 (Target)
- Nome do database
- Hostname/IP e porta
- Usuário: cdcapply
- Senha do usuário
- Schema(s) de destino

### Acesso Remoto
- ✅ **VPN** configurada (se necessário)
- ✅ **RDP** habilitado para Windows
- ✅ **SSH** habilitado para Linux
- ✅ Credenciais fornecidas com **antecedência**

---

## 6. INFORMAÇÕES DO AMBIENTE

### Documentação a Fornecer
- ✅ **Diagrama de rede** (se disponível)
- ✅ **Endereços IP** de todos os servidores
- ✅ **Hostnames** e FQDNs
- ✅ **Detalhes dos bancos de dados**:
  - Nomes dos databases
  - Versões do DB2
  - Schemas existentes
  - Tabelas a serem replicadas (se já definidas)
  - Volume estimado de dados
  - Taxa de transações (aproximada)

---

## 7. JANELA DE MANUTENÇÃO

### Disponibilidade Necessária
- ✅ **Janela de manutenção** de pelo menos **8 horas** para:
  - Instalação dos componentes
  - Configuração inicial
  - Testes de conectividade
- ✅ **Período de testes** de **3-5 dias** para:
  - Configuração de subscrições
  - Replicação inicial (refresh)
  - Testes de replicação contínua
  - Validação de dados

### Coordenação com Equipes
- ✅ Equipe de **Infraestrutura** disponível
- ✅ Equipe de **Banco de Dados** disponível
- ✅ Equipe de **Rede** disponível (se necessário)

---

## 8. CHECKLIST DE VALIDAÇÃO

### ☐ Máquina Windows Preparada
- [ ] Hardware conforme especificações
- [ ] Windows Server instalado e atualizado
- [ ] Software pré-requisito instalado (.NET, Java, VC++)
- [ ] Firewall configurado (portas abertas)
- [ ] Usuário administrativo criado
- [ ] Conectividade de rede validada

### ☐ Máquina Linux Red Hat Preparada
- [ ] Hardware conforme especificações
- [ ] RHEL instalado e atualizado
- [ ] Pacotes necessários instalados
- [ ] Limites do sistema configurados (ulimits)
- [ ] Parâmetros do kernel ajustados (sysctl)
- [ ] Usuário cdcuser criado com permissões
- [ ] Firewall configurado (portas abertas)
- [ ] SELinux configurado
- [ ] SSH habilitado

### ☐ Banco de Dados DB2 Preparado
- [ ] DB2 instalado (source e target)
- [ ] Log archiving habilitado (source)
- [ ] Usuários de replicação criados (cdccapture, cdcapply)
- [ ] Permissões concedidas
- [ ] Bancos catalogados no servidor Linux
- [ ] Conectividade testada
- [ ] Espaço em disco suficiente

### ☐ Rede e Conectividade
- [ ] Resolução de nomes configurada (DNS ou hosts)
- [ ] Portas abertas entre todos os componentes
- [ ] Latência de rede validada (< 10ms)
- [ ] Largura de banda adequada (≥ 100 Mbps)
- [ ] Teste de ping entre servidores bem-sucedido

### ☐ Acessos e Documentação
- [ ] Credenciais fornecidas (Windows, Linux, DB2)
- [ ] Acesso remoto configurado (VPN, RDP, SSH)
- [ ] Documentação de ambiente entregue
- [ ] Contatos das equipes definidos
- [ ] Janela de manutenção agendada

---

## 9. CONTATOS DO CLIENTE

Por favor, fornecer os seguintes contatos:

| Função | Nome | E-mail | Telefone |
|--------|------|--------|----------|
| Gerente de Projeto | | | |
| Administrador Windows | | | |
| Administrador Linux | | | |
| DBA DB2 | | | |
| Administrador de Rede | | | |

---

## 10. OBSERVAÇÕES IMPORTANTES

1. **Ambiente de Teste**: Esta PoC deve ser executada em ambiente de **não-produção** ou com dados de teste.

2. **Backup**: Recomenda-se realizar **backup completo** dos bancos de dados antes de iniciar a PoC.

3. **Impacto**: A captura de dados pode ter impacto mínimo no desempenho do banco origem. Será monitorado durante a PoC.

4. **Espaço em Disco**: O espaço para staging (200 GB) é uma estimativa. Pode precisar de ajuste conforme volume de transações.

5. **Suporte**: Ter contrato de suporte IBM ativo é recomendado para abertura de casos se necessário.

6. **Prazo**: Todos os requisitos devem estar prontos **antes do início da PoC** para evitar atrasos.

---

## 11. PRÓXIMOS PASSOS

Após confirmação de que todos os requisitos foram atendidos:

1. ✅ Cliente valida checklist completo
2. ✅ Cliente fornece todas as credenciais e acessos
3. ✅ Cliente fornece documentação do ambiente
4. ✅ Agendar data de início da PoC
5. ✅ Executar kickoff e validação de acesso
6. ✅ Iniciar instalação e configuração

---

**Documento preparado em**: 05/06/2026  
**Versão**: 1.0  
**Validade**: Este documento deve ser validado pelo cliente antes do início da PoC

---

## APROVAÇÃO DO CLIENTE

| Nome | Cargo | Assinatura | Data |
|------|-------|------------|------|
| | | | |
| | | | |
