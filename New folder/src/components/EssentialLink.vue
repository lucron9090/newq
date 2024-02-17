<template>
  <div>
    <h6 align="center">
      <div v-for="(item, index) in menuData" :key="index">
        {{ item.title }}
        <div>
          <div v-for="(child, index) in item.child" :key="index">
            {{ child.title }}
            <ul>
              <li v-for="(generator, index) in child.generator" :key="index">
                <q-btn align="between" class="btn-fixed-width" outline :color="index % 2 === 0 ? 'primary' : 'secondary'" :label="generator.name" :to="generator.slug" btn-fixed-width="50px"/>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </h6>
  </div>
</template>


<script>
import menuData from './MenuData.json'

export default {
  data() {
    return {
      menuData
    }
  }
}
</script>
<style>

.btn-fixed-width {
  min-width: 200px;
  max-width: 200px;
}

</style>
