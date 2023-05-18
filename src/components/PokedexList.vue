<script setup>
    // declaration
    import axios from "axios";
    import { computed, onMounted, reactive, ref } from "vue";

    // data
    const POKEDEX_LIMIT = 150;
    const PAGE_LIMIT = 30;
    const PAGE_COUNT = Math.ceil(POKEDEX_LIMIT / PAGE_LIMIT);

    const pokemonList = ref([]);
    const viewType = ref("all"); // values: all, page
    const currentPage = ref(1);
    const pageList = ref([]);

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

    function formatPokemonName(sName) {
        return sName.charAt(0).toUpperCase() + sName.slice(1)
    }

    function changeViewType(sView) {
        let sCurrentView = viewType.value;
        viewType.value = sView;

        if (sCurrentView !== sView) {
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

        for(let i = 1; i <= PAGE_COUNT; i++) {
            pageList.value.push(i);
        }
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
            <div class="col py-2" v-for="pokemon in pokemonList" :key="pokemon.id">
                <div class="card px-1">
                    <div class="card-body d-flex">
                        <div class="flex-shrink-0">
                            <img :src="pokemon.image" style="height: 60px; width: 60px; object-fit: contain;" :alt="pokemon.name" />
                        </div>
                        <div class="flex-grow-1 mx-3">
                            <span class="badge rounded-pill text-bg-dark">
                                {{ '#' + String(pokemon.id).padStart(3, '0') }}
                            </span>
                            <p class="card-title">
                                <strong>{{ formatPokemonName(pokemon.name) }}</strong>
                            </p>
                            <p class="card-text">
                                <span class="badge text-bg-success me-1" v-for="elemtype in pokemon.types" :key="elemtype.slot">
                                    {{ elemtype.type.name.toUpperCase() }}&nbsp;
                                </span>
                            </p>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="justify-content-center py-2" :class="[ showLoadMoreButton === true ? 'd-flex' : 'd-none' ]">
            <button type="button" class="btn btn-outline-primary" @click="addMorePokemon">Load More</button>
        </div>
        <div class="pagination justify-content-center py-2" :class="[ showPagination === true ? 'd-flex' : 'd-none' ]">
            <ul class="pagination">
                <li class="page-item" :class="{ active: currentPage === page }" v-for="page in pageList" :key="page" @click="changePagination(page)">
                    <a href="#" class="page-link">{{ page }}</a>
                </li>
            </ul>
        </div>
    </div>
</template>