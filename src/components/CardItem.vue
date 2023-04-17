<template>
  <div class="card-container bg-white">
    <img class="my-image" :src="props.item.images.portrait[0]" alt="NO Image"/>
    <!-- <div class="my-image">image replacer</div> -->
    <div class="card-author-section">
      {{ props.item.author }}
    </div>
    <div>{{ props.item.title }}</div>
    <div class="card-like-section">
      <div>button</div>
      <div>likes</div>
    </div>
  </div>
</template>

<style lang="css">

.card-container {
  display: flex;
  border-radius: .25rem;
  flex-direction: column;
  padding: .75rem;
  width: max(calc(100% - 1.5rem), calc(228px - 1.5rem));

}

.card-container > * + * {
  margin-top: 0.7rem;
}

.my-image {
  height: 5rem;
  margin: -0.75rem -0.75rem 0 -0.75rem;
  border-top-left-radius: .25rem;
  border-top-right-radius: .25rem;
  background-color: aqua;
}

.card-like-section {
  display: flex;
  justify-content: space-between;
}

/* .card-content {
  background-color: white;
  width: max(100%, 228px);
} */

</style>

<script setup>
import { onMounted } from 'vue' // , computed, ref,

const props = defineProps({
  item: {
    type: Object,
    require: true
  }
})

onMounted(() => {
  console.log('cards moun', props.item.images.portrait[0])
})
</script>
