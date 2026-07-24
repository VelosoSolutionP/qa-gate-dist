# QA-Gate

### Pare de mandar bug pra homologação. Mate no commit. 🚦

**Simulação em browser real antes de cada commit.** Injeta o bug, exige que o sistema responda com mensagem amigável no DOM real e console limpo. Só deixa commitar no verde.

> Teste unitário passa e a tela quebra mesmo assim — hidratação errada, lib JS que não subiu, mensagem de erro que não aparece no modal. QA-Gate fecha essa brecha exercendo o fluxo num Chrome de verdade, do jeito que o usuário faz.

**O que você ganha:**
- ✅ **Bug de UI barrado no commit** — não chega no QA nem na homologação
- 🔒 **Governança automática** — commit, branch, teste e documentação no mesmo padrão, sem depender de disciplina manual
- 🧠 **Contexto preservado** — a IA carrega arquitetura, gotchas e feedback do seu projeto a cada sessão, sem re-explicar
- 🧪 **Teste unitário obrigatório** — tocou código de produção, tem teste cobrindo; senão não commita
- 🌐 **Simulação em Chrome real** — pega lib quebrada e hidratação errada que o teste unitário deixa passar
- 🔐 **Licença offline** — seu código nunca sai da sua máquina
- 💸 **Menos retrabalho, menos bug em produção** — custo previsível

Feito para o **Claude Code**. Núcleo portável a outros agentes/CI via **MCP**.

