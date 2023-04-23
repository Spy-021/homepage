<template>
  <div class="card-container bg-white">
    <!-- <img class="my-image" :src="imagePath" alt="NO Image"/> -->
    <img class="my-image" :id="imgID" alt="NO Image"/>
    <div class="card-author-section">
      <div class="author-avatar bg-primary-variant text-white">
        {{ props.item.avatar }}
      </div>
      <div class="date-name-container">
        <div>
          {{ props.item.author }}
        </div>
        <div class="text-accent font-size-sm">
          {{ props.item.dateAdded }}
        </div>
      </div>
    </div>
    <div class="card-title-section font-size-lg">{{ props.item.title }}</div>
    <div class="card-like-section">
      <button
        type="button"
        @click="toggleLike()"
        class="like-button"
        :class="{ 'like-button-active' : isLiked }"
      >
      LIKE
      </button>
      <div class="text-accent likes-container font-size-sm">
        <img src="@/assets/heart-icon.svg" alt=""/>
        <span>{{ localLikes }}</span>
        <span>likes</span>
      </div>
    </div>
  </div>
</template>

<style lang="css">


.card-container {
  /* --pa-cards: 1.25rem; */
  display: flex;
  border-radius: .25rem;
  flex-direction: column;
  padding: calc(var(--pa-cards) - .5rem) var(--pa-cards);
  width: max(calc(100% - calc(var(--pa-cards) * 2)), calc(228px - calc(var(--pa-cards) * 2)));

}

.card-container > * + * {
  margin-top: 0.7rem;
}

.my-image {
  height: 7rem;
  margin: calc(var(--pa-cards) * -1) calc(var(--pa-cards) * -1) 0 calc(var(--pa-cards) * -1);
  border-top-left-radius: .25rem;
  border-top-right-radius: .25rem;
  background-color: rgb(200, 224, 224);
}

.card-author-section {
  display: flex;
  align-items: center;

}

.card-author-section > * + * {
  margin-left: .75rem;
}

.author-avatar {
  --radios: 1.25rem;

  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: var(--radios);
  height: calc(var(--radios) * 2);
  width: calc(var(--radios) * 2);
}

.date-name-container {
  display: flex;
  flex-direction: column;
}

.card-title-section {
  margin-bottom: 0.7rem;
}

.card-like-section {
  display: flex;
  justify-content: space-between;
  margin-top: auto;
}

.likes-container {
  display: flex;
  justify-content: center;
  align-items: center;
}
.likes-container > * + * {
  margin-left: .3rem;
}

.like-button {
  padding: 0.2rem 1rem;
  color: var(--primary);
  background-color: transparent;
  /* color: white;
  background-color: var(--primary); */
  border: 1.5px var(--primary) solid;
  border-radius: .25rem;
  cursor: pointer;
}
.like-button[type=button]:hover {
  background-color: #e8f9ff;
  /* background-color: #019ad1; */
  -webkit-transition: background-color 100ms linear;
  -ms-transition: background-color 100ms linear;
  transition: background-color 100ms linear;
}

.like-button-active {
  color: white;
  background-color: var(--primary);
}

.like-button-active[type=button]:hover {
  background-color: #019ad1;
  -webkit-transition: background-color 100ms linear;
  -ms-transition: background-color 100ms linear;
  transition: background-color 100ms linear;
}


/* .card-content {
  background-color: white;
  width: max(100%, 228px);
} */

@media (min-width: 320px) and (max-width: 767px) {
  /* .my-image {
    height: 18rem;
  } */
}
/* @media (min-width: 768px) and (max-width: 1024px) {
  .my-image {
    height: 14rem;
  }
} */

</style>

<script setup>
import { ref, computed, onBeforeMount } from 'vue' // , computed, ref,
import { v4 as uuidv4 } from 'uuid'
import axios from 'axios'

const props = defineProps({
  item: {
    type: Object,
    require: true
  }
})

const imgID = uuidv4()

const isLiked = ref(false)
const localLikes = computed(() => {
  return isLiked.value ? props.item.likes + 1 : props.item.likes
})

function toggleLike () {
  isLiked.value = !isLiked.value
}
/**
 * p1 => portrait, landscape
 * p2 => 0, 1
 */
async function getImage () {
  const imgP = getPath('landscape', 1)
  // console.log('imgP', imgP)
  axios.get(imgP, {
    responseType: 'blob' // arraybuffer
  })
  .then((res) => {
    // const strIamge = Buffer.from(res.data, 'binary').toString('base64')
    // const strIamge = Buffer.from(res.data, 'binary')
    const strIamge = res.data
    // console.log('strIamge', strIamge)
    const urlCreator = window.URL || window.webkitURL;
    const imageUrl = urlCreator.createObjectURL(strIamge)
    const imgTag = document.getElementById(imgID)
    imgTag.src = imageUrl
  })
}
const getPath = (imgMode, imgCode) => {
  if (props.item?.images[0]?.[imgMode][imgCode])
    return props.item?.images[0]?.[imgMode][imgCode]
  if (props.item?.images?.[imgMode][imgCode]) {
    return props.item?.images?.[imgMode][imgCode]
  }
  console.warn('no valid image path found!')
  return 'somewhere'
}
// const imagePath = ref('')
onBeforeMount(getImage)
</script>
