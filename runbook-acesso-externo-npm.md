### 📘 Fase 2 Detalhada: Servidor Nginx Proxy Manager (NPM)

Nesta etapa, vamos subir a "Torre de Controle" que vai direcionar o tráfego da internet para os containers corretos dos alunos.

#### Passo 1: Criando o Container e o "Pulo do Gato" (Nesting)

Por padrão, o Proxmox bloqueia a execução do Docker dentro de um container LXC por motivos de segurança. Para o Docker funcionar, precisamos ativar uma permissão chamada _Nesting_.

1. No Proxmox, crie um novo container LXC usando o template do **Debian 12**.
    
2. **Configuração:** 1 Core de CPU, 512 MB de RAM, 8 GB de Disco.
    
3. **Rede:** IP `20.20.0.202/20` e Gateway `20.20.0.1`.
    
4. **O Segredo:** Antes de ligar o container, clique nele na lista à esquerda, vá na aba **Options** (Opções) no painel central, dê um duplo clique em **Features** (Recursos) e marque a caixinha **Nesting**. Dê OK.
    
5. Agora sim, clique em **Start** e abra o **Console**.
    

#### Passo 2: Instalando o Docker

Logue no console com o usuário `root` e a senha que você definiu. Vamos instalar o Docker usando o script oficial, que é o método mais rápido e seguro.

Execute estes comandos, um por vez:

Bash

```bash
# Atualizar a lista de pacotes do sistema
apt update && apt upgrade -y

# Baixar o script de instalação do Docker
curl -fsSL https://get.docker.com -o get-docker.sh

# Rodar a instalação (pode levar uns 2 minutinhos)
sh get-docker.sh
```

#### Passo 3: Criando a Estrutura do Proxy

Com o Docker instalado, vamos criar uma pasta organizada para o Nginx Proxy Manager e montar o arquivo de configuração (`docker-compose.yml`).

Bash

```bash
# Criar uma pasta chamada npm e entrar nela
mkdir /opt/npm
cd /opt/npm

# Criar e abrir o arquivo de configuração
nano docker-compose.yml
```

Cole o código exato abaixo dentro do arquivo. Ele diz ao Docker para baixar a imagem do NPM, abrir as portas de web (80 e 443) e a porta do painel administrativo (81):

YAML

```yaml
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

_Salve apertando `CTRL + O` (Enter) e saia com `CTRL + X`._

#### Passo 4: Subindo o Serviço

Ainda dentro da pasta `/opt/npm`, digite o comando para iniciar o servidor em segundo plano (modo detached):

Bash

```bash
docker compose up -d
```

O Docker vai baixar as peças e iniciar o Proxy. Quando ele terminar e voltar para a linha de comando normal, o servidor estará pronto!

#### Passo 5: O Primeiro Acesso

Vá para o navegador do seu notebook ou de algum aluno e digite o IP seguido da porta 81: 👉 **`http://20.20.0.202:81`**

O painel de login prateado do Nginx Proxy Manager vai aparecer. Os dados de fábrica para o primeiro acesso são:

- **Email:** `admin@example.com`
    
- **Senha:** `changeme`
    

Logo ao entrar, o sistema pedirá para você alterar esse e-mail genérico para o seu próprio e cadastrar uma nova senha forte.

Pronto! A Torre de Controle está no ar e no IP perfeito. O próximo passo será configurar o Cloudflare Tunnel para publicar os sites na internet!
