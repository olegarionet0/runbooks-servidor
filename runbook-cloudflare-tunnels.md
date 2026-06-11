### 📘 Runbook: Acesso Externo via Cloudflare Tunnels (cloudflared)

Este runbook orienta a implantação do **Cloudflare Tunnels** (`cloudflared`) integrado ao contêiner do Nginx Proxy Manager (NPM). Essa abordagem permite publicar os sites WordPress dos squads e o Moodle na internet sem a necessidade de abrir portas no roteador do laboratório (CGNAT/Firewall Friendly).

---

## 1. Como Funciona a Arquitetura

O túnel cria uma conexão persistente e segura de saída entre o laboratório físico e a rede global da Cloudflare.

```
[ Usuário na Internet ] 
          │ (HTTPS)
          ▼
[ Rede Cloudflare (Edge) ]
          │ (Túnel Criptografado de Saída)
          ▼
[ Container cloudflared ] (Roda no LXC do NPM)
          │ (HTTP local)
          ▼
[ Nginx Proxy Manager ] (Porta 80)
          │ (HTTP local)
          ├─► [ WP-Squad1 (20.20.0.210) ]
          ├─► [ WP-Squad2 (20.20.0.211) ]
          └─► [ Moodle (20.20.0.X) ]
```

---

## 2. Pré-requisitos Administrativos (Painel Cloudflare)

> [!IMPORTANT]
> **Controle de Acesso:** A conta da Cloudflare e o domínio ativo (ex: `escola.local` ou `seudominio.com`) devem estar sob o controle do **Professor/Administrador**. Os alunos não devem possuir acesso a estas credenciais.

1. O Administrador acessa o **Cloudflare Zero Trust Dashboard** (`https://one.dash.cloudflare.com/`).
2. Vai em **Networks** -> **Tunnels** -> **Create a Tunnel**.
3. Seleciona o conector **Cloudflare** (`cloudflared`).
4. Nomeia o túnel (ex: `lab-infra-2026`).
5. Copia o **Tunnel Token** gerado (uma longa chave alfanumérica).

---

## 3. Configuração no Container do NPM (Squad de Infraestrutura)

Os alunos deverão atualizar o arquivo `docker-compose.yml` da "Torre de Controle" (NPM) para incluir o agente do túnel.

1. Acesse o console do container NPM (`20.20.0.202`).
2. Entre no diretório de configuração:
   ```bash
   cd /opt/npm
   nano docker-compose.yml
   ```
3. Atualize o arquivo para adicionar o serviço `cloudflared` ao lado do `app` (NPM):

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

  cloudflared:
    image: cloudflare/cloudflared:latest
    restart: unless-stopped
    command: tunnel --no-autoupdate run --token <SEU_TUNNEL_TOKEN_FORNECIDO_PELO_PROFESSOR>
```

> [!WARNING]
> Substitua `<SEU_TUNNEL_TOKEN_FORNECIDO_PELO_PROFESSOR>` pelo token gerado pelo professor. Não compartilhe ou publique esse token no GitHub.

4. Reinicie os serviços para aplicar as alterações:
   ```bash
   docker compose down
   docker compose up -d
   ```

5. Verifique se o container está rodando e conectado:
   ```bash
   docker compose logs cloudflared
   ```

---

## 4. Roteamento de Subdomínios (DNS Cloudflare)

No painel do Cloudflare Zero Trust, o Administrador deve configurar o roteamento (Public Hostnames) para apontar os subdomínios para o Nginx Proxy Manager interno:

| **Public Hostname** | **Service URL** | **Descrição** |
| :--- | :--- | :--- |
| `*.lab.seudominio.com` | `http://app:80` | Wildcard para todos os subdomínios dos alunos |
| `moodle.lab.seudominio.com` | `http://app:80` | Subdomínio dedicado para o Moodle |

> [!NOTE]
> Como o `cloudflared` está na mesma rede Docker que o NPM (`app`), ele pode referenciar o container diretamente pelo nome de serviço `http://app:80`.

---

## 5. Configuração no Painel do Nginx Proxy Manager (NPM)

Com o túnel direcionando todo o tráfego do subdomínio `*.lab.seudominio.com` para o NPM, os alunos podem cadastrar seus Proxy Hosts livremente:

1. Acesse `http://20.20.0.202:81`.
2. Vá em **Hosts** -> **Proxy Hosts** -> **Add Proxy Host**.
3. Configure o site do Squad:
   * **Domain Names:** `squad1.lab.seudominio.com`
   * **Scheme:** `http`
   * **Forward Hostname / IP:** `20.20.0.210`
   * **Forward Port:** `80`
   * **Block Common Exploits:** Ativado.
4. Na aba **SSL**, selecione **None** (pois a Cloudflare já provê criptografia HTTPS de borda até o navegador do usuário final de forma transparente).
5. Salve a configuração. O site agora estará acessível mundialmente sob a URL criptografada.
