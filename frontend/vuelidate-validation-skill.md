---
name: "vuelidate-i18n"
description: "Vuelidate validation with i18n integration. Invoke when user wants to add/translate validation rules, create validation messages, or work with form validation."
---

# Vuelidate + i18n Validation

Form validation with Vuelidate and internationalized error messages.

## Project Structure

```
src/
├── utilities/
│   └── validationErrorMessage.js    # Error message generator
├── composables/
│   └── validation.js                 # Validation composable
├── mixins/
│   ├── validationMixin.js           # Options API mixin (If the component is Vue 2.x)
│   └── validationStatusMixin.js     # Status mixin
└── locales/
    └── {tr,en,ro}/
        └── shared/
            ├── validations.json     # All validation messages
            └── validation_rules.json # Rule-specific values
```

## Validation Messages (validations.json)

### Turkish (tr)
```json
{
  "required": "{field} alanı zorunludur",
  "email": "Geçerli bir email adresi giriniz!",
  "minLength": "{field} en az {min_length} karakter olmalıdır.",
  "maxLength": "{field} en fazla {max_length} karakter olmalıdır.",
  "minValue": "{field} en az {min_value} değerinde olmalıdır.",
  "maxValue": "{field} en fazla {max_value} değerinde olmalıdır.",
  "sameAs": "{field} ve tekrarı aynı olmalıdır.",
  "sameAsPassword": "Parola ve tekrarı aynı olmalıdır.",
  "selectbox_required": "{field} seçimi zorunludur!",
  "numeric": "{field} alanı pozitif sayı olmalıdır!",
  "integer": "{field} alanı tam sayı olmalıdır.",
  "url": "{field} mutlaka bir URL olmalıdır.",
  "iban": "IBAN hatalı!",
  "invalid_phone_number": "Geçersiz telefon numarası!",
  "must_choose_file": "Bir dosya seçmelisiniz.",
  "error": "Girdiğiniz bilgiler eksik veya hatalı. Lütfen kontrol ediniz."
}
```

### English (en)
```json
{
  "required": "{field} field is mandatory",
  "email": "Please enter a valid email address!",
  "minLength": "{field} should be at least {min_length} characters.",
  "maxLength": "{field} should be at most {max_length} characters.",
  "minValue": "{field} should be at least {min_value}.",
  "maxValue": "{field} should be at most {max_value}.",
  "sameAs": "{field} and the repetition should be the same.",
  "sameAsPassword": "Password and the repetition should be the same.",
  "selectbox_required": "{field} selection is mandatory!",
  "numeric": "{field} should be a positive number!",
  "integer": "{field} should be an integer!",
  "url": "{field} must be a valid URL.",
  "iban": "Invalid IBAN!",
  "invalid_phone_number": "Invalid phone number!",
  "must_choose_file": "You must select a file.",
  "error": "The information you entered is incomplete or incorrect. Please check."
}
```

### Romanian (ro)
```json
{
  "required": "Câmpul {field} este obligatoriu",
  "email": "Vă rugăm să introduceți o adresă de email validă!",
  "minLength": "{field} trebuie să aibă cel puțin {min_length} caractere.",
  "maxLength": "{field} trebuie să aibă cel mult {max_length} caractere.",
  "minValue": "{field} trebuie să fie cel puțin {min_value}.",
  "maxValue": "{field} trebuie să fie cel mult {max_value}.",
  "sameAs": "{field} și repetarea trebuie să fie la fel.",
  "sameAsPassword": "Parola și repetarea trebuie să fie la fel.",
  "selectbox_required": "Selecția {field} este obligatorie!",
  "numeric": "Câmpul {field} trebuie să fie un număr pozitiv!",
  "integer": "Câmpul {field} trebuie să fie un număr întreg!",
  "url": "{field} trebuie să fie un URL valid.",
  "iban": "IBAN invalid!",
  "invalid_phone_number": "Număr de telefon invalid!",
  "must_choose_file": "Trebuie să selectați un fișier.",
  "error": "Informațiile introduse sunt incomplete sau incorecte. Vă rugăm să verificați."
}
```

## Built-in Vuelidate Validators

```bash
# Install
npm install @vuelidate/validators
```

```js
import {
  required,
  email,
  minLength,
  maxLength,
  minValue,
  maxValue,
  sameAs,
  requiredIf,
  helpers
} from '@vuelidate/validators'
```

## Basic Usage

### Setup (Composition API)

```vue
<script setup>
import { ref } from 'vue'
import { useVuelidate } from '@vuelidate/core'
import { required, email, minLength } from '@vuelidate/validators'
import { errorMessageWithParams } from '@/utilities/validationErrorMessage.js'

const form = ref({
  email: '',
  password: ''
})

const rules = {
  email: { required, email, $lazy: true },
  password: { required, minLength: minLength(8), $lazy: true }
}

const v$ = useVuelidate(rules, form)
const emailErrors = computed(() => errorMessageWithParams(v$.value.email, 'views.admin.users.email'))
</script>
```

