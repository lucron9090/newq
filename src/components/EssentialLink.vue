<template>
  <!-- Container for the component with a specific style -->
  <div class="q-pa-md" style="max-width: 350px">
    <!-- List component with borders and rounded corners -->
    <q-list bordered class="rounded-borders">
      <!-- Show the spinner if isLoading is true -->
      <q-linear-progress v-if="isLoading" dark query color="cyan" class="q-mt-sm" />
      <!-- Show the content if isLoading is false -->
      <q-item v-else v-for="(item, index) in menuData" :key="index" > <!-- id="`list-${id}`"> -->
        <!-- Expansion item with a specific label and model -->
        <q-expansion-item expand-separator :label="item.title" v-model="item.open" :icon="`img:https://oldie.veriftools.ru/media/${item.icon}`">
          <div v-if="item.open">
            <!-- Loop through child items -->
            <div v-for="(child, childIndex) in item.child" :key="childIndex">
              <q-item-section avatar>
                <q-avatar size="16px">
                  <!-- Child item icon -->
                </q-avatar>
              </q-item-section>

              <q-expansion-item :header-inset-level="1" expand-separator :label="child.title" v-model="child.open" :icon="`img:https://oldie.veriftools.ru/media/${child.icon}`">
                <div v-if="child.open">
                  <div v-if="child.generator && child.generator.length > 0">
                    <!-- Loop through child generators -->
                    <q-card v-for="(generator, generatorIndex) in child.generator" :key="generatorIndex" :icon="`img:https://oldie.veriftools.ru/media/${generator.icon}`">
                      <q-avatar size="16px">
                        <img :src="'https://oldie.veriftools.ru/media/' + generator.icon">
                      </q-avatar>
                      <q-btn flat :style="{ color: generatorIndex % 2 === 0 ? '#FF0080' : '#0000FF' }"
                        :label="generator.name" @click="handleSlugSelection(generator.slug)"></q-btn>
                    </q-card>
                  </div>
                </div>
              </q-expansion-item>
            </div>
            <div v-if="item.generator && item.generator.length > 0">
              <!-- Loop through item generators -->
              <q-card v-for="(igenerator, igeneratorIndex) in item.generator" :key="igeneratorIndex" :icon="`img:https://oldie.veriftools.ru/media/${igenerator.icon}`" >
                <q-avatar size="16px">
                  <img :src="'https://oldie.veriftools.ru/media/' + igenerator.icon">
                </q-avatar>
                <q-btn flat :style="{ color: igeneratorIndex % 2 === 0 ? '#FF0080' : '#0000FF' }" :label="igenerator.name"
                  @click="handleSlugSelection(igenerator.slug)"></q-btn>
              </q-card>
            </div>
          </div>
        </q-expansion-item>
      </q-item>
    </q-list>
  </div>
</template>

<script>
import menuData from './MenuData.json' // Import menu data from a JSON file
import { bus } from './Event-Bus' // Import bus from Event-Bus file
export default {
  data() {
    return {
      menuData, // Initialize menuData from imported JSON
      isLoading: false // Initialize isLoading as false
    }
  },
  created() {

  },
  methods: {
    handleSlugSelection(slug) {
      bus.emit('slugSelected', slug) // Emit event with selected slug
    },
  }
}
</script>
<style></style>
