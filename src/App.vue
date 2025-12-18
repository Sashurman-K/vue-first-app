<template>
  <div id="app">
    <nav>
      <router-link to="/">🎨 Генератор</router-link>
      <router-link to="/library">📚 Библиотека</router-link>
    </nav>
    <router-view />
  </div>
</template>

<style>
#app {
  font-family: 'Segoe UI', system-ui, sans-serif;
  min-height: 100vh;
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
}

nav {
  padding: 1.5rem;
  background: white;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

nav a {
  margin: 0 1.5rem;
  padding: 0.75rem 1.5rem;
  text-decoration: none;
  font-weight: 600;
  color: #64748b;
  border-radius: 8px;
  transition: all 0.3s;
}

nav a:hover {
  background: #f1f5f9;
  color: #475569;
}

nav a.router-link-exact-active {
  background: #3b82f6;
  color: white;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.4);
}
</style>