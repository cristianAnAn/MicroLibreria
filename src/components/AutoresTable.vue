<template>
  <div class="table-container">
    <div class="search-container">
      <input
        ref="searchInput"
        type="text"
        v-model="searchQuery"
        @input="handleInput"
        @keydown.down="moveDown"
        @keydown.up="moveUp"
        @keydown.enter="selectAuthor"
        @blur="hideSuggestions"
        @focus="showSuggestions = true"
        placeholder="Buscar por nombre o apellido..."
        class="search-input"
      >
      <ul v-if="showSuggestions && autocompleteSuggestions.length" class="autocomplete-list">
        <li
          v-for="(suggestion, index) in autocompleteSuggestions"
          :key="index"
          @mousedown.prevent="selectAuthor(suggestion)"
          :class="{ 'highlighted': index === highlightedIndex }"
          class="autocomplete-item"
        >
          {{ suggestion.nombre }} {{ suggestion.apellido }}
        </li>
      </ul>
    </div>

    <div v-if="loading" class="loading">Cargando autores...</div>
    <div v-if="error" class="error">Error: {{ error }}</div>

    <table v-if="filteredAutores.length > 0" class="autores-table">
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Apellido</th>
          <th>Fecha Nacimiento</th>
          <th>Grados Académicos</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="autor in filteredAutores" :key="autor.autorLibroGuid">
          <td>{{ autor.nombre }}</td>
          <td>{{ autor.apellido }}</td>
          <td>{{ formatFecha(autor.fechaNacimiento) }}</td>
          <td>
            <div v-for="(grado, index) in autor.gradosAcademicos" :key="index" class="grado-academico">
              <p><strong>{{ grado.nombre }}</strong></p>
              <p>{{ grado.centroAcademico }}</p>
              <p>{{ formatFecha(grado.fechaGrado) }}</p>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="!loading && filteredAutores.length === 0" class="no-data">
      No se encontraron autores
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      autores: [],
      searchQuery: '',
      highlightedIndex: -1,
      loading: true,
      error: null,
      debounceTimer: null,
      showSuggestions: false
    };
  },
  computed: {
    filteredAutores() {
      if (!this.searchQuery) return this.autores;
      const searchTerms = this.searchQuery.toLowerCase().split(' ');
      return this.autores.filter(autor => {
        const nombreCompleto = `${autor.nombre?.toLowerCase()} ${autor.apellido?.toLowerCase()}`;
        return searchTerms.every(term => 
          nombreCompleto.includes(term)
        );
      });
    },
    autocompleteSuggestions() {
      const query = this.searchQuery.toLowerCase().trim();
      if (!query) return [];
      
      return this.autores.filter(autor => {
        const nombre = autor.nombre?.toLowerCase() || '';
        const apellido = autor.apellido?.toLowerCase() || '';
        return nombre.startsWith(query) || 
               apellido.startsWith(query) || 
               `${nombre} ${apellido}`.startsWith(query);
      }).slice(0, 5);
    }
  },
  mounted() {
    this.fetchAutores();
  },
  methods: {
    async fetchAutores() {
      try {
        const response = await axios.get('http://autoreslibros.somee.com/api/AutorLibro');
        this.autores = response.data;
      } catch (err) {
        this.error = err.message || 'Error al cargar los autores';
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
    handleInput() {
      this.showSuggestions = true;
      if (this.debounceTimer) clearTimeout(this.debounceTimer);
      this.debounceTimer = setTimeout(() => {
        this.highlightedIndex = -1;
      }, 300);
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
    selectAuthor(suggestion) {
      if (suggestion?.nombre && suggestion?.apellido) {
        this.searchQuery = `${suggestion.nombre} ${suggestion.apellido}`;
      } else if (this.highlightedIndex >= 0) {
        const selected = this.autocompleteSuggestions[this.highlightedIndex];
        if (selected) {
          this.searchQuery = `${selected.nombre} ${selected.apellido}`;
        }
      }
      
      this.showSuggestions = false;
      this.highlightedIndex = -1;
      this.$nextTick(() => {
        this.$refs.searchInput.focus();
      });
    }
  }
};
</script>

<style scoped>
.search-container {
  position: relative;
  margin-bottom: 1.5rem;
  max-width: 400px;
}

.search-input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #4a5568;
  border-radius: 4px;
  font-size: 1rem;
  transition: border-color 0.3s ease;
}

.search-input:focus {
  outline: none;
  border-color: #4299e1;
  box-shadow: 0 0 0 2px rgba(66, 153, 225, 0.2);
}

.autocomplete-list {
  position: absolute;
  width: 100%;
  max-height: 200px;
  overflow-y: auto;
  margin: 0;
  padding: 0;
  list-style: none;
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  z-index: 1000;
}

.autocomplete-item {
  padding: 0.75rem;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.autocomplete-item:hover,
.highlighted {
  background-color: #ebf8ff;
  color: #2b6cb0;
}

.autores-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 1rem;
}

.autores-table th,
.autores-table td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

.autores-table th {
  background-color: #4a5568;
  color: white;
}

.grado-academico {
  padding: 8px;
  margin: 4px 0;
  background-color: #f7fafc;
  border-radius: 4px;
}

.grado-academico p {
  margin: 4px 0;
}

.loading {
  color: #4299e1;
  padding: 1rem;
}

.error {
  color: #f56565;
  padding: 1rem;
}

.no-data {
  color: #a0aec0;
  padding: 1rem;
}
</style>