<template>
  <div id="poke-form">
    <h1>Aquí puedes buscar tus Pokemons Preferidos</h1>
    <button v-on:click="showPokemon">Muestra los Pokémon</button>
    <div id="wait">
      <hr />
      <h2>
        Por favor, espere... tratar con una API, le rogamos paciencia, a veces puede demorar un poco.
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
      pokemonsInfo: []
    };
  },
  methods: {
    sendGetRequest: async function () {
      try {
        const resp = await axios.get(apiURL);
        this.info = resp;
      } catch (err) {
        // If there's 404 error, It means there's no more pokemon to show, so I stop the loop.
        if (err.response && err.response.status === 404) {
          console.error(err);
          this.totalPokemons = 1;
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
        // We get a list of all pokemons, and filter by who have only one encounter
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
        // Here It's collected the Name & Type from each pokemon to an array of objects 
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
      } catch (err) {
        console.error(err);
      }
    },
    getPokemonGeneration: async function () {
      try {
        // Here It's collected the info of from which generation is each pokemon who have only one encounter
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
              this.pokemonsInfo[x].generation = i;
              break;
            }
          }
        }
      } catch (err) {
        console.error(err);
      }
    },
    checkNameTypos: async function () {
      // Here It's a cleaning of data, because API had some typos
      let tempSplittedName;
      for (var i = 0; i < this.pokemonsInfo.length; i++) {
        if (this.pokemonsInfo[i].name.includes("-normal")) {
          tempSplittedName = this.pokemonsInfo[i].name.split("-");
          this.pokemonsInfo[i].name = tempSplittedName[0];
        }
      }
    },
    splitTypes: async function () {
      // Here I split the types between 2, to a correct sort & show later 
      for (var i = 0; i < this.pokemonsInfo.length; i++) {
        this.pokemonsInfo[i].type1 = this.pokemonsInfo[i].types[0];
        if (this.pokemonsInfo[i].types.length > 1) {
          this.pokemonsInfo[i].type2 = this.pokemonsInfo[i].types[1];
        }
      }
    },
    filterAndSortPokemons: async function () {
      let sortedGenerations1 = [];
      let sortedGenerations2 = [];
      let sortedGenerations3 = [];

      // Sort by Type
      this.pokemonsInfo.sort((a, b) => a.type1.localeCompare(b.type1));

      // Cloning generations sorted
      for (var i = 0; i < this.pokemonsInfo.length; i++) {
        switch (this.pokemonsInfo[i].generation) {
          case 1:
            sortedGenerations1.push(this.pokemonsInfo[i]);
            break;

          case 2:
            sortedGenerations2.push(this.pokemonsInfo[i]);
            break;

          case 3:
            sortedGenerations3.push(this.pokemonsInfo[i]);
            break;
        }
      }
      this.pokemonsInfo = [];

      // Copy to the final array sorted
      for (var a = 0; a < sortedGenerations1.length; a++) {
        this.pokemonsInfo.push(sortedGenerations1[a]);
      }
      for (var e = 0; e < sortedGenerations2.length; e++) {
        this.pokemonsInfo.push(sortedGenerations2[e]);
      }
      for (var o = 0; o < sortedGenerations3.length; o++) {
        this.pokemonsInfo.push(sortedGenerations3[o]);
      }
      document.getElementById("wait").style.display = "none";
      document.getElementById("pokemonList").style.display = "inline";
    },
    showPokemon: async function () {
      // Here It's the order of priority for each method to execute
      await this.getPokemonCounter();
      await this.getPokemonList();
      await this.getPokemonNameAndType();
      await this.checkNameTypos();
      await this.getPokemonGeneration();
      await this.splitTypes();
      await this.filterAndSortPokemons();
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