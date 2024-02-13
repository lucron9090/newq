<template>
  <div>
    <form @submit.prevent="submitForm">
      <div v-for="(step, stepIndex) in formSteps" :key="stepIndex">
        <div v-for="(field, fieldIndex) in step.fields" :key="fieldIndex">
          <label :for="field.inputName">{{ field.inputLabel }}</label>
          <input
            :type="getFieldType(field.type)"
            :placeholder="field.inputPlaceholderDefault"
            :name="field.inputName"
            :required="field.required"
          />
        </div>
      </div>
      <button type="submit">Submit</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'GenForm',
  data() {
    return {
      formSteps: [],
      isLoading: false, // Added a loading state
      fetchError: null, // State to store fetch errors
    };
  },
  mounted() {
    this.fetchFormData();
  },
  methods: {
    async fetchFormData() {
      this.isLoading = true; // Indicate loading state
      try {
        const response = await fetch('https://oldie.veriftools.ru/api/integration/generator-full-information/dl_texas');
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`); // Better error handling
        }
        const data = await response.json();
          this.formSteps = data.steps;
        } catch (error) {
          console.error('Error fetching form data:', error);
        this.fetchError = error; // Store error
      } finally {
        this.isLoading = false; // Reset loading state
      }
    },
    submitForm() {
      // Handle form submission logic here
    },
    getFieldType(type) {
      // Improved readability using an object map
      const fieldTypeMap = {
        'string': 'text',
        'number': 'number',
      };
      return fieldTypeMap[type] || 'text';
    }
  }
};
</script>
