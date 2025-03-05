<template>
  <div class="form-container">
    <h2>Agregar Nuevo Libro</h2>
    
    <form @submit.prevent="submitForm">
      <div class="form-group">
        <label>Título:</label>
        <input 
          type="text" 
          v-model="nuevoLibro.titulo"
          required
        >
      </div>

      <div class="form-group">
        <label>Fecha de Publicación:</label>
        <input
          type="datetime-local"
          v-model="nuevoLibro.fechaPublicacion"
          required
        >
      </div>

      <div class="form-group">
        <label>Autor:</label>
        <select
          v-model="nuevoLibro.autorLibro"
          required
          class="custom-select"
          :disabled="loadingAutores"
        >
          <option 
            v-if="loadingAutores" 
            value="" 
            disabled
          >
            Cargando autores...
          </option>
          <option 
            v-for="autor in autores" 
            :key="autor.autorLibroGuid" 
            :value="autor.autorLibroGuid"
          >
            {{ autor.nombre }} {{ autor.apellido }}
          </option>
          <option 
            v-if="errorAutores" 
            value="" 
            disabled
          >
            Error cargando autores
          </option>
        </select>
      </div>

      <button 
        type="submit" 
        :disabled="loading"
        class="submit-btn"
      >
        {{ loading ? 'Enviando...' : 'Agregar Libro' }}
      </button>

      <div v-if="successMessage" class="success-message">
        {{ successMessage }}
      </div>

      <div v-if="error" class="error-message">
        {{ error }}
      </div>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      nuevoLibro: {
        titulo: '',
        fechaPublicacion: '',
        autorLibro: ''
      },
      autores: [],
      loading: false,
      loadingAutores: false,
      errorAutores: false,
      successMessage: '',
      error: ''
    };
  },
  mounted() {
    this.fetchAutores();
  },
  methods: {
    async fetchAutores() {
      this.loadingAutores = true;
      try {
        const response = await axios.get('http://autoreslibros.somee.com/api/AutorLibro');
        this.autores = response.data;
      } catch (error) {
        console.error('Error fetching autores:', error);
        this.errorAutores = true;
      } finally {
        this.loadingAutores = false;
      }
    },
    async submitForm() {
      this.loading = true;
      this.error = '';
      this.successMessage = '';

      try {
        const payload = {
          ...this.nuevoLibro,
          fechaPublicacion: new Date(this.nuevoLibro.fechaPublicacion).toISOString()
        };

        await axios.post(
          'http://libreriamaterial.somee.com/api/LibroMaterial',
          payload,
          {
            headers: {
              'Content-Type': 'application/json'
            }
          }
        );

        this.successMessage = '¡Libro agregado exitosamente!';
        this.nuevoLibro = {
          titulo: '',
          fechaPublicacion: '',
          autorLibro: ''
        };

        setTimeout(() => {
          this.$router.push('/libros');
        }, 1000);

      } catch (err) {
        this.error = err.response?.data || 'Error al agregar el libro';
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.form-container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 2rem;
  background: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

h2 {
  color: #2c3e50;
  text-align: center;
  margin-bottom: 1.5rem;
}

.form-group {
  margin-bottom: 1.5rem;
  position: relative;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  color: #4a5568;
  font-weight: 500;
}

input, .custom-select {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #e2e8f0;
  border-radius: 4px;
  font-size: 1rem;
  background-color: white;
  transition: border-color 0.3s ease;
  box-sizing: border-box;
}

.custom-select {
  appearance: none;
  padding-right: 2.5rem;
  background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='currentColor' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
  background-repeat: no-repeat;
  background-position: right 0.75rem center;
  cursor: pointer;
}

.custom-select:focus {
  outline: none;
  border-color: #4299e1;
  box-shadow: 0 0 0 1px #4299e1;
}

.custom-select:disabled {
  background-color: #f8f9fa;
  cursor: not-allowed;
  opacity: 0.7;
}

.custom-select option {
  padding: 0.5rem;
  line-height: 1.5;
}

.submit-btn {
  width: 100%;
  padding: 1rem;
  background-color: #48bb78;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.3s;
}

.submit-btn:disabled {
  background-color: #cbd5e0;
  cursor: not-allowed;
}

.submit-btn:hover:not(:disabled) {
  background-color: #38a169;
}

.success-message {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #c6f6d5;
  color: #22543d;
  border-radius: 4px;
}

.error-message {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #fed7d7;
  color: #742a2a;
  border-radius: 4px;
}
</style>