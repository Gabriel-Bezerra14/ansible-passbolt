# **⚙️ Automação de Deploy do Passbolt com Ansible**

Este projeto utiliza Ansible para automatizar completamente a instalação e configuração do **Passbolt Community Edition** em um servidor Debian, usando Docker.
O objetivo é ter um processo de deploy rápido, seguro e 100% reprodutível, seguindo as melhores práticas de Infraestrutura como Código (IaC).

## **🚀 Como Funciona?**

O processo é orquestrado por um *playbook* principal (playbook.yml) que executa três *roles* (capítulos) em sequência:

1. **docker**: Garante que o Docker esteja instalado, configurado e rodando no servidor alvo.  
2. **pip**: Instala as dependências Python necessárias para que o Ansible possa se comunicar com o Docker.  
3. **passbolt**: Realiza o deploy da aplicação Passbolt, configurando os arquivos, a rede e iniciando os contêineres.

O *playbook* é **idempotente**, o que significa que você pode executá-lo várias vezes sem causar problemas. Ele verifica o que já está feito e executa apenas as tarefas necessárias.

## **📋 Pré-requisitos**

Antes de executar a automação, você precisa ter duas coisas instaladas na **sua máquina de controle** (de onde você vai rodar o Ansible):

- **Ansible**: 
```sh
    sudo apt install ansible
```
- **sshpass**: Necessário para a autenticação SSH via senha.
```sh
   sudo apt update && sudo apt install sshpass
```

## **⚙️ Configuração**

Antes da primeira execução, você precisa configurar 3 arquivos principais:

### **1\. Inventário (inventory.ini)**

Este arquivo informa ao Ansible em qual servidor ele deve trabalhar.

\# inventory.ini
```ini
[passbolt_servers]
servidor_passbolt ansible_host=ip-sua-vm

[passbolt_servers:vars]
ansible_user=user
ansible_password=password
```

### **2\. Configuração do Ansible (ansible.cfg)**

Este arquivo resolve o problema de segurança do SSH na primeira conexão.

\# ansible.cfg
```cfg
[defaults]
# Desabilita a verificação da chave do host SSH.
host_key_checking = False
```

### **3\. Variáveis do Passbolt (roles/passbolt/defaults/main.yml)**

Aqui você personaliza a sua instalação do Passbolt.

\# roles/passbolt/defaults/main.yml

```yml
# Variáveis de configuração
passbolt_install_dir: "/seu/diretorio/passbolt"
passbolt_network_name: "passbolt-net"
passbolt_app_url: "http://ip-sua-vm"
mysql_user: "passbolt_user"
mysql_database: "passbolt"
passbolt_admin_email: "admin@adm.com"
passbolt_admin_firstname: "Admin"
passbolt_admin_lastname: "User"
mysql_root_password: 12345678
mysql_password: 12345
```

## **▶️ Como Executar**

Com tudo configurado, basta executar um único comando na raiz do projeto:
```sh
ansible-playbook \-i inventory.ini playbook.yml
```
O Ansible cuidará de todo o resto. Na primeira execução, ao final, ele exibirá o link para você registrar o usuário administrador no navegador.

## **🔧 Manutenção**

A estrutura em *roles* torna a manutenção muito simples:

* Precisa mudar a URL do Passbolt? Edite **apenas** o arquivo roles/passbolt/defaults/main.yml.  
* Precisa atualizar a versão do Docker? Mude a versão do pacote **apenas** no arquivo roles/docker/tasks/main.yml.  
* Precisa adicionar um *role* para fazer backup? Crie uma nova pasta roles/backup sem interferir nas outras.