### Setup (Options API)

```vue
<script>
import { email, minLength, required } from '@vuelidate/validators'
import { useVuelidate } from '@vuelidate/core'
import { errorMessageWithParams } from '@/utilities/validationErrorMessage.js'

export default {
  data() {
    return {
      v$: useVuelidate(),
      user: {
        email: '',
        password: ''
      }
    }
  },
  validations() {
    return {
      user: {
        email: { required, email, $lazy: true },
        password: { required, minLength: minLength(8), $lazy: true }
      }
    }
  },
  computed: {
    emailErrorMessages() {
      return errorMessageWithParams(this.v$.user.email, 'views.admin.users.email')
    },
    passwordErrorMessages() {
      return errorMessageWithParams(this.v$.user.password, 'views.admin.users.password')
    }
  }
}
</script>
```

## Template Usage

### Basic Error Display

```vue
<template>
  <base-input
    v-model="form.email"
    :error="v$.email.$error"
    :error-messages="emailErrorMessages"
  />
</template>
```

### Password Strength Indicator

```vue
<template>
  <ul class="password-requirements">
    <li :class="{ isValid: v$.password.minLength.$response }">
      {{ $t('shared.validations.minLengthPassword') }}
    </li>
    <li :class="{ isValid: v$.password.containsNumber.$response }">
      {{ $t('shared.validations.containsNumber') }}
    </li>
    <li :class="{ isValid: v$.password.containsUppercase.$response }">
      {{ $t('shared.validations.containsUppercase') }}
    </li>
    <li :class="{ isValid: v$.password.containsLowercase.$response }">
      {{ $t('shared.validations.containsLowercase') }}
    </li>
  </ul>
</template>
```

## Custom Validators

### Simple Custom Validator

```vue
<script setup>
import { helpers } from '@vuelidate/validators'
import i18n from '@/i18n/i18n.js'

const { t } = i18n.global

const mustBePositive = helpers.withMessage(
  t('shared.validations.numeric'),
  value => /^\d+$/.test(value)
)

const rules = {
  amount: { mustBePositive }
}
</script>
```

### Custom Date Validator

```vue
<script setup>
import { helpers } from '@vuelidate/validators'
import moment from 'moment'
import { useI18n } from 'vue-i18n'

const { t } = useI18n()

const isValidDate = value => {
  if (!value) return true
  if (value === 'Invalid date') return false
  return moment(value, 'YYYY-MM-DD', true).isValid()
}

const validDate = helpers.withMessage(
  t('views.admin.holidays.invalid_date_error'),
  isValidDate
)

const rules = {
  start_date: { required, validDate, $lazy: true }
}
</script>
```

### Custom Regex Validator

```vue
<script setup>
import { helpers } from '@vuelidate/validators'
import country from '@/utilities/country.js'

const phoneValidator = helpers.regex(
  country.is('tr') ? /^\([0-9]{3}\) [0-9]{3} [0-9]{2} [0-9]{2}$/ : /^\([0-9]{3}\) [0-9]{3} [0-9]{3}$/
)

const rules = {
  phone: { required, phoneValidator }
}
</script>
```

## Common Validation Patterns

### Login Form

```vue
<script setup>
import { email, minLength, required } from '@vuelidate/validators'
import { useVuelidate } from '@vuelidate/core'
import { errorMessageWithParams } from '@/utilities/validationErrorMessage.js'

const form = ref({ email: '', password: '' })

const rules = {
  email: { required, email, $lazy: true },
  password: { required }
}

const v$ = useVuelidate(rules, form)

const handleSubmit = async () => {
  const result = await v$.value.$validate()
  if (result) {
    // Submit form
  }
}
</script>
```

### Registration Form with Password

```vue
<script setup>
import { email, helpers, minLength, required, sameAs } from '@vuelidate/validators'

const form = ref({
  email: '',
  password: '',
  password_confirmation: ''
})

const rules = {
  email: { required, email, $lazy: true },
  password: {
    required,
    minLength: minLength(8),
    containsUppercase: value => /[A-Z]/.test(value),
    containsLowercase: value => /[a-z]/.test(value),
    containsNumber: value => /[0-9]/.test(value),
    containsSpecial: value => /[ !"@#£$%&{()}<>=+'|^,;-]/.test(value)
  },
  password_confirmation: {
    required,
    sameAsPassword: sameAs(form.value.password)
  }
}
</script>
```

### Phone Number Validation

```vue
<script setup>
import { required, minLength } from '@vuelidate/validators'
import i18n from '@/i18n/i18n.js'

const { t } = i18n.global

const phoneRules = {
  phone: {
    required,
    minLength: minLength(t('shared.validation_rules.phone_min_length')),
    mustBeSecondCharacterSeven: value => !(value?.length && value[1] !== '7')
  }
}
</script>
```

