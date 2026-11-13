# PostgreSQL Security Hardening: Isolamento Multi-Tenant, RBAC e Auditoria Forense

Implementação prática de defesa em profundidade (*Defense in Depth*) aplicada a um banco de dados relacional PostgreSQL 16 em arquitetura multi-tenant (`fintech_corp`). O projeto simula a proteção de dados financeiros e sensíveis (PII/LGPD) contra interceptação de tráfego, escalonamento de privilégios e vazamento cruzado entre clientes (*cross-tenant data leak*).

### Pilares de Segurança Implementados
* **Transporte e Perímetro:** Bloqueio explícito de conexões sem criptografia (`hostnossl reject`), exigência de TLS 1.2+ e autenticação `SCRAM-SHA-256`.
* **Controle de Acesso (RBAC):** Aplicação estrita do Princípio do Menor Privilégio, revogação de permissões padrão do schema `public` e segregação de funções administrativas e operacionais.
* **Isolamento Granular (RLS & CLS):** Segregação de linhas por *tenant* em nível de motor de banco (`Row-Level Security`) e mascaramento/restrição de colunas sensíveis (`Column-Level Security` / Views).
* **Auditoria e Detecção (Forensics):** Rastreamento granular de comandos DDL, DML, DCL e leituras críticas via `pgaudit`, validado por simulações práticas de ataque.

---

## 1. Preparação do Ambiente (Infraestrutura Base)

O ambiente foi isolado em uma Máquina Virtual (VirtualBox) utilizando apenas ferramentas *open-source*, garantindo contenção do raio de explosão (*blast radius*) e simulando um servidor de produção.

### Especificações da VM
* **Hypervisor:** Oracle VirtualBox
* **Sistema Operacional:** Ubuntu Server 24.04 LTS / Debian 12 *(sem interface gráfica para reduzir a superfície de ataque, serviços em segundo plano e consumo de RAM)*.
* **Hardware Virtual:**
  * **RAM:** 4 GB
  * **CPU:** 2 vCPUs
  * **Disco:** 30 GB VDI dinâmico *(dimensionado para suportar logs forenses do `pgaudit` sem risco de indisponibilidade/DoS por exaustão de disco)*.

### Isolamento de Rede (Defesa de Perímetro)
A rede foi configurada em modo **NAT** (evitando exposição na rede local via *Bridge*), com redirecionamento de portas restrito ao *loopback* (`127.0.0.1`) da máquina hospedeira:

| Serviço | Protocolo | IP Hospedeiro | Porta Hospedeiro | Porta Convidado | Finalidade |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SSH** | TCP | `127.0.0.1` | `2222` | `22` | Administração remota segura |
| **PostgreSQL** | TCP | `127.0.0.1` | `5433` | `5433` | Acesso exclusivo ao banco de dados |

> **Decisão de Segurança:** Fixar `127.0.0.1` impede que o VirtualBox abra as portas em `0.0.0.0`, bloqueando varreduras ou tentativas de conexão vindas de outros dispositivos na mesma rede local.

### Boas Práticas Aplicadas na Instalação
* **Menor Privilégio:** Criação de usuário administrativo comum (sem operação direta como `root`).
* **Superfície Mínima:** Instalação apenas do `OpenSSH Server` durante o setup do sistema.
* **Runtime de Contêineres:** Docker Engine instalado via repositório oficial `apt` (evitando pacotes *Snap* que conflitam com permissões restritas de chaves privadas TLS).
