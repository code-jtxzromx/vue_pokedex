<script setup>
    import { computed } from "vue";

    const props = defineProps({
        pokemon: Object
    });

    const bgColor = computed(() => props.pokemon.types[0].type.name);

    function formatPokemonName(sName) {
        return sName.charAt(0).toUpperCase() + sName.slice(1)
    }
</script>

<template>
    <div class="col py-2">
        <div class="card px-1" :class="[ 'card-' + bgColor ]" style="cursor: pointer;;">
            <div class="card-body d-flex">
                <div class="flex-shrink-0">
                    <img class="object-fit-cover border rounded-circle bg-white px-2 py-2" :src="props.pokemon.image" :alt="props.pokemon.name" style="height: 80px; width: 80px;" />
                </div>
                <div class="flex-grow-1 mx-3">
                    <span class="badge rounded-pill text-bg-dark">
                        {{ '#' + String(props.pokemon.id).padStart(3, '0') }}
                    </span>
                    <p class="cart-title">
                        <strong>{{ formatPokemonName(props.pokemon.name) }}</strong>
                    </p>
                    <p class="card-text">
                        <span class="badge me-1" :class="[ 'type-badge-' + elemtype.type.name ]" v-for="elemtype in props.pokemon.types" :key="elemtype.slot">
                            {{ elemtype.type.name.toUpperCase() }}
                        </span>
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>