# Ghost NginX Compose

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white) ![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white) ![Ghost](https://img.shields.io/badge/ghost-%2315171A.svg?style=for-the-badge&logo=ghost&logoColor=white) ![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)

## 📖 Descrição do Projeto

Este repositório contém uma solução para provisionar um ambiente de blog **Ghost** utilizando **Docker Compose**. A arquitetura inclui um banco de dados **MySQL** para persistência dos dados e um proxy reverso **Nginx** para gerenciar o tráfego e habilitar a comunicação segura via HTTPS.

## ✅ Funcionalidades e Implementações

-   [x] **Orquestração com `docker-compose`:** Todos os serviços são definidos e gerenciados em um único arquivo `docker-compose.yml`.
-   [x] **Stack:** A solução utiliza a stack Ghost (Node.js) e MySQL.
-   [x] **Persistência de Dados:** Todos os dados críticos do banco de dados e do Ghost são persistidos em volumes nomeados do Docker.
-   [x] **Ordem de Inicialização:** O serviço do Ghost aguarda o banco de dados estar totalmente saudável (`healthy`) antes de iniciar.
-   [x] **Proxy Reverso com Nginx:** O Nginx atua como o único ponto de entrada para o ambiente, com configurações personalizadas.
-   [x] **HTTPS Habilitado:** A comunicação é criptografada via HTTPS, com redirecionamento automático de todas as requisições HTTP.
-   [x] **Certificado Autoassinado:** Um certificado SSL autoassinado é gerado localmente, com o comando devidamente documentado.

## 🛠️ Pré-requisitos

Antes de começar, garanta que você tenha os seguintes softwares instalados na sua máquina:

* **Docker**
* **Docker Compose v2+**
* **OpenSSL**

## 🚀 Como Executar

Siga estes passos abaixo para subir o ambiente localmente.

### 1. Clonar o Repositório
```bash
git clone <url_do_repo>
cd <nome_da_pasta>
```

### 2. Configurar Variáveis de Ambiente
Crie um arquivo `.env` a partir do exemplo fornecido e preencha com senhas de sua preferência.

```bash
cp .env.example .env
```

### 3. Configurar o Domínio Local
Mapeie o domínio `gtblog.local` para o seu endereço de loopback.

**Adicione a seguinte linha ao seu arquivo `hosts`:**
* **macOS / Linux:** `/etc/hosts`
* **Windows:** `C:\Windows\System32\drivers\etc\hosts` (edite como Admin)

```
127.0.0.1 gtblog.local
```

### 4. Gerar o Certificado SSL
Foi utilizado um certificado autoassinado para HTTPS. Execute o comando abaixo na raiz do projeto para gerá-lo:
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout nginx/certs/selfsigned.key -out nginx/certs/selfsigned.crt -subj "/CN=gtblog.local"
```

### 5. Subir o Ambiente
Agora use o comando abaixo para subir o ambiente:
```bash
docker compose up -d
```

## 🌐 Acessando o Blog

* **URL do Site:** **[https://gtblog.local](https://gtblog.local)**
* **Painel de Administração:** **[https://gtblog.local/ghost](https://gtblog.local/ghost)**

**Aviso:** Ao acessar o site pela primeira vez, o navegador exibirá um alerta de segurança ("Sua conexão não é particular"). **Isso é esperado**, pois o certificado foi assinado por mim e não por uma autoridade que o navegador confia. Clique em "Avançado" e depois em "Ir para gtblog.local (não seguro)" para prosseguir.

## 🏛️ Arquitetura da Solução

A arquitetura é composta por três serviços principais que se comunicam através de uma rede interna do Docker:
1.  **`nginx`:** O único serviço exposto ao host. Recebe todo o tráfego nas portas 80 e 443, gerencia os certificados SSL, redireciona HTTP para HTTPS e atua como proxy reverso para a aplicação Ghost.
2.  **`ghost`:** Contém a aplicação Node.js do Ghost. Não é exposta diretamente e apenas recebe tráfego do Nginx. Conecta-se ao banco de dados para ler e escrever conteúdo.
3.  **`db`:** O banco de dados MySQL, responsável por armazenar de forma persistente todo o conteúdo do blog. Apenas o serviço do Ghost tem acesso a ele.

## 📝 Decisões e Notas

* **Versionamento de Imagens:** Todas as imagens no docker-compose.yml possuem versões fixas (ex: mysql:8.0, ghost:5-alpine) para garantir a reprodutibilidade. A tag :latest foi evitada. Para Ghost e Nginx, foram utilizadas as variantes alpine por serem mais leves e seguras. Para o MySQL, foi usada a imagem oficial padrão, pois não há uma variante alpine com suporte oficial.
* **Certificado SSL:** A escolha por um **certificado autoassinado** foi usada, pois cumpre o requisito de habilitar HTTPS e permite que o projeto seja 100% autocontido e executável localmente sem dependências externas.
* **Cabeçalhos de Segurança:** Foi adicionado o cabeçalho `Strict-Transport-Security` na configuração do Nginx como uma boa prática de segurança para forçar os navegadores a se comunicarem apenas via HTTPS.
