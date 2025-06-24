<template>
  <div>
    <h1>Movie Search</h1>
    <input
      v-model="query"
      @keyup.enter="search"
      placeholder="Type a movie and press Enter"
      class="movie-search"
    />

    <div v-if="watchlist.length" class="watchlist-section">
      <h2>Your Watchlist</h2>
      <div class="movies-container">
        <div v-for="movie in watchlist" :key="movie.id" class="movie-wrapper">
          <img :src="movie.poster || '/default.svg'" class="movie-image" />
          <div class="movie-info-wrap">
            <p class="movie-title">{{ movie.title }} ({{ movie.year }})</p>
          </div>
          <button @click="removeFavorite (movie)">Remove Favorites</button>
        </div>
      </div>
    </div>
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
        <button @click="addFavorite (movie)">Add to Favorites</button>
      </div>
    </div>
    <div v-else-if="!loading && searched">No results found.</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const query = ref('')
const results = ref([])
const loading = ref(false)
const searched = ref(false)
const watchlist = ref([]);


onMounted(() => {
  const stored = JSON.parse(localStorage.getItem('watchlist') || '[]');
  watchlist.value = stored;
});

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


function addFavorite (movie){
  const stored = JSON.parse(localStorage.getItem('watchlist') || '[]');

  if (!stored.some(m => m.id === movie.id)) {
    stored.push(movie);
    localStorage.setItem('watchlist', JSON.stringify(stored));
    watchlist.value = watchlist.value.filter(m => m.id !== movie.id);

  }
}

function removeFavorite(movie) {
  localStorage.setItem('watchlist', JSON.stringify(
    JSON.parse(localStorage.getItem('watchlist') || '[]')
      .filter(m => m.id !== movie.id)
  ));
}

function isEnglish(title) {
  return /^[\x00-\x7F]+$/.test(title) && /\b(the|of|lord|ring|bird|hunt|evil|empire)\b/i.test(title)
}
</script> 