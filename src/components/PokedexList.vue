<script setup>
    // declaration
    import axios from "axios";
    import { onMounted, ref } from "vue";

    // data
    const POKEDEX_LIMIT = 150;
    const PAGE_LIMIT = 30;
    const PAGE_COUNT = Math.ceil(POKEDEX_LIMIT / PAGE_LIMIT);

    const pokemonList = ref([]);
    const viewType = ref("all"); // values: all, page
    const currentPage = ref(1);
    const pageList = ref([]);

    // methods
    async function getPokemonList(iOffset = 0, iLimit = 50) {
        let oReturnPokemonList = { result: null, error: true };
        let oPokedexRequest = await sendApiRequest(`https://pokeapi.co/api/v2/pokemon?offset=${iOffset}&limit=${iLimit}`);
        if (oPokedexRequest.result === null) {
            return oReturnPokemonList;
        }

        let oPromises = oPokedexRequest.result.results.map(oPokemon => sendApiRequest(oPokemon.url));
        await Promise.all(oPromises).then(oResponse => {
            if (oResponse.filter(oResult => oResult.error !== null).length < 1) {
                oReturnPokemonList = {
                    result: oResponse.map(oResult => ({
                        id: oResult.result.id,
                        name: oResult.result.name,
                        types: oResult.result.types,
                        image: oResult.result.sprites.other['official-artwork'].front_default,
                        height: oResult.result.height,
                        weight: oResult.result.weight
                    })),
                    error: false
                };
            }
        });

        return oReturnPokemonList;
    }

    async function sendApiRequest(sUrl) {
        try {
            const oResponse = await axios.get(sUrl);

            return { result: oResponse.data, error: null }
        } catch (oError) {
            return { result: null, error: oError }
        }
    }

    function formatPokemonName(sName) {
        return sName.charAt(0).toUpperCase() + sName.slice(1)
    }

    function changeViewType(sView) {
        let sCurrentView = viewType.value;
        viewType.value = sView;

        if (sCurrentView !== sView) {
            sView === "all" ? initialLoadPokemonList() : changePagination(1);
            currentPage.value = 1;
        }
    }

    // using "load more" view
    async function initialLoadPokemonList() {
        let oPokemonList = await getPokemonList();

        if (oPokemonList.error === false) {
            pokemonList.value = oPokemonList.result;
        } else {
            alert("Something went wrong!");
        }
    }

    async function addMorePokemon() {
        let iLimit = 15;
        if (pokemonList.value.length + 15 > POKEDEX_LIMIT) {
            iLimit = POKEDEX_LIMIT - pokemonList.value.length;
        }

        let oPokemonList = await getPokemonList(pokemonList.value.length, iLimit);

        if (oPokemonList.error === false) {
            oPokemonList.result.forEach(oPokemon => {
                pokemonList.value.push(oPokemon);
            })
        } else {
            alert("Something went wrong!");
        }
    }

    // using "pagination" view
    async function changePagination(iPage) {
        let oPokemonList = await getPokemonList((iPage - 1) * PAGE_LIMIT, PAGE_LIMIT);

        if (oPokemonList.error === false) {
            pokemonList.value = oPokemonList.result;
        } else {
            alert("Something went wrong!");
        }
    }

    // state hooks
    onMounted(() => {
        initialLoadPokemonList();

        for(let i = 1; i <= PAGE_COUNT; i++) {
            pageList.value.push(i);
        }
    })
</script>

<template>
    <button @click="changeViewType('all')">View All Pokemon</button>
    <button @click="changeViewType('page')">View Pokemon by Pages</button>
    <ol type="1">
        <li v-for="pokemon in pokemonList" :key="pokemon.id">
            <img :src="pokemon.image" style="height: 60px; width: 60px; object-fit: contain;" />
            <p>{{ formatPokemonName(pokemon.name) }}</p>
            <p>
                <span v-for="elemtype in pokemon.types" :key="elemtype.slot">
                    {{ elemtype.type.name.toUpperCase() }}&nbsp;
                </span>
            </p>
            <button>View Details</button>
        </li>
        <!-- <li v-for="pokemon in pokemonList" :key="pokemon.id">{{ formatPokemonName(pokemon.name) }}</li> -->
    </ol>
    <button v-show="viewType === 'all' && pokemonList.length < POKEDEX_LIMIT" @click="addMorePokemon">Load More</button>
    <div v-show="viewType === 'page'">
        <p>
            <button v-for="page in pageList" :key="page" @click="changePagination(page)">{{ page }}</button>
        </p>
    </div>
</template>