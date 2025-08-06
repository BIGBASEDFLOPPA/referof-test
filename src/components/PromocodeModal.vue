<template>
  <div v-if="isOpen" class="modal-wrapper">
    <div class="wrapper">
      <div class="header-wrapper">
        <h2 class="form-title">Создание промокода</h2>
        <img
            src="/iconClose.svg"
            alt="Закрыть"
            @click="closeModal"
            class="close-button"
        />
      </div>
      <div class="steps">
        <div class="step" :class="{ active: currentStep === 0 }">Шаг 1: Основное</div>
        <div class="step" :class="{ active: currentStep === 1 }">Шаг 2: Настройки промокода</div>
      </div>

      <StepOne
          v-if="currentStep === 0"
          :formValues="formValues"
          @next="handleNext"
          @back="handleBack"
          @cancel="closeModal"
      />
      <StepTwo
          v-else-if="currentStep === 1"
          :formValues="formValues"
          @submit="handleSubmit"
          @back="handleBack"
          @cancel="closeModal"
      />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import StepOne from './StepOne.vue';
import StepTwo from './StepTwo.vue';

const isOpen = ref(true);
const currentStep = ref(0);

const formValues = reactive({
  name: '',
  title: '',
  description: '',
  points: '',
  startDate: '',
  endDate: '',
  hasNoEndDate: false,
  limit: 1,
  sendToAll: false,
});

function closeModal() {
  isOpen.value = false;
}

function handleNext(values: any) {
  Object.assign(formValues, values);
  currentStep.value++;
}

function handleBack() {
  if (currentStep.value > 0) currentStep.value--;
}

function handleSubmit(values: any) {
  const finalData = { ...formValues, ...values };
  if (finalData.hasNoEndDate) finalData.endDate = null;

  console.log('Итоговый объект промокода:', finalData);

  closeModal();
}
</script>

<style>
.header-wrapper {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

.wrapper {
  background: white;
  width: 516px;
  padding: 24px;
  border-radius: 16px;
}

.form-title {
  font-size: 20px;
  font-weight: 600;
  margin-bottom: 24px;
  color: #222;
}

.steps {
  font-size: 14px;
  display: flex;
  margin-bottom: 32px;
  user-select: none;
  border-bottom: 2px solid #ccc;
  position: relative;
  align-items: flex-end;
  height: 34px;
}

.step {
  flex: 1;
  white-space: nowrap;
  font-weight: 600;
  font-size: 16px;
  padding-bottom: 8px;
  color: #77777C;
  pointer-events: none;
  position: relative;
  text-align: center;
}

.step.active {
  color: #107FD1;
}

.step.active::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 0;
  right: 0;
  height: 3px;
  background-color: #107FD1;
  border-radius: 3px 3px 0 0;
}
.close-button{
  padding-bottom: 20px;
  cursor: pointer;
}
</style>
