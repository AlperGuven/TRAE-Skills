---
name: "vue-i18n"
description: "Vue 3 i18n implementation guide for vue-i18n. Invoke when user wants to add/translate text, create new locale files, or work with translations."
---

# Vue i18n (vue-i18n)

Internationalization implementation for Vue 3 applications using vue-i18n.

## When to Use

- Adding new user-facing text to components
- Creating new locale files (tr/en/ro)
- Translating hardcoded strings
- Working with date/number formatting
- Changing language/locale

## Project Structure

```
src/
├── i18n/
│   ├── i18n.js              # Main i18n configuration
│   ├── dateTimeFormats.js   # Date/time formatting per locale
│   └── numberFormats.js     # Number formatting per locale
└── locales/
    ├── tr/                  # Turkish (default)
    ├── en/                  # English
    └── ro/                  # Romanian
        ├── store/           # Store-related translations
        │   ├── shared.json
        │   └── admin/
        │       └── application.json
        ├── views/           # Page/view translations
        │   └── retailer/
        │       └── application.json
        ├── components/      # Component translations
        ├── constants/       # Constant translations
        └── shared/          # Shared/common translations
```

## Core Concepts

### 1. Translation Function

```vue
<script setup>
import { useI18n } from 'vue-i18n'

const { t, locale } = useI18n()

// Simple translation
const message = t('store.shared.added', { element: t('store.application.product') })
// Output (tr): "Ürün eklendi!"
</script>
```

### 2. Template Usage

```vue
<template>
  <!-- Simple key -->
  <span>{{ t('store.shared.cancel') }}</span>

  <!-- With interpolation -->
  <span>{{ t('store.shared.added', { element: 'Ürün' }) }}</span>

  <!-- Pluralization -->
  <span>{{ t('store.shared.items_count', { count: items.length }) }}</span>
</template>
```

### 3. Script Options API

```vue
<script>
export default {
  methods: {
    submit() {
      const message = this.$t('store.shared.saved', { element: this.$t('common.product') })
    }
  }
}
</script>
```

## Adding New Translations

### Step 1: Identify the Namespace

| Content Type | Location |
|--------------|----------|
| Store-related messages | `locales/{locale}/store/` |
| Page content | `locales/{locale}/views/` |
| Reusable components | `locales/{locale}/components/` |
| Shared constants | `locales/{locale}/constants/` |
| Common/shared | `locales/{locale}/shared/` |

### Step 2: Add to All Locale Files

**`locales/tr/store/application.json`** (Turkish):
```json
{
  "product": "Ürün",
  "add_product": "Ürün Ekle",
  "product_added": "{product} eklendi!"
}
```

**`locales/en/store/application.json`** (English):
```json
{
  "product": "Product",
  "add_product": "Add Product",
  "product_added": "{product} added!"
}
```

**`locales/ro/store/application.json`** (Romanian):
```json
{
  "product": "Produs",
  "add_product": "Adaugă Produs",
  "product_added": "{product} adăugat!"
}
```

### Step 3: Use in Component

```vue
<script setup>
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

const successMessage = t('store.application.product_added', {
  product: t('store.application.product')
})
</script>
```

## Interpolation & Placeholders

### Basic Interpolation

```json
{
  "welcome": "Merhaba {name}!",
  "item_count": "{count} adet ürün"
}
```

### HTML Interpolation (v-html)

```json
{
  "link_text": "Detaylı bilgi için <a href='/help'>tıklayın</a>"
}
```

```vue
<span v-html="t('link_text')"></span>
```

### List Interpolation

```json
{
  "users_online": "{0} kullanıcı çevrimiçi"
}
```

```vue
<span>{{ t('users_online', [userCount]) }}</span>
```

## Pluralization

### JSON Format

```json
{
  "item_count": "{n} ürün | {n} ürünler"
}
```

### Usage

```vue
<span>{{ t('item_count', { n: items.length }, items.length) }}</span>
```

### Named Pluralization

```json
{
  "message": "{count} adet ürün seçildi | {count} adet ürün seçildi"
}
```

## Date/Time Formatting

### Define Format

**`src/i18n/dateTimeFormats.js`**:
```js
export default {
  'tr-TR': {
    short: { year: 'numeric', month: 'short', day: 'numeric' },
    long: { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long', hour: 'numeric', minute: 'numeric' }
  },
  'en-US': {
    short: { year: 'numeric', month: 'short', day: 'numeric' },
    long: { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long', hour: 'numeric', minute: 'numeric' }
  }
}
```

### Use in Component

```vue
<script setup>
import { useI18n } from 'vue-i18n'

const { d } = useI18n()

const formattedDate = d(new Date(), 'short')
const formattedDateTime = d(new Date(), 'long')
</script>
```

## Number Formatting

### Define Format

**`src/i18n/numberFormats.js`**:
```js
export default {
  'tr-TR': {
    currency: { style: 'currency', currency: 'TRY' },
    decimal: { style: 'decimal', minimumFractionDigits: 2 }
  },
  'en-US': {
    currency: { style: 'currency', currency: 'USD' },
    decimal: { style: 'decimal', minimumFractionDigits: 2 }
  }
}
```

### Use in Component

```vue
<script setup>
import { useI18n } from 'vue-i18n'

const { n } = useI18n()

const price = n(1000.50, 'currency')
const rate = n(0.15, 'percent')
</script>
```

## Changing Locale

### In Component

```vue
<script setup>
import { useI18n } from 'vue-i18n'

const { locale } = useI18n()

const changeLanguage = (lang) => {
  locale.value = lang
  localStorage.setItem('language', lang)
}
</script>
```

### In Store (Pinia)

```js
import { defineStore } from 'pinia'
import i18n from '@/i18n/i18n.js'

export const useSettingsStore = defineStore('settings', () => {
  const changeLocale = (locale) => {
    i18n.global.locale.value = locale
    localStorage.setItem('language', locale)
  }

  return { changeLocale }
})
```

## Common Patterns

### Error Messages with Interpolation

```json
{
  "fetching_problem": "{element} çekilirken bir sorun oluştu!",
  "save_error": "{element} kaydedilirken bir sorun oluştu!",
  "delete_error": "{element} silinirken bir sorun oluştu!"
}
```

```vue
<script setup>
const { t } = useI18n()

try {
  await api.save()
} catch (err) {
  errorMessage.value = t('store.shared.save_error', {
    element: t('store.application.product')
  })
}
</script>
```

### Notification Messages

```json
{
  "added": "{element} eklendi!",
  "updated": "{element} güncellendi!",
  "deleted": "{element} silindi!"
}
```

```vue
<script setup>
import { useNotificationStore } from '@/stores/helpers/notification.js'
import { useI18n } from 'vue-i18n'

const notificationStore = useNotificationStore()
const { t } = useI18n()

const addProduct = async () => {
  await productStore.addProduct(data)
  notificationStore.add({
    type: 'success',
    message: t('store.shared.added', { element: t('store.application.product') })
  })
}
</script>
```

## Best Practices

| Practice | Why |
|----------|-----|
| Use dot notation for keys | `store.application.product` not `storeApplicationProduct` |
| Use interpolation for variables | Never concatenate: `"Hello " + name` ❌ → `"Hello {name}"` ✓ |
| Group related keys | `application.product.add`, `application.product.edit` |
| Add to ALL locales | Always update tr/en/ro together |
| Use shared translations | Common words in `shared/` folder |
| Keep translations flat | Nested max 3 levels deep |

## Naming Conventions

```json
{
  "store": {
    "shared": {
      "added": "{element} eklendi!",
      "fetching_problem": "{element} çekilirken sorun oluştu!",
      "save_error": "{element} kaydedilirken sorun oluştu!"
    },
    "application": {
      "product": "Ürün",
      "farmer": "Çiftçi",
      "add_product": "Ürün Ekle"
    }
  },
  "views": {
    "retailer": {
      "application": {
        "create": {
          "title": "Yeni Satış"
        }
      }
    }
  }
}
```

## Vue-i18n Version Notes

This project uses **vue-i18n v9** with Composition API:

```js
// Correct (Composition API)
const { t, locale, d, n } = useI18n()

// Legacy (Options API)
this.$t('key')
this.$i18n.locale
```

## Quick Reference

```bash
# Translation function
t('store.shared.added', { element: 'Ürün' })

# Date formatting
d(new Date(), 'short')

# Number formatting
n(1000.50, 'currency')

# Change locale
locale.value = 'en'
```

