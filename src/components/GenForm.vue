<template>
  <q-responsive :ratio="16 / 9" class="col">
    <div>
      <div class="q-pa-md q-gutter-sm">
        <q-bar dense class="text-white">
          <q-linear-progress query color="warning" />
          <div>{{ taskStatus }} <q-linear-progress query color="warning" /></div>
          <q-space />
          <q-icon name="near_me" />
          <div>100%</div>
        </q-bar>
        <q-spinner v-if="isLoading" color="primary" size="40px" />
        <form @submit.prevent="formSubmissionHandler">
          <q-stepper v-model="step" @finish="formSubmissionHandler">
            <q-step v-for="(step, index) in steps" :key="index" :name="step.name" :label="step.name" :title="step.name"
              :done="step.done">
              <div v-for="(field, i) in step.fields" :key="i">
                <q-input :type="mapFieldType(field.type)" :label="field.input_label" :placeholder="field.input_placeholder"
                  :name="field.input_name" v-model="field.value" :rules="[val => (field.is_required && !val ? 'Field is required' : true)]" />
              </div>
              <q-stepper-navigation>
                <q-btn v-if="index !== 0" label="Back" color="secondary" @click="previousStep" />
                <q-btn :label="index === steps.length - 1 ? 'Generate' : 'Next'" color="primary"
                  @click.prevent="index === steps.length - 1 ? formSubmissionHandler() : nextStep()" />
              </q-stepper-navigation>
            </q-step>
          </q-stepper>
        </form>
      </div>
    </div>
  </q-responsive>
</template>

<script>
import { defineComponent, ref, onBeforeUnmount } from 'vue'
import { Dialog, QSpinner, QResponsive, QBar, QLinearProgress, QSpace, QIcon, QBtn, QStepper, QStep, QStepperNavigation, QInput } from 'quasar';
import { bus } from './Event-Bus'
export default defineComponent({
  name: 'DynamicForm',
  components: { QSpinner, QResponsive, QBar, QLinearProgress, QSpace, QIcon, QBtn, QStepper, QStep, QStepperNavigation, QInput },
  setup() {
    const alert = ref(false);
    return {
      alert,
    }
  },
  data() {
    return {
      steps: [],
      step: null,
      isLoading: false,
      isGenerating: false,
      error: null,
      responseStatus: null,
      intervalId: null,
      taskStatus: null,
      selectedGenerator: null,
      selGenID: null,
      loopcounter: 0, // Added loopcounter
    };
  },
  created() {
    bus.on('selectedGenerator', this.handleSelectedGenerator);
  },
  beforeUnmount() { // Changed from 'destroyed' for Vue 3 compatibility
    clearInterval(this.intervalId);
    bus.off('selectedGenerator', this.handleSelectedGenerator);
  },
  methods: {
    nextStep() {
      const currentIndex = this.steps.findIndex(s => s.name === this.step);
      if (currentIndex < this.steps.length - 1) {
        this.step = this.steps[currentIndex + 1].name;
      }
    },
    previousStep() {
      const currentIndex = this.steps.findIndex(s => s.name === this.step);
      if (currentIndex > 0) {
        this.step = this.steps[currentIndex - 1].name;
      }
    },
    filterSteps() {
      return this.steps.reduce((acc, step) => {
        step.fields.forEach(field => {
          acc[field.input_name] = field.value;
        });
        return acc;
      }, {});
    },
    validateForm() {
      for (let step of this.steps) {
        for (let field of step.fields) {
          if (field.is_required && !field.value) {
            return false;
          }
        }
      }
      return true;
    },
    async formSubmissionHandler() {
  if (!this.validateForm()) {
    Dialog.create({
      title: 'Error',
      message: 'Please fill in all required fields before submitting.',
    });
    return;
  }
  this.isGenerating = true;
  let tParam = ''; // Initialize the t parameter
  try {
    const formData = new FormData();
    formData.append('generator', this.selectedGenerator.id);
    formData.append('data', JSON.stringify(this.filterSteps()));
    const response = await fetch('http://localhost:9000/api/frontend/generator-item/', {
      method: 'POST',
      headers: {
        'Accept': 'application/json, text/plain, */*',
      },
      body: formData,
    });
    const data = await response.json();
    if (data) {
      const taskId = data.id;
      this.intervalId = setInterval(async () => {
        // Append the t parameter if it exists
        const statusUrl = `http://localhost:9000/api/frontend/result-image/${taskId}/?t=${tParam}`;
        const statusResponse = await fetch(statusUrl);
        const statusData = await statusResponse.json();
        this.taskStatus = statusData.task_status;

        // Extract the t parameter from the response URL if it exists
        const tMatch = statusResponse.url.match(/[?&]t=([^&]+)/);
        tParam = tMatch ? tMatch[1] : '';

        this.loopcounter += 1;
        if (this.taskStatus === 'end' || this.loopcounter >= 10) {
          clearInterval(this.intervalId);
          this.isGenerating = false;
        }
      }, 3000);
      setTimeout(() => {
        clearInterval(this.intervalId);
        this.isGenerating = false;
      }, 30000);
    }
  } catch (error) {
    this.error = error;
    Dialog.create({
      title: 'Error',
      message: 'An error occurred during form submission.',
    });
  }
},
    async handleSelectedGenerator(generator) {
      if (!generator) return;
      this.selGenID = generator.id.toString();
      this.isLoading = true;
      this.steps = []; // reset steps
      try {
        const response = await fetch(`http://localhost:9000/api/integration/generator-full-information/${generator.slug}/`);
        const { steps, id } = await response.json();
        if (steps) {
          this.steps = steps.map(step => ({ ...step, done: false }));
          this.step = this.steps[0].name;
        } else {
          console.error('No data received from API');
        }
      } catch (error) {
        this.error = error;
      } finally {
        this.isLoading = false;
      }
    },
    mapFieldType(type) {
      const fieldTypeMap = {
        string: 'text',
        number: 'number',
      };
      return fieldTypeMap[type] || 'text';
    },
  }
})
</script>
