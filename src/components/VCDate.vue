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
        required: true,
        default: ''
    }
});

const emit = defineEmits("update:modelValue");

const selected = ref(props.modelValue);

const open = ref(false);
const everOpened = ref(false);


// current year if modelValue is empty, if not, the year of the modelValue
const selectedYear = ref(props.modelValue ? new Date(props.modelValue).getFullYear() : new Date().getFullYear());
const selectedMonth = ref(props.modelValue ? new Date(props.modelValue).getMonth() : new Date().getMonth());
const selectedDecade = ref(props.modelValue ? parseInt(new Date(props.modelValue).getFullYear()/10) : parseInt(new Date().getFullYear()/10));

//const selectedDay = ref(props.modelValue ? new Date(props.modelValue).getDate() : new Date().getDate());

const months = [
    'enero',
    'febrero',
    'marzo',
    'abril',
    'mayo',
    'junio',
    'julio',
    'agosto',
    'septiembre',
    'octubre',
    'noviembre',
    'diciembre'
];


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
        const heightList = 320;
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



function getCalendarDay(index, year, month) {
    const day = new Date(year, month, getRealDateIndex(index, year, month)).getDate();

    return day;
}

function getRealDateIndex(index, year, month) {
    const weekDay = new Date(year, month, 1).getDay();
    const previousMonthDays = weekDay === 0 ? 6 : weekDay - 1;
    return index + 1 - previousMonthDays;
}

function daysInMonth(year, month) {
    return new Date(year, month + 1, 0).getDate();
}

function previousMonth(month) {
    if (month === 0) {
        return 11;
    } else {
        return month - 1;
    }
}

function nextMonth(month) {
    if (month === 11) {
        return 0;
    } else {
        return month + 1;
    }
}

function previousYear(year, month) {
    if (month === 0) {
        return year - 1;
    } else {
        return year;
    }
}

function nextYear(year, month) {
    if (month === 11) {
        return year + 1;
    } else {
        return year;
    }
}

function updateNextMonth() {
    if (selectedMonth.value === 11) {
        selectedMonth.value = 0;
        selectedYear.value++;
    } else {
        selectedMonth.value++;
    }
}

function updatePreviousMonth() {
    if (selectedMonth.value === 0) {
        selectedMonth.value = 11;
        selectedYear.value--;
    } else {
        selectedMonth.value--;
    }
}

function updateNextDecade() {
    selectedDecade.value++;
}

function updatePreviousDecade() {
    selectedDecade.value--;
}

const nextMonthTransitionTimeout = ref(null);
const previousMonthTransitionTimeout = ref(null);
const nextDecadeTransitionTimeout = ref(null);
const previousDecadeTransitionTimeout = ref(null);

function handleNextMonth() {
    if (transitioningToNextMonth.value || transitioningToPreviousMonth.value) {
        return;
    }
    transitioningToNextMonth.value = true;
    nextMonthTransitionTimeout.value = setTimeout(() => {
        updateNextMonth();
        transitioningToNextMonth.value = false;
    }, 250);

}

function handlePreviousMonth() {
    if (transitioningToPreviousMonth.value || transitioningToNextMonth.value) {
        return;
    }
    transitioningToPreviousMonth.value = true;
    previousMonthTransitionTimeout.value = setTimeout(() => {
        updatePreviousMonth();
        transitioningToPreviousMonth.value = false;
    }, 250);
}

function handleNextDecade() {
    if (transitioningToNextDecade.value || transitioningToPreviousDecade.value) {
        return;
    }
    transitioningToNextDecade.value = true;
    nextDecadeTransitionTimeout.value = setTimeout(() => {
        updateNextDecade();
        transitioningToNextDecade.value = false;
    }, 250);
}

function handlePreviousDecade() {
    if (transitioningToPreviousDecade.value || transitioningToNextDecade.value) {
        return;
    }
    transitioningToPreviousDecade.value = true;
    previousDecadeTransitionTimeout.value = setTimeout(() => {
        updatePreviousDecade();
        transitioningToPreviousDecade.value = false;
    }, 250);
}

const transitioningToNextMonth = ref(false);
const transitioningToPreviousMonth = ref(false);

const transitioningToNextDecade = ref(false);
const transitioningToPreviousDecade = ref(false);

function displayMonthYear() {
    if (transitioningToNextMonth.value) {
        return months[nextMonth(selectedMonth.value)] + ' ' + nextYear(selectedYear.value);
    } else if (transitioningToPreviousMonth.value) {
        return months[previousMonth(selectedMonth.value)] + ' ' + previousYear(selectedYear.value);
    } else {
        return months[selectedMonth.value] + ' ' + selectedYear.value;
    }
}


const currentWindow = ref("month");

function handleHeaderClick() {
    if (currentWindow.value === "month") {
        currentWindow.value = "year";
    } else if (currentWindow.value === "year") {
        currentWindow.value = "multiyear";
    } else {
        currentWindow.value = "month";
    }
}

function handleNextArrow() {
    if (currentWindow.value === "month") {
        handleNextMonth();
    } else if (currentWindow.value === "multiyear") {
        handleNextDecade();
    }
}

function handlePreviousArrow() {
    if (currentWindow.value === "month") {
        handlePreviousMonth();
    } else if (currentWindow.value === "multiyear") {
        handlePreviousDecade();
    }
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
                'hidden': !everOpened
            }"
                class="w-[300px] bg-gradient-to-r from-cyan-500 to-blue-500 rounded-[10px] p-[1px] absolute z-50 overflow-hidden flex gap-[1px]">
                <div class="bg-white h-[320px] rounded-[9px] p-2 overflow-hidden grid gap-[1px] w-full grid-rows-2"
                    style="grid-template-rows: 40px auto">
                    <div class="flex justify-between items-center h-full gap-1">
                        <span
                            class="text-gray-400 px-[10px] w-full hover:bg-slate-100 rounded-[10px] h-full flex items-center justify-start"
                            @click="handleHeaderClick">
                            {{ currentWindow === 'month' ? displayMonthYear() : currentWindow === 'year' ? selectedYear
                                : selectedYear - 10 + ' - ' + (selectedYear + 9) }}
                        </span>
                        <div class="flex gap-1 h-full" v-if="currentWindow === 'month' || currentWindow === 'multiyear'">
                            <span
                                class="text-gray-400 rotate-180 h-full flex justify-center items-center w-8 hover:bg-slate-100 rounded-[10px]"
                                @click="handlePreviousArrow">
                                <svg class="h-5 w-5 text-gray-400 transition-all duration-200" fill="none"
                                    stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M19 9l-7 7-7-7">
                                    </path>
                                </svg>
                            </span>
                            <span
                                class="text-gray-400 h-full flex justify-center items-center w-8 hover:bg-slate-100 rounded-[10px]"
                                @click="handleNextArrow">
                                <svg class="h-5 w-5 text-gray-400 transition-all duration-200" fill="none"
                                    stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M19 9l-7 7-7-7">
                                    </path>
                                </svg>
                            </span>
                        </div>
                    </div>
                    <div class="relative">
                        <div :class="{
                            'slide-in-bck-center': currentWindow === 'month',
                            'scale-out-center': currentWindow === 'year',
                            'hidden': currentWindow === 'multiyear'
                        }"
                        class="grid grid-rows-2 h-full" style="grid-template-rows: 25px auto">
                            <div class="grid grid-cols-7 w-full font-semibold">
                                <span class="flex items-center justify-center">L</span>
                                <span class="flex items-center justify-center">M</span>
                                <span class="flex items-center justify-center">X</span>
                                <span class="flex items-center justify-center">J</span>
                                <span class="flex items-center justify-center">V</span>
                                <span class="flex items-center justify-center">S</span>
                                <span class="flex items-center justify-center">D</span>
                            </div>
                            <div class="h-full relative overflow-hidden">
                                <div :class="{
                                    'transition-to-previous-month-as-previous-month': transitioningToPreviousMonth,
                                    'top-[-100%]': !transitioningToPreviousMonth,
                                }" class="grid grid-rows-6 grid-cols-7 w-full h-full absolute">
                                    <span v-for="(day, index) in 42" class="flex items-center justify-center">
                                        {{ getCalendarDay(index, previousYear(selectedYear, selectedMonth),
                                            previousMonth(selectedMonth)) }}
                                    </span>
                                </div>
                                <div :class="{
                                    'transition-to-previous-month-as-current-month': transitioningToPreviousMonth,
                                    'transition-to-next-month-as-current-month': transitioningToNextMonth,
                                    'top-0': !transitioningToPreviousMonth || !transitioningToNextMonth,
                                }" class="grid grid-rows-6 grid-cols-7 w-full h-full absolute">
                                    <span v-for="(day, index) in 42"
                                        class="flex items-center justify-center hover:bg-slate-100 rounded-[50%]"
                                        :class="{
                                            'text-gray-400': getRealDateIndex(index, selectedYear, selectedMonth) <= 0 || getRealDateIndex(index, selectedYear, selectedMonth) > daysInMonth(selectedYear, selectedMonth),
                                        }">
                                        {{ getCalendarDay(index, selectedYear, selectedMonth) }}
                                    </span>
                                </div>
                                <div :class="{
                                    'transition-to-next-month-as-next-month': transitioningToNextMonth,
                                    'top-[100%]': !transitioningToNextMonth,
                                }" class="grid grid-rows-6 grid-cols-7 w-full h-full absolute">
                                    <span v-for="(day, index) in 42" class="flex items-center justify-center">
                                        {{ getCalendarDay(index, nextYear(selectedYear, selectedMonth),
                                            nextMonth(selectedMonth)) }}
                                    </span>
                                </div>
                            </div>
                        </div>
                        <div
                            :class="{
                                'slide-in-bck-center': currentWindow === 'year',
                                'scale-out-center': currentWindow === 'multiyear',
                                'hidden': currentWindow === 'month'
                            }"
                            class="absolute top-0 w-full h-full grid grid-cols-3 grid-rows-4 gap-1">
                            <div v-for="month in 12" class="flex items-center justify-center hover:bg-slate-100 rounded-[10px]">
                                {{ months[month - 1].slice(0, 3) }}.
                            </div>
                        </div>
                        <div :class="{
                                'slide-in-bck-center': currentWindow === 'multiyear',
                                'scale-out-center': currentWindow === 'month',
                                'hidden': currentWindow === 'year'
                            }"
                            class="absolute top-0 w-full h-full">
                            <div class="relative w-full h-full overflow-hidden">
                                <div
                                    :class="{
                                        'transition-to-previous-month-as-previous-month': transitioningToPreviousDecade,
                                        'top-[-100%]': !transitioningToPreviousDecade,
                                    }"
                                    class="absolute w-full h-full grid grid-cols-2 grid-rows-5">
                                    <span v-for="(year, index) in 10" class="flex items-center justify-center hover:bg-slate-100 rounded-[10px]">
                                        {{ (selectedDecade - 1) * 10 + index }}
                                    </span>
                                </div>
                                <div
                                    :class="{
                                        'transition-to-previous-month-as-current-month': transitioningToPreviousDecade,
                                        'transition-to-next-month-as-current-month': transitioningToNextDecade,
                                        'top-0': !transitioningToPreviousDecade || !transitioningToNextDecade,
                                    }"
                                    class="absolute w-full h-full grid grid-cols-2 grid-rows-5">
                                    <span v-for="(year, index) in 10" class="flex items-center justify-center hover:bg-slate-100 rounded-[10px]">
                                        {{ selectedDecade * 10 + index }}
                                    </span>
                                </div>
                                <div
                                    :class="{
                                        'transition-to-next-month-as-next-month': transitioningToNextDecade,
                                        'top-[100%]': !transitioningToNextDecade,
                                    }"
                                    class="absolute w-full h-full grid grid-cols-2 grid-rows-5">
                                    <span v-for="(year, index) in 10" class="flex items-center justify-center hover:bg-slate-100 rounded-[10px]">
                                        {{ (selectedDecade + 1) * 10 + index }}
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.scale-out-center {
    animation: scale-out-center 0.2s cubic-bezier(0.550, 0.085, 0.680, 0.530) both;
}

.scale-in-center {
	animation: scale-in-center 0.2s cubic-bezier(0.250, 0.460, 0.450, 0.940) both;
}

.slide-in-bck-center {
	animation: slide-in-bck-center 0.2s cubic-bezier(0.250, 0.460, 0.450, 0.940) both;
}

@keyframes slide-in-bck-center {
  0% {
    transform: scale(2);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes scale-in-center {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes scale-out-center {
    0% {
        transform: scale(1);
        opacity: 1;
    }

    100% {
        transform: scale(0);
        opacity: 1;
        display: none;
    }
}

.transition-to-previous-month-as-previous-month {
    top: 0;
    transition: top 0.25s;
}

.transition-to-previous-month-as-current-month {
    top: 100%;
    transition: top 0.25s;
}

.transition-to-next-month-as-next-month {
    top: 0;
    transition: top 0.25s;
}

.transition-to-next-month-as-current-month {
    top: -100%;
    transition: top 0.25s;
}

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
