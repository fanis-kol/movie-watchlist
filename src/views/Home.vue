<template>
  <div>
    <h1>Movie Search</h1>
    <input
      v-model="query"
      @keyup.enter="search"
      placeholder="Type a movie and press Enter"
    />
    <div v-if="loading">Loading...</div>
    <ul v-if="results.length">
      <li v-for="movie in results" :key="movie.id">
        {{ movie.title }} ({{ movie.year }})
      </li>
    </ul>
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
          year: item['#YEAR']
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