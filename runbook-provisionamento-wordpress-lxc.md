### 📘 Runbook: Provisionamento de Servidores Web (WordPress)

# Runbook: Infraestrutura de Squads - WordPress LXC

## 1. Contexto da Infraestrutura

- **Host:** Proxmox VE (Lenovo 16GB RAM)
    
- **Rede:** `20.20.0.0/20` (Máscara: `255.255.240.0`)
    
- **Gateway:** `20.20.0.1` (Padrão da Escola)
    
- **DNS:** `8.8.8.8` / `8.8.4.4`

- **Acesso ao Proxmox:** `<SOLICITAR_AO_PROFESSOR>`

- **Login Admin WordPress (Modelo):** `admin2`

- **Senha Admin WordPress (Modelo):** `lab123`
    

## 2. Inventário de IPs (Planejamento)

| **Unidade**    | **Nome Proxmox** | **Endereço IP** | **Link do Site** | **Painel Admin (wp-admin)** |
| -------------- | ---------------- | --------------- | ----------------- | --------------------------- |
| Servidor Host  | `pve-lab`        | `20.20.0.200`   | `https://20.20.0.200:8006/` | - |
| Servidor Host2 | `pve`            | `20.20.0.201`   | `https://20.20.0.201:8006/` | - |
| Proxy (NPM)    | `NPM-Proxy`      | `20.20.0.202`   | `http://20.20.0.202:81/`  | - |
| Squad 1        | `WP-Squad1`      | `20.20.0.210`   | [Acessar Site](https://20.20.0.210/) | [Acessar Admin](https://20.20.0.210/wp-login.php) |
| Squad 2        | `WP-Squad2`      | `20.20.0.211`   | [Acessar Site](https://20.20.0.211/) | [Acessar Admin](https://20.20.0.211/wp-login.php) |
| Squad 3        | `WP-Squad3`      | `20.20.0.212`   | [Acessar Site](https://20.20.0.212/) | [Acessar Admin](https://20.20.0.212/wp-login.php) |
| Squad 4        | `WP-Squad4`      | `20.20.0.213`   | [Acessar Site](https://20.20.0.213/) | [Acessar Admin](https://20.20.0.213/wp-login.php) |
| Squad 5        | `WP-Squad5`      | `20.20.0.214`   | [Acessar Site](https://20.20.0.214/) | [Acessar Admin](https://20.20.0.214/wp-login.php) |
| Squad 6        | `WP-Squad6`      | `20.20.0.215`   | [Acessar Site](https://20.20.0.215/) | [Acessar Admin](https://20.20.0.215/wp-login.php) |
| Squad 7        | `WP-Squad7`      | `20.20.0.216`   | [Acessar Site](https://20.20.0.216/) | [Acessar Admin](https://20.20.0.216/wp-login.php) |
| Squad 8        | `WP-Squad8`      | `20.20.0.217`   | [Acessar Site](https://20.20.0.217/) | [Acessar Admin](https://20.20.0.217/wp-login.php) |
| Squad 9        | `WP-Squad9`      | `20.20.0.218`   | [Acessar Site](https://20.20.0.218/) | [Acessar Admin](https://20.20.0.218/wp-login.php) |

---
## 3. Passo a Passo do Provisionamento

### Fase A: Criação do Molde (Template)

1. Baixar o template LXC `debian-12-turnkey-wordpress` em `local (pve-lab)` -> `CT Templates`.
    
2. Criar o Container (CT): **2 Cores CPU / 1GB RAM / 10GB SSD**.
    
3. Configurar Rede: Bridge `vmbr0`, IPv4 `20.20.0.210/20`, Gateway `20.20.0.1`.
    
4. No primeiro boot (Console), definir senhas e pular serviços de Hub/Backup.
    

### Fase B: Ajuste Técnico (Troubleshooting SSL/Redirect)

Para evitar o erro de redirecionamento infinito (Redirect Loop) e habilitar o login via HTTPS, o arquivo `wp-config.php` deve ser editado.

**Comando:**

`nano /var/www/wordpress/wp-config.php` (ou `/var/www/html/wp-config.php`)

**Código a ser inserido (ACIMA da linha "That's all, stop editing"):**

PHP

```php
// Forçar HTTPS e IP Dinâmico para Clones
$_SERVER['HTTPS'] = 'on';
define( 'FORCE_SSL_ADMIN', true );
define( 'WP_HOME', 'https://' . $_SERVER['HTTP_HOST'] );
define( 'WP_SITEURL', 'https://' . $_SERVER['HTTP_HOST'] );

// Configurações de Performance e Escrita
define( 'WP_CACHE', false ); 
define( 'FS_METHOD', 'direct' );
```

### Fase C: Linha de Montagem de Clones

1. Desligar o container `WP-Squad1`.
    
2. Clicar com o botão direito -> **Clone**.
    
3. Selecionar **Full Clone**.
    
4. Após clonar, alterar o IP na aba **Network** conforme a tabela do inventário.
    

## 4. Comandos Úteis de Terminal (CLI)

- **Reiniciar o Apache (Servidor Web):** `systemctl restart apache2`
    
- **Criar novo usuário admin via CLI:**
    
    `wp user create mestre mestre@escola.local --role=administrator --user_pass=admin123 --allow-root`
    
- **Limpar cache do WordPress:** `wp cache flush --allow-root`
    

---

**Documento atualizado em:** 09/06/2025

**Responsável:** Professor Olegário Neto & Equipe de Infraestrutura
