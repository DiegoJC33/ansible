# Projeto de Automação de Infraestrutura com Ansible & Docker

Bem-vindo a este projeto de Infraestrutura como Código! O objetivo aqui é demonstrar o uso do Ansible para automatizar, do zero, a configuração de servidores para um ambiente de desenvolvimento. Para agilidade e portabilidade, todo o ambiente é provisionado localmente usando Docker, simulando servidores reais.

Este repositório é o resultado de um ciclo completo de desenvolvimento e depuração, resolvendo problemas comuns de conectividade, permissões e compatibilidade de ambiente.

## 🚀 Entregas de Automação

Os playbooks neste repositório realizam duas configurações críticas para qualquer servidor base:

1.  **Sincronização de Tempo (NTP):** Garante que os servidores tenham o relógio precisamente sincronizado, um requisito fundamental para a consistência de logs, agendamento de tarefas (cron) e operações distribuídas.
2.  **Servidor Web (Nginx):** Provisiona um servidor web Nginx de alta performance, deixando-o pronto para hospedar aplicações ou atuar como um proxy reverso.

## 🛠️ Stack de Tecnologias

*   **Ansible:** O coração da automação, usado para o gerenciamento de configuração declarativo.
*   **Docker:** Para criar ambientes de servidor Linux (Ubuntu) isolados e descartáveis, acelerando o ciclo de desenvolvimento e teste.
*   **Git & GitHub:** Para o versionamento de todo o código da infraestrutura, permitindo rastreabilidade, colaboração e integração com fluxos de CI/CD.

## ⚙️ Configuração do Ambiente Local (Pré-requisitos)

Para executar este projeto, sua máquina precisa ter:

1.  **Git:** Para clonar este repositório.
2.  **Docker e Docker Desktop (ou Docker Engine):** Para executar os containers que simulam nossos servidores. [Instruções de instalação](httpss://www.docker.com/get-started).
3.  **Python 3 e Pip:** A base para a execução do Ansible.
4.  **Ansible:** A ferramenta de automação.
    ```bash
    # Exemplo de instalação em sistemas baseados em Debian/Ubuntu
    sudo apt update
    sudo apt install software-properties-common
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible
    ```
5.  **Biblioteca Docker para Python:** Permite que o Ansible se comunique com a API do Docker.
    ```bash
    pip install docker
    ```

## 📋 Guia de Execução Completo

Siga este guia para provisionar, configurar e validar a infraestrutura.

### Fase 1: Clonar o Repositório

Primeiro, obtenha o código da automação em sua máquina local.

```bash
git clone https://github.com/DiegoJC33/ansible.git
cd ansible
```
Fase 2: Provisionar os "Servidores" com Docker
Vamos criar nossos dois servidores de desenvolvimento, dev-server1 e dev-server2, como containers Docker. Usamos a imagem rastasheep/ubuntu-sshd, que nos fornece um ambiente Ubuntu mínimo com um servidor SSH pré-configurado (útil para inspeção manual).
Comandos de criação:
Generated bash
# Para o dev-server1: Mapeia a porta 8081 (HTTP local) para 80 (HTTP container) e 2222 para 22 (SSH)
docker run --name dev-server1 -d -p 8081:80 -p 2222:22 rastasheep/ubuntu-sshd

# Para o dev-server2: Mapeia a porta 8082 (HTTP local) para 80 (HTTP container) e 2223 para 22 (SSH)
docker run --name dev-server2 -d -p 8082:80 -p 2223:22 rastasheep/ubuntu-sshd

Por que fazemos isso? O mapeamento de portas (-p) cria uma ponte entre o seu localhost e o ambiente isolado do container. Sem isso, os serviços rodando dentro do container seriam inacessíveis.
Fase 3: Testar a Conexão Ansible-Docker
Nosso inventário (inventories/dev.ini) está configurado para usar a conexão nativa do Docker (ansible_connection=docker). Esta é uma decisão de design importante, pois é mais eficiente e robusta do que usar SSH para gerenciar containers locais, evitando a necessidade de gerenciar chaves ou senhas.
Vamos verificar se o Ansible consegue se comunicar com os containers:
Generated bash
ansible all -i inventories/dev.ini -m ping
Use code with caution.
Bash
A saída esperada é um SUCCESS verde para ambos os servidores, com a mensagem "ping": "pong". Isso confirma que a ponte de comunicação está funcionando.

Fase 4: Executar os Playbooks de Configuração
Agora, vamos aplicar nossas configurações automatizadas.
1. Configurar o NTP:
Este playbook instala e configura o serviço NTP. Note o uso de become: yes no playbook, que instrui o Ansible a executar as tarefas com privilégios de administrador (sudo).
Generated bash
ansible-playbook -i inventories/dev.ini playbooks/ntp.yml

2. Configurar o Nginx:
Este playbook instala, habilita e inicia o Nginx. Ele foi cuidadosamente construído para ser compatível com o ambiente mínimo dos containers.
Generated bash
ansible-playbook -i inventories/dev.ini playbooks/web.yml
Use code with caution.
Bash
Fase 5: Validar a Infraestrutura Provisionada
A etapa final é verificar se os serviços estão realmente funcionando como esperado.
1. Verificação via Navegador:
Abra seu navegador e acesse os seguintes endereços. Em ambos, você deve ver a página de boas-vindas "Welcome to nginx!".
Acessar Servidor 1: http://localhost:8081
Acessar Servidor 2: http://localhost:8082
2. (Opcional) Verificação via Linha de Comando:
Você pode inspecionar o status dos serviços diretamente dentro dos containers.
Generated bash
# Verificar status do Nginx no dev-server1
docker exec dev-server1 service nginx status

# Verificar status do NTP no dev-server2
docker exec dev-server2 service ntp status
Use code with caution.
Bash
🧠 Lições Aprendidas e Decisões de Design
Durante o desenvolvimento deste projeto, vários desafios comuns foram superados, moldando o design final dos playbooks:
Conectividade: O erro inicial No route to host nos ensinou que não podemos acessar o IP interno de um container Docker diretamente do host. A solução foi adotar o conector nativo ansible_connection=docker.
Gerenciamento de Serviços em Containers: O erro Service is in unknown state revelou que muitas imagens Docker não possuem um systemd completo. Por isso, os playbooks foram adaptados para usar o módulo ansible.builtin.command com comandos diretos (service nginx start), que é uma abordagem muito mais robusta para ambientes de container.
Idempotência e Handlers: Foi necessário compreender como o Ansible lida com tarefas que não mudam (changed=0) e como os handlers são (ou não) acionados, utilizando notify para garantir que os serviços sejam iniciados apenas após uma instalação bem-sucedida.
Organização com Roles: A lógica foi separada em roles (ntp_hardening, nginx_web) para criar componentes de automação limpos, reutilizáveis e fáceis de manter, seguindo as melhores práticas do Ansible.
🧹 Limpando o Ambiente
Quando terminar, você pode parar e remover os containers para liberar os recursos da sua máquina.
Generated bash
docker stop dev-server1 dev-server2
docker rm dev-server1 dev-server2
