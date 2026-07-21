# Configuração MCP — OZI-UI (Windows)

Guia para configurar MCP servers no Claude Code para este projeto.

---

## O que é MCP

MCP (Model Context Protocol) permite conectar o Claude Code a ferramentas externas: bancos de dados, sistemas de arquivos, APIs, etc. Os servidores MCP são configurados no `settings.json` do projeto.

---

## Localização dos arquivos de configuração

| Arquivo | Escopo | Localização |
|---|---|---|
| `settings.json` | Projeto | `E:/xampp/www/ozi-ui/ozi-ui-pkg/.claude/settings.json` |
| `settings.local.json` | Local (não commitar) | `E:/xampp/www/ozi-ui/ozi-ui-pkg/.claude/settings.local.json` |
| `settings.json` global | Usuário | `C:/Users/Thomaz/.claude/settings.json` |

---

## Instalação do Claude Code (se necessário)

```powershell
npm install -g @anthropic-ai/claude-code
```

Verificar instalação:
```powershell
claude --version
```

---

## MCPs úteis para este projeto

### Filesystem (acesso a arquivos)
Permite ao Claude ler/escrever arquivos diretamente — útil para navegar entre os repos.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "E:/xampp/www/ozi"
      ]
    }
  }
}
```

### MariaDB / MySQL (acesso ao banco)
Útil para projetos de sandbox que precisam de queries ao banco.

```json
{
  "mcpServers": {
    "mysql": {
      "command": "npx",
      "args": ["-y", "@benborla29/mcp-server-mysql"],
      "env": {
        "MYSQL_HOST": "127.0.0.1",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "root",
        "MYSQL_PASS": "",
        "MYSQL_DB": "nome_do_banco"
      }
    }
  }
}
```

---

## Verificar MCPs ativos

No Claude Code, rodar:
```
/mcp
```

---

## Troubleshooting

**MCP não aparece:**
- Verificar se o `settings.json` está no lugar correto (dentro do diretório raiz do projeto)
- Reiniciar o Claude Code após alterar configurações
- Verificar se o `npx` está disponível: `npx --version`

**Erro de permissão:**
- Adicionar o comando ao `allowedTools` no `settings.json`
- Ou usar `settings.local.json` para permissões locais que não vão para o git
