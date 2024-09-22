<script setup>
import { defineProps } from 'vue'

const props = defineProps({
    title: {
        type: String,
        required: true
    }
});

</script>

<template>
    <div>
        <h2 class="text-lg font-medium text-white rounded-lg rounded-bl-none rounded-br-none w-fit bg-teal-800 px-3">
            {{ title }}
        </h2>
        <div class="rounded-lg rounded-tl-none bg-white p-4 grid gap-5">
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
                    class="text-lg font-medium text-white rounded-lg mb-2 w-fit bg-gray-800 px-3">
                    50% width
                </h2>
                <div class="rounded-lg rounded-tl-none bg-white w-[50%]">
                    <slot />
                </div>
            </div>
            <div>
                <h2
                    class="text-lg font-medium text-white rounded-lg mb-2 w-fit bg-gray-800 px-3">
                    25% width
                </h2>
                <div class="rounded-lg rounded-tl-none bg-white w-[25%]">
                    <slot />
                </div>
            </div>
        </div>
    </div>
</template>