<template>
  <div v-if="isLoading">
    Loading...
  </div>
  <div v-else-if="isGenerating">
    Generating...
  </div>
  <div v-else-if="error">
    Error: {{ error.message }}
  </div>
  <div v-else-if="responseStatus === 'Done'">
    Success! Your file is ready.
  </div>
  <div v-else>
    Magic Hand is idle
  </div>
</template>

<script>
export default {
  props: {
    isLoading: { type: Boolean, default: false },
    isGenerating: { type: Boolean, default: false },
    progress: { type: Number, default: 0.0 },
    taskStatus: { type: String, default: ''},
    error: { type: Object, default: null },
    responseStatus: { type: String, default: '' },
  }
}
</script>
