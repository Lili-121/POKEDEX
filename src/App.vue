<template>
<div class="contenedor">
  <div class="search">
    <div id="logo">
      <img src="https://imgs.search.brave.com/gFZo0uYWvt3LnlIgniFe2oGIFvsBc6DHEgMkb1ssGaQ/rs:fit:860:0:0/g:ce/aHR0cHM6Ly93d3cu/cG5nYWxsLmNvbS93/cC1jb250ZW50L3Vw/bG9hZHMvNC9Qb2tl/YmFsbC1QTkctSW1h/Z2VzLnBuZw">
    </div>
    <div class="search-container">
      <q-input
        v-model="pokeSearch"
        label="ID o nombre de Pokemon"
        :error="!!errorMsg"
        :error-message="errorMsg"
        color="white"
        label-color="white"
        text-color="white"
        input-class="white-text"
      />
      <div class="btn-search">
        <q-btn color="blue-4" text-color="black" label="Buscar" @click="traer" />
      </div>
    </div>
  </div>
  <div :style="{ backgroundColor: backgroundColor }" class="container">
    <div v-if="loading" class="loading">Cargando...</div>
    <div class="poke-container" v-if="pokeLoaded">
      <div class="pokeImg">
        <div class="imgPoke">
          <img :src="imgPokemon" alt="" />
        </div>
        <div class="pokeId">
          <p>#{{ pokeID }}</p>
        </div>
      </div>
      <p id="pokename1">{{ pokeName }}</p>
      <div class="pokeInfo">
        <h5>ID</h5>
        <h5>Peso</h5>
        <h5>Altura</h5>
        <h6>#{{ pokeID }}</h6>
        <h6>{{ pokeHeight }}</h6>
        <h6>{{ pokeWeight }}</h6>
      </div>
      <div class="tipos">
        <div class="tipoPokemon">
          <div
            class="tipoPokemonDiv"
            v-for="(typeObj, index) in pokeType"
            :key="index"
            :style="{ backgroundColor: getColor(typeObj.type) }"
          >
            <h6 class="tipoPokemonText">{{ typeObj.type }}</h6>
          </div>
        </div>
      </div>
    </div>
    <div class="pokeStats" v-if="pokeLoaded">
      <div class="pokeStat" v-for="stat in pokeStatsArray" :key="stat.name">
        <div class="statinfo">
        <p id="numStat">{{ stat.label }}</p>
        <h2 id="nomStat">{{ stat.value }}</h2>
      </div>
        <q-linear-progress :value="stat.value / 100" :color="stat.color" />
      </div>
    </div>
  </div>
</div>
</template>

<script setup>
import axios from "axios";
import { ref, watch } from "vue";

let pokeLoaded = ref(false);
let loading = ref(false);
let imgPokemon = ref("");
let pokeSearch = ref("");
let pokeName = ref("");
let pokeID = ref("");
let pokeHeight = ref("");
let pokeWeight = ref("");
let pokeType = ref([]);
let pokeStatsArray = ref([]);
let errorMsg = ref("");
let backgroundColor = ref("#E83030");

const coloresTipos = {
  normal: "#A8A878",
  fire: "#F08030",
  water: "#6890F0",
  grass: "#78C850",
  electric: "#F8D030",
  ice: "#98D8D8",
  fighting: "#C03028",
  poison: "#A040A0",
  ground: "#E0C068",
  flying: "#A890F0",
  psychic: "#F85888",
  bug: "#A8B820",
  rock: "#B8A038",
  ghost: "#705898",
  dragon: "#7038F8",
  dark: "#705848",
  steel: "#B8B8D0",
};

function getColor(type) {
  return coloresTipos[type] || "white";
}

function updateBackgroundColor() {
  if (pokeType.value.length > 0) {
    const type = pokeType.value[0].type.toLowerCase();
    const color = coloresTipos[type] || "#E83030";
    backgroundColor.value = color + "cc";
  } else {
    backgroundColor.value = "#E83030";
  }
}

watch(pokeType, updateBackgroundColor);

async function traer() {
  if (!pokeSearch.value) {
    errorMsg.value = "Ingrese un ID o nombre de Pokemon";
    return;
  }

  loading.value = true;
  try {
    let res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${pokeSearch.value}`);
    errorMsg.value = "";
    pokeLoaded.value = true;
    imgPokemon.value = res.data.sprites.other["official-artwork"].front_default;
    pokeName.value = res.data.name;
    pokeID.value = res.data.id;
    pokeHeight.value = res.data.height;
    pokeWeight.value = res.data.weight;
    pokeType.value = res.data.types.map((typeName) => ({
      type: typeName.type.name,
    }));
    pokeStatsArray.value = [
      { label: "Hp", value: res.data.stats[0].base_stat, color: "red" },
      { label: "Ataque", value: res.data.stats[1].base_stat, color: "warning" },
      { label: "Defensa", value: res.data.stats[2].base_stat, color: "secondary" },
      { label: "Ataque Especial", value: res.data.stats[3].base_stat, color: "blue" },
      { label: "Defensa Especial", value: res.data.stats[4].base_stat, color: "purple" },
      { label: "Velocidad", value: res.data.stats[5].base_stat, color: "pink" },
    ];
  } catch (error) {
    errorMsg.value = "No se encontró el Pokémon. Intente con otro nombre o ID.";
    console.error(error);
  } finally {
    loading.value = false;
  }
}

</script>

<style src="./style.css"></style>
