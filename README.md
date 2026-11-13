# PostgreSQL Security Hardening: Isolamento Multi-Tenant, RBAC e Auditoria Forense

Implementação prática de defesa em profundidade (*Defense in Depth*) aplicada a um banco de dados relacional PostgreSQL 16 em arquitetura multi-tenant (`fintech_corp`). O projeto simula a proteção de dados financeiros e sensíveis (PII/LGPD) contra interceptação de tráfego, escalonamento de privilégios e vazamento cruzado entre clientes (*cross-tenant data leak*).

### Pilares de Segurança Implementados
* **Transporte e Perímetro:** Bloqueio explícito de conexões sem criptografia (`hostnossl reject`), exigência de TLS 1.2+ e autenticação `SCRAM-SHA-256`.
* **Controle de Acesso (RBAC):** Aplicação estrita do Princípio do Menor Privilégio, revogação de permissões padrão do schema `public` e segregação de funções administrativas e operacionais.
* **Isolamento Granular (RLS & CLS):** Segregação de linhas por *tenant* em nível de motor de banco (`Row-Level Security`) e mascaramento/restrição de colunas sensíveis (`Column-Level Security` / Views).
* **Auditoria e Detecção (Forensics):** Rastreamento granular de comandos DDL, DML, DCL e leituras críticas via `pgaudit`, validado por simulações práticas de ataque.