# TRAE-Skills

A collection of skill files for [Trae IDE](https://trae.ai/) that enhance Vue 3 / Vite / Pinia based frontend development workflows.

You can put your agent skills into your project main path to ".trae" folder

## Skills

### 1. agent-browser

**Purpose**: Browser automation CLI for UI testing, E2E tests, and page interaction.

**Use cases**:
- Test UI components and forms
- Automated E2E testing without heavy frameworks
- Page interaction testing
- Screenshot capture for visual verification

**Setup**:
```bash
npm install -g agent-browser
agent-browser install
```

**Key features**:
- Lightweight alternative to Playwright
- Accessibility tree-based element selection
- Snapshot inspection for element refs
- Keyboard and mouse automation
- Screenshot capture

---

### 2. vue-i18n

**Purpose**: Internationalization implementation guide for vue-i18n v9 with Vue 3 Composition API.

**Use cases**:
- Adding new user-facing text to components
- Creating new locale files (tr/en/ro)
- Translating hardcoded strings
- Date/number formatting per locale
- Language switching

**Key concepts**:
- Translation function: `t('key.path')`
- Interpolation: `t('key', { field: 'value' })`
- Pluralization: `{n} | {n}` format
- Date formatting: `d(date, 'short')`
- Number formatting: `n(amount, 'currency')`

**File structure**:
```
src/
├── i18n/
│   ├── i18n.js
│   ├── dateTimeFormats.js
│   └── numberFormats.js
└── locales/
    ├── tr/
    ├── en/
    └── ro/
```

---

### 3. vuelidate-i18n

**Purpose**: Form validation with Vuelidate and internationalized error messages.

**Use cases**:
- Adding validation rules to forms
- Creating localized validation messages
- Custom validator development
- Error message display patterns

**Built-in validators**:
- `required`, `email`, `minLength`, `maxLength`
- `minValue`, `maxValue`, `sameAs`
- `requiredIf`, `numeric`, `integer`

**Error message mapping**:
| Validator | i18n Param |
|-----------|------------|
| `minLength(8)` | `{min_length: 8}` |
| `minValue(0)` | `{min_value: 0}` |
| `maxLength(50)` | `{max_length: 50}` |

---

## Installation

Copy the skill files to your Trae configuration directory:

```
~/.trae/skills/
├── agent-browser/
│   └── SKILL.md
├── vue-i18n/
│   └── SKILL.md
└── vuelidate-i18n/
    └── SKILL.md
```

Or reference them in your project's `.trae/skills/` directory for team sharing.

---

## Usage in Trae

When you invoke these skills, Trae will provide context-aware guidance based on the skill files.

Example prompts:
- "Login sayfasına validation ekle"
- "Yeni bir locale dosyası oluştur"
- "Ürün ekleme formunu test et"
- "Bu component'e çeviri anahtarları ekle"

---

## Target Stack

These skills are designed for projects using:

- **Vue 3.5+** (Composition API & Options API)
- **Pinia 3.x** (State management)
- **Vue Router 4** (Routing)
- **Vite 7** (Build tool)
- **Vue-i18n** (Internationalization)
- **Vuelidate** (Form validation)

---

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

---

## License

This project is licensed under the **MIT License** - see the LICENSE file for details.

---

## Credits

Created for Vue 3 / Vite / Pinia enterprise applications in any domain.
