<template>
  <q-dialog v-model="visible">
    <q-card>
      <q-card-section>
        <q-card-title>Error</q-card-title>
        <q-card-main>
          <p>{{ message }}</p>
        </q-card-main>
        <q-card-actions align="right">
          <q-btn label="Close" color="primary" @click="closeModal" />
        </q-card-actions>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script>
export default {
  props: {
    message: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      visible: false
    };
  },
  methods: {
    openModal() {
      this.visible = true;
    },
    closeModal() {
      this.visible = false;
    }
  }
};
</script>
