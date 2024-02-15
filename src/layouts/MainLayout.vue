<template>
  <q-layout view="hHh lpR fFf">
    <q-page-container>
      <div class="q-pa-md justify-center content-contain">

        <div class="row justify-center">
          <div class="col">
            <EssentialLink />
          </div>
          <div class="col">
            <router-view />
          </div>
        </div>
      </div>
    </q-page-container>
  </q-layout>
</template>

<style>
.content-container {
  max-width: 1440px;
  align-self: start;
  width: 100%;
}
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
