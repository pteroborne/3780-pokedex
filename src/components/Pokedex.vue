<template>
  <div>
    <h1>Pokedex</h1>
    <div class="filters">
      <input v-model="search" placeholder="Search for a pokemon...">
      <label for="poke-type">Type</label>
      <select v-model="selectedType" name="poke-type">
        <option value="">All</option>
        <option v-for="type in types" :key="type" :value="type">{{ type }}</option>
      </select>
      <button @click="toggleFavorites">{{ showFavorites ? 'Show all' : 'Show favorites' }}</button>
    </div>
    <div class="cards">
      <div v-for="pokemon in filteredPokemon" :key="pokemon.name" class="card">
        <img :src="pokemon.image" :alt="pokemon.name">
        <h3>{{ pokemon.name }}</h3>
        <p>Type: {{ pokemon.type }}</p>
        <p>Description: {{ pokemon.description }}</p>
        <button v-if="!pokemon.isFavorite" @click="addToFavorites(pokemon)">Add to favorites</button>
        <button v-else @click="removeFromFavorites(pokemon)">Remove from favorites</button>
      </div>
    </div>
    <h2 v-if="showFavorites">Favorites</h2>
    <div class="cards" v-if="showFavorites">
      <div v-for="favorite in favorites" :key="favorite.name" class="card">
        <img :src="favorite.image" :alt="favorite.name">
        <h3>{{ favorite.name }}</h3>
        <p>Type: {{ favorite.type }}</p>
        <p>Description: {{ favorite.description }}</p>
        <button @click="removeFromFavorites(favorite)">Remove from favorites</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      pokemon: [],
      search: '',
      selectedType: '',
      favorites: [],
      showFavorites: false
    }
  },
  async mounted() {
    const {data} = await axios.get('https://pokeapi.co/api/v2/pokemon?limit=151')
    this.pokemon = await Promise.all(data.results.map(async (p, index) => {
      const {data: pokemonData} = await axios.get(p.url)
      const type = pokemonData.types[0].type.name
      const {data: speciesData} = await axios.get(`https://pokeapi.co/api/v2/pokemon-species/${index + 1}`)
      const description = speciesData.flavor_text_entries.find(entry => entry.language.name === 'en').flavor_text.replace(/[\n\f]/g, ' ')
      return {
        name: p.name,
        url: p.url,
        image: pokemonData.sprites.front_default,
        type,
        description,
        isFavorite: false
      }
    }))
  },
  computed: {
    filteredPokemon() {
      const allPokemon = this.pokemon.filter(p => {
        const nameMatch = p.name.toLowerCase().includes(this.search.toLowerCase())
        const typeMatch = !this.selectedType || p.type === this.selectedType
        return nameMatch && typeMatch
      })
      if (this.showFavorites) {
        return allPokemon.filter(p => p.isFavorite)
      } else {
        return allPokemon
      }
    },
    types() {
      const types = new Set()
      this.pokemon.forEach(p => types.add(p.type))
      return Array.from(types).sort()
    }
  },
  methods: {
    addToFavorites(pokemon) {
      pokemon.isFavorite = true
      this.favorites.push(pokemon)
    },
    removeFromFavorites(pokemon) {
      pokemon.isFavorite = false
      this.favorites = this.favorites.filter(favorite => favorite.name !== pokemon.name)
    },
    toggleFavorites() {
      this.showFavorites = !this.showFavorites
    }
  }
}
</script>

<style>
.filters {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin-bottom: 16px;
}

.cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 16px;
}

.card {
  display: flex;
  flex-direction: column;
  padding: 1rem;
  background-color: #f1f1f1;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

img {
  width: 128px;
  height: 128px;
  object-fit: contain;
  margin-bottom: 16px;
}

h2 {
  margin-top: 32px;
  margin-bottom: 16px;
}

button {
  padding: 8px 16px;
  font-size: 16px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 16px;
}
</style>
