<template>
  <q-responsive :ratio="16/9" class="col">
   <div >
    <q-spinner v-if="isLoading" color="primary" size="40px" />

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
            v-model="field.value"
          />
          <!--:rules="[val => !!val || 'Field is required']"-->
        </div>
        <q-stepper-navigation>
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
      </q-stepper-navigation>
      </q-step>
    </q-stepper>


  </div>
</q-responsive>
</template>

<script>
import { defineComponent } from 'vue'
import { bus } from './Event-Bus'

export default defineComponent({
  name: 'DynamicForm',
  data() {
    return {
      steps: [],
      step: null,
      isLoading: false,
      isGenerating: false,
      error: null,
      isCurrentStepValid: false,
    }
  },
  created() {
    bus.on('slugSelected', this.handleSlugSelected)
  },
  destroyed() {
    bus.off('slugSelected', this.handleSlugSelected)
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
    async formSubmissionHandler() {

      this.isGenerating = true
      try {
        const response = await fetch('http://localhost:9000/api/integration/generate/', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(this.steps),
        })
        const data = await response.json()
        if (data) {
          console.log(data)
        } else {
          console.error('No data received from API')
        }
      } catch (error) {
        this.error = error
      } finally {
        this.isGenerating = false
      }
    },
    async handleSlugSelected(slug) {
      if(!slug) return;
      this.isLoading = true
      this.steps = [];
      try {
        const response = await fetch(`http://localhost:9000/api/integration/generator-full-information/${slug}/`)
        const { steps } = await response.json()
        if (steps) {
          {
            this.steps = steps.map(step => ({ ...step, done: false }))
            this.step = this.steps[0].name
          }
        } else {
          console.error('No data received from API')
        }
      } catch (error) {
        this.error = error
      } finally {
        this.isLoading = false
      }
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
