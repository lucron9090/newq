<template>
  <div class="q-pa-md q-gutter-sm">
    <q-input ref="filterRef" filled v-model="filter" label="Filter">
      <template v-slot:append>
        <q-icon v-if="filter !== ''" name="clear" class="cursor-pointer" @click="resetFilter" />
      </template>
    </q-input>

    <q-tree
      :nodes="data"
      node-key="title"
      child-node-key="child"
      :children-key="child"
      label-key="title"
      no-connectors
      :filter="filter"
      default-expand-all
    />
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import MenuData from './MenuData.json'

export default {
  setup () {
    const filter = ref('')
    const filterRef = ref(null)
    const data = ref([])


    function convertData(data) {
      return data.map(item => {
        return {
          title: item.title,
          child: convertData(item.child || [])
        };
      });
    }



    onMounted(() => {
      data.value = convertData(MenuData)
    })

    return {
      filter,
      filterRef,
      data,

      resetFilter () {
        filter.value = ''
        filterRef.value.focus()
      }
    }
  }
}
</script>
