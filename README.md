# Trabalho Prático: Monitoramento com Zabbix e Grafana

Este repositório contém os passos e configurações para a implementação de uma solução de monitoramento de infraestrutura utilizando Zabbix e Grafana, como parte da avaliação da disciplina de Gerenciamento de Redes de Computadores da Universidade Estadual de Maringá.

## Objetivo do Projeto

Os objetivos deste trabalho são instalar, configurar e monitorar um dispositivo de rede com a ferramenta Zabbix integrada à ferramenta Grafana.  
Ambas as ferramentas serão instaladas em uma máquina virtual com Linux, que será o próprio dispositivo monitorado.

## Tecnologias Utilizadas

* **Sistema Operacional:** Ubuntu Server (22.04 LTS ou superior)
* **Monitoramento:** Zabbix
* **Visualização:** Grafana
* **Banco de Dados:** MySQL
* **Servidor Web:** Apache2

---

## Guia de Instalação e Configuração

Siga os passos abaixo para replicar o ambiente de monitoramento.

### Parte 1: Instalação do Zabbix Server (com MySQL)

#### 1.1 Adicionar o Repositório Oficial do Zabbix

Primeiro, adicione o repositório do Zabbix para ter acesso aos pacotes de instalação mais recentes.

```bash
# Baixe o pacote de configuração do repositório
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu$(lsb_release -rs)_all.deb

# Instale o pacote
sudo dpkg -i zabbix-release_6.4-1+ubuntu$(lsb_release -rs)_all.deb

# Atualize a lista de pacotes
sudo apt update
```

#### 1.2 Instalar Componentes do Zabbix

Instale o servidor Zabbix, a interface web (frontend), o agente de monitoramento e os scripts para MySQL.

```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

#### 1.3 Criar e Configurar o Banco de Dados

Acesse o MySQL para criar o banco de dados e o usuário que o Zabbix utilizará.

```bash
# Instale o MySQL Server, caso não tenha
sudo apt install mysql-server

# Acesse o shell do MySQL
sudo mysql -uroot -p

# Execute os comandos SQL abaixo (substitua 'sua_senha_segura' por uma senha forte)
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user 'zabbix'@'localhost' identified by 'sua_senha_segura';
grant all privileges on zabbix.* to 'zabbix'@'localhost';
set global log_bin_trust_function_creators = 1;
quit;
```

#### 1.4 Importar o Schema Inicial do Zabbix

Importe a estrutura inicial de tabelas e dados para o banco de dados recém-criado.

```bash
sudo zcat /usr/share/doc/zabbix-sql-scripts/mysql/server.sql.gz | mysql -uzabbix -p zabbix
```

*Você precisará digitar a senha criada no passo anterior.*

#### 1.5 Configurar a Senha no Zabbix Server

Edite o arquivo de configuração do Zabbix (`/etc/zabbix/zabbix-server.conf`) para informar a senha do banco de dados.

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Encontre a linha `DBPassword=` e adicione sua senha:

```conf
DBPassword=sua_senha_segura
```

#### 1.6 Iniciar os Serviços

Reinicie e habilite os serviços para que iniciem com o sistema.

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

#### 1.7 Configuração via Interface Web

1. Acesse `http://<ip_do_seu_servidor>/zabbix` no navegador.  
2. Siga as instruções do instalador web. Na tela de configuração do banco de dados, utilize as credenciais que você criou.  
3. Ao finalizar, faça login com o usuário padrão:
   * **Usuário:** `Admin`
   * **Senha:** `zabbix`

-----

### Parte 2: Instalação do Grafana

#### 2.1 Adicionar o Repositório Oficial do Grafana

```bash
sudo apt install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

#### 2.2 Instalar e Iniciar o Grafana

```bash
# Atualize os pacotes e instale o Grafana
sudo apt update
sudo apt install grafana

# Inicie e habilite o serviço
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

O Grafana estará disponível em `http://<ip_do_seu_servidor>:3000`.  
O login padrão é `admin` / `admin`.

-----

### Parte 3: Integração do Zabbix com o Grafana

#### 3.1 Instalar o Plugin do Zabbix no Grafana

Execute o comando no terminal do seu servidor para instalar o plugin de integração.

```bash
sudo grafana-cli plugins install alexanderzobnin-zabbix-app
```

Reinicie o Grafana para aplicar a alteração:

```bash
sudo systemctl restart grafana-server
```

#### 3.2 Configurar o Data Source

1. Acesse a interface do Grafana.  
2. No menu, vá em **Connections > Data sources** e clique em **"Add new data source"**.  
3. Pesquise e selecione **Zabbix**.  
4. No campo **URL**, insira o endereço da API do Zabbix:  
   `http://<ip_do_seu_servidor>/zabbix/api_jsonrpc.php`  
5. Na seção **Zabbix API details**, insira o usuário (`Admin`) e a senha do Zabbix.  
6. Clique em **"Save & test"** para verificar a conexão.

Com isso, o ambiente está pronto para ser utilizado.