<template>
  <div id="app">
    <div id="nav">
      <!-- <router-link to="/">Home</router-link> | -->
      <router-link to="/about">About</router-link> |
      <router-link to="/mymap">Map</router-link>
    </div>
    <router-view />
  </div>
</template>

<style>
:root {
  --background: rgb(4, 31, 37);
  --content-background: rgba(21, 21, 21, 0.6);
  --header: rgba(130, 155, 252, 0.185);
  --footer: rgb(20, 38, 54);
  --markedText: rgb(110, 5, 5);

  --title: rgb(121, 252, 210);
  --text: rgb(255, 255, 255);
  --something: rgb(47, 115, 66);

  --markerSize: 30px;
  --markerShadow: rgb(211, 93, 93);
  --shadowWidth: 15px;
  --borderWidth: 3px;
  --border: rgb(58, 58, 58);

  --spotlightColor: rgba(0, 0, 0, 0.95);
  --spotlightSize: 150px;
  --spotlightTransparent: 100px;
}

 @font-face {
  font-family: FilmNoir;
  src: url(font/Film-Noir.otf);
}

#app {
  font-family: FilmNoir, Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: var(--text);
}

#nav {
  padding: 30px;
}

#nav a {
  font-size: 1.5em;
  position: relative;
  z-index: 10;
  /* font-weight: bold; */
  color: #e1bc29;
  transition: 0.2s all;
}

#nav a.router-link-exact-active {
  font-size: 2em;
  color: #e1bc29;
}

.body {
  background-color: black;
}
</style>
