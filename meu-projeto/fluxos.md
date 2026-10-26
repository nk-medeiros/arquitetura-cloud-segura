# Fluxos de Comunicação

## Fluxos entre componentes

| # | Origem | Destino | Protocolo | Porta | Finalidade | Status |
|---|---|---|---|---|---|---|
| 1 | Cliente (Internet) | WAF + ALB | HTTPS | 443 | Acesso à aplicação | Permitido |
| 2 | ALB | App/API (privada) | HTTP | 8080 | Encaminhar requisições | Permitido |
| 3 | App/API | RDS | TCP | 5432 | Consultas ao banco | Permitido |
| 4 | App/API | S3 via VPC Endpoint | HTTPS | 443 | Upload/download de arquivos | Permitido |
| 5 | App/API | Secrets Manager | HTTPS | 443 | Ler credenciais do banco | Permitido |
| 6 | App/API | CloudWatch Logs | HTTPS | 443 | Enviar logs | Permitido |
| 7 | Administrador | Client VPN | UDP/TCP | 443 | Conexão segura de manutenção | Permitido |
| 8 | Client VPN | App (SSH) | TCP | 22 | Manutenção (só do SG da VPN) | Permitido |
| 9 | Client VPN | RDS | TCP | 5432 | Manutenção do banco | Permitido |
| 10 | App (privada) | Internet via NAT | HTTPS | 443 | Atualizações de pacotes | Permitido |
| 11 | Internet | App/API | Qualquer | Qualquer | Acesso direto | **Bloqueado** |
| 12 | Internet | RDS | Qualquer | Qualquer | Acesso direto | **Bloqueado** |
| 13 | Internet | SSH/RDP | TCP | 22 / 3389 | Acesso remoto | **Bloqueado** |
| 14 | Internet | S3 (acesso público) | HTTPS | 443 | Leitura pública | **Bloqueado** (Block Public Access) |
| 15 | RDS | Internet | Qualquer | Qualquer | Saída | **Bloqueado** |

## Regras dos Security Groups

| Security Group | Entrada permitida | Saída |
|---|---|---|
| `sg-alb` | 443 de `0.0.0.0/0` | 8080 para `sg-app` |
| `sg-app` | 8080 de `sg-alb`; 22 de `sg-vpn` | 5432 para `sg-rds`; 443 para endpoints e NAT |
| `sg-rds` | 5432 de `sg-app` e de `sg-vpn` | Nenhuma |
| `sg-vpn` | 443 do IP/rede do administrador | 22 para `sg-app`; 5432 para `sg-rds` |