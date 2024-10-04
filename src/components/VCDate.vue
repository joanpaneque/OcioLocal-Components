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
    },
    type: {
        type: String,
        required: false,
        default: 'date'
    },
    max: {
        type: String,
        required: false,
        default: '2022-05-13'
    },
    min: {
        type: String,
        required: false,
        default: '2021-12-20'
    }
});

const emit = defineEmits("update:modelValue");

const selected = ref(props.modelValue);

const open = ref(false);
const everOpened = ref(false);

const currentWindow = ref(props.type === 'birthday' ? 'multiyear' : 'month');


// current year if modelValue is empty, if not, the year of the modelValue


const selectedYear = ref(props.modelValue ? new Date(props.modelValue).getFullYear() : null);
const selectedMonth = ref(props.modelValue ? new Date(props.modelValue).getMonth() : null);
const selectedDay = ref(props.modelValue ? new Date(props.modelValue).getDate() : null);


const calendarYear = ref(null);
const calendarMonth = ref(null);
const calendarDecade = ref(null);

resetCalendar();

function resetCalendar() {
    const currentDate = new Date();
    const currentYear = currentDate.getFullYear();
    const currentMonth = currentDate.getMonth();

    const minDate = props.min ? new Date(props.min) : null;
    const maxDate = props.max ? new Date(props.max) : null;
    const minYear = minDate ? minDate.getFullYear() : null;
    const maxYear = maxDate ? maxDate.getFullYear() : null;
    const minMonth = minDate ? minDate.getMonth() : null;
    const maxMonth = maxDate ? maxDate.getMonth() : null;

    // Calcular el año del calendario
    if (selectedYear.value != null) {
        calendarYear.value = selectedYear.value;
    } else if (maxYear != null && maxYear < currentYear) {
        calendarYear.value = maxYear;
    } else if (minYear != null && minYear > currentYear) {
        calendarYear.value = minYear;
    } else {
        calendarYear.value = currentYear;
    }

    // Calcular el mes del calendario
    if (selectedMonth.value != null) {
        calendarMonth.value = selectedMonth.value;
    } else if (calendarYear.value === maxYear && maxMonth != null && maxMonth < currentMonth) {
        calendarMonth.value = maxMonth;
    } else if (calendarYear.value === minYear && minMonth != null && minMonth > currentMonth) {
        calendarMonth.value = minMonth;
    } else {
        calendarMonth.value = currentMonth;
    }

    // Calcular la década del calendario
    if (selectedYear.value != null) {
        calendarDecade.value = Math.floor(selectedYear.value / 10);
    } else if (maxYear != null && Math.floor(maxYear / 10) < Math.floor(currentYear / 10)) {
        calendarDecade.value = Math.floor(maxYear / 10);
    } else if (minYear != null && Math.floor(minYear / 10) > Math.floor(currentYear / 10)) {
        calendarDecade.value = Math.floor(minYear / 10);
    } else {
        calendarDecade.value = Math.floor(currentYear / 10);
    }
}



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
    open.value = !open.value;
    everOpened.value = true;
    resetCalendar();

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
    if (calendarMonth.value === 11) {
        calendarMonth.value = 0;
        calendarYear.value++;
    } else {
        calendarMonth.value++;
    }
}

function updatePreviousMonth() {
    if (calendarMonth.value === 0) {
        calendarMonth.value = 11;
        calendarYear.value--;
    } else {
        calendarMonth.value--;
    }
}

function updateNextDecade() {
    calendarDecade.value++;
}

function updatePreviousDecade() {
    calendarDecade.value--;
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
        return months[nextMonth(calendarMonth.value)] + ' ' + nextYear(calendarYear.value);
    } else if (transitioningToPreviousMonth.value) {
        return months[previousMonth(calendarMonth.value)] + ' ' + previousYear(calendarYear.value);
    } else {
        return months[calendarMonth.value] + ' ' + calendarYear.value;
    }
}

function displayDecade() {
    if (transitioningToNextDecade.value) {
        return (calendarDecade.value + 1) * 10 + ' - ' + ((calendarDecade.value + 2) * 10 - 1);
    } else if (transitioningToPreviousDecade.value) {
        return (calendarDecade.value - 1) * 10 + ' - ' + (calendarDecade.value * 10 - 1);
    } else {
        return calendarDecade.value * 10 + ' - ' + ((calendarDecade.value + 1) * 10 - 1);
    }
}

function handleHeaderClick() {
    resetCalendar();
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

function handleDayClick(day) {
    console.log(day);
    selectedYear.value = calendarYear.value;
    selectedMonth.value = calendarMonth.value;
    selectedDay.value = day + 1;
    if (selectedYear.value !== null && selectedMonth.value !== null && selectedDay.value !== null) {
        selected.value = new Date(selectedYear.value, selectedMonth.value, selectedDay.value).toISOString().slice(0, 10);
        emit('update:modelValue', selected.value);
    }
}

function isOutOfThisMonth(index, year, month) {
    return getRealDateIndex(index, year, month) <= 0 || getRealDateIndex(index, year, month) > daysInMonth(year, month);
}

function isSelectedDay(index, year, month) {
    return getRealDateIndex(index, year, month) === selectedDay.value - 1 && year === selectedYear.value && month === selectedMonth.value;
}

function isToday(index, year, month) {
    return getRealDateIndex(index, year, month) === new Date().getDate() && year === new Date().getFullYear() && month === new Date().getMonth();
}

function convertDate(date) {
    return date.split('-').reverse().join('/');
}

function handleMonthClick(month) {
    selectedMonth.value = month;
    calendarMonth.value = month - 1;
    currentWindow.value = "month";
    if (selectedYear.value !== null && selectedMonth.value !== null && selectedDay.value !== null) {
        selected.value = new Date(selectedYear.value, selectedMonth.value, selectedDay.value).toISOString().slice(0, 10);
        emit('update:modelValue', selected.value);
    }
}

function handleYearClick(year) {
    selectedYear.value = year;
    calendarYear.value = year;
    currentWindow.value = "year";
    if (selectedYear.value !== null && selectedMonth.value !== null && selectedDay.value !== null) {
        selected.value = new Date(selectedYear.value, selectedMonth.value, selectedDay.value).toISOString().slice(0, 10);
        emit('update:modelValue', selected.value);
    }
}

function maxYear() {
    if (props.max) {
        return new Date(props.max).getFullYear();
    } else {
        return null;
    }
}

function maxMonth() {
    if (props.max) {
        return new Date(props.max).getMonth();
    } else {
        return null;
    }
}

function maxDay() {
    if (props.max) {
        return new Date(props.max).getDate();
    } else {
        return null;
    }
}

function minYear() {
    if (props.min) {
        return new Date(props.min).getFullYear();
    } else {
        return null;
    }
}

function minMonth() {
    if (props.min) {
        return new Date(props.min).getMonth();
    } else {
        return null;
    }
}

function minDay() {
    if (props.min) {
        return new Date(props.min).getDate();
    } else {
        return null;
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
                            v-if="selected">
                            {{ convertDate(selected) }}
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
                            {{ currentWindow === 'month' ? displayMonthYear() : currentWindow === 'year' ? calendarYear
                                : displayDecade() }}
                        </span>
                        <div class="flex gap-1 h-full" v-if="currentWindow === 'month' || currentWindow === 'multiyear'">
                            <span
                                class="rotate-180 h-full flex justify-center items-center w-8 hover:bg-slate-100 rounded-[10px]"
                                :class="{
                                    'text-gray-200 pointer-events-none':
                                        (currentWindow === 'multiyear' && calendarDecade * 10 < minYear()) ||
                                        (currentWindow === 'month' && minYear() && calendarYear === minYear() && minMonth() === calendarMonth)
                                }"
                                @click="handlePreviousArrow">
                                <svg class="h-5 w-5 transition-all duration-200" fill="none"
                                    stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                        d="M19 9l-7 7-7-7">
                                    </path>
                                </svg>
                            </span>
                            <span
                                class="h-full flex justify-center items-center w-8 hover:bg-slate-100 rounded-[10px]"
                                :class="{
                                    'text-gray-200 pointer-events-none':
                                        (currentWindow === 'multiyear' && maxYear() && calendarDecade * 10 + 10 > maxYear()) ||
                                        (currentWindow === 'month' && maxYear() && calendarYear === maxYear() && maxMonth() === calendarMonth)                                }"
                                @click="handleNextArrow">
                                <svg
                                    class="h-5 w-5 transition-all duration-200" fill="none"
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
                                    <span v-for="(day, index) in 42"
                                        :class="{
                                            'text-gray-400': isOutOfThisMonth(index, previousYear(calendarYear, calendarMonth), previousMonth(calendarMonth)),
                                            'bg-slate-300 hover:bg-slate-300 text-black': isSelectedDay(index, previousYear(calendarYear, calendarMonth), previousMonth(calendarMonth)),
                                        }"
                                        class="flex items-center justify-center">
                                        {{ getCalendarDay(index, previousYear(calendarYear, calendarMonth),
                                            previousMonth(calendarMonth)) }}
                                    </span>
                                </div>
                                <div :class="{
                                    'transition-to-previous-month-as-current-month': transitioningToPreviousMonth,
                                    'transition-to-next-month-as-current-month': transitioningToNextMonth,
                                    'top-0': !transitioningToPreviousMonth || !transitioningToNextMonth,
                                }" class="grid grid-rows-6 grid-cols-7 w-full h-full absolute">
                                    <span v-for="(day, index) in 42"
                                        @click="handleDayClick(getRealDateIndex(index, calendarYear, calendarMonth))"
                                        class="flex items-center justify-center hover:bg-slate-100 rounded-[50%]"
                                        :class="{
                                            'text-gray-400':
                                                isOutOfThisMonth(index, calendarYear, calendarMonth) &&
                                                !(maxYear() && calendarYear === maxYear() && calendarMonth === maxMonth() && getRealDateIndex(index, calendarYear, calendarMonth) > maxDay()) &&
                                                !(minYear() && calendarYear === minYear() && calendarMonth === minMonth() && getRealDateIndex(index, calendarYear, calendarMonth) < minDay()),
                                            'bg-slate-300 hover:bg-slate-300 text-black': isSelectedDay(index, calendarYear, calendarMonth),
                                            'font-bold': isToday(index, calendarYear, calendarMonth),
                                            'pointer-events-none text-gray-200':
                                                maxYear() && calendarYear === maxYear() && calendarMonth === maxMonth() && getRealDateIndex(index, calendarYear, calendarMonth) > maxDay() ||
                                                minYear() && calendarYear === minYear() && calendarMonth === minMonth() && getRealDateIndex(index, calendarYear, calendarMonth) < minDay()

                                        }">
                                        {{ getCalendarDay(index, calendarYear, calendarMonth) }}
                                    </span>
                                </div>
                                <div :class="{
                                    'transition-to-next-month-as-next-month': transitioningToNextMonth,
                                    'top-[100%]': !transitioningToNextMonth,
                                }" class="grid grid-rows-6 grid-cols-7 w-full h-full absolute">
                                    <span v-for="(day, index) in 42"
                                        :class="{
                                            'text-gray-400': isOutOfThisMonth(index, nextYear(calendarYear, calendarMonth), nextMonth(calendarMonth)),
                                            'bg-slate-300 hover:bg-slate-300 text-black': isSelectedDay(index, nextYear(calendarYear, calendarMonth), previousMonth(calendarMonth)),
                                        }"
                                        class="flex items-center justify-center rounded-[10px]">
                                        {{ getCalendarDay(index, nextYear(calendarYear, calendarMonth),
                                            nextMonth(calendarMonth)) }}
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
                            <div v-for="(month) in 12"
                                @click="handleMonthClick(month)"
                                :class="{
                                    'font-bold': month === new Date().getMonth()+1 && calendarYear === new Date().getFullYear(),
                                    'bg-slate-300 hover:bg-slate-300 text-black': selectedMonth === month,
                                    'text-gray-300 pointer-events-none':
                                    maxYear() && calendarYear === maxYear() && month - 1 > maxMonth() ||
                                    minYear() && calendarYear === minYear() && month - 1 < minMonth()
                                }"
                                class="flex items-center justify-center hover:bg-slate-100 rounded-[10px]">
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
                                        {{ (calendarDecade - 1) * 10 + index }}
                                    </span>
                                </div>
                                <div
                                    :class="{
                                        'transition-to-previous-month-as-current-month': transitioningToPreviousDecade,
                                        'transition-to-next-month-as-current-month': transitioningToNextDecade,
                                        'top-0': !transitioningToPreviousDecade || !transitioningToNextDecade,
                                    }"
                                    class="absolute w-full h-full grid grid-cols-2 grid-rows-5">
                                    <span v-for="(year, index) in 10"
                                    @click="handleYearClick((calendarDecade * 10) + index)"
                                    :class="{
                                        'font-bold': new Date().getFullYear() === calendarDecade * 10 + index,
                                        'bg-slate-300 hover:bg-slate-300 text-black': selectedYear === calendarDecade * 10 + index,
                                        'text-gray-300 pointer-events-none': maxYear() && (calendarDecade * 10 + index) > maxYear() || minYear() && (calendarDecade * 10 + index) < minYear()
                                    }"
                                    class="flex items-center justify-center hover:bg-slate-100 rounded-[10px]">
                                        {{ calendarDecade * 10 + index }}
                                    </span>
                                </div>
                                <div
                                    :class="{
                                        'transition-to-next-month-as-next-month': transitioningToNextDecade,
                                        'top-[100%]': !transitioningToNextDecade,
                                    }"
                                    class="absolute w-full h-full grid grid-cols-2 grid-rows-5">
                                    <span v-for="(year, index) in 10"
                                    class="flex items-center justify-center hover:bg-slate-100 rounded-[10px]">
                                        {{ (calendarDecade + 1) * 10 + index }}
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
