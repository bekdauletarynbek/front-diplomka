<template>
<div>
  <p class="text-gray-800 text-lg">Добро пожаловать в центр мониторинга Дамбами в Казахстане</p>
  <p class="text-gray-500 text-sm">Система для отслеживания возможных дыр в дамбах</p>
  <div class="list">
    <div class="item border border-solid border-gray-400 py-4 px-2 rounded-xl" v-for="item in oblasts" :key="item.id">
      <p>{{item.title}}</p>
      <div :class="`item_${item.id}`"></div>
    </div>
  </div>
</div>
</template>

<script>
export default {
  name: "listOf",
  data() {
    return {
      oblasts: [
        {
          id: 1,
          title: 'Караганда'
        },
        {
          id: 2,
          title: "Астана"
        }
      ]
    }
  },
  methods: {
    getDataOf() {

    }
  }
}
</script>

<style scoped>

</style>
