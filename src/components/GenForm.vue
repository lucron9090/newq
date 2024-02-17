<template>
  <q-responsive :ratio="16 / 9" class="col">
    <div>
      <div class="q-pa-md q-gutter-sm">
        <q-bar dense class=" text-white">
          <q-linear-progress query color="warning" />

          <div>{{ taskStatus }} <q-linear-progress query color="warning" /></div>

          <q-space />
          <q-icon name="near_me" />
          <div>100%</div>
        </q-bar>

        <q-spinner v-if="isLoading" color="primary" size="40px" />

        <q-stepper v-model="step" @finish="handleFormSubmission">
          <q-step v-for="(step, index) in steps" :key="index" :name="step.name" :label="step.name" :title="step.name"
            :done="step.done">
            <div v-for="(field, i) in step.fields" :key="i">
              <q-input :type="mapFieldType(field.type)" :label="field.input_label" :placeholder="field.input_placeholder"
                :name="field.input_name" v-model="field.value" />
              <!--:rules="[val => !!val || 'Field is required']"-->
            </div>
            <q-stepper-navigation>
              <q-btn label="Back" color="secondary" @click="previousStep" :disabled="step === steps[0].name" />
              <q-btn :label="index === steps.length - 1 ? 'Generate' : 'Next'" color="primary"
                @click="index === steps.length - 1 ? formSubmissionHandler() : nextStep()" />
            </q-stepper-navigation>
          </q-step>
        </q-stepper>


      </div>
      </div>
  </q-responsive>
</template>

<script>
import { ErrorCodes, defineComponent, ref } from 'vue'
import { Dialog } from 'quasar';
import { bus } from './Event-Bus'

export default defineComponent({
  name: 'DynamicForm',
  setup() {
    return {
      alert: ref(false),
    }
  },
  data() {
    return {
      steps: [],
      step: null,
      isLoading: false,
      isGenerating: false,
      error: null,
      isCurrentStepValid: false,
      responseStatus: null,
      intervalId: null,
      taskStatus: null,
      selectedGenerator: null,
    };
  },
  created() {
    bus.on('selectedGenerator', this.handleSelectedGenerator);
  },
  destroyed() {
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
      this.isGenerating = true;

      try {
        const formData = new FormData();
        formData.append('generator', this.selectedGenerator.id); // replace with your actual generator id
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
            const statusResponse = await fetch(`http://localhost:9000/api/frontend/result-image/${taskId}`); // include task_id in URL
            const statusData = await statusResponse.json();

            this.taskStatus = statusData.task_status;
            if (this.taskStatus === 'end' && loopcounter < 10) {
              clearInterval(this.intervalId);
              this.isGenerating = false;
            }
          }, 3000);

          setTimeout(() => {
            clearInterval(this.intervalId);
            this.isGenerating = false;
          }, 30000);

        }
      }
      catch (error) {
        this.error = error;
      }
    },
    showGenerateModal() {
      Dialog.create({
        title: 'Generate',
        message: 'Are you sure you want to generate?',
        cancel: true,
        persistent: true,
        ok: {
          label: 'Yes, generate',
          color: 'primary',
          handler: () => {
            this.formSubmissionHandler();
          }
        },
        cancel: {
          label: 'Cancel',
          color: 'negative'
        }
      })
    },
    showStatusModal(status) {
      Dialog.create({
        title: 'Generation Status',
        message: `The generation task has started: ${status}`,
      })
    },
    async handleSelectedGenerator(generator) {
      if (!generator)
        return;
      this.isLoading = true;
      this.steps = []; // reset steps
      try {
        const response = await fetch(`http://localhost:9000/api/integration/generator-full-information/${generator.slug}/`);
        const { steps, id } = await response.json();
        if (steps) {
          this.steps = steps.map(step => ({ ...step, done: false }));
          this.step = this.steps[0].name;
        }
        else {
          console.error('No data received from API');
        }
      }
      catch (error) {
        this.error = error;
      }
      finally {
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

  },
  components: { ErrorCodes }
})
</script>
