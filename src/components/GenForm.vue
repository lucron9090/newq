<template>
  <q-stepper v-model="step" @finish="handleFormSubmission">
    <q-step
      v-for="(step, index) in steps"
      :key="index"
      :name="step.name"
      :label="step.name"
      :title="step.name"
      :done="step.done"
    >
      <div v-for="(field, i) in step.fields" :key="i">
        <q-input
          :type="mapFieldType(field.type)"
          :label="field.input_label"
          :placeholder="field.input_placeholder"
          :name="field.input_name"
          :rules="[val => !!val || 'Field is required']"
          v-model="field.value"
        />
      </div>
      <q-btn
      label="Back"
      color="secondary"
      @click="previousStep"
      :disabled="step === steps[0].name"
    />
    <q-btn
      :label="step === steps[steps.length - 1].name ? 'Generate' : 'Next'"
      color="primary"
      @click="step === steps[steps.length - 1].name ? formSubmissionHandler() : nextStep()"
    />

    </q-step>
    <q-stepper-navigation>
      <q-btn
        label="Submit"
        type="submit"
        color="primary"
        @click="handleFormSubmission"
      />
    </q-stepper-navigation>
  </q-stepper>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'DynamicForm',
  data() {
    return {
      steps: [],
      step: null,
      isLoading: false,
      error: null,
      isCurrentStepValid: false,
    }
  },
  mounted() {
    this.fetchFormData()
  },
  methods: {
    nextStep() {
      const currentIndex = this.steps.findIndex(s => s.name === this.step)
      if (currentIndex < this.steps.length - 1) {
        this.step = this.steps[currentIndex + 1].name
      }
    },
    previousStep() {
      const currentIndex = this.steps.findIndex(s => s.name === this.step)
      if (currentIndex > 0) {
        this.step = this.steps[currentIndex - 1].name
      }
    },
    formSubmissionHandler() {
      // Implement your form submission logic here
    },
    async fetchFormData() {
      this.isLoading = true
      try {
        const response = await fetch('http://localhost:9000/api/integration/generator-full-information/dl_texas/')
        const { steps } = await response.json()
        this.steps = steps.map(step => ({ ...step, done: false }))
        this.step = this.steps[0].name
      } catch (error) {
        console.error('Error fetching form data:', error)
        this.error = error
      } finally {
        this.isLoading = false
      }
    },
    handleFormSubmission() {
      // Add form submission logic here
    },
    mapFieldType(type) {
      const fieldTypeMap = {
        string: 'text',
        number: 'number',
      }
      return fieldTypeMap[type] || 'text'
    },
  },
})
</script>