Por [DevPoint Innovation](https://devpointinnovation.com.br/) — Consultoria em IA Aplicada. Empresa sólida, +12 anos de mercado.

> ℹ️ Este repositório é o **artefato de distribuição** (código empacotado). Requer **licença** para rodar.

---

## Suporte a assistentes de IA (Claude / ChatGPT / Gemini)

O QA-Gate é **agnóstico de assistente** no que importa: o gate roda no `git commit` (pre-commit) e não liga pra qual IA escreveu o código — quem commitar código ruim é barrado, seja qual for o assistente.

| Camada | Claude Code | ChatGPT | Gemini | Cursor / outros |
|---|---|---|---|---|
| **MCP** (tools `qa_run_gate`, `qa_simulate`…) | ✅ nativo | ✅ via cliente MCP | ✅ via Gemini CLI (MCP) | ✅ qualquer cliente MCP |
| **Gate no pre-commit (CLI)** | ✅ | ✅ agnóstico | ✅ agnóstico | ✅ |
| **Governança de fluxo** (muro de tarefa, guard de branch/commit, timer) | ✅ hooks nativos | 🟡 via git hook | 🟡 via git hook | 🟡 via git hook |

- **Nativo total:** Claude Code (plugin + hooks + MCP).
- **Demais assistentes:** consomem o **MCP** (as tools) e/ou o **pre-commit hook** (`node qa-gate.mjs`) — a qualidade e o bloqueio continuam valendo porque o enforcement é no **git**, não na IA.
- **Licença offline** em todos: o código nunca sai da máquina.

---

## Requisitos

- **Node 18+**
- **Google Chrome** instalado (usa o Chrome do sistema — não baixa chromium)
- App rodando local com dados (Docker de paridade ou `php artisan db:seed`)
- Uma **licença** (chave) — em [devpointinnovation.com.br](https://devpointinnovation.com.br/)

---

## Instalação

### Opção A — Plugin do Claude Code (recomendado)

```
# no Claude Code
/plugin marketplace add VelosoSolutionP/qa-gate-dist
/plugin install qa-gate
```

Depois configure a licença e o projeto:

```bash
export QA_GATE_LICENSE="<sua-chave>"
```
```
# no Claude Code
/qa-init      # detecta o stack e cria config + hook + seeder
/qa-gate      # roda a simulação
```

### Opção B — CLI / hook (qualquer editor ou CI)

```bash
git clone https://github.com/VelosoSolutionP/qa-gate-dist.git qa-gate
cd qa-gate && npm install        # instala as dependências (playwright etc.)
export QA_GATE_LICENSE="<sua-chave>"
```

No seu projeto, crie `qa-gate.config.json` (ver abaixo) e o hook de pre-commit:

```bash
# .githooks/pre-commit
node /caminho/para/qa-gate/qa-gate.mjs

git config core.hooksPath .githooks
```

A partir daí, cada `git commit` roda o gate. Backend puro pula; UI tocada simula.

### MCP (motor de simulação)

Aponte o MCP do seu agente para o servidor:

```json
{
  "mcpServers": {
    "qa-gate": {
      "command": "node",
      "args": ["/caminho/para/qa-gate/mcp/server.mjs"],
      "env": { "QA_GATE_LICENSE": "<sua-chave>" }
    }
  }
}
```

### Hooks (governança)

O plugin auto-descobre os hooks. Para setup manual, use `hooks/settings.example.json` como base (aponte os comandos para os `hooks/*.mjs` deste artefato).

---

## Licença

Exige uma chave válida (`QA_GATE_LICENSE`). Validação **offline** (assinatura Ed25519) — não manda seu código pra lugar nenhum, não precisa de internet a cada run. A chave **expira** (ex.: trial de 30 dias). Compre / renove em [devpointinnovation.com.br](https://devpointinnovation.com.br/).

---

## Configuração da empresa — `qa-gate.company.json`

Define o padrão de branch/commit/documentação da sua empresa. Sem o arquivo, vale o default embutido (quem não configurar não quebra). Resolução: env `QA_GATE_COMPANY_CONFIG` → `qa-gate.company.json` subindo do repo → `~/.qa-gate/company.json` → default.

```json
{
  "autor": "nome.sobrenome",
  "branchPattern": "<tipo>/<autor>/<numero>",
  "commitScope": "numero",
  "tipos": ["fix", "feat", "perf", "refactor", "chore", "test", "docs"],
  "doc": { "template": "redmine", "required": true },
  "notify": {
    "whatsapp": { "enabled": false, "provider": "callmebot", "phone": "xxx", "apikey": "xxx", "helpTimeoutMin": 15 }
  }
}
```

`notify.whatsapp` (opt-in): comprovante do gate + pedido de ajuda por WhatsApp. Deixe `enabled:false` para não usar.

---

## Configuração do gate — `qa-gate.config.json`

Na raiz do seu projeto:

```json
{
  "baseUrl": "http://localhost:9080",
  "healthPath": "/login",
  "chromeChannel": "chrome",
  "login": {
    "path": "/login",
    "email": "qa.gate@seu-dominio.local",
    "password": "SuaSenhaQA",
    "emailSel": "input[name=email]",
    "passSel": "input[name=password]",
    "submitSel": "form[action*=login] [type=submit]"
  },
  "uiGlobs": ["**/*.blade.php", "resources/views/**", "app/Http/Livewire/**", "resources/js/**"],
  "flows": [
    { "name": "cliente-create", "watch": ["**cliente**"], "path": "/clientes/create", "submitText": "Salvar", "expectFriendlyError": true },
    { "name": "cliente-lista", "watch": ["**ClienteTable**"], "path": "/clientes", "mode": "read", "expectSelector": "table tbody tr", "expectMinCount": 1 }
  ]
}
```

- **form** (default): injeta submit inválido/vazio e exige mensagem amigável. Para formulários/modais.
- **read**: sem submit — carrega a rota e exige conteúdo renderizado + console limpo. Para listagem/visualização.

---

## Como funciona

```
git commit
   │
   ▼
diff staged → tocou UI? ── não ──► pula browser ✔ (backend puro)
   │ sim
   ▼
app no ar? ── não ──► bloqueia (a IA sobe o ambiente e re-tenta)
   │ sim
   ▼
Chrome real: login → abre fluxo → injeta bug (submit vazio)
   │
   ▼
mensagem amigável visível? + console SEVERE=0 + zero request falho?
   ├─ sim → VERDE ✔ commita
   └─ não → VERMELHO ✖ bloqueia + screenshot da falha
```

---

## Comandos (plugin)

| Comando | O que faz |
|---|---|
| `/qa-init` | Detecta o stack e cria `qa-gate.config.json`, hook e seeder |
| `/qa-gate` | Roda o gate no diff staged e mostra verde/vermelho + screenshot |

## Tools (MCP)

| Tool | O que faz |
|---|---|
| `qa_check_app` | App local está no ar? |
| `qa_list_flows` | Lista os fluxos configurados |
| `qa_simulate` | Simula UM fluxo (injeta bug) + screenshot inline |
| `qa_run_gate` | Gate completo a partir do diff staged (igual ao pre-commit) |

---

## Emergência

`git commit --no-verify` burla o gate. Use só em emergência real.

---

© DevPoint Innovation. Uso mediante licença. · [devpointinnovation.com.br](https://devpointinnovation.com.br/)
