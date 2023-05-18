<script setup>
    import { computed, ref } from "vue";

    const props = defineProps({
        pokemon: Object
    });

    const useShinyVersion = ref(false);
    
    const formattedName = computed(() => props.pokemon.name.charAt(0).toUpperCase() + props.pokemon.name.slice(1));
    const indexNo = computed(() => '#' + String(props.pokemon.id).padStart(3, '0'));
    const imageUrl = computed(() => useShinyVersion.value === true ? props.pokemon?.image.front_shiny : props.pokemon?.image.front_default )
</script>

<template>
    <div class="modal fade" id="pokemonDetails" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="pokemonDetailsLabel" aria-hidden="true">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h1 class="modal-title fs-5" id="pokemonDetailsLabel">Pokemon Details</h1>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <div class="d-flex flex-column py-2">
                        <div class="d-flex justify-content-center">
                            <img class="object-fit-cover border rounded bg-white px-2 py-2" :src="imageUrl" :alt="props.pokemon.name" style="height: 300px; width: 300px;" />
                        </div>
                        <div class="d-flex justify-content-center">
                            <input class="form-check-input" type="checkbox" id="checkShiny" v-model="useShinyVersion">
                            <label class="form-check-label" for="checkShiny">
                                Display Shiny version
                            </label>
                        </div>                        
                    </div>
                    <table class="table table-borderless mt-3">
                        <tr>
                            <th scope="row">Pokedex No.</th>
                            <td class="py-2">{{ indexNo }}</td>
                        </tr>
                        <tr>
                            <th scope="row">Name</th>
                            <td class="py-2">{{ formattedName }}</td>
                        </tr>
                        <tr>
                            <th scope="row">Type</th>
                            <td class="py-2">
                                <span class="badge me-1" :class="[ 'type-badge-' + elemtype.type.name ]" v-for="elemtype in props.pokemon.types" :key="elemtype.slot">
                                    {{ elemtype.type.name.toUpperCase() }}
                                </span>
                            </td>
                        </tr>
                        <tr>
                            <th scope="row">Height</th>
                            <td class="py-2">{{ props.pokemon.height }}</td>
                        </tr>
                        <tr>
                            <th scope="row">Weight</th>
                            <td class="py-2">{{ props.pokemon.weight }}</td>
                        </tr>
                    </table>
                </div>
            </div>
        </div>
    </div>
</template>