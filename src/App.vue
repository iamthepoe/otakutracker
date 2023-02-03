<script setup>
  import axios from 'axios';
  import { ref, onMounted, computed } from 'vue'
  const query = ref('')
  const my_list = ref([])
  const search_results = ref([])

  const my_list_asc = computed(() => {
    return my_list.value.sort((a, b) => {
      return a.title.localeCompare(b.title)
    });
  });

  const searchAnime = async () => {
    const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
    let res = await axios.get(url);
    search_results.value = res.data.data;
}

  const emptyInput = (e) =>{
    if(!e.target.value)
      search_results.value = [];
  }

  const addAnime = (anime) => {
    search_results.value = [];
    query.value = '';

    my_list.value.push({
      id: anime.mal_id,
      title: anime.title,
      image: anime.images.jpg.image_url,
      total_episodes: anime.episodes,
      watched_episodes: 0
    });

    localStorage.setItem('my_list', JSON.stringify(my_list.value));
  }

  const increaseWatch = (anime) =>{
    anime.watched_episodes++;
    localStorage.setItem('my_list', JSON.stringify(my_list.value));
  }

  const decreaseWatch = (anime) =>{
    anime.watched_episodes--;
    localStorage.setItem('my_list', JSON.stringify(my_list.value));
  }

  onMounted(()=>{
    my_list.value = JSON.parse(localStorage.my_list) || [];
  });
  document.body.style.backgroundColor = "#131318";

</script>

<template>
	<main>
		<h1>OTAKU<span style="color:#8424d3;">TRACKER</span></h1>

		<form @submit.prevent="searchAnime">
			<input type="text" placeholder="Procure por um anime..." v-model="query" @input="emptyInput" />
			<button type="submit" class="button">Buscar</button>
		</form>

		<div class="results" v-if="search_results.length > 0">
			<div v-for="anime in search_results" class="result">
				<img :src="anime.images.jpg.image_url" />
				<div class="details">
					<h3>{{ anime.title }}</h3>
					<p :title="anime.synopsis" v-if="anime.synopsis">{{ anime.synopsis.slice(0, 120) }}...</p>
					<span class="flex-1"></span>
					<button @click="addAnime(anime)" class="button">Add to My Anime</button>
				</div>
			</div>
		</div>

		<div class="myanime" v-if="my_list.length > 0">
			<div v-for="anime in my_list_asc" class="anime">
				<img :src="anime.image" />
				<h3>{{ anime.title }}</h3>
				<div class="flex-1"></div>
				<span class="episodes">{{ anime.watched_episodes }} / {{ anime.total_episodes }}</span>
				<button 
					v-if="anime.total_episodes !== anime.watched_episodes" 
					@click="increaseWatch(anime)" class="button">+</button>
				<button 
					v-if="anime.watched_episodes > 0"
					@click="decreaseWatch(anime)" class="button">-</button>
			</div>
		</div>
	</main>
</template>

<style scoped>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
	font-family: 'Fira Sans', sans-serif;
}

::-webkit-scrollbar{
    width: 12px;
}

::-webkit-scrollbar-thumb{
    background: linear-gradient(transparent, purple);
    border-radius: 6px;
}

main {
	margin: 0 auto;
	max-width: 768px;
	padding: 1.5rem;
}
h1 {
	text-align: center;
	margin-bottom: 1.5rem;
	color: #FFF;
}
form {
	display: flex;
	max-width: 480px;
	margin: 0 auto 1.5rem;
}
form input {
	appearance: none;
	outline: none;
	border: none;
	background: #3E3E3E;
	display: block;
	color: #FFF;
	font-size: 1.125rem;
	padding: 0.5rem 1rem;
	width: 100%;
}
.button {
	appearance: none;
	outline: none;
	border: none;
	background: none;
	cursor: pointer;
	display: block;
	padding: 0.5rem 1rem;
	background-image: linear-gradient(to right, #8A2BE2 50%, #9B30FF 50%);
	background-size: 200%;
	color: white;
	font-size: 1.125rem;
	font-weight: bold;
	text-transform: uppercase;
	transition: 0.4s;
}
.button:hover {
	background-position: right;
}
.results {
	background-color: #131318;
	border-radius: 0.5rem;
	box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	max-height: 480px;
	overflow-y: scroll;
	margin-bottom: 1.5rem;
}
.result {
	display: flex;
	margin: 1rem;
	padding: 1rem;
	border: 1px solid rgb(215, 1, 198);
	border-radius: 5px;
	transition: 0.4s;
}
.result img {
	width: 100px;
	border-radius: 1rem;
	margin-right: 1rem;
}
.details {
	flex: 1 1 0%;
	display: flex;
	align-items: flex-start;
	flex-direction: column;
}
.details h3 {
	font-size: 1.25rem;
	margin-bottom: 0.5rem;
	color: #FFF;
}
.details p {
	font-size: 0.875rem;
	margin-bottom: 1rem;
	color: #FFF;
}
.details .button {
	margin-left: auto;
}
.flex-1 {
	display: block;
	flex: 1 1 0%;
}
.myanime h2 {
	color: #FFF;
	font-weight: 400;
	margin-bottom: 1.5rem;
}
.myanime .anime {
display: flex;
align-items: center;
margin-bottom: 1.5rem;
background-color: #1d1d24;
padding: 1rem;
border-radius: 0.5rem;
box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.5);
}

.anime img {
width: 72px;
height: 72px;
object-fit: cover;
border-radius: 1rem;
margin-right: 1rem;
}

.anime h3 {
color: #fff;
font-size: 1.125rem;
}

.anime .episodes {
margin-right: 1rem;
color: #fff;
}

.anime .button:first-of-type {
margin-right: 0.5rem;
background-color: #9a00e0;
color: #fff;
}

.anime .button:last-of-type {
margin-right: 0;
background-color: #9a00e0;
color: #fff;
}
</style>
