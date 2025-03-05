<template>
  <div class="form-container">
    <div class="form-header">
      <p class="form-subtitle">Complete los campos para agregar un nuevo autor y sus grados académicos.</p>
    </div>

    <form @submit.prevent="submitForm" class="author-form">
      <!-- Sección Información del Autor -->
      <fieldset class="form-section">
        <legend><i class="fas fa-id-card"></i> Información del Autor</legend>
        
        <div class="form-grid">
          <div class="form-group">
            <label>Nombre <span class="required">*</span></label>
            <input 
              type="text" 
              v-model="autor.nombre"
              required
              placeholder="Ingrese el nombre"
            >
          </div>

          <div class="form-group">
            <label>Apellido <span class="required">*</span></label>
            <input 
              type="text" 
              v-model="autor.apellido"
              required
              placeholder="Ingrese el apellido"
            >
          </div>

          <div class="form-group">
            <label>Fecha de Nacimiento <span class="required">*</span></label>
            <input 
              type="date" 
              v-model="autor.fechaNacimiento"
              required
            >
          </div>
        </div>
      </fieldset>

      <!-- Sección Grados Académicos -->
      <fieldset class="form-section">
        <legend><i class="fas fa-graduation-cap"></i> Grados Académicos</legend>
        
        <div 
          v-for="(grado, index) in autor.gradosAcademicos" 
          :key="index" 
          class="degree-card"
        >
          <div class="degree-header">
            <h3>Grado Académico {{ autor.gradosAcademicos.length > 1 ? index + 1 : '' }}</h3>
            <button 
              type="button" 
              @click="removeGrado(index)" 
              class="btn-remove"
              v-if="autor.gradosAcademicos.length > 1"
            >
              <i class="fas fa-trash"></i> Eliminar
            </button>
          </div>

          <div class="form-grid">
            <div class="form-group">
              <label>Título <span class="required">*</span></label>
              <input 
                type="text" 
                v-model="grado.nombre"
                required
                placeholder="Ej: Licenciatura en Ciencias"
              >
            </div>

            <div class="form-group">
              <label>Institución <span class="required">*</span></label>
              <input 
                type="text" 
                v-model="grado.centroAcademico"
                required
                placeholder="Ej: Universidad Nacional"
              >
            </div>

            <div class="form-group">
              <label>Fecha de Obtención <span class="required">*</span></label>
              <input 
                type="date" 
                v-model="grado.fechaGrado"
                required
              >
            </div>
          </div>
        </div>

        <button type="button" @click="addGrado" class="btn-add">
          <i class="fas fa-plus-circle"></i> Agregar otro grado académico
        </button>
      </fieldset>

      <!-- Sección de Envío -->
      <div class="form-actions">
        <button type="submit" class="submit-btn">
          <i class="fas fa-save"></i> Registrar Autor
        </button>
      </div>
    </form>

    <!-- Mensaje de éxito -->
    <div v-if="successMessage" class="success-message">
      <i class="fas fa-check-circle"></i> {{ successMessage }}
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      autor: {
        nombre: '',
        apellido: '',
        fechaNacimiento: '',
        gradosAcademicos: [{
          nombre: '',
          centroAcademico: '',
          fechaGrado: ''
        }] // Un grado académico por defecto al inicio
      },
      successMessage: ''
    }
  },
  methods: {
    addGrado() {
      this.autor.gradosAcademicos.push({
        nombre: '',
        centroAcademico: '',
        fechaGrado: ''
      })
    },
    removeGrado(index) {
      if (this.autor.gradosAcademicos.length > 1) {
        this.autor.gradosAcademicos.splice(index, 1)
      }
    },
    async submitForm() {
      try {
        const payload = {
          nombre: this.autor.nombre,
          apellido: this.autor.apellido,
          fechaNacimiento: new Date(this.autor.fechaNacimiento).toISOString(),
          gradosAcademicos: this.autor.gradosAcademicos.map(grado => ({
            nombre: grado.nombre,
            centroAcademico: grado.centroAcademico,
            fechaGrado: new Date(grado.fechaGrado).toISOString()
          }))
        }

        await axios.post('http://autoreslibros.somee.com/api/AutorLibro', payload, {
          headers: {
            'Content-Type': 'application/json'
          }
        })

        this.successMessage = 'Autor registrado exitosamente!'
        this.resetForm()
        
      } catch (err) {
        console.error('Error al registrar el autor:', err)
      }
    },
    resetForm() {
      this.autor = {
        nombre: '',
        apellido: '',
        fechaNacimiento: '',
        gradosAcademicos: [{
          nombre: '',
          centroAcademico: '',
          fechaGrado: ''
        }]
      }
    }
  }
}
</script>

<style scoped>
.form-container {
  max-width: 1000px;
  margin: 2rem auto;
  padding: 2rem;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.form-header {
  text-align: center;
  margin-bottom: 2rem;
}

.form-subtitle {
  color: #718096;
  font-size: 1rem;
}

.form-section {
  margin-bottom: 2rem;
  padding: 1.5rem;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  background: #f8fafc;
}

.form-section legend {
  padding: 0 1rem;
  font-weight: 600;
  color: #4a5568;
  font-size: 1.1rem;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-top: 1rem;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  color: #4a5568;
  font-weight: 500;
}

input {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #e2e8f0;
  border-radius: 6px;
  font-size: 1rem;
  transition: border-color 0.3s ease;
}

input:focus {
  border-color: #4299e1;
  outline: none;
  box-shadow: 0 0 0 3px rgba(66, 153, 225, 0.1);
}

.degree-card {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  margin-bottom: 1.5rem;
  border: 1px solid #e2e8f0;
}

.degree-card + .degree-card {
  margin-top: 2rem;
}

.degree-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #e2e8f0;
}

.degree-header h3 {
  color: #2c3e50;
  font-size: 1.2rem;
}

.btn-remove {
  background: #fed7d7;
  color: #c53030;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.3s ease;
}

.btn-remove:hover {
  background: #feb2b2;
}

.btn-add {
  background: #f7fafc;
  color: #4299e1;
  border: 2px dashed #cbd5e0;
  padding: 1rem;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
  width: 100%;
  margin-top: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.btn-add:hover {
  background: #ebf8ff;
  border-color: #4299e1;
}

.btn-add i {
  font-size: 1.2rem;
}

.submit-btn {
  background: #48bb78;
  color: white;
  border: none;
  padding: 1rem 2rem;
  border-radius: 6px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: opacity 0.3s ease;
  width: 100%;
  margin-top: 1.5rem;
}

.required {
  color: #e53e3e;
}

.success-message {
  background: #c6f6d5;
  color: #22543d;
  padding: 1rem;
  border-radius: 6px;
  margin-top: 1.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  animation: fadeIn 0.5s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>