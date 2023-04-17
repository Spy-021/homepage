<template>
  <main class="page-background">
    <div
      class="header bg-primary-variant"
    >
    </div>

    <div class="page">
      <!-- flex-grow -->
      <div class="page-content">
        <div class="xl-font-size text-white">dev articles</div>
        <div class="menu bg-white">
          <span class="menu-items">
            <img src="@/assets/search-icon.svg" alt=""/>
            <input
              v-model="filterText"
              placeholder="Filter by title, author..."
              class="input-text-filter font-size-inherit"
            >
          </span>
          <span class="menu-items">
            <!-- placeholder="All Results" -->
            <select
              v-model="sortOption"
              class="font-size-inherit text-accent sort-dropdown"
            >
              <option disabled value="">All Results</option>
              <option
                v-for="option in options"
                :value="option.value"
                :key="option.value"
              >
                {{ option.text }}
              </option>

            </select>
          </span>
          <span class="menu-items">
            <input type="checkbox" id="checkbox" v-model="latestCheckbox" />
            <label class="text-accent" for="checkbox">Latest only</label>
          </span>
        </div>
        <div class="cards-container bg-primary">
          <CardItem
            v-for="(itm, i) of items"
            :key="i"
            :item="itm"
          />
        </div>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted } from 'vue' // , computed
import CardItem from '@/components/CardItem.vue'
import axios from 'axios'

const DEV_MODE = process.env.NODE_ENV !== 'production'
const API_URI = DEV_MODE
  ? 'http://localhost:8080/api.json'
  : 'https://myposter.de/web-api/job-interview'

const filterText = ref('')
const latestCheckbox = ref('')
const sortOption = ref('')
const options = [
  {
    text: 'Authors A - Z',
    value: 'authorAZ'
  },
  {
    text: 'Authors Z - A',
    value: 'authorZA'
  },
  {
    text: 'Date ASC',
    value: 'dateASC'
  },
  {
    text: 'Date DESC',
    value: 'dateDESC'
  }
]
const items = ref([])

onMounted(() => {
  axios({
    method: 'get',
    url: API_URI
  }).then(({ data }) => {
    const { message, payload } = data
    // console.log('fetch message', message)
    if (message.status === 'success') {
      if (DEV_MODE) {
        console.log('first item', payload.data[0])
        for (let i = 0; i < payload.data.length * 3; i++) {
          items.value.push(...payload.data)
        }
      } else {
        items.value = payload.data
      }
    }
    // console.log('fetch payload', payload)
  })
})
</script>


<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  components: {
    // HelloWorld
  }
}
</script>

<style scoped>

.sort-dropdown {
  padding: .5rem;
  border: none;
}

input[type="checkbox"] {
  appearance: none;
  background-color: #EEE;
  margin: 0;
  font: inherit;
  width: 1.15em;
  height: 1.15em;
  border: 0;
  border-radius: 0;
  transform: translateY(-0.075em);

  display: grid;
  place-content: center;
}
input[type="checkbox"]::before {
  content: "";
  width: 1em;
  height: 1em;
  transform: scale(0);
  transition: 120ms transform ease-in-out;
  box-shadow: inset 1em 1em var(--primary-variant);
}

input[type="checkbox"]:checked::before {
  transform: scale(1);
}

.input-text-filter {
  padding: .5rem;
  border: none;
}

.menu-items {
  display: flex;
  align-items: center;
  font-size: 1rem;
}
.menu-items > * + * {
  margin-left: .35rem;
}

.space-y > :not([hidden]) ~ :not([hidden]) {
  --tw-space-y-reverse: 0;
  --space: 1rem;
  margin-top: calc(1rem * calc(1 - var(--tw-space-y-reverse)));
  margin-bottom: calc(1rem * var(--tw-space-y-reverse));
  /* margin-top: calc(var(--space) * calc(1 - var(--tw-space-y-reverse)));
  margin-bottom: calc(var(--space) * var(--tw-space-y-reverse)); */
}

/* .space-x-4 > :not([hidden]) ~ :not([hidden]) {
  --tw-space-x-reverse: 0;
  margin-right: calc(1rem * var(--tw-space-x-reverse));
  margin-left: calc(1rem * calc(1 - var(--tw-space-x-reverse)));
} */
.page-background {
  /* position: relative;
  display: flex;
  flex-direction: column; */
  height: 100vh;

  /* @apply space-y; */
  /* background-color: red; */
}

.menu {
  /* background-color: red; */
  margin-top: 2rem;
  border-radius: .3rem;
  padding: 1.5rem;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
}
.menu-items + .menu-items {
  margin-left: .75rem;
}

.page {
  position: absolute;
  /* padding: 1rem; */
  height: 100vh;
  width: 100vw;
  top: 0;
  left: 0;

  display: flex;
  justify-content: center;
  /* flex-direction: column; */

  /* background-color: aqua; */
}

.page-content {
  margin-top: 3rem;
  margin-bottom: 3rem;

  width: clamp(320px, 70%, 920px);
  /* background-color: rgb(22, 17, 17); */
}

.flex-grow {
  flex-grow: 1
}

.header {
  height: 9rem;
}

.cards-container {
  margin-top: 2rem;
  display: grid;
  row-gap: 1.75rem;
  column-gap: 1rem;
  grid-template-columns: repeat(3, minmax(0, 1fr));
}

/** Mobile */
@media (max-width: 767px) {
  .menu {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
  .menu-items + .menu-items {
    margin-top: .75rem;
  }

  .cards-container {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
}

/** Tablet */
@media (min-width: 768px) and (max-width: 1024px) {
  .cards-container {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

</style>
