<template>
  <router-view :key="$route.fullPath" />
</template>

<script setup></script>

<style>
@import url("./assets/font/kanit-900.css");
@import url("./assets/font/long-cang.css");

::-webkit-scrollbar {
  height: 14px;
  width: 4px;
}

::-webkit-scrollbar-thumb {
  background-color: rgba(153, 153, 153, 0.8);
  border-radius: 10px;
}

::-webkit-scrollbar-track {
  border-radius: 10px;
}

body {
  overflow: overlay;
  background-color: #f3f6f7;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen,
    Ubuntu, Cantarell, "Open Sans", "Helvetica Neue", sans-serif;
}

:root {
  --theme-color: #19b1f5;
  --text-color: #4c4948;
  --card-box-shadow: 0 3px 8px 6px rgba(7, 17, 27, 0.05);
}

::selection {
  background: var(--theme-color);
  color: #fff;
}
</style>
