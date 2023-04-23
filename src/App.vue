<template>
  <main class="page-background">
    <div
      class="header bg-primary-variant"
    >
    </div>

    <div class="page">
      <!-- flex-grow -->
      <div class="page-content">
        <div class="font-size-xl text-white">dev articles</div>
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
        <!-- bg-primary -->
        <div class="cards-container">
          <CardItem
            v-for="(obj, i) of filteredItems"
            :key="i"
            :item="obj.item"
          />
        </div>
      </div>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, onBeforeMount } from 'vue' // , computed, onMounted
import CardItem from '@/components/CardItem.vue'
import axios from 'axios'

const DEV_MODE = process.env.NODE_ENV !== 'production'
const API_URI = DEV_MODE
  ? 'http://localhost:8080/real_data.json'
  : 'https://myposter.de/web-api/job-interview'

const filterText = ref('')
const latestCheckbox = ref(false)
const sortOption = ref('')
const options = [
  {
    text: 'Authors A - Z',
    value: 'author-asc'
  },
  {
    text: 'Authors Z - A',
    value: 'author-desc'
  },
  {
    text: 'Date ASC',
    value: 'dateAdded-asc'
  },
  {
    text: 'Date DESC',
    value: 'dateAdded-desc'
  }
]

const items = ref([])
const dateLastYear = (() => {
  const nD = new Date()
  return nD.setFullYear(nD.getFullYear() - 1) && nD
})()

// debounced!
const filteredItems = computed(() => {
  // step 1 => filter title
  // step 2 => if latest one year ?
  // step 3 => sort
  // console.log('sortOption', sortOption.value) // last step is sorting!
  // console.log('latestCheckbox', latestCheckbox.value)

  const filtered = items.value.filter((c) => {
    const titleAuthorFiltered = !filterText.value || (c.orig.author.includes(filterText.value) || c.orig.title.includes(filterText.value))
    const latestFiltered = !latestCheckbox.value || (() => {
      const registDate = new Date(c.orig.dateAdded)
      return !isNaN(registDate.getTime()) && dateLastYear < registDate
    })()

    return titleAuthorFiltered && latestFiltered
  })

  const [sortBy, dir ] = (() => {
    const [sortBy, asc ] = sortOption.value.split('-')
    return [sortBy, (asc === 'asc' ? 1 : -1) ]
  })()
  // console.log('sortBy', sortBy)
  // console.log('dir', dir)

  return !sortOption.value ? filtered : filtered.sort((a, b) => {
    const [A, B] = [a.orig[sortBy], b.orig[sortBy]]
    if (A < B) return -1 * dir
    if (A > B) return 1 * dir
    return 0
  })
})

function capStr (s) {
  return s.charAt(0)?.toUpperCase() + s.slice(1) || s // fallback if empty str!
}

function capAuthor (p) {
  return p.trim().split(' ').map((s) => capStr(s)).join(' ')
}

const dateFormatter = new Intl.DateTimeFormat('default', { day: 'numeric',  month: 'short' }) // year: 'numeric'
function dateConverter (d) { // d => dateString e.g. '1970-01-01'
  const md = new Date(d) // myDate
  if (isNaN(md.getTime())) return d

  // de-DE, default
  // formatToParts or reverse it!
  const res = dateFormatter.format(md).split(' ').reverse().join(' ')
  return res
}

function getAuthorAbrev (p) {
  if (!(typeof p === 'string' && p.length > 0)) return 'N/A'

  // console.log('func getAuthorAbrev', p)
  const firstLetters = p.trim().split(' ').map((e) => e[0]).join('')

  // What about people with 3+ names ?! // {2,}
  if (/^[a-zA-Z]{2}$/.test(firstLetters)) return firstLetters.toUpperCase()
  return 'N/A'
}

let count = 0
function prepareCardValues (p) {
  // p.author = capAuthor('pan max' + ++count)
  p.author = capAuthor(p.author)
  p.dateAdded = dateConverter(p.dateAdded)
  p.avatar = getAuthorAbrev(p.author)
  p.title = capStr(p.title)
  return p
}

function deepClone (p) {
  return JSON.parse(JSON.stringify(p))
}

onBeforeMount(() => {
  axios({
    method: 'get',
    url: API_URI
  }).then(({ data }) => {
    const { message, payload } = data
    // console.log('fetch message', message)
    if (message.status === 'success') {
      items.value.length = 0 // reset

      if (!DEV_MODE) {
        for (let i = 0; i < payload.data.length * 3; i++) {
          // items.value.push(
          //   ...payload.data.map((card) => ({ orig: card, display: true, item: prepareCardValues({ ...card }) }))
          // )
          items.value.push(
            ...payload.data.map((card) => {
              count++
              if (count % 2 === 0) card.dateAdded = new Date('2022-11-12')
              return { orig: card, display: true, item: prepareCardValues({ ...card }) }
            })
          )

        }
      } else {
        items.value = payload.data.map((card) => ({ orig: card, display: true, item: prepareCardValues(deepClone(card)) }))
      }
    }
    // console.log('fetch payload', payload)
    // console.log('first item', items.value[0])

  }).catch((err) => {
    console.warn('fecth data failed', err)
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
  justify-content: center;
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
  grid-template-columns: repeat(3, minmax(auto, 1fr));
}
.menu-items + .menu-items {
  margin-left: .75rem;
}

.page {
  position: absolute;
  /* padding: 1rem; */
  height: 100vh;
  width: 100%;
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
  margin-bottom: 2rem;
  display: grid;
  row-gap: 1.75rem;
  column-gap: 1rem;
  grid-template-columns: repeat(3, minmax(0, 1fr));
}

/** Mobile */
@media (min-width: 320px) and (max-width: 767px) {
  .menu {
    grid-template-columns: repeat(1, minmax(0, 1fr));
  }
  .menu-items {
    justify-content: flex-start;
  }
  .menu-items + .menu-items {
    margin-top: 1.25rem;
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
