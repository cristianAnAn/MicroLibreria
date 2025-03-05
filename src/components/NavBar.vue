<template>
  <nav class="navbar">
    <div class="nav-container">
      <a href="/" class="logo">🏛 Librería Material</a>
      <div class="nav-links">
        <a href="/libros/" class="nav-link">Catálogo</a>
        <a href="/autores/" class="nav-link">Autores</a>
        <a href="/agregar-libro/" class="nav-link">Nuevo Libro</a>
        <a href="/agregar-autor/" class="nav-link">Nuevo Autor</a>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  mounted() {
    this.updateActiveLink();
    window.addEventListener('popstate', this.updateActiveLink);
  },
  methods: {
    updateActiveLink() {
      const currentPath = window.location.pathname;
      this.$el.querySelectorAll('.nav-link').forEach(link => {
        link.classList.toggle('active', link.getAttribute('href') === currentPath);
      });
    }
  }
}
</script>

<style scoped>
.navbar {
  background-color: #2c3e50;
  padding: 1rem 0;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.nav-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  color: white;
  font-size: 1.5rem;
  text-decoration: none;
  font-weight: 600;
}

.nav-links {
  display: flex;
  gap: 2rem;
}

.nav-link {
  color: #ecf0f1;
  text-decoration: none;
  font-weight: 500;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  transition: all 0.3s;
}

.nav-link:hover {
  background-color: #34495e;
}

.nav-link.active {
  background-color: #3498db;
}
</style>