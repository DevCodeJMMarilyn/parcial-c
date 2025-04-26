<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios'

const buscarPorTipo = async () => {
    if (!selectedType.value) return;

    loading.value = true;
    typePokemons.value = [];
    seleccionarDetallesPokemon.value = null;

    try {
        const res = await axios.get(`https://pokeapi.co/api/v2/type/${selectedType.value}`)
        
        const pokemonPromises = res.data.pokemon.slice(0, 5).map(p =>
            axios.get(p.pokemon.url).then(res => res.data)
        );

        typePokemons.value = await Promise.all(pokemonPromises);
    } catch (e) {
        console.error(e);
        error.value = "Error al buscar pokemón por tipo";
    } finally {
        loading.value = false;
    }
}

// detalles pokemon
const mostrarDetalles = (pokemon) => {
    seleccionarDetallesPokemon.value = pokemon;
}

const selectedType = ref('');
const typePokemons = ref([]);
const seleccionarDetallesPokemon = ref(null);
const loading = ref(false);

const tipos = computed(() => {
    return pokemon.value?.types?.map(t => t.type.name) || []
})

const sortedPokemons = computed(() => {
    return [...typePokemons.value].sort((a, b) => a.name.localeCompare(b.name))
})

</script>

<template>
    <div class="p-4 bg-white rounded-lg flex-1">
        <h1 class="text-2xl font-bold mb-4">Buscar Pokemon por tipo</h1>
        <div class="flex gap-2 mb-4">
            <select v-model="selectedType" class="border p-2 rounded w-full" @change="buscarPorTipo">
                <option value="">Selecciona un tipo</option>
                <option value="ghost">Ghost</option>
                <option value="fighting">Fighting</option>
                <option value="flying">Flying</option>
            </select>
        </div>

        <div v-if="loading" class="text-center py-4">
            Cargando...
        </div>

        <div v-if="sortedPokemons.length > 0" class="mt-4">
            <h3 class="text-lg font-semibold mb-2">Pokemón ordenados :</h3>
            <div class="grid grid-cols-1 gap-4">
                <div v-for="pokemon in sortedPokemons" :key="pokemon.id" @click="mostrarDetalles(pokemon)"
                    class="bg-white p-3 rounded shadow cursor-pointer hover:bg-gray-50">
                    <div class="flex items-center gap-3">
                        <img :src="pokemon.sprites.front_default" alt="" class="w-16 h-16">
                        <span class="capitalize font-medium">{{ pokemon.name }}</span>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="seleccionarDetallesPokemon" class="mt-6 bg-white shadow-md rounded p-4">
            <h3 class="text-xl font-bold capitalize mb-2">{{ seleccionarDetallesPokemon.name }}</h3>
            <p>Altura: {{ seleccionarDetallesPokemon.height / 10 }} m</p>
            <p>Peso: {{ seleccionarDetallesPokemon.weight / 10 }} kg</p>
            <p>
                Tipos:
                <span v-for="type in seleccionarDetallesPokemon.types" :key="type.slot"
                    class="inline-block bg-orange-500 rounded text-white px-2 py-1 mx-1 capitalize">
                    {{ type.type.name }}
                </span>
            </p>
        </div>
    </div>
</template>