
<template>
  <q-layout view="hhr Lpr ffr">
    <q-page-container class="fit row wrap justify-center items-start content-start">
      <div class="q-pa-md">
        <div class="row justify-center q-gutter-sm">
          <q-intersection class="example-item">
            <q-card flat bordered class="q-ma-sm">
              <EssentialLink />

              <q-card-section>
                <router-view />
              </q-card-section>
            </q-card>
          </q-intersection>
        </div>
      </div>
    </q-page-container>
  </q-layout>
</template>
<style lang="sass" scoped>
.example-item
  width: 290px
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
  setup () {
    const leftDrawerOpen = ref(false)
    const rightDrawerOpen = ref(false)

    return {
      leftDrawerOpen,
      toggleLeftDrawer () {
        leftDrawerOpen.value = !leftDrawerOpen.value
      },

      rightDrawerOpen,
      toggleRightDrawer () {
        rightDrawerOpen.value = !rightDrawerOpen.value
      }
    }
  }
});

</script>
