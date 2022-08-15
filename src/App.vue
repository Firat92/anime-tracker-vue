<script setup>
import { ref, computed, onMounted, watch } from 'vue';

const query = ref('');
const my_anime = ref([]);
const search_results = ref([]);

const my_anime_asc = computed(() => {
	return my_anime.value.sort((a, b) => {
		return a.title.localeCompare(b.title);
	});
});

const searchAnime = async () => {
	const res = await fetch(`https://api.jikan.moe/v4/anime?q=${query.value}`);
	const data = await res.json();
	search_results.value = data.data;
	console.log(data);
};

const handleInput = (e) => {
	if (!e.target.value) {
		search_results.value = [];
	}
};

watch(query, (animeVal) => {
	localStorage.setItem('query', animeVal);
});

watch(
	my_anime,
	(animeVal) => {
		localStorage.setItem('my_anime', JSON.stringify(animeVal));
	},
	{
		deep: true,
	}
);

const addAnime = (anime) => {
	search_results.value = [];
	query.value = '';

	my_anime.value.push({
		id: anime.mal_id,
		title: anime.title,
		image: anime.images.jpg.image_url,
		total_episodes: anime.episodes,
		watched_episodes: 0,
	});

	localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
};

const increaseWatch = (anime) => {
	anime.watched_episodes++;
	localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
};

const decreaseWatch = (anime) => {
	anime.watched_episodes--;
	localStorage.setItem('my_anime', JSON.stringify(my_anime.value));
};

onMounted(() => {
	my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || [];
});

const RemoveAnime = (anime) => {
	my_anime.value = my_anime.value.filter((a) => a !== anime);
	localStorage.removeItem('my_anime', my_anime.value[0]);
};

const ResetAll = () => {
	my_anime.value = [];
	localStorage.removeItem('my_anime', JSON.stringify(my_anime.value));
};
</script>

<template>
	<body class="bg-white m-0 p-0 box-border font-sans">
		<main class="my-0 mx-auto max-w-3xl p-6">
			<h1 class="text-center mb-6">Anime Tracker</h1>

			<form @submit.prevent="searchAnime" class="flex max-w-lg my-6 mx-auto">
				<input
					class="appearance-none outline-none border-none bg-blue-300 rounded-md block text-lg py-2 px-4 w-full mx-2"
					type="text"
					placeholder="Search for an anime..."
					v-model="query"
					@input="handleInput"
				/>
				<button type="submit" class="button">Search</button>
			</form>

			<div
				class="results bg-white rounded-lg shadow-md mb-6 overflow-y-scroll max-h-[30rem]"
				v-if="search_results.length > 0"
			>
				<div
					class="result flex m-4 p-4 rounded-md border border-solid border-white duration-500"
					v-for="anime in search_results"
				>
					<img
						:src="anime.images.jpg.image_url"
						class="w-24 rounded-2xl mr-4"
					/>
					<div class="details flex flex-1 flex-col items-start">
						<h3 class="text-xl mb-2 text-red-300">{{ anime.title }}</h3>
						<p
							class="text-sm mb-4"
							:title="anime.synopsis"
							v-if="anime.synopsis"
						>
							{{ anime.synopsis.slice(0, 120) }}...
						</p>
						<span class="asd block flex-1"></span>
						<button @click="addAnime(anime)" class="ml-auto button"
							>Add to My Anime</button
						>
					</div>
				</div>
			</div>

			<div class="myanime" v-if="my_anime.length > 0">
				<h2
					class="bg-slate-50 font-normal mb-6 flex items-center justify-center"
					>My anime</h2
				>

				<div
					class="anime flex items-center mb-6 bg-white p-4 rounded-lg shadow-md"
					v-for="anime in my_anime_asc"
				>
					<img
						class="w-20 h-20 object-cover rounded-2xl mr-4"
						:src="anime.image"
					/>
					<h3 class="bg-slate-50 text-sm">{{ anime.title }}</h3>
					<div class="flex-1"></div>
					<span class="episodes mr-4 bg-slate-50">
						{{ anime.watched_episodes }} / {{ anime.total_episodes }}
					</span>
					<button
						class="button type"
						v-if="anime.total_episodes !== anime.watched_episodes"
						@click="increaseWatch(anime)"
					>
						+
					</button>
					<button
						class="button type"
						v-if="anime.watched_episodes > 0"
						@click="decreaseWatch(anime)"
					>
						-
					</button>
					<button @click="RemoveAnime(anime)" class="button ml-2"
						>Remove Anime</button
					>
				</div>

				<button @click="ResetAll" class="button mx-auto"
					>Reset All Anime</button
				>
			</div>
		</main>
	</body>
</template>
