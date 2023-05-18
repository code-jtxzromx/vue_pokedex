<script setup>
    import { computed } from "vue";

    const props = defineProps({
        pokemon: Object
    });
    const emit = defineEmits(['openPopup']);

    const bgColor = computed(() => props.pokemon.types[0].type.name);
    const formattedName = computed(() => props.pokemon.name.charAt(0).toUpperCase() + props.pokemon.name.slice(1));
    const indexNo = computed(() => '#' + String(props.pokemon.id).padStart(3, '0'));

    function openPopup() {
        emit('openPopup', props.pokemon);
    }
</script>

<template>
    <div class="col py-2">
        <div class="card px-1" :class="[ 'card-' + bgColor ]" style="cursor: pointer;" @click="openPopup"  data-bs-toggle="modal" data-bs-target="#pokemonDetails">
            <div class="card-body d-flex">
                <div class="flex-shrink-0">
                    <img class="object-fit-cover border rounded-circle bg-white px-2 py-2" :src="props.pokemon.image.front_default" :alt="props.pokemon.name" style="height: 80px; width: 80px;" />
                </div>
                <div class="flex-grow-1 ms-2">
                    <span class="badge rounded-pill text-bg-dark">{{ indexNo }}</span>
                    <p class="cart-title"><strong>{{ formattedName }}</strong></p>
                    <p class="card-text">
                        <span class="badge me-1" :class="[ 'type-badge-' + elemtype.type.name ]" v-for="elemtype in props.pokemon.types" :key="elemtype.slot">
                            {{ elemtype.type.name.toUpperCase() }}
                        </span>
                    </p>
                </div>
                <div class="float-right">
                    <i class="bi bi-box-arrow-up-right float-right"></i>
                </div>
            </div>
        </div>
    </div>
</template>