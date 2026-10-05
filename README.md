# TRAE-Skills

A skill library following the open [Agent Skills](https://agentskills.io/) standard. Authored and tuned for [Trae](https://trae.ai/), portable as-is to Claude Code, Codex CLI, Cursor, Gemini CLI and OpenCode.

Every skill is a directory containing a `SKILL.md` file. Nothing else is required.

The library started as a frontend-only collection and is growing beyond it. Contributions are welcome — see [Contributing](#contributing).

## Repository layout

```
.agents/skills/
├── agent-browser/SKILL.md      # frontend
├── vue-i18n/SKILL.md           # frontend
├── vuelidate-i18n/SKILL.md     # frontend
└── laravel-tdd/SKILL.md        # backend
```

`.agents/skills/` is the vendor-neutral discovery path. Categories live in each skill's `metadata.category` frontmatter field rather than in the directory tree, because the spec requires `name` to match the parent directory exactly.

## Catalogue

### Frontend

| Skill            | Purpose                                                       | Trigger                                                                                      |
| ---------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `agent-browser`  | Drives a real Chrome instance through the `agent-browser` CLI | Verifying a running UI, reproducing a frontend bug, lightweight E2E checks                   |
| `vue-i18n`       | vue-i18n v9 internationalization for Vue 3 Composition API    | Adding or translating text, creating locale files, pluralization, date and number formatting |
| `vuelidate-i18n` | Vuelidate validation with localized error messages            | Writing validation rules, custom validators, wiring validation state into inputs             |

### Backend

| Skill         | Purpose                                                       | Trigger                                                                                  |
| ------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `laravel-tdd` | Red-green-refactor workflow for Laravel with PHPUnit and Pest | Writing or refactoring Laravel tests, adding endpoints or models, fixing bugs test-first |

## Installation

### Trae

Trae reads `.agents/skills/` natively once the directory is enabled.

1. Open **Settings → Skills & Commands**.
2. Under **Import Settings**, toggle on **Enable .agents Skills Directory**.

To install into a project or globally instead, copy the skill directories:

```bash
# Project scope
cp -R .agents/skills/laravel-tdd /path/to/project/.trae/skills/

# Global scope
cp -R .agents/skills/laravel-tdd ~/.trae/skills/
```

A skill in `.trae/skills/` takes priority over one of the same name in `.agents/skills/`.

### Claude Code

```bash
# Project scope
cp -R .agents/skills/laravel-tdd /path/to/project/.claude/skills/

# Global scope
cp -R .agents/skills/laravel-tdd ~/.claude/skills/
```

### Codex CLI, Gemini CLI, OpenCode

These already read `.agents/skills/` directly. For global scope:

```bash
cp -R .agents/skills/laravel-tdd ~/.agents/skills/
```

### Using the skills CLI

```bash
# List available skills
npx skills add AlperGuven/TRAE-Skills --list

# Install one skill for a specific agent
npx skills add AlperGuven/TRAE-Skills --skill laravel-tdd -a trae
npx skills add AlperGuven/TRAE-Skills --skill vue-i18n -a claude-code

# Install globally
npx skills add AlperGuven/TRAE-Skills --skill agent-browser -g
```

### Import through the Trae UI

**Settings → Skills & Commands → Create**, then upload the `SKILL.md` file or a zip of the skill directory.

## Usage

Invoke by natural language, by `#skill-name` in the Trae chat box, or by referencing the file with `@`:

- "Login sayfasına validation ekle" → `vuelidate-i18n`
- "Yeni bir locale dosyası oluştur" → `vue-i18n`
- "Ürün ekleme formunu test et" → `agent-browser`
- "Write a failing test for the project create endpoint" → `laravel-tdd`

Trae scans every skill's `name` and `description` at session start, then loads the full body only when a task matches. This is why descriptions state both what a skill does and when to use it.

## Authoring conventions

Rules enforced across this library:

| Rule                                                                                                       | Reason                                                               |
| ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `name` matches the directory name, kebab-case, max 64 chars                                                | Required by the spec; a mismatch makes the skill unloadable          |
| `description` max 1024 chars, third person, states what **and** when                                       | It is the only text the agent sees before deciding to load the skill |
| No angle brackets in frontmatter                                                                           | They get misread as markup when injected into the system prompt      |
| Only spec frontmatter keys: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools` | Unknown keys are rejected by validators                              |
| Body under 500 lines                                                                                       | The body is loaded whole into context on activation                  |
| Skill written in English                                                                                   | Keeps it usable across agents and locales                            |
| `scripts/`, `references/`, `assets/` stay one level deep                                                   | Flat layout keeps on-demand resources predictable to locate          |

Heavy detail belongs in a `references/` file linked from the body, not in `SKILL.md`.

### Security

Never commit credentials into a skill. Examples that need logins read them from the environment:

```bash
HOME=/tmp agent-browser fill input[type='email'] "$E2E_ADMIN_EMAIL"
```

## Target stack

**Frontend** — Vue 3.5+, Pinia 3.x, Vue Router 4, Vite 7, vue-i18n v9, Vuelidate

**Backend** — PHP 8.5, Laravel 12, PHPUnit 12, Pest, Larastan 3, Rector 2

## Contributing

1. Create `.agents/skills/<skill-name>/SKILL.md`.
2. Fill in `name`, `description`, `license` and `metadata.category`.
3. Keep the body under 500 lines and split deep reference material into `references/`.
4. Add the skill to the catalogue table above.

## License

MIT — see [LICENSE](LICENSE).
