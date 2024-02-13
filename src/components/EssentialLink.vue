<template>
  <div class="q-pa-md" style="max-width: 350px">
    <q-list bordered class="rounded-borders">
      <div v-for="(item, index) in menuData" :key="index">
        <q-avatar size="16px">
                          <img :src="'https://oldie.veriftools.ru/media/' + item.icon">
                        </q-avatar>
        <q-expansion-item expand-separator :label="item.title" v-model="item.open">



          <div v-if="item.open">
            <div v-for="(child, childIndex) in item.child" :key="childIndex">
              <q-item-section avatar>
              <q-avatar size="16px">
                          <img :src="'https://oldie.veriftools.ru/media/' + child.icon">
                        </q-avatar>
                      </q-item-section>


              <q-expansion-item :header-inset-level="1" expand-separator :label="child.title" v-model="child.open">

                <div v-if="child.open">
                  <div v-if="child.generator && child.generator.length > 0">
                    <q-card v-for="(generator, generatorIndex) in child.generator" :key="generatorIndex">
                        <q-avatar size="16px">
                          <img :src="'https://oldie.veriftools.ru/media/' + generator.icon">
                        </q-avatar>
                       <q-btn flat :style="{ color: generatorIndex % 2 === 0 ? '#FF0080' : '#0000FF' }" :label="generator.name" @click="handleSlugSelection(igenerator.slug)"></q-btn>
                    </q-card>
                  </div>
                </div>

              </q-expansion-item>


            </div>
            <div v-if="item.generator && item.generator.length > 0">
                  <q-card v-for="(igenerator, igeneratorIndex) in item.generator" :key="igeneratorIndex">
                    <q-avatar size="16px"><img :src="'https://oldie.veriftools.ru/media/' + igenerator.icon"></q-avatar>
                    <q-btn flat :style="{ color: igeneratorIndex % 2 === 0 ? '#FF0080' : '#0000FF' }" :label="igenerator.name"  @click="handleSlugSelection(igenerator.slug)"></q-btn>
                  </q-card>
                </div>
          </div>



        </q-expansion-item>
      </div>
    </q-list>
  </div>
</template>


<script>
import menuData from './MenuData.json'
import Vue from 'vue'
import { EventBus } from './EventBus.js'
export default {
  data() {
    return {
      menuData
    }
  },

  methods: {
    handleSlugSelection(slug) {
      EventBus.$emit('slugSelected', slug)
    },
  async fetchFormData() {
      this.isLoading = true
      try {
        const response = await fetch('http://localhost:9000/api/frontend/category/') //https://oldie.veriftools.ru/api/frontend/category/ http://localhost:9000/api/frontend/category/
        const { steps } = await response.json()
        this.steps = steps.map(step => ({ ...step, done: false }))
        this.step = this.steps[0].name
      } catch (error) {
        console.error('Error fetching form data:', error)
        this.error = error
      } finally {
        this.isLoading = false
      }
    }
  }

}
</script>
<style>

</style>
