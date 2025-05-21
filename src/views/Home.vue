<template>
  <div>
    <h1>Movie Search</h1>
    <input v-model="query" @keyup.enter="search" placeholder="Search for a movie..." />
    <div v-if="loading">Searching...</div>
    <div v-else-if="results.length">
      <ul>
        <li v-for="movie in results" :key="movie.id">
          {{ movie.title }} ({{ movie.release_date?.slice(0, 4) }})
        </li>
      </ul>
    </div>
    <div v-else>No results</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { searchMovies } from '../utils/tmdb.js'

const query = ref('')
const results = ref([])
const loading = ref(false)

async function search() {
  if (!query.value.trim()) return
  loading.value = true
  results.value = await searchMovies(query.value)
  loading.value = false
}
</script>