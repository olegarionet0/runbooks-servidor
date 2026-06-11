# 📖 Central de Runbooks - Cluster Acadêmico 2026

Bem-vindo ao repositório central de documentação e procedimentos operacionais (Runbooks) para a infraestrutura do laboratório! Este espaço é mantido pelos squads de alunos para organizar o provisionamento, a rede e a conectividade de nossos servidores.

---

## 🚀 Como Proceder (Fluxo de Trabalho)

Para implantar e expor seus serviços no laboratório, siga estes passos em ordem:

### 1️⃣ Consulte o Índice Central
Toda a documentação ativa está mapeada no arquivo **[[indice-cluster-academico]]**. Use-o como seu ponto de partida no Obsidian ou GitHub para navegar entre os guias.

### 2️⃣ Provisione a Aplicação (WordPress)
Se o seu squad precisa de uma instância WordPress:
* Acesse o runbook **[[runbook-provisionamento-wordpress-lxc]]**.
* Siga o procedimento de criação da máquina modelo (LXC Turnkey no Proxmox).
* **Muito Importante:** Execute a *Fase B (Ajuste Técnico)* para editar o `wp-config.php` e evitar loops de redirecionamento HTTPS infinito quando o site for publicado na internet.
* Clone a máquina e configure o IP específico do seu Squad conforme a tabela de inventário de IPs.

### 3️⃣ Exponha sua Aplicação na Rede Local (NPM)
Com o WordPress rodando localmente no IP do seu Squad:
* Acesse o runbook **[[runbook-acesso-externo-npm]]**.
* A equipe de infraestrutura utilizará este guia para configurar a "Torre de Controle" (Nginx Proxy Manager) no IP `20.20.0.202`.
* Você criará um *Proxy Host* no painel administrativo do NPM (`http://20.20.0.202:81`) apontando o domínio do seu squad (ex: `squad1.lab.seudominio.com`) para o IP interno do seu contêiner na porta `80`.

### 4️⃣ Ative o Acesso Externo Seguro (Cloudflare Tunnels)
Para que pessoas fora da rede da escola consigam acessar o seu site WordPress ou o Moodle:
* Acesse o runbook **[[runbook-cloudflare-tunnels]]**.
* A equipe de infraestrutura integrará o contêiner do `cloudflared` ao Docker Compose do NPM.
* **Atenção:** Solicite o **Tunnel Token** diretamente ao **Professor/Administrador**. Nunca publique o token real no código ou no GitHub!

---

## 🔒 Diretrizes e Boas Práticas

* **Respeite o IPAM:** Nunca utilize um IP que não esteja explicitamente atribuído ao seu Squad na tabela de inventário. Conflitos de IP derrubam os serviços dos seus colegas.
* **Segurança de Credenciais:** Nunca salve senhas, chaves privadas SSH ou tokens de serviços online nos arquivos markdown ou códigos que serão commitados no GitHub. Use placeholders (ex: `<SEU_TOKEN_AQUI>`).
* **Mantenha a Documentação Viva:** Encontrou um bug ou um erro nos comandos durante o laboratório? Faça a correção no arquivo markdown correspondente e notifique o professor para atualizarmos o repositório público.
