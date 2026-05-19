# Copilot Angular Skillset (v20 → v21)

Sada konfiguračních souborů, které dramaticky zvedají kvalitu GitHub Copilotu
na velkých Angular projektech (v20 s plánem na v21). Funguje ve VS Code
i ve WebStormu / IntelliJ (JetBrains Copilot plugin).

## Co to obsahuje

```
.github/
├── copilot-instructions.md            # globální kontext repa (vždy se načítá)
├── instructions/                      # path-scoped pravidla (auto dle glob)
│   ├── components.instructions.md
│   ├── templates.instructions.md
│   ├── services.instructions.md
│   ├── stores.instructions.md
│   ├── routing.instructions.md
│   ├── guards-resolvers.instructions.md
│   ├── pipes-directives.instructions.md
│   ├── styles.instructions.md
│   └── tests.instructions.md
└── prompts/                           # reusable slash commandy
    ├── new-component.prompt.md
    ├── new-feature.prompt.md
    ├── convert-to-signals.prompt.md
    ├── migrate-to-control-flow.prompt.md
    ├── migrate-ngmodule-to-standalone.prompt.md
    ├── refactor-to-inject.prompt.md
    ├── add-unit-tests.prompt.md
    ├── review-component.prompt.md
    ├── fix-onpush-issues.prompt.md
    └── angular-21-upgrade.prompt.md
.copilotignore                         # co Copilot NEMÁ indexovat
```

## Jak to nainstalovat do existujícího projektu

1. Zkopíruj celou složku `.github/` do kořene svého Angular repa.
   (Pokud už máš `.github/` s workflows, jenom přidej `copilot-instructions.md`,
    `instructions/` a `prompts/`.)
2. Zkopíruj `.copilotignore` do kořene repa.
3. Otevři `.github/copilot-instructions.md` a uprav sekce v hranatých závorkách
   `[ ... ]` — state mgmt, styling, testing framework atd. — podle reality tvého projektu.
4. Projdi jednotlivé `instructions/*.md` soubory a sluč s tvými konvencemi.
5. (WebStorm) restartuj IDE, aby Copilot načetl nové instrukce.

## Jak to používat

### Globální instrukce
`copilot-instructions.md` se načítá automaticky pro každou Copilot interakci
(inline completion, chat, edit mode, agent mode). Nemusíš dělat nic.

### Path-scoped instrukce
Soubory v `instructions/` mají frontmatter `applyTo: "<glob>"`. Copilot je
automaticky aktivuje, když pracuješ v souboru, který matchuje glob.

### Prompt files
V Copilot Chat napiš `/<jméno-promptu>`, např. `/convert-to-signals`.
Můžeš je volat na vybraný soubor / složku / selection.

### Doporučený workflow pro migraci v20 → v21

1. Spusť `/angular-21-upgrade` v chatu (Agent mode) — projde závislosti, build config.
2. Po feature folderech spusť `/migrate-ngmodule-to-standalone`, pak
   `/migrate-to-control-flow`, pak `/convert-to-signals`, pak `/refactor-to-inject`.
3. Po každém kroku `/add-unit-tests` na změněné komponenty.
4. `/review-component` jako finální QA.

## Dodatečné doporučení

- **Přepni model** v Copilot Chat na **Claude Sonnet 4.6** nebo **GPT-5** —
  pro Angular refactory je výrazně lepší než default.
- **Agent mode > inline completions** pro multi-file migrace.
- **MCP servers**: přidej `context7` pro fresh Angular/RxJS docs a Angular CLI MCP.
- **.copilotignore** drasticky čistí kontext — vyhoď `dist/`, generated, vendored libs.

## Licence

MIT — používej, forkuj, uprav. Žádná atribuce nutná.
