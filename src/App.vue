<template>
  <div
    id="nav"
    class="bg-red-200 p-8 flex flex-col w-full items-center justify-center"
  >
    <!-- <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link> -->
    <router-view />
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0;
  padding: 0;
}

#nav {
  /* padding: 30px; */
  min-height: 100vh;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
