<template>
  <div>
    <h1>Movie Search</h1>
    <input
      v-model="query"
      @keyup.enter="search"
      placeholder="Type a movie and press Enter"
      class="movie-search"
    />
    <div v-if="loading">Loading...</div>
    <div v-if="results.length" class="movies-container">
      <div v-for="movie in results" :key="movie.id" class="movie-wrapper">
        <a :href="movie.url" target="_blank" rel="noopener" class="movie-link">
        <img :src="movie.poster || '/default.svg'" class="movie-image">
        <div class="movie-info-wrap">
          <p class="movie-title">
            {{ movie.title }} ({{ movie.year }})
          </p>
        </div>  
        </a>
      </div>
    </div>
    <div v-else-if="!loading && searched">No results found.</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const query = ref('')
const results = ref([])
const loading = ref(false)
const searched = ref(false)

async function search() {
  if (!query.value.trim()) return

  loading.value = true
  searched.value = false

  try {
    const res = await fetch(`https://search.imdbot.workers.dev/?q=${encodeURIComponent(query.value)}`)
    const data = await res.json()

    results.value = (data && data.description)
    ? data.description
        .filter(item => isEnglish(item['#TITLE']))
        .map(item => ({
          id: item['#IMDB_ID'],
          title: item['#TITLE'],
          year: item['#YEAR'],
          url: `https://www.imdb.com/title/${item['#IMDB_ID']}`,
          poster: item['#IMG_POSTER']
        }))
    : []
    } catch (e) {
      console.error('API search failed:', e)
      results.value = []
    } finally {
      loading.value = false
      searched.value = true
    }
}

function isEnglish(title) {
  return /^[\x00-\x7F]+$/.test(title) && /\b(the|of|lord|ring|bird|hunt|evil|empire)\b/i.test(title)
}
</script> 