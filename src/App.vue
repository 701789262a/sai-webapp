<template>
  <div class="container">
    <Sidebar /> <!-- Sidebar component -->
    <Map />     <!-- Map component -->
  </div>
</template>

<script>
import Map from './components/Map.vue';
import Sidebar from './components/Sidebar.vue';

export default {
  name: "App",
  components: {
    Map,
    Sidebar
  }
};
</script>

<style lang="scss">
/* Flex container for sidebar and map */
:root {
	--primary: #4ade80;
	--primary-alt: #22c55e;
	--grey: #64748b;
	--dark: #1e293b;
	--dark-alt: #334155;
	--light: #f1f5f9;
	--sidebar-width: 310px;
}

* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'Fira sans', sans-serif;
}

body {
	background: var(--light);
}

button {
	cursor: pointer;
	appearance: none;
	border: none;
	outline: none;
	background: none;
}

.app {
	height: 100%;
	display: flex;

	main {
		flex: 1 1 0;
		padding: 2rem;

		@media (max-width: 1024px) {
			padding-left: 6rem;
		}
	}
}

.container {
  display: flex;
  height: 100%;
  width: 100%;
  flex-direction: row;

}

html, body {
  height: 100%;
  margin: 0;
  overflow: hidden; /* Prevent unwanted scrolling */
}
</style>
