<template>
  <div class="shadow shadow-lg mb-3">
    <div class="w-full bg-white flex items-center text-green-800">
      <div class="h-full w-12 border-r border-grey-400 text-center flex items-center">
        <div class="flex-1 opacity-50 text-3xl px-2">{{ park.resultNumber }}</div>
      </div>
      <div class="h-full flex-grow flex items-center p-2">
        <div class="text-lg flex-1 my-4">
          <span class="block uppercase font-medium">{{ park.name }}</span>
          <span v-if="park.rating" class="text-sm opacity-75">{{ park.rating }}</span>
          <span v-if="park.user_ratings_total" class="text-sm opacity-50"> ({{ park.user_ratings_total }} reviews)</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ParkResult",
  props: ['park'],
  data() {
    return {
     
    };
  }
};
</script>

<style scoped>
</style>