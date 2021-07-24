<template>
  <div id="poke-form">
    <h1>Aquí puedes buscar tus Pokemons Preferidos</h1>
    <button v-on:click="showPokemon">Muestra los Pokémon</button>
    <div id="wait">
      <hr />
      <h2>
        Por favor, espere... tratar con una API, a veces puede demorar un poco.
      </h2>
      <img src="../assets/images/working.gif" />
    </div>
    <div id="pokemonList">
      <hr />
      <h2>Lista de Pokemons con un único encuentro:</h2>
      <br />
      <table style="width: 100%" id="pokemonTable">
        <tr>
          <th>Nombre</th>
          <th>Generación</th>
          <th>Tipo 1</th>
          <th>Tipo 2</th>
        </tr>
        <tr v-for="pokemon in pokemonsInfo" :key="pokemon.name">
          <td>{{ pokemon.name }}</td>
          <td>{{ pokemon.generation }}</td>
          <td>{{ pokemon.type1 }}</td>
          <td>{{ pokemon.type2 }}</td>
        </tr>
      </table>
    </div>
  </div>
</template>

<script>
import axios from "axios";
var apiURL;
export default {
  name: "PokeForm",
  data() {
    return {
      totalPokemons: null,
      info: null,
      uniqueEncounter: [],
      pokemonsInfo: [],
    };
  },
  methods: {
    sendGetRequest: async function () {
      try {
        const resp = await axios.get(apiURL);
        this.info = resp;
      } catch (err) {
        if (err.response && err.response.status === 404) {
          console.error(err);
          this.totalPokemons = 1;
          // If there's an 404 error, It means there's no more pokemon to show, so I stop the loop.
        }
      }
    },
    getPokemonCounter: async function () {
      try {
        apiURL = "https://pokeapi.co/api/v2/pokemon/";
        await this.sendGetRequest();
        this.totalPokemons = this.info.data.count;
      } catch (err) {
        console.error(err);
      }
    },
    getPokemonList: async function () {
      try {
        document.getElementById("wait").style.display = "inline";
        for (var i = 1; i < this.totalPokemons; i++) {
          apiURL = "https://pokeapi.co/api/v2/pokemon/" + i + "/encounters";
          await this.sendGetRequest();
          if (this.info.data === undefined || this.info.data.length === 0) {
            continue;
          }
          if (
            this.info.data[0].version_details[0].encounter_details[0].method
              .name == "only-one"
          ) {
            this.uniqueEncounter.push(i);
          }
        }
      } catch (err) {
        console.error(err);
      }
    },
    getPokemonNameAndType: async function () {
      try {
        let pokemon = {};
        for (var i = 0; i < this.uniqueEncounter.length; i++) {
          apiURL =
            "https://pokeapi.co/api/v2/pokemon/" + this.uniqueEncounter[i];
          await this.sendGetRequest();
          pokemon.name = this.info.data.name;
          pokemon.number = this.uniqueEncounter[i];
          pokemon.types = [];

          for (var x = 0; x < this.info.data.types.length; x++) {
            pokemon.types.push(this.info.data.types[x].type.name);
          }
          this.pokemonsInfo.push(pokemon);
          pokemon = {};
        }
        document.getElementById("wait").style.display = "none";
        document.getElementById("pokemonList").style.display = "inline";
      } catch (err) {
        console.error(err);
      }
    },
    getPokemonGeneration: async function () {
      try {
        for (var x = 0; x < this.pokemonsInfo.length; x++) {
          for (var i = 1; i < 9; i++) {
            apiURL = "https://pokeapi.co/api/v2/generation/" + i;
            await this.sendGetRequest();
            let tempVar;
            tempVar = this.info.data.pokemon_species.find(
              (tempPoke) => tempPoke.name === this.pokemonsInfo[x].name
            );
            if (tempVar === undefined || tempVar.length === 0) {
              continue;
            } else {
              this.pokemonsInfo[x].generation = i.toString();
              break;
            }
          }
        }
        console.log(this.pokemonsInfo);
      } catch (err) {
        console.error(err);
      }
    },
    checkNameTypos: async function () {
      let tempSplittedName;
      for (var i = 0; i < this.pokemonsInfo.length; i++) {
        if (this.pokemonsInfo[i].name.includes("-normal")) {
          tempSplittedName = this.pokemonsInfo[i].name.split("-");
          this.pokemonsInfo[i].name = tempSplittedName[0];
        }
      }
    },
    splitTypes: async function () {
      for (var i = 0; i < this.pokemonsInfo.length; i++) {
          this.pokemonsInfo[i].type1 = this.pokemonsInfo[i].types[0];
        if (this.pokemonsInfo[i].types.length > 1) {
          this.pokemonsInfo[i].type2 = this.pokemonsInfo[i].types[1];
        }
      }
    },
    showTable: async function () {

    },
    showPokemon: async function () {
      await this.getPokemonCounter();
      await this.getPokemonList();
      await this.getPokemonNameAndType();
      await this.checkNameTypos();
      await this.getPokemonGeneration();
      await this.splitTypes();
    },
  },
};
</script>

<style scoped>
#wait {
  display: none;
}

#pokemonList {
  display: none;
}

table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

td,
th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}
</style>