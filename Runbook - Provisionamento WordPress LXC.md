### 📘 Runbook: Provisionamento de Servidores Web (WordPress)

Abaixo está o conteúdo formatado para você copiar e colar em uma nova nota no Obsidian.

# Runbook: Infraestrutura de Squads - WordPress LXC

## 1. Contexto da Infraestrutura

- **Host:** Proxmox VE (Lenovo 16GB RAM)
    
- **Rede:** `20.20.0.0/20` (Máscara: `255.255.240.0`)
    
- **Gateway:** `20.20.0.1` (Padrão da Escola)
    
- **DNS:** `8.8.8.8` / `8.8.4.4`

- Senha: serverlab
    

## 2. Inventário de IPs (Planejamento)

| **Unidade**    | **Nome Proxmox** | **Endereço IP** |
| -------------- | ---------------- | --------------- |
| Servidor Host  | `pve-lab`        | `20.20.0.200`   |
| Servidor Host2 | `pve`            | `20.20.0.201`   |
| Squad 1        | `WP-Squad1`      | `20.20.0.210`   |
| Squad 2        | `WP-Squad2`      | `20.20.0.215`   |
| Squad 3        | `WP-Squad3`      | `20.20.0.220`   |
| Squad 4        | `WP-Squad4`      | `20.20.0.225`   |
| Squad 5        | `WP-Squad5`      | `20.20.0.230`   |
| Squad 6        | `WP-Squad6`      | `20.20.0.235`   |
| Squad 7        | `WP-Squad7`      | `20.20.0.240`   |
- login adm wordpress: https://20.20.0.210/wp-login.php
- senha: 
  ```
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

```
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

**Documento atualizado em:** 08/05/2026

**Responsável:** Professor Olegário Neto & Equipe de Infraestrutura