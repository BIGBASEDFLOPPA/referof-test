<template>
  <Form @submit="onSubmit" class="form" v-slot="{ errors, meta }">
    <div class="date-title">
      Срок действия промокода<span class="required-form">*</span>
    </div>

    <div class="date-row">
      <div
          class="date-group"
          :class="{ 'form-input--error': dateErrors.startDate || (meta.submitCount > 0 && errors.limit) }"
      >
        <label class="form-label">Дата начала</label>

        <div class="datetime-wrapper">
          <div class="input-date-wrapper">
            <img
                src="/iconCalendar.svg"
                class="calendar-icon"
                alt="calendar icon"
                @click="focusInput('startDate')"
            />
            <input
                type="date"
                class="datetime-input input-date with-icon"
                v-model="startDateDate"
                ref="startDateInput"
            />
          </div>

          <div class="input-time-wrapper" @click="focusInput('startTime')">
            <input
                type="time"
                class="datetime-input input-time"
                v-model="startDateTime"
                ref="startDateTimeInput"
            />
          </div>
        </div>
        <span v-if="dateErrors.startDate" class="form-error">{{ dateErrors.startDate }}</span>
      </div>

      <div
          class="date-group"
          :class="{ 'form-input--error': dateErrors.endDate }"
          :style="{ opacity: hasNoEndDate ? 0.6 : 1 }"
          :aria-disabled="hasNoEndDate"
      >
        <label class="form-label">Дата окончания</label>
        <div class="datetime-wrapper">
          <div class="input-date-wrapper">
            <img
                src="/iconCalendar.svg"
                class="calendar-icon"
                alt="calendar icon"
                @click="focusInput('endDate')"
            />
            <input
                type="date"
                class="datetime-input input-date with-icon"
                v-model="endDateDate"
                :disabled="hasNoEndDate"
                ref="endDateInput"
            />
          </div>

          <div class="input-time-wrapper" @click="focusInput('endTime')">
            <input
                type="time"
                class="datetime-input input-time"
                v-model="endDateTime"
                :disabled="hasNoEndDate"
                ref="endDateTimeInput"
            />
          </div>
        </div>
        <span v-if="dateErrors.endDate" class="form-error">{{ dateErrors.endDate }}</span>
      </div>
    </div>

    <div class="form-group form-group-checkbox">
      <input
          id="hasNoEndDate"
          type="checkbox"
          class="form-checkbox"
          v-model="hasNoEndDate"
      />
      <label class="form-label-inline" for="hasNoEndDate">Без даты конца</label>
    </div>

    <div class="line-separator"></div>

    <div class="form-group">
      <label class="form-label limit-title">
        Введите лимит активаций (шт.)<span class="required-form">*</span>
      </label>
      <Field
          name="limit"
          type="number"
          class="form-input"
          :class="{ 'form-input--error': meta.submitCount > 0 && errors.limit }"
          :validate-on-change="false"
          :validate-on-blur="false"
      />
      <span v-if="meta.submitCount > 0 && errors.limit" class="form-error">{{ errors.limit }}</span>
    </div>

    <div class="form-group form-group-radio">
      <label class="form-label promo-settings-title">Настроить получение промокода</label>
      <div class="form-radio-option">
        <Field
            id="sendToAllFalse"
            name="sendToAll"
            type="radio"
            :value="false"
            class="form-radio"
            :validate-on-change="false"
            :validate-on-blur="false"
        />
        <label for="sendToAllFalse" class="form-label-inline">Создать промокод без отправки</label>
      </div>
      <div class="form-radio-option">
        <Field
            id="sendToAllTrue"
            name="sendToAll"
            type="radio"
            :value="true"
            class="form-radio"
            :validate-on-change="false"
            :validate-on-blur="false"
        />
        <label for="sendToAllTrue" class="form-label-inline">Отправить промокод всем пользователям</label>
      </div>
      <span v-if="meta.submitCount > 0 && errors.sendToAll" class="form-error">{{ errors.sendToAll }}</span>
    </div>

    <div class="form-buttons">
      <button type="button" @click="$emit('back')" class="form-button form-button-cancel">Назад</button>
      <button type="submit" class="form-button form-button-submit">Создать</button>
    </div>
  </Form>
