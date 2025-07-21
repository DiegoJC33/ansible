# Projeto de Automação de Infraestrutura com Ansible

Este repositório contém playbooks Ansible para automatizar a configuração de servidores de desenvolvimento. O objetivo é garantir que os ambientes sejam provisionados de forma rápida, consistente e repetível, utilizando os princípios de Infraestrutura como Código.

O ambiente de destino é simulado utilizando containers Docker, permitindo testes locais sem a necessidade de máquinas virtuais ou servidores físicos.

## 🚀 Entregas Realizadas

Este projeto automatiza a configuração de dois serviços essenciais:

1.  **Sincronização de Tempo (NTP):** Instala e configura o serviço NTP para garantir que o relógio do servidor esteja sempre sincronizado, o que é crucial para logs e operações agendadas.
2.  **Servidor Web (Nginx):** Instala e configura o servidor web Nginx, deixando-o pronto para hospedar aplicações.

## 🛠️ Tecnologias Utilizadas

*   **Ansible:** Ferramenta de automação para provisionamento e gerenciamento de configuração.
*   **Docker:** Para criar ambientes de servidor isolados e leves para desenvolvimento e teste.
*   **Git:** Para o versionamento de todo o código da infraestrutura.

## ⚙️ Pré-requisitos

Antes de começar, garanta que você tenha as seguintes ferramentas instaladas em sua máquina local:

*   [Docker](httpss://www.docker.com/get-started) e Docker Desktop (ou Docker Engine no Linux)
*   [Ansible](httpss://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
*   Python 3 e Pip
*   A biblioteca Docker para Python:
    ```bash
    pip install docker
    ```

## 📋 Passo a Passo para Execução

Siga os passos abaixo para provisionar e configurar os servidores do zero.

### 1. Preparar o Ambiente (Containers Docker)

Os servidores `dev-server1` e `dev-server2` são simulados por containers Docker. O comando abaixo irá criá-los, mapeando as portas necessárias para acesso via SSH (para debug) e HTTP (para o serviço web).

Execute os seguintes comandos no seu terminal:

```bash
# Para o dev-server1: Mapeia a porta 8081 (HTTP) e 2222 (SSH)
docker run --name dev-server1 -d -p 8081:80 -p 2222:22 rastasheep/ubuntu-sshd

# Para o dev-server2: Mapeia a porta 8082 (HTTP) e 2223 (SSH)
docker run --name dev-server2 -d -p 8082:80 -p 2223:22 rastasheep/ubuntu-sshd
