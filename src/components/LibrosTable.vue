<template>
  <div class="table-container">
    <!-- Buscador de libros -->
    <div class="search-container">
      <input
        ref="searchInput"
        type="text"
        v-model="searchQuery"
        @input="handleInput"
        @keydown.down.prevent="moveDown"
        @keydown.up.prevent="moveUp"
        @keydown.enter.prevent="handleEnter"
        @blur="hideSuggestions"
        @focus="showSuggestions = true"
        placeholder="Buscar libro por título..."
        class="search-input"
      >
      <ul v-if="showSuggestions && autocompleteSuggestions.length" class="autocomplete-list">
        <li
          v-for="(suggestion, index) in autocompleteSuggestions"
          :key="suggestion.libreriaMateriaID"
          @mousedown.prevent="() => selectBook(suggestion)"
          :class="{ 'highlighted': index === highlightedIndex }"
          class="autocomplete-item"
        >
          {{ suggestion.titulo }}
        </li>
      </ul>
    </div>

    <!-- Mensajes de estado -->
    <div v-if="loading" class="loading">Cargando libros...</div>
    <div v-if="error" class="error">Error: {{ error }}</div>

    <!-- Tabla de libros -->
    <table v-if="filteredLibros.length > 0" class="libros-table">
      <thead>
        <tr>
          <th>Título</th>
          <th>Fecha de Publicación</th>
          <th>Autor</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="libro in filteredLibros" :key="libro.libreriaMateriaID">
          <td>{{ libro.titulo }}</td>
          <td>{{ formatFecha(libro.fechaPublicacion) }}</td>
          <td>{{ obtenerNombreAutor(libro.autorLibro) }}</td>
        </tr>
      </tbody>
    </table>

    <div v-if="!loading && filteredLibros.length === 0" class="no-data">
      No se encontraron libros
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import '../LibrosTable.css'
export default {
  data() {
    return {
      libros: [],
      autores: [],
      searchQuery: '',
      loading: true,
      highlightedIndex: -1,
      error: null,
      showSuggestions: false,
      debounceTimer: null
    };
  },
  computed: {
    filteredLibros() {
      if (!this.searchQuery) return this.libros;
      const searchTerms = this.searchQuery.toLowerCase().split(' ');
      return this.libros.filter(libro => {
        const titulo = libro.titulo?.toLowerCase() || '';
        return searchTerms.every(term => titulo.includes(term));
      });
    },
    autocompleteSuggestions() {
      const query = this.searchQuery.toLowerCase().trim();
      if (!query) return [];
      
      return this.libros
        .filter(libro => {
          const titulo = libro.titulo?.toLowerCase() || '';
          return titulo.includes(query);
        })
        .slice(0, 5);
    }
  },
  mounted() {
    this.fetchDatos();
  },
  methods: {
    async fetchDatos() {
      try {
        const [librosResponse, autoresResponse] = await Promise.all([
          axios.get('https://libreriamaterial.somee.com/api/LibroMaterial'),
          axios.get('https://autoreslibros.somee.com/api/AutorLibro')
        ]);
        
        this.libros = librosResponse.data;
        this.autores = autoresResponse.data;
        
      } catch (err) {
        this.error = err.message || 'Error al cargar los datos';
        console.error('Error fetching data:', err);
      } finally {
        this.loading = false;
      }
    },
    formatFecha(fecha) {
      return new Date(fecha).toLocaleDateString('es-ES', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    },
    obtenerNombreAutor(autorId) {
      const autor = this.autores.find(a => a.autorLibroGuid === autorId);
      return autor ? `${autor.nombre} ${autor.apellido}` : 'Desconocido';
    },
    handleInput() {
      this.showSuggestions = true;
      this.highlightedIndex = -1;
      clearTimeout(this.debounceTimer);
    },
    hideSuggestions() {
      setTimeout(() => {
        this.showSuggestions = false;
      }, 200);
    },
    moveDown() {
      if (this.highlightedIndex < this.autocompleteSuggestions.length - 1) {
        this.highlightedIndex++;
      }
    },
    moveUp() {
      if (this.highlightedIndex > 0) {
        this.highlightedIndex--;
      }
    },
    handleEnter() {
      if (this.autocompleteSuggestions.length > 0) {
        const selected = this.highlightedIndex >= 0 
          ? this.autocompleteSuggestions[this.highlightedIndex]
          : this.autocompleteSuggestions[0];
        
        this.selectBook(selected);
      } else {
        this.showSuggestions = false;
        this.$refs.searchInput.blur();
      }
    },
    selectBook(suggestion) {
      if (suggestion?.titulo) {
        this.searchQuery = suggestion.titulo;
        this.showSuggestions = false;
        this.highlightedIndex = -1;
        this.$nextTick(() => {
          this.$refs.searchInput.focus();
        });
      }
    }
  }
};
</script>

<style scoped>

</style>