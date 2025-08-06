<template>
  <Form
      :validation-schema="schema"
      :initial-values="formValues"
      @submit="onSubmit"
      v-slot="{ errors, values, meta }"
      class="form"
  >
    <div class="form-group">
      <label class="form-label promo-title">Название промокода<span class="form-required">*</span></label>
      <Field name="name" v-slot="{ field, meta: fieldMeta, errorMessage }" :validate-on-blur="false" :validate-on-change="false">
        <input
            v-bind="field"
            type="text"
            class="form-input"
            :class="{ 'form-input--error': fieldMeta.touched && errorMessage }"
            placeholder="Введи название"
        />
      </Field>
      <span v-if="errors.name" class="form-error">{{ errors.name }}</span>
    </div>

    <div class="form-group">
      <label class="form-label name-title">Заголовок<span class="form-required">*</span></label>
      <Field name="title" v-slot="{ field, meta: fieldMeta, errorMessage }" :validate-on-blur="false" :validate-on-change="false">
        <input
            v-bind="field"
            type="text"
            class="form-input"
            :class="{ 'form-input--error': fieldMeta.touched && errorMessage }"
            placeholder="Введи заголовок"
        />
      </Field>
      <span v-if="errors.title" class="form-error">{{ errors.title }}</span>
    </div>

    <div class="form-group">
      <label class="form-label text-title">Сопроводительный текст</label>
      <Field name="description" v-slot="{ field }" :validate-on-blur="false" :validate-on-change="false">
        <textarea
            v-bind="field"
            v-model="description"
            class="form-textarea"
            maxlength="250"
            placeholder="Например: «Ты попал в число счастливчиков! Дарим 300 баллов»"
        />
      </Field>
      <div class="form-counter-wrapper">
        <span
            class="form-counter"
            :class="{ 'form-counter--error': meta.submitCount > 0 && description.length === 0 }"
        >
          {{ description.length }}/250
        </span>
      </div>
    </div>

    <div class="form-group">
      <label class="form-label">Укажи количество баллов<span class="form-required">*</span></label>
      <Field name="points" v-slot="{ field, meta: fieldMeta, errorMessage }" :validate-on-blur="false" :validate-on-change="false">
        <input
            v-bind="field"
            type="number"
            class="form-input form-input--icon"
            :class="{ 'form-input--error': fieldMeta.touched && errorMessage }"
            placeholder="100"
        />
      </Field>
      <span v-if="errors.points" class="form-error">{{ errors.points }}</span>
    </div>

    <div class="form-buttons">
      <button type="button" @click="$emit('back')" class="form-button form-button-cancel">Назад</button>
      <button type="submit" class="form-button form-button-submit">Далее</button>
    </div>
  </Form>
</template>

<script setup lang="ts">
import { Form, Field } from 'vee-validate';
import * as yup from 'yup';
import { ref } from 'vue';

const props = defineProps<{ formValues: any }>();
const emit = defineEmits(['next', 'back', 'cancel']);

const description = ref(props.formValues.description || '');

const schema = yup.object({
  name: yup.string().required('Введите название').max(30),
  title: yup.string().required('Введите заголовок').max(120),
  description: yup.string().max(250),
  points: yup
      .number()
      .typeError('Введите кол-во(только числа)')
      .required('Введите количество баллов'),
});

function onSubmit(values: any) {
  emit('next', { ...values, description: description.value });
}
</script>


<style scoped>
.promo-title,
.name-title,
.promo-title{
  font-weight: 600;
}
.form-label {
  padding-bottom: 6px;
  display: block;
}

.form-required {
  color: red;
}

.form-group {
  padding-bottom: 16px;
}

.form-input,
.form-textarea {
  padding: 13px 12px;
  border-radius: 12px;
  width: 100%;
  border: none;
  background-color: #f5f5f5;
}

.form-input:focus,
.form-textarea:focus {
  outline: none;
}

.form-input--error,
.form-input--error:focus {
  border: 1px solid red;
}

.form-textarea {
  resize: none;
  overflow: hidden;
  min-height: 90px;
}

.form-error {
  color: red;
  font-size: 13px;
  margin-top: 4px;
  display: block;
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
  background-color: #05263f;
  color: #ffffff;
}

.form-button-cancel {
  height: 40px;
  border-radius: 12px;
  background-color: #f2f1f6;
  color: #77777c;
}

.form-counter-wrapper {
  display: flex;
  justify-content: flex-end;
  margin-top: 4px;
}

.form-counter {
  color: #888;
  font-size: 12px;
}

.form-counter--error {
  color: red;
}

.form-input--icon {
  background-image: url('/iconPoints.svg');
  background-repeat: no-repeat;
  background-position: 12px center;
  background-size: 20px 20px;
  padding-left: 42px;
}
input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input[type="number"] {
  -moz-appearance: textfield;
}
</style>
