# AGENTS.md — externí integrace pro Copilot Agent mode

Tento dokument popisuje, jak nastavit prostředí, aby prompty z `.github/prompts/`,
které sahají na **externí systémy** (Jira, GitHub PR, …), fungovaly.

Pravidlo: **každý uživatel má vlastní credentials.** Nikdy je necommituj
do repa. Patří do shell profilu nebo do `.envrc` (gitignored).

---

## Jira (`/from-jira`)

`/from-jira` načte ticket přes Jira REST API v3 a vrátí plán implementace.
**Nepíše kód** — to dělají až follow-up prompty (`/new-component`, `/new-feature`, …).

### 1. Vygeneruj Atlassian API token

1. https://id.atlassian.com/manage-profile/security/api-tokens
2. **Create API token** → opiš si ho (zobrazuje se jen jednou).
3. Token je vázaný na tvůj Atlassian účet, ne na konkrétní projekt. Práva má
   stejná jako tvůj uživatel.

### 2. Nastav env proměnné

Do `~/.zshrc` / `~/.bashrc`:

```bash
export JIRA_BASE_URL="https://<workspace>.atlassian.net"
export JIRA_EMAIL="you@example.com"
export JIRA_API_TOKEN="<token-z-bodu-1>"
```

Pak `source ~/.zshrc` (nebo nový terminál). Ověř:

```bash
curl -s -u "$JIRA_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/myself" | head -c 200
```

Měl bys vidět JSON s tvým profilem. 401 = špatný email/token. 403 = token
nemá přístup k API. 404 = špatné `JIRA_BASE_URL`.

### 3. (Volitelně) Atlassian CLI

`/from-jira` umí použít [oficiální Atlassian CLI (`acli`)](https://developer.atlassian.com/cloud/acli/),
když je nainstalovaný — je to pohodlnější než `curl`, ale není povinný.

```bash
brew install atlassian/acli/acli
acli auth login
acli jira workitem view PROJ-123
```

Pokud `acli` chybí, prompt automaticky padá na `curl`.

### 4. (Volitelně) `jq`

Strukturovaný výpis JSONu z `curl`. Bez něj to taky funguje, ale je to ošklivější.

```bash
brew install jq
```

---

## GitHub PR (`/review-pr`)

Vyžaduje [`gh` CLI](https://cli.github.com/).

```bash
brew install gh
gh auth login
```

Pro privátní repa potřebuješ scope `repo`. `gh auth status` ověří stav.

---

## Bezpečnostní pravidla

- **Token nikdy nepiš do chatu Copilota.** Prompty používají jen `$JIRA_API_TOKEN`
  reference, takže token nikdy nejde do logu konverzace.
- **Nepatří do repa.** `.envrc`, `.env*` musí být v `.gitignore`. `direnv`
  je doporučená cesta pro per-projekt env (nainstaluj přes `brew install direnv`).
- **Token revokuj**, pokud opouštíš projekt nebo zařízení.
- **Read scope stačí.** `/from-jira` jen čte. Když chceš někdy psát komentáře
  zpátky do Jiry, vyrob si **druhý** token s tím účelem a používej ho jen
  ve write promptech (zatím žádný v této sadě není).

---

## MCP varianta (nepovinná)

Pokud chceš jít přes [Model Context Protocol](https://modelcontextprotocol.io/)
místo CLI, Copilot Chat (VS Code 1.95+) MCP podporuje. Atlassian má oficiální
MCP server — viz https://www.atlassian.com/platform/remote-mcp-server.

Konfigurace v `.vscode/mcp.json` repa (commitnutelné, bez tokenů):

```json
{
  "servers": {
    "atlassian": {
      "command": "npx",
      "args": ["-y", "@atlassian/mcp-server"],
      "env": {
        "ATLASSIAN_SITE_URL": "${input:atlassian_site}",
        "ATLASSIAN_API_TOKEN": "${input:atlassian_token}",
        "ATLASSIAN_EMAIL": "${input:atlassian_email}"
      }
    }
  },
  "inputs": [
    { "id": "atlassian_site", "type": "promptString", "description": "https://<workspace>.atlassian.net" },
    { "id": "atlassian_email", "type": "promptString", "description": "your atlassian email" },
    { "id": "atlassian_token", "type": "promptString", "description": "API token", "password": true }
  ]
}
```

Když je MCP nakonfigurované, prompty z této sady ho **použijí přednostně**
(Copilot Agent mode si MCP toolset načte automaticky) a `curl` fallback se
neuplatní. Funkčně se report neliší.

---

## Co když nic z toho nemám

- `/from-jira` bez tokenu = nepoužitelné. Není fallback "vlož popis ticketu do
  chatu" — to už je obyčejné `/new-feature` s popisem v Copilot Chat.
- `/review-pr` bez `gh` = nepoužitelné. Použij `/review-branch` po lokálním
  fetchnutí PR větve: `git fetch origin pull/<num>/head:pr-<num>`.
- Všechny ostatní prompty (refactory, scaffolding, code review jednoho
  souboru) externí přístup **nepotřebují**.