### CNP Validation (Romania)

```vue
<script setup>
import { required, minLength, maxLength } from '@vuelidate/validators'
import { Cnp } from '@/utilities/cnp.js'
import moment from 'moment'

const cnpRules = {
  cnp: {
    required,
    minLength: minLength(13), // This 13 is not constant, you can pick according to situtation
    maxLength: maxLength(13),
    minAge: value => {
      if (value?.length < 13) return true
      if (value) {
        const birthDate = new Cnp(value).getBirthDate()
        return moment().diff(birthDate, 'years', true) >= 18
      }
      return false
    },
    maxAge: value => {
      if (value?.length < 13) return true
      if (value) {
        const birthDate = new Cnp(value).getBirthDate()
        return moment().diff(birthDate, 'years', true) <= 100
      }
      return false
    }
  }
}
</script>
```

## Validation Composable & Mixin

### Composables (src/composables/validation.js)

```js
import { useNotificationStore } from '@/stores/helpers/notification.js'
import i18n from '@/i18n/i18n.js'

const { t } = i18n.global

export default function useValidation() {
  const validationStatuses = (validation, enableDirtyClass = true) => {
    return {
      error: validation.$error,
      dirty: enableDirtyClass ? validation.$dirty : false
    }
  }

  const validationErrorNotification = () => {
    const notificationStore = useNotificationStore()
    notificationStore.add({
      type: 'danger',
      message: t('shared.validations.error')
    })
  }

  return { validationStatuses, validationErrorNotification }
}
```

### Usage

```vue
<script setup>
import useValidation from '@/composables/validation.js'

const { validationStatuses, validationErrorNotification } = useValidation()

const status = computed(() => validationStatuses(v$.value.email))
</script>
```

## Error Message Generator

**src/utilities/validationErrorMessage.js**

```js
import { useI18n } from 'vue-i18n'

export const errorMessageWithParams = (validation, fieldName) => {
  const { t } = useI18n()
  const messages = []

  const rules = Object.keys(validation ?? {}).filter(item => item.startsWith('$') === false)

  const items = rules.reduce((obj, key) => {
    obj[key] = validation[key]
    return obj
  }, {})

  for (const key in items) {
    if (items[key]?.$invalid === true && items[key]?.$pending === false) {
      const params = {
        ...items[key].$params,
        field: t(fieldName)
      }

      // Parameter name mapping
      if (key === 'minLength') params.min_length = params.min
      if (key === 'minValue') params.min_value = params.min
      if (key === 'maxValue') params.max_value = params.max
      if (key === 'maxLength') params.max_length = params.max

      messages.push(t(`shared.validations.${key}`, params))
    }
  }

  // Fallback to required
  if (messages.length === 0) {
    messages.push(t('shared.validations.required', { field: t(fieldName) }))
  }

  return messages
}
```

## Best Practices

| Practice | Why |
|----------|-----|
| Use `$lazy: true` | Prevents validation until first blur/submit |
| Use `errorMessageWithParams` | Centralized i18n message handling |
| Always add `$lazy: true` | Performance optimization |
| Use computed for error messages | Reactive updates |
| Add blur/submit handlers | Proper UX for validation |

## Quick Reference

### Validation Keys (validations.json)

| Key | Description |
|-----|-------------|
| `required` | Field is mandatory |
| `email` | Valid email format |
| `minLength` | Minimum character count |
| `maxLength` | Maximum character count |
| `minValue` | Minimum numeric value |
| `maxValue` | Maximum numeric value |
| `sameAs` | Fields must match |
| `numeric` | Numbers only |
| `integer` | Whole numbers only |
| `url` | Valid URL format |
| `iban` | Valid IBAN |
| `selectbox_required` | Dropdown selection required |

### Parameter Mapping

| Validator | i18n Param |
|-----------|------------|
| `minLength(8)` | `{min_length: 8}` |
| `maxLength(50)` | `{max_length: 50}` |
| `minValue(0)` | `{min_value: 0}` |
| `maxValue(100)` | `{max_value: 100}` |

## Adding New Validation Rules

### 1. Add to validations.json (all locales)

```json
// locales/tr/shared/validations.json
{
  "newRule": "{field} için yeni kural mesajı"
}

// locales/en/shared/validations.json
{
  "newRule": "New rule message for {field}"
}

// locales/ro/shared/validations.json
{
  "newRule": "Mesaj nou pentru {field}"
}
```

### 2. Use in Component

```vue
<script setup>
import { helpers } from '@vuelidate/validators'
import i18n from '@/i18n/i18n.js'

const { t } = i18n.global

const isValidCustom = value => /* logic */
const customValidator = helpers.withMessage(
  t('shared.validations.newRule'),
  isValidCustom
)
</script>
```