</template>

<script setup lang="ts">
import { ref, reactive, watch } from 'vue'
import { useForm, Field, Form } from 'vee-validate'
import * as yup from 'yup'

const props = defineProps<{ formValues: any }>()
const emit = defineEmits(['submit', 'back', 'cancel'])

const schema = yup.object({
  limit: yup.number().required('Введите лимит').typeError('Только числа'),
  sendToAll: yup.boolean().required('Выберите вариант'),
})

const { errors, setFieldValue, meta, handleSubmit } = useForm({
  validationSchema: schema,
  initialValues: props.formValues,
  validateOnBlur: false,
  validateOnChange: false,
})

const startDateDate = ref('')
const startDateTime = ref('00:00')
const endDateDate = ref('')
const endDateTime = ref('00:00')

const hasNoEndDate = ref(!!props.formValues.hasNoEndDate)

const dateErrors = reactive<{ startDate?: string; endDate?: string }>({})

if (props.formValues.startDate) {
  const [datePart, timePart = '00:00'] = props.formValues.startDate.split('T')
  startDateDate.value = datePart
  startDateTime.value = timePart.slice(0, 5)
}
if (props.formValues.endDate) {
  const [datePart, timePart = '00:00'] = props.formValues.endDate.split('T')
  endDateDate.value = datePart
  endDateTime.value = timePart.slice(0, 5)
}

watch(hasNoEndDate, (val) => {
  setFieldValue('hasNoEndDate', val)
  if (val) {
    endDateDate.value = ''
    endDateTime.value = '00:00'
    dateErrors.endDate = ''
  }
})

function validateDates() {
  dateErrors.startDate = ''
  dateErrors.endDate = ''

  let valid = true

  if (!startDateDate.value) {
    dateErrors.startDate = 'Укажите дату начала'
    valid = false
  }
  if (!startDateTime.value) {
    dateErrors.startDate = dateErrors.startDate
        ? dateErrors.startDate + '; Укажите время начала'
        : 'Укажите время начала'
    valid = false
  }

  if (!hasNoEndDate.value) {
    if (!endDateDate.value) {
      dateErrors.endDate = 'Укажите дату окончания'
      valid = false
    }
    if (!endDateTime.value) {
      dateErrors.endDate = dateErrors.endDate
          ? dateErrors.endDate + '; Укажите время окончания'
          : 'Укажите время окончания'
      valid = false
    }
  }

  if (
      !hasNoEndDate.value &&
      startDateDate.value &&
      startDateTime.value &&
      endDateDate.value &&
      endDateTime.value
  ) {
    const startISO = `${startDateDate.value}T${startDateTime.value}`
    const endISO = `${endDateDate.value}T${endDateTime.value}`

    if (new Date(startISO) > new Date(endISO)) {
      dateErrors.endDate = dateErrors.endDate
          ? dateErrors.endDate + '; Дата окончания должна быть позже даты начала'
          : 'Дата окончания должна быть позже даты начала'
      valid = false
    }
  }

  return valid
}

const onSubmit = handleSubmit(async (formValues) => {
  const datesValid = validateDates()
  if (!datesValid) return

  emit('submit', {
    ...formValues,
    startDate: `${startDateDate.value}T${startDateTime.value}`,
    endDate: hasNoEndDate.value ? null : `${endDateDate.value}T${endDateTime.value}`,
    hasNoEndDate: hasNoEndDate.value,
  })
})

const refs = {
  startDateInput: ref<HTMLInputElement | null>(null),
  startDateTimeInput: ref<HTMLInputElement | null>(null),
  endDateInput: ref<HTMLInputElement | null>(null),
  endDateTimeInput: ref<HTMLInputElement | null>(null),
}

function focusInput(field: 'startDate' | 'startTime' | 'endDate' | 'endTime') {
  switch (field) {
    case 'startDate':
      refs.startDateInput.value?.focus()
      break
    case 'startTime':
      refs.startDateTimeInput.value?.showPicker()
      break
    case 'endDate':
      refs.endDateInput.value?.focus()
      break
    case 'endTime':
      refs.endDateTimeInput.value?.showPicker()
      break
  }
}
</script>

<style scoped>
.form-label {
  display: block;
  padding-bottom: 6px;
}

.required-form {
  color: red;
}

.form-input {
  padding: 13px 12px;
  border-radius: 12px;
  width: 100%;
  border: none;
  transition: border-color 0.2s ease;
  background-color: #F5F5F5;
}

.form-input--error,
.form-input--error:focus {
  border-color: red;
}

.promo-settings-title {
  font-weight: 600;
}

.date-title {
  margin-bottom: 12px;
}

.limit-title {
  font-weight: 600;
}

.date-row {
  display: flex;
  flex-wrap: nowrap;
  overflow: hidden;
}

.datetime-wrapper {
  display: flex;
  gap: 4px;
  width: 100%;
  min-width: 0;
  align-items: center;
}

.input-date-wrapper {
  position: relative;
  width: 60%;
  min-width: 0;
}

.input-date {
  width: 100%;
  height: 44px;
  border-radius: 12px;
  border: none;
  background-color: #f5f5f5;
  box-sizing: border-box;
  outline: none;
  padding: 11px 13px;
  position: relative;
  z-index: 2;
}

.input-time-wrapper {
  width: 80px;
  position: relative;
  cursor: pointer;
}

.input-time {
  height: 44px;
  border-radius: 12px;
  border: none;
  background-color: #f5f5f5;
  box-sizing: border-box;
  outline: none;
  padding: 11px 13px;
  cursor: pointer;
}

.datetime-input:focus {
  border: none;
  outline: none;
}

.form-checkbox {
  cursor: pointer;
  vertical-align: middle;
  margin-right: 6px;
}

.form-radio {
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 3px solid #C7C7CC;
  background-color: transparent;
  position: relative;
  transition: all 0.2s ease;
  cursor: pointer;
  margin-right: 6px;
}

.form-radio:checked {
  border: 5px solid #107FD1;
  background-color: transparent;
}

.form-error {
  padding-top: 4px;
  display: block;
  color: red;
}

.form-buttons {
  display: flex;
  gap: 12px;
  padding-top: 4px;
}

.form-button {
  font-weight: 600;
  flex: 1;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.form-button-submit {
  height: 40px;
  border-radius: 12px;
  background-color: #05263F;
  color: #FFFFFF;
}

.form-button-cancel {
  height: 40px;
  border-radius: 12px;
  background-color: #F2F1F6;
  color: #77777C;
}

.form-group-checkbox {
  margin-top: 8px;
}

.form-group-radio {
  display: flex;
  flex-direction: column;
  gap: 16px;
  flex-wrap: nowrap;
  margin-top: 20px;
  margin-bottom: 16px;
}

.form-radio-option {
  display: flex;
  align-items: center;
  gap: 6px;
}

.line-separator {
  height: 1px;
  background-color: #EBEBF0;
  width: 100%;
  margin: 20px 0;
}

input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}

.calendar-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  width: 24px;
  height: 24px;
  cursor: pointer;
  z-index: 3;
  pointer-events: auto;
}

.with-icon {
  padding-left: 44px !important;
}

input[type="date"]::-webkit-calendar-picker-indicator {
  opacity: 0;
  position: absolute;
  right: 0;
  height: 100%;
  cursor: pointer;
  z-index: 4;
}

input[type="date"] {
  position: relative;
  z-index: 2;
}

input[type="time"]::-webkit-calendar-picker-indicator {
  background: none;
  display: none;
}
</style>