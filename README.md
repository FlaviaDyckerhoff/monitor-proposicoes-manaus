# 🏛️ Monitor Proposições — CMM (Câmara Municipal de Manaus)

Monitora automaticamente a API SAPL da Câmara Municipal de Manaus e envia email quando há proposições novas. Roda **4x por dia** via GitHub Actions (8h, 12h, 17h e 21h, horário de Brasília).

---

## Como funciona

1. O GitHub Actions roda o script nos horários configurados
2. O script chama a API REST pública da CMM (`sapl.cmm.am.gov.br/api`)
3. Compara as proposições recebidas com as já registradas no `estado.json`
4. Se há proposições novas → envia email com a lista organizada por tipo
5. Salva o estado atualizado no repositório

---

## Estrutura do repositório

```
monitor-proposicoes-manaus/
├── monitor.js
├── package.json
├── estado.json
├── README.md
└── .github/
    └── workflows/
        └── monitor.yml
```

---

## Setup

### PARTE 1 — Gmail App Password

> Se já tem uma senha de app de outro monitor, pode reutilizá-la. Pule para a Parte 2.

1. Acesse [myaccount.google.com/security](https://myaccount.google.com/security)
2. Confirme que **Verificação em duas etapas** está ativa
3. Busque por **"Senhas de app"**, crie com o nome `monitor-cmm`
4. Copie a senha de 16 letras — aparece só uma vez

### PARTE 2 — Criar repositório

1. [github.com](https://github.com) → **+ → New repository**
2. Nome: `monitor-proposicoes-manaus` | Visibility: **Private**
3. **Create repository**

### PARTE 3 — Upload dos arquivos

1. **"uploading an existing file"** → suba `monitor.js`, `package.json`, `README.md` → Commit
2. **Add file → Create new file** → nome: `.github/workflows/monitor.yml` → cole o conteúdo → Commit

### PARTE 4 — Secrets

**Settings → Secrets and variables → Actions → New repository secret**

| Name | Valor |
|------|-------|
| `EMAIL_REMETENTE` | seu Gmail |
| `EMAIL_SENHA` | senha de 16 letras (sem espaços) |
| `EMAIL_DESTINO` | email de destino dos alertas |

### PARTE 5 — Testar

**Actions → Monitor Proposições Manaus → Run workflow → Run workflow**

Log esperado no primeiro run:
```
📊 Total de proposições em 2026: 250 (3 páginas)
📊 Total coletado: 250 proposições
🆕 Proposições novas: 250
✅ Email enviado com 250 proposições novas.
```

---

## API utilizada

```
URL Base:  https://sapl.cmm.am.gov.br/api
Endpoint:  GET /materia/materialegislativa/?ano=2026&page=1&page_size=100&o=-data_apresentacao
Docs:      https://sapl.cmm.am.gov.br/api/schema/swagger-ui/
```

Sistema SAPL 3.1 (Interlegis), API REST pública sem autenticação.

---

## Resetar o estado

1. Clique em `estado.json` → lápis → substitua por:
```json
{"proposicoes_vistas":[],"ultima_execucao":""}
```
2. Commit → rode o workflow manualmente

---

## Problemas comuns

**"Authentication failed"** → verifique `EMAIL_SENHA` sem espaços

**Workflow não aparece em Actions** → confirme que está em `.github/workflows/monitor.yml`

**"0 proposições encontradas"** → verifique `https://sapl.cmm.am.gov.br/api/materia/materialegislativa/?ano=2026&page=1&page_size=5` no browser

**Autor aparece como "-"** → comportamento esperado quando a API retorna autor como URL aninhada e está lenta; o email ainda é enviado normalmente
