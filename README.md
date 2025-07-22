# Projeto 01 - DevOps com Vagrant e Ansible

**Disciplina:** Administração de Sistemas Abertos  
**Período:** 2025.1  
**Professor:** Leonidas Lima  
**Instituição:** IFPB - Campus João Pessoa

## 👥 Integrantes

- **Nilson Vinícius Aurélio Chaves** — 20221380002
- **Wellington [Sobrenome Não Informado]** — 20221380031

---

## 🎯 Objetivo

Este projeto visa aplicar conceitos de **Infraestrutura como Código (IaC)** e automação de ambientes usando **Vagrant** e **Ansible**, simulando uma infraestrutura de quatro máquinas virtuais integradas.

---

## 🧱 Infraestrutura Criada com Vagrant

| VM    | Hostname                      | RAM (MB) | IP Privado         | Função             |
|-------|-------------------------------|----------|---------------------|---------------------|
| arq   | arq.nilson.wellington.devops  | 512      | 192.168.56.102      | Servidor de arquivos / DHCP / NFS / LVM |
| db    | db.nilson.wellington.devops   | 512      | 192.168.56.27 (DHCP) | Servidor MariaDB + NFS Cliente |
| app   | app.nilson.wellington.devops  | 512      | 192.168.56.28 (DHCP) | Servidor Web + NFS Cliente |
| cli   | cli.nilson.wellington.devops  | 1024     | 192.168.56.29 (DHCP) | Cliente gráfico + NFS Cliente |

---

## ⚙️ Automatizações com Ansible

### Comum a todas as VMs:
- Atualização do sistema
- Timezone: `America/Recife`
- NTP configurado com `chrony` (pool.ntp.br)
- Grupo `ifpb` e usuários `nilson` e `wellington`
- SSH:
  - Autenticação por chave pública
  - Root bloqueado
  - Acesso só para `vagrant` e `ifpb`
  - Mensagem de acesso
- Cliente NFS
- Sudo sem senha para o grupo `ifpb`

### Servidor `arq`:
- DHCP autoritativo
- LVM com 3 discos de 10 GB → VG `dados`, LV `ifpb` (15 GB)
- Montagem automática em `/dados`
- Servidor NFS exportando `/dados/nfs`
- Usuário exclusivo `nfs-ifpb` com UID/GID 65534

### Servidor `db`:
- Instalação do `mariadb-server`
- Autofs montando `/dados/nfs` do `arq` em `/var/nfs`

### Servidor `app`:
- Instalação do Apache2
- Substituição do `index.html` com dados do projeto
- Autofs montando `/dados/nfs` em `/var/nfs`

### Cliente `cli`:
- Instalação de `firefox-esr` e `xauth`
- SSH com `X11Forwarding` habilitado
- Montagem automática do NFS em `/var/nfs`

---

## 📁 Estrutura de Diretórios

```bash
projeto_devops/
├── ansible/
│   ├── comum-playbook.yml
│   ├── arq-playbook.yml
│   ├── db-playbook.yml
│   ├── app-playbook.yml
│   ├── cli-playbook.yml
│   ├── inventory.ini
│   └── keys/
├── .vagrant/           # Discos dinâmicos criados para a VM 'arq'
├── Vagrantfile         # Define as VMs e provisionamento
└── README.md


