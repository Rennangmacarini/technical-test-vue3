<template>
  <div class="container">
    <div class="search-container">
      <img src="../assets/logo.png" alt="">
      <input type="text" v-model="searchQuery" @input="searchGifs" placeholder="Search GIFs" />
    </div>
    <div class="content">
      <div v-for="gif in Giphy" :key="gif.id">
        <img :src="gif.images.fixed_height.url" :alt="gif.title" @click="openModal(gif)" />
      </div>
    </div>
    <div class="modal" v-if="modalOpen" @click="closeModal">
      <img :src="gif.images.fixed_height.url" :alt="gif.title" />
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted } from 'vue'
import { GiphyFetch } from '@giphy/js-fetch-api'
import '../css/app.css'

const giphyFetch = new GiphyFetch('sXpGFDGZs0Dv1mmNFvYaGUvYwKX0PWIh')

export default {
  name: 'IndexPage',
  setup () {
    const gif = ref(null)
    const Giphy = ref([])
    const searchQuery = ref('')
    const modalOpen = ref(false)

    onMounted(async () => {
      try {
        const gifData = await fetchRandomGif()
        gif.value = gifData

        const GiphyData = await fetchRandomGifs(20)
        Giphy.value = GiphyData
      } catch (error) {
        console.error('Failed to fetch GIFs', error)
      }
    })

    onMounted(() => {
      window.addEventListener('scroll', handleScroll)
    })

    onUnmounted(() => {
      window.removeEventListener('scroll', handleScroll)
    })

    async function fetchRandomGif () {
      const { data } = await giphyFetch.random()
      return data
    }

    async function fetchRandomGifs (limit) {
      const { data } = await giphyFetch.trending({ limit })
      return data
    }

    async function searchGifs () {
      try {
        if (searchQuery.value === '') {
          const GiphyData = await fetchRandomGifs(20)
          Giphy.value = GiphyData
          return
        }

        const searchResults = await fetchSearchGifs(searchQuery.value, 20)
        Giphy.value = searchResults
      } catch (error) {
        console.error('Failed to search GIFs', error)
      }
    }

    async function fetchSearchGifs (query, limit) {
      const { data } = await giphyFetch.search(query, { limit })
      return data
    }

    async function loadMoreGifs () {
      try {
        const moreGifs = await fetchRandomGifs(20)
        Giphy.value = [...Giphy.value, ...moreGifs]
      } catch (error) {
        console.error('Failed to load more GIFs', error)
      }
    }

    function openModal (selectedGif) {
      gif.value = selectedGif
      modalOpen.value = true
    }

    function closeModal () {
      modalOpen.value = false
    }

    function handleScroll () {
      const scrollHeight = document.documentElement.scrollHeight
      const scrollTop = window.pageYOffset || document.documentElement.scrollTop
      const clientHeight = document.documentElement.clientHeight

      if (scrollHeight - scrollTop === clientHeight) {
        loadMoreGifs()
      }
    }

    return {
      gif,
      Giphy,
      searchQuery,
      searchGifs,
      modalOpen,
      openModal,
      closeModal,
    }
  }
}
</script>

<style>

.container {
  padding: 25px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.8);
  z-index: 999;
}

.modal img {
  max-width: 100%;
  max-height: 100%;
}

.content {
  column-count: 4;
  column-gap: 25px;
  margin-top: 3rem;
}

.content img {
  width: 100%;
  margin-bottom: 25px;
  cursor: pointer;
}

.content img:hover{
  transition: 0.3s ease-in-out;
 transform: scale(1.1);
}

.search-container {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.search-container img {
 width: 200px;
}

.search-container input {
  width: 450px;
  height: 50px;
  padding: 0.8rem;
  margin: 3rem 0rem;
  margin-right: 8px;
  font-size: 25px;
  border: 1px solid rgb(156, 155, 155);
  border-radius: 18px;
}

.search-container button {
  padding: 4px 8px;
}

</style>
