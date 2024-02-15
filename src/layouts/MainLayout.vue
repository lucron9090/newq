
<template>
  <q-layout view="hHh Lpr fFf">
  <div class="q-pa-md example-row-stacked-to-horizontal">

    <div class="row">
      <div class="col-12 col-md-4"><EssentialLink /></div>
      <div class="col-12 col-md-8"><q-page-container><router-view /></q-page-container></div>
    </div>
  </div>
  </q-layout>
</template>
<style lang="sass">
.example-row-stacked-to-horizontal
  .row > div
  .row + .row
    margin-top: 1rem
</style>
<script>
import { defineComponent, ref } from 'vue'
import EssentialLink from 'components/EssentialLink.vue';
export default defineComponent({
  name: 'MainLayout',

  components: {
    EssentialLink,
  },

  data() {
    return {
      errorMessage: ""
    };
  },
  setup() {
    const leftDrawerOpen = ref(false)
    const rightDrawerOpen = ref(false)

    return {
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value
      },

      rightDrawerOpen,
      toggleRightDrawer() {
        rightDrawerOpen.value = !rightDrawerOpen.value
      }
    }
  }
});

</script>
