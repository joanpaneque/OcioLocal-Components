<script setup>
import { defineProps, ref } from 'vue'

const props = defineProps({
    title: {
        type: String,
        required: true
    },
    input: {
        type: String,
        required: true
    }
});

const floatRight1 = ref(false);
const floatRight2 = ref(false);


</script>

<template>
    <div>
        <h2 class="text-lg font-medium text-white rounded-lg rounded-bl-none rounded-br-none w-fit bg-teal-800 px-3">
            {{ title }}
        </h2>
        <div class="rounded-lg rounded-tl-none bg-white p-4 grid gap-5">
            <span v-if="input">{{ input }}</span>
            <span v-else class="text-red-500">No input</span>
            <div>
                <h2
                    class="text-lg font-medium text-white rounded-lg mb-2 w-fit bg-gray-800 px-3">
                    100% width
                </h2>
                <div class="rounded-lg rounded-tl-none bg-white">
                    <slot />
                </div>
            </div>
            <div>
                <h2
                    @click="floatRight1 = !floatRight1"
                    class="text-lg font-medium text-white rounded-lg mb-2 w-fit bg-gray-800 px-3">
                    50% width
                </h2>
                <div class="rounded-lg rounded-tl-none bg-white w-[50%]" :class="{'float-right': floatRight1}">
                    <slot />
                </div>
            </div>
            <div>
                <h2
                    @click="floatRight2 = !floatRight2"
                    class="text-lg font-medium text-white rounded-lg mb-2 w-fit bg-gray-800 px-3">
                    25% width
                </h2>
                <div class="rounded-lg rounded-tl-none bg-white w-[25%]" :class="{'float-right': floatRight2}">
                    <slot />
                </div>
            </div>
        </div>
    </div>
</template>