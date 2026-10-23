# Banco de Dados Chave-Valor Open Source: Valkey

Documentação do ambiente instalado para trabalho em grupo.

## O que é Valkey

**Valkey** é um banco de dados chave-valor (key-value store) de alto desempenho,
open source (licença BSD-3-Clause). É um fork do Redis, criado após a Redis Inc.
alterar a licença do Redis para SSPL/RSALv2 (não aprovada pela OSI).

- Site: https://valkey.io/
- Repositório: https://github.com/valkey-io/valkey
- Compatível 100% com o protocolo Redis (mesmos comandos e clientes)

## Instalação no Linux (CachyOS / Arch)

```bash
sudo pacman -S valkey
sudo systemctl enable --now valkey
```

Verificar:
```bash
redis-cli ping   # → PONG
```

## Comandos básicos (CLI)

```bash
# Conectar
redis-cli

# Inserir / atualizar
SET nome "João"
SET idade 25

# Consultar
GET nome        # → "João"
GET idade       # → "25"

# Verificar existência
EXISTS nome     # → 1 (existe) / 0 (não existe)

# Excluir
DEL nome

# Listar todas as chaves
KEYS *

# Definir expiração (segundos)
SETEX sessao 3600 "token123"

# Incrementar
INCR contador

# Hash (objeto)
HSET usuario:1 nome "Maria" idade 30
HGET usuario:1 nome   # → "Maria"
HGETALL usuario:1

# Lista
LPUSH fila "item1"
RPOP fila

# Conjunto
SADD tags "redis" "nosql"
SMEMBERS tags
```

## Interface Gráfica (GUI) — Open Source

### Another Redis Desktop Manager (recomendado)
- Licença: MIT (open source)
- Plataformas: Windows, Linux, macOS
- Download: https://github.com/qishibo/AnotherRedisDesktopManager/releases

Conexão padrão:
- Host: `localhost`
- Porta: `6379`
- Senha: (vazia, se não configurada)

### Tiny RDM
- Licença: MIT
- Plataformas: Windows, Linux, macOS
- Site: https://redis.tinycraft.cc/

### kvdb-gui
- Open source
- Plataformas: Windows, Linux, macOS
- Repositório: https://github.com/yizhentec/kvdb-gui

## Configuração

Arquivo de configuração: `/etc/valkey/valkey.conf`

Configurações comuns:
```conf
# Porta
port 6379

# Bind (escutar em todas as interfaces)
bind 0.0.0.0

# Senha
requirepass minha_senha

# Persistência (RDB)
save 900 1
save 300 10
save 60 10000

# Memória máxima
maxmemory 256mb
maxmemory-policy allkeys-lru
```

Após alterar, reiniciar:
```bash
sudo systemctl restart valkey
```

## Serviço systemd

```bash
# Iniciar
sudo systemctl start valkey

# Parar
sudo systemctl stop valkey

# Reiniciar
sudo systemctl restart valkey

# Status
sudo systemctl status valkey

# Habilitar no boot
sudo systemctl enable valkey

# Desabilitar no boot
sudo systemctl disable valkey

# Logs
sudo journalctl -u valkey -f
```

## Arquitetura

```
┌─────────────┐     TCP :6379     ┌──────────────┐
│   Cliente   │ ◄──────────────► │   Valkey     │
│  (CLI/GUI)  │                  │  (servidor)  │
└─────────────┘                  └──────┬───────┘
                                        │
                                 ┌──────┴───────┐
                                 │  Memória     │
                                 │  + Disco     │
                                 │  (RDB/AOF)   │
                                 └──────────────┘
```

## Tipos de dados suportados

| Tipo | Descrição | Exemplo |
|---|---|---|
| String | Texto ou binário | `SET chave "valor"` |
| Hash | Objeto (campo→valor) | `HSET obj campo valor` |
| List | Lista ordenada | `LPUSH lista item` |
| Set | Conjunto único | `SADD set item` |
| Sorted Set | Conjunto ordenado por score | `ZADD zset 1 item` |
| Stream | Log de eventos | `XADD stream * campo valor` |
| Bitmap | Array de bits | `SETBIT bitmap 0 1` |
| HyperLogLog | Contagem aproximada | `PFADD hll item` |

## Backup e restauração

```bash
# Backup manual (RDB)
redis-cli BGSAVE

# O arquivo dump.rdb fica em:
ls /var/lib/valkey/dump.rdb

# Restaurar: copiar dump.rdb para /var/lib/valkey/ e reiniciar
sudo systemctl stop valkey
sudo cp backup.rdb /var/lib/valkey/dump.rdb
sudo chown valkey:valkey /var/lib/valkey/dump.rdb
sudo systemctl start valkey
```

## Comparação com o projeto anterior

| | Projeto anterior (kv-server) | Valkey |
|---|---|---|
| Licença | MIT (código próprio) | BSD-3-Clause (open source) |
| Persistência | ❌ Memória apenas | ✅ Disco (RDB + AOF) |
| GUI | ❌ CLI apenas | ✅ Múltiplas GUIs |
| Produção | ❌ Acadêmico | ✅ Produção |
| Compatibilidade | Protocolo próprio | Protocolo Redis (universal) |
| Clientes | 1 (compilado) | Centenas (todas linguagens) |

## Links úteis

- Documentação oficial: https://valkey.io/docs/
- Comandos: https://valkey.io/commands/
- GitHub: https://github.com/valkey-io/valkey
- Redis Insight (GUI oficial): https://redis.io/insight/
- Another Redis Desktop Manager: https://github.com/qishibo/AnotherRedisDesktopManager
