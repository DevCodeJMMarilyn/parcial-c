<script setup>
import { ref, computed } from 'vue';
import axios from 'axios'

const idPokemon = ref('');
const pokemon = ref(null);
const pokemonMoves = ref([]);

const error = ref('');

const tipos = computed(() => {
    return pokemon.value?.types?.map(t => t.type.name) || []
})

const buscarPokemon = async () => {
    error.value = ""
    pokemon.value = null


    //validaciones del ejercicio 1
    if (!idPokemon.value) {
        error.value = "Por favor escribe el numero de un pokemon"
        return
    }

    if (idPokemon.value > 151) {
        error.value = "El numero de un pokemon no puede ser mayor a 151"
        return
    }

    try {
        const res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${idPokemon.value}`)
        pokemon.value = res.data
        pokemonMoves.value = res.data.moves.slice(0, 4).map(move => move.move.name)

    } catch (e) {
        console.log(e)
        error.value = e.response?.status === 404
            ? "Pokemon no encontrado"
            : "Error al buscar pokemon"
    }
}

</script>

<template>
    <div class="p-4 flex-1">
        <h1 class="text-2xl font-bold mb-4">Buscar Pokemon por ID</h1>
        <input class="border p-2 rounded w-full" v-model="idPokemon" type="number" value=""
            placeholder="Ingresa el numero del pokemon">
        <button @click="buscarPokemon"
            class="mt-2 bg-red-500 text-amber-50 px-4 py-2 rounded hover:bg-orange-600">Buscar</button>

        <div v-if="pokemon" class="mt-6 bg-white shadow-md rounded p-4 text-center">
            <div class="flex justify-center gap-4 mb-4">
                <img :src="pokemon.sprites.front_default" alt="imagen normal" class="w-48 h-48">
                <img :src="pokemon.sprites.front_shiny" alt="imagen shiny" class="w-48 h-48">
            </div>
            <h2 class="text-xl font-bold capitalize">{{ pokemon.name }}</h2>
            <p>Altura: {{ pokemon.height }}</p>
            <p>Peso: {{ pokemon.weight }}</p>
            <p>
                Tipo:
                <span v-for="tipo in tipos" :key="tipo"
                    class="inline-block bg-red-600 rounded text-amber-50 px-2 py-1 mx-1">
                    {{ tipo }}
                </span>
            </p>

            <h2 class="mb-2 text-lg font-semibold text-gray-900 dark:text-white">Movimientos:</h2>
            <ul class="max-w-md space-y-1 text-gray-500 list-disc list-inside dark:text-gray-400"
                v-for="pokemonMove in pokemonMoves">
                <li>
                    Movimiento: {{ pokemonMove }}
                </li>
            </ul>
        </div>
        <p v-if="error" class="text-red">
            {{ error }}
        </p>
    </div>



</template>