<script setup>
    // declaration
    import axios from "axios";
    import { computed, onMounted, ref } from "vue";

    // components
    import Paginator from './Paginator.vue';
    import PokemonCard from "./PokemonCard.vue";

    // data
    const POKEDEX_LIMIT = 150;
    const PAGE_LIMIT = 30;

    const pokemonList = ref([]);
    const viewType = ref("all"); // values: all, page
    const currentPage = ref(1);

    // computed
    const showLoadMoreButton = computed(() => viewType.value === 'all' && pokemonList.value.length < POKEDEX_LIMIT);
    const showPagination = computed(() => viewType.value === 'page');

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

    function changeViewType(sView) {
        let sCurrentView = viewType.value;
        viewType.value = sView;

        if (sCurrentView !== sView) {
            pokemonList.value = [];
            sView === "all" ? initialLoadPokemonList() : changePagination(1);
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
            currentPage.value = iPage;
        } else {
            alert("Something went wrong!");
        }
    }

    // state hooks
    onMounted(() => {
        initialLoadPokemonList();
    })
</script>

<template>
    <div class="container py-4 px-3 auto">
        <div class="d-flex justify-content-center py-4">
            <h2 style="text-center">Vue Pokedex</h2>
        </div>
        <div class="d-flex btn-group py-2">
            <input type="radio" class="btn-check" name="viewoption" autocomplete="off" value="all" v-model="viewType" />
            <button type="button" class="btn btn-outline-primary" @click="changeViewType('all')">
                View All Pokemon
            </button>
            <input type="radio" class="btn-check" name="viewoption" autocomplete="off" value="page" v-model="viewType" />
            <button type="button" class="btn btn-outline-primary" @click="changeViewType('page')">
                View Pokemon by Pages
            </button>
        </div>

        <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 row-cols-xl-4">
            <PokemonCard v-for="pokemon in pokemonList" :key="pokemon.id" :pokemon="pokemon" />
        </div>

        <div class="justify-content-center py-2" :class="[ showLoadMoreButton === true ? 'd-flex' : 'd-none' ]">
            <button type="button" class="btn btn-outline-primary" @click="addMorePokemon">Load More</button>
        </div>
        <div class="pagination justify-content-center py-2" :class="[ showPagination === true ? 'd-flex' : 'd-none' ]">
            <Paginator :current-page="currentPage" :item-count="POKEDEX_LIMIT" :page-limit="PAGE_LIMIT" @change-page="changePagination" />
        </div>
    </div>
</template>