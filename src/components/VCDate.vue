<script setup>
import { defineProps, ref, onMounted, onBeforeUnmount, defineEmits } from 'vue';

const props = defineProps({
    placeholder: {
        type: String,
        required: true,
        default: 'Selecciona una fecha'
    },
    modelValue: {
        type: String,
        required: true
    }
});

const emit = defineEmits("update:modelValue");

const selected = ref(props.modelValue);

const open = ref(false);
const everOpened = ref(false);


const selectBox = ref(null);
const selectList = ref(null);

const VCDateContainer = ref(null);

const unfoldedPosition = ref('bl'); // bl, br, tl, tr

const handleToggle = () => {
    console.log("TOGGLE");
    open.value = !open.value;
    everOpened.value = true;

    if (!open.value) {
        return;
    }

    unfoldedPosition.value = 'bl';

    setTimeout(() => {
        const leftList = selectList.value.getBoundingClientRect().left;
        const heightList = 250;
        const widthList = selectList.value.getBoundingClientRect().width;
        const topBox = selectBox.value.getBoundingClientRect().top;
        const heightBox = selectBox.value.getBoundingClientRect().height;

        // Space available from selectBox to the bottom of the screen taking in mind 5px of margin
        const spaceBottom = window.innerHeight - topBox - heightBox - 5;

        // Space available from selectBox to the top of the screen taking in mind 5px of margin
        const spaceTop = topBox - 5;

        // Space available from selectBox to the right of the screen
        const spaceRight = window.innerWidth - leftList - widthList;

        // Space available from selectBox to the left of the screen
        const spaceLeft = leftList;


        let position = '';

        if (spaceBottom > heightList) {
            position += 'b';
        } else if (spaceTop > heightList) {
            position += 't';
        } else {
            position += 'b';
        }

        if (spaceRight > widthList) {
            position += 'l';
        } else if (spaceLeft > widthList) {
            position += 'r';
        } else {
            position += 'l';
        }

        console.log("B", spaceBottom, "T", spaceTop, "R", spaceRight, "L", spaceLeft, "P", position);
        console.log(heightList);


        if (position.length !== 2) {
            position = 'bl';
        }


        unfoldedPosition.value = position;

    }, 1);
};

// Cerrar el desplegable cuando se hace clic fuera
const handleClickOutside = (event) => {
    if (!VCDateContainer.value.contains(event.target)) {
        open.value = false;
    }
};

function handleSelect(key) {
    selected.value = key;
    emit('update:modelValue', key);
    open.value = false;
}

// Agregar y eliminar event listener en el ciclo de vida de Vue
onMounted(() => {
    document.addEventListener('click', handleClickOutside);
});

onBeforeUnmount(() => {
    document.removeEventListener('click', handleClickOutside);
});

function handleSelectMinute(minute) {
    selectedMinute.value = minute;

    if (selectedHour.value === null || selectedMinute.value === null) {
        emit('update:modelValue', '');
        return;
    }
    emit('update:modelValue', `${selectedHour.value}:${selectedMinute.value}`);
}

function handleSelectHour(hour) {
    selectedHour.value = hour;


    if (selectedHour.value === null || selectedMinute.value === null) {
        emit('update:modelValue', '');
        return;
    }
    emit('update:modelValue', `${selectedHour.value}:${selectedMinute.value}`);
}

</script>

<template>
    <div ref="VCDateContainer">
        <div class="relative select-none">
            <div :class="{
                'bg-gradient-to-r from-cyan-500 to-blue-500': selected,
                'bg-gradient-to-r from-gray-300 to-gray-500': !selected
            }" class="p-[1px] rounded-[10px]" @click="handleToggle" ref="selectBox">
                <div class="bg-white rounded-[9px] h-[45px] relative">
                    <div class="absolute pointer-events-none transition-all duration-200 left-[10px]" :class="{
                        'top-[calc(50%)] transform -translate-y-1/2': !selected,
                        'top-[-8px] text-xs bg-white px-2 rounded-[20px]': selected,
                    }">
                        <span class="text-gray-400 select-none" :class="{
                            'bg-gradient-to-r from-cyan-500 to-blue-500 bg-clip-text text-transparent': selected,
                        }">
                            {{ placeholder }}
                        </span>
                    </div>
                    <div
                        class="absolute left-[10px] top-[50%] transform -translate-y-1/2 flex items-center gap-2 pr-5 w-full">
                        <span class="text-ellipsis overflow-hidden whitespace-nowrap pr-5"
                            v-if="selectedHour || selectedMinute">
                            {{ selectedHour ?? '--' }}:{{ selectedMinute ?? '--' }}
                        </span>
                    </div>
                    <div class="absolute right-[10px] top-[50%] transform -translate-y-1/2">
                        <svg :class="{
                            'rotate-[270deg]': !open,
                            'rotate-[360deg]': open && unfoldedPosition === 'bl' || unfoldedPosition === 'br',
                            'rotate-[180deg]': open && unfoldedPosition === 'tl' || unfoldedPosition === 'tr',
                        }" class="h-5 w-5 text-gray-400 transition-all duration-200" fill="none" stroke="currentColor"
                            viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7">
                            </path>
                        </svg>
                    </div>
                </div>
            </div>
            <div ref="selectList" :class="{
                'top-[47px] mt-[5px] swing-in-top-fwd': unfoldedPosition === 'bl' || unfoldedPosition === 'br',
                'bottom-[47px] mb-[5px] swing-in-bottom-bck': unfoldedPosition === 'tl' || unfoldedPosition === 'tr',
                'right-[0px]': unfoldedPosition === 'tr' || unfoldedPosition === 'br',
                'left-[0px]': unfoldedPosition === 'tl' || unfoldedPosition === 'bl',
                'top-[47px] mt-[5px] pointer-events-none swing-out-top-bck': !open && (unfoldedPosition === 'bl' || unfoldedPosition === 'br'),
                'bottom-[47px] mb-[5px] pointer-events-none swing-out-bottom-bck': !open && (unfoldedPosition === 'tl' || unfoldedPosition === 'tr'),
                'hidden': !everOpened,

            }"
                class="w-[250px] bg-gradient-to-r from-cyan-500 to-blue-500 rounded-[10px] p-[1px] absolute z-50 overflow-hidden flex gap-[1px]">
                <div class="bg-white h-[250px] rounded-[9px] overflow-hidden grid gap-[1px] w-full grid-rows-3" style="grid-template-rows: 30px 20px auto">
                    <div class="flex justify-between items-center p-2">
                        <span class="text-gray-400">&lt;</span>
                        <span class="text-gray-400">Enero</span>
                        <span class="text-gray-400">&gt;</span>
                    </div>      
                    <div class="grid grid-cols-7 w-full font-bold">
                        <span class="flex items-center justify-center">L</span>
                        <span class="flex items-center justify-center">M</span>
                        <span class="flex items-center justify-center">X</span>
                        <span class="flex items-center justify-center">J</span>
                        <span class="flex items-center justify-center">V</span>
                        <span class="flex items-center justify-center">S</span>
                        <span class="flex items-center justify-center">D</span>
                    </div>
                    <div>

                    </div>                
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
/* Scrollbar */
::-webkit-scrollbar {
    width: 5px;
}

::-webkit-scrollbar-track {
    background: transparent;
    width: 20px;
}

::-webkit-scrollbar-thumb {
    /* gradient from blue-500 to cyan-500 */
    background: #3B82F6;

}

.swing-in-top-fwd {
    animation: swing-in-top-fwd 0.5s cubic-bezier(0.175, 0.885, 0.320, 1.275) both;
}


.swing-in-bottom-bck {
    animation: swing-in-bottom-bck 0.6s cubic-bezier(0.175, 0.885, 0.320, 1.275) both;
}

.swing-out-top-bck {
    animation: swing-out-top-bck 0.45s cubic-bezier(0.175, 0.885, 0.320, 1.275) both;
}

.swing-out-bottom-bck {
    animation: swing-out-bottom-bck 0.45s cubic-bezier(0.175, 0.885, 0.320, 1.275) both;
}

@keyframes swing-in-bottom-bck {
    0% {
        transform: rotateX(-70deg);
        transform-origin: bottom;
        opacity: 0;
    }

    100% {
        transform: rotateX(0);
        transform-origin: bottom;
        opacity: 1;
    }
}

@keyframes swing-in-top-fwd {
    0% {
        transform: rotateX(-100deg);
        transform-origin: top;
        opacity: 0;
    }

    100% {
        transform: rotateX(0deg);
        transform-origin: top;
        opacity: 1;
    }
}

@keyframes swing-out-top-bck {
    0% {
        transform: rotateX(0deg);
        transform-origin: top;
        opacity: 1;
    }

    100% {
        transform: rotateX(-100deg);
        transform-origin: top;
        opacity: 0;
    }
}

@keyframes swing-out-bottom-bck {
    0% {
        transform: rotateX(0);
        transform-origin: bottom;
        opacity: 1;
    }

    100% {
        transform: rotateX(100deg);
        transform-origin: bottom;
        opacity: 0;
    }
}
</style>
