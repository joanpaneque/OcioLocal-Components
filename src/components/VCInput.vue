<script setup>
import VCInput from '@/components/VCInput.vue'
import { defineProps, ref, watch, defineEmits } from 'vue';

const props = defineProps({
    placeholder: {
        type: String,
        required: false,
        default: 'Type here...'
    },
    placeholder2: {
        type: String,
        required: false,
        default: "Type here..."
    },
    error: {
        type: String,
        required: false,
        default: null
    },
    maxlength: {
        type: Number,
        required: false,
        default: 20
    },
    minlength: {
        type: Number,
        required: false
    },
    modelValue: {
        type: String,
        required: false,
        default: ''
    },
    validation: {
        type: Function,
        required: false,
        default: null
    },
    forceuppercase: {
        type: Boolean,
        required: false,
        default: false
    },
    allowedcharacters: {
        type: String,
        required: false,
        default: null
    },
    type: {
        type: String,
        required: false,
        default: 'text'
    },
    originalpassword: {
        type: String,
        required: false,
        default: ''
    }
});

const emit = defineEmits(['update:modelValue', 'update:error']);


const forceUppercase = ref(props.forceuppercase);
const allowedCharacters = ref(props.allowedcharacters);
const minlength = ref(props.minlength);
const type = ref(props.type.toUpperCase());
const maxlength = ref(props.maxlength);

const showPassword = ref(false);

if (type.value === 'DNI' || type.value === 'NIE' || type.value === 'DNINIE') {
    forceUppercase.value = true;
    allowedCharacters.value = '0123456789TRWAGMYFPDXBNJZSQVHLCKE';
    maxlength.value = 9;
}

if (type.value === 'IBAN') {
    forceUppercase.value = true;
    allowedCharacters.value = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    maxlength.value = 34;
    minlength.value = 15;
}

if (type.value === 'EMAIL') {
    forceUppercase.value = false;
    allowedCharacters.value = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789@.-_@';
    maxlength.value = 254;
}

if (type.value === 'PASSWORD' || type.value === 'PASSWORDCONFIRM') {
    forceUppercase.value = false;
    allowedCharacters.value = null;
    minlength.value = 8;
    maxlength.value = 30;
}


const incomingErrorFromParent = ref(false);

// watch when props.error changes
watch(() => props.error, (newValue) => {
    console.log('error changed');
    validateChange();
    incomingErrorFromParent.value = true;
    setError(newValue);
});

const error = ref(props.error);
const closingError = ref(false);

const passwordConfirmError = ref("");



const tempError = ref(null);

const tempErrorTimeout = ref(null);
const errorTimeout = ref(null);

function setTempError(message) {
    if (tempError.value == message) {
        return;
    } else {
        clearTimeout(tempErrorTimeout.value);
    }
    tempError.value = message;
    tempErrorTimeout.value = setTimeout(() => {
        tempError.value = null;
    }, 1000);
}

function stopTempError() {
    clearTimeout(tempErrorTimeout.value);
    tempError.value = null;
}

function stopError() {
    clearTimeout(errorTimeout.value);
    setError(null);
}


const focus = ref(false);
const input = ref(props.modelValue);
const oldInput = ref(props.modelValue);

const inputPasswordConfirm = ref("");

watch(() => props.originalpassword, (newValue) => {
    if (input.value === "") {
        return;
    }
    validateChange();
});


function setError(message) {
    clearTimeout(errorTimeout.value);
    if (!message) {
        closingError.value = true;
        errorTimeout.value = setTimeout(() => {
            error.value = null;
            closingError.value = false;
        }, 300);
    } else {
        error.value = message;
        clearTimeout(errorTimeout.value);
        emit('update:error', message);
    }
}




function DNITempValidation(value) {
    // si el valor es menor que 9 y hay una letra, dar error
    if (value.length < 9 && isNaN(value)) {
        return { ok: false, error: 'Faltan más números de DNI' };
    }
    // si el valor es de 9 y no hay letra dar error
    if (value.length === 9 && !isNaN(value)) {
        return { ok: false, error: 'Falta la letra del DNI' };
    }

    return { ok: true };
}

function NIETempValidation(value) {

    // Validar que el primer carácter sea (X, Y o Z) o vacío
    if (!['X', 'Y', 'Z', ''].includes(value.charAt(0))) {
        return { ok: false, error: 'Carácter inválido. Usa X, Y o Z.' }
    }

    // Validar que los 7 caracteres siguientes sean números. Si hay menos de 7, no pasa nada, validalo igualmente
    if (!/^\d{0,7}$/.test(value.substring(1, 8))) {
        return { ok: false, error: 'Faltan números del NIE.' };
    }

    // Si value.charAt(8) es un numero, dar error
    if (value.length == 9 && !isNaN(value.charAt(8))) {
        return { ok: false, error: 'Falta la ultima letra del NIE.' };
    }


    return { ok: true };
}

function DNINIETempValidation(value) {
    // Primero hay que diferenciar si es un DNI o un NIE
    // Si el primer caracter es un número, es un DNI
    if (!isNaN(value.charAt(0))) {
        return DNITempValidation(value);
    } else {
        return NIETempValidation(value);
    }
}

function IBANTempValidation(value) {
    // Eliminar todos los espacios en blanco
    value = value.replace(/\s/g, '');

    // El primer caracter debe ser una letra o estar vacío
    if (!/^[A-Z]{0,1}$/.test(value.charAt(0))) {
        return { ok: false, error: 'Falta la primera letra.' };
    }

    // El segundo caracter debe ser una letra o estar vacío
    if (!/^[A-Z]{0,1}$/.test(value.charAt(1))) {
        return { ok: false, error: 'Falta la segunda letra.' };
    }

    // El tercer caracter debe ser un número o estar vacío
    if (!/^\d{0,1}$/.test(value.charAt(2))) {
        return { ok: false, error: 'Falta el tercer número.' };
    }

    // El cuarto caracter debe ser un número o estar vacío
    if (!/^\d{0,1}$/.test(value.charAt(3))) {
        return { ok: false, error: 'Falta el cuarto número.' };
    }

    return { ok: true };
}

function EMAILTempValidation(value) {
    // Si el primer caracter esta vacio, validar
    if (value.charAt(0) === '') {
        return { ok: true };
    }

    // Si el primer caracter es un punto, un guion o una arroba, dar error
    if (['.', '-', '@'].includes(value.charAt(0))) {
        return { ok: false, error: `Carácter inicial inválido: ${value.charAt(0)}` };
    }

    // Si hay mas de un arroba, dar error
    if (value.split('@').length > 2) {
        return { ok: false, error: 'Demasiadas arrobas' };
    }

    // Si hay dos puntos seguidos, dar error
    if (value.includes('..')) {
        return { ok: false, error: 'Demasiados puntos seguidos' };
    }

    // Si hay dos guiones seguidos, dar error
    if (value.includes('--')) {
        return { ok: false, error: 'Demasiados guiones seguidos' };
    }

    // Si hay un punto seguido de un guion, dar error
    if (value.includes('.-')) {
        return { ok: false, error: 'Punto seguido de guion' };
    }

    // Si hay un guion seguido de un punto, dar error
    if (value.includes('-.')) {
        return { ok: false, error: 'Guion seguido de punto' };
    }

    // Si hay un guion seguido de un arroba, dar error
    if (value.includes('-@')) {
        return { ok: false, error: 'Guion seguido de arroba' };
    }

    // Si hay un punto seguido de un arroba, dar error
    if (value.includes('.@')) {
        return { ok: false, error: 'Punto seguido de arroba' };
    }

    // Si hay un arroba seguido de un punto, dar error
    if (value.includes('@.')) {
        return { ok: false, error: 'Arroba seguido de punto' };
    }

    // Si hay un arroba seguido de un guion, dar error
    if (value.includes('@-')) {
        return { ok: false, error: 'Arroba seguido de guion' };
    }

    return { ok: true };
}

function DNIValidation(value) {
    // comprueba si la longitud es 9
    if (value.length !== 9) {
        return { ok: false, error: 'Número de documento incorrecto' };
    }

    // comprueba si los 8 primeros caracteres son números
    if (isNaN(value.substring(0, 8))) {
        return { ok: false, error: 'Número de documento incorrecto' };
    }

    // comprueba si el último caracter es una letra válida
    let letter = value.substring(8).toUpperCase();
    let number = parseInt(value.substring(0, 8));
    let validLetter = 'TRWAGMYFPDXBNJZSQVHLCKE'.charAt(number % 23);
    if (letter !== validLetter) {
        return { ok: false, error: 'Número de documento incorrecto' };
    }

    return { ok: true };
}

function NIEValidation(value) {
    // Eliminamos espacios y convertimos a mayúsculas para normalizar la entrada
    value = value.toUpperCase().trim();

    // Expresión regular para validar el formato del NIE
    const nieRegex = /^[XYZ]\d{7}[A-Z]$/;

    // Validar que el NIE cumple con el patrón básico
    if (!nieRegex.test(value)) {
        return { ok: false, error: "El formato del NIE es incorrecto" };
    }

    // Convertimos la letra inicial a un número:
    // X = 0, Y = 1, Z = 2
    const letraInicial = value.charAt(0);
    let numeroBase = value.slice(1, -1); // Elimina la letra inicial y la letra final

    if (letraInicial === 'X') {
        numeroBase = '0' + numeroBase;
    } else if (letraInicial === 'Y') {
        numeroBase = '1' + numeroBase;
    } else if (letraInicial === 'Z') {
        numeroBase = '2' + numeroBase;
    }

    // Calcular el dígito de control
    const letrasControl = 'TRWAGMYFPDXBNJZSQVHLCKE';
    const numero = parseInt(numeroBase, 10);
    const letraCorrecta = letrasControl[numero % 23];

    // Comparar la letra final del NIE con la letra de control calculada
    const letraFinal = value.charAt(8);
    if (letraFinal !== letraCorrecta) {
        return { ok: false, error: "La letra de control es incorrecta" };
    }

    // Si pasa todas las validaciones, el NIE es válido
    return { ok: true };
}

function DNINIEValidation(value) {
    // Primero hay que diferenciar si es un DNI o un NIE
    // Si el primer caracter es un número, es un DNI
    if (!isNaN(value.charAt(0))) {
        return DNIValidation(value);
    } else {
        return NIEValidation(value);
    }
}

function IBANValidation(value) {
    // Eliminar todos los espacios en blanco
    value = value.replace(/\s/g, '');

    // Los dos primeros caracteres deben ser letras
    if (!/^[A-Z]{2}$/.test(value.substring(0, 2))) {
        return { ok: false, error: 'Faltan las dos primeras letras.' };
    }

    // Los dos siguientes caracteres deben ser números
    if (!/^\d{2}$/.test(value.substring(2, 4))) {
        return { ok: false, error: 'Faltan los dos primeros números.' };
    }

    return { ok: true };
}

function EMAILValidation(value) {
    // Si está vacío, dar error
    if (value === '') {
        return { ok: false, error: 'Email vacío' };
    }

    // Verificar que haya exactamente un arroba
    const atIndex = value.indexOf('@');
    if (atIndex === -1) {
        return { ok: false, error: 'Falta el símbolo @' };
    }
    if (value.split('@').length !== 2) {
        return { ok: false, error: 'Demasiadas arrobas' };
    }

    const localPart = value.slice(0, atIndex);
    const domainPart = value.slice(atIndex + 1);

    // Validar parte local (antes del @)
    if (localPart.length > 64) {
        return { ok: false, error: 'La parte local supera los 64 caracteres permitidos' };
    }
    if (localPart.charAt(0) === '.' || localPart.charAt(localPart.length - 1) === '.') {
        return { ok: false, error: 'Punto seguido de arroba' };
    }

    // Si el primer carácter es un punto, un guion o una arroba, dar error
    if (['.', '-', '@'].includes(localPart.charAt(0))) {
        return { ok: false, error: `Carácter inicial inválido: ${localPart.charAt(0)}` };
    }

    // Verificar puntos consecutivos, guiones consecutivos, combinaciones de punto y guion
    if (localPart.includes('..')) {
        return { ok: false, error: 'Demasiados puntos seguidos en la parte local' };
    }
    if (localPart.includes('--')) {
        return { ok: false, error: 'Demasiados guiones seguidos en la parte local' };
    }
    if (localPart.includes('.-') || localPart.includes('-.')) {
        return { ok: false, error: 'Punto seguido de guion o guion seguido de punto en la parte local' };
    }

    // Validar parte del dominio
    if (domainPart.length > 253) {
        return { ok: false, error: 'El dominio supera los 253 caracteres permitidos' };
    }
    if (!domainPart.includes('.')) {
        return { ok: false, error: 'El dominio debe contener al menos un punto' };
    }

    // Verificar formato de dominio válido (solo letras, números, guiones y puntos)
    const domainParts = domainPart.split('.');
    for (let i = 0; i < domainParts.length; i++) {
        const label = domainParts[i];
        if (label.length === 0) {
            return { ok: false, error: 'El dominio no tiene etiquetas válidas' };
        }
        if (label.charAt(0) === '-' || label.charAt(label.length - 1) === '-') {
            return { ok: false, error: 'No puede terminar con guion' };
        }
    }

    // Validar que el dominio no empiece ni termine con un punto o guion
    if (domainPart.charAt(0) === '-' || domainPart.charAt(domainPart.length - 1) === '-') {
        return { ok: false, error: 'El dominio no puede comenzar o terminar con un guion' };
    }
    if (domainPart.charAt(0) === '.' || domainPart.charAt(domainPart.length - 1) === '.') {
        return { ok: false, error: 'El dominio no puede comenzar o terminar con un punto' };
    }

    return { ok: true };
}

function PASSWORDValidation(value) {
    // Si está vacío, dar error
    if (value.length === 0) {
        return { ok: false, error: 'Contraseña vacía' };
    }

    // La contraseña debe contener al menos 1 letra minúscula
    if (!/[a-z]/.test(value)) {
        return { ok: false, error: 'Falta una letra minúscula' };
    }

    // La contraseña debe contener al menos 1 letra mayúscula
    if (!/[A-Z]/.test(value)) {
        return { ok: false, error: 'Falta una letra mayúscula' };
    }

    // La contraseña debe contener al menos 1 número
    if (!/\d/.test(value)) {
        return { ok: false, error: 'Falta un número' };
    }

    // La contraseña debe contener al menos 1 carácter especial
    if (!/[^a-zA-Z0-9]/.test(value)) {
        return { ok: false, error: 'Falta un carácter especial' };
    }

    return { ok: true };
}

function handleInput(e) {
    if (input.value.length > maxlength.value) {
        setTempError(`Longitud máxima de ${maxlength.value}`);
        input.value = oldInput.value;
        return;
    }

    if (forceUppercase.value) {
        input.value = input.value.toUpperCase();
    }

    if (allowedCharacters.value) {
        // dont use regex, use includes
        for (let i = 0; i < input.value.length; i++) {
            if (!allowedCharacters.value.includes(input.value.charAt(i))) {
                setTempError(`Carácter inválido: ${input.value.charAt(i)}`);
                input.value = oldInput.value;
                return;
            }
        }
    }

    if (type.value === 'DNI') {
        let validation = DNITempValidation(input.value);
        if (!validation.ok) {
            setTempError(validation.error);
            input.value = oldInput.value;
            return;
        }
    }

    if (type.value === 'NIE') {
        let validation = NIETempValidation(input.value);
        if (!validation.ok) {
            setTempError(validation.error);
            input.value = oldInput.value;
            return;
        }
    }

    if (type.value === 'DNINIE') {
        let validation = DNINIETempValidation(input.value);
        if (!validation.ok) {
            setTempError(validation.error);
            input.value = oldInput.value;
            return;
        }
    }

    if (type.value === 'IBAN') {
        let validation = IBANTempValidation(input.value);
        if (!validation.ok) {
            setTempError(validation.error);
            input.value = oldInput.value;
            return;
        }
    }

    if (type.value === 'EMAIL') {
        let validation = EMAILTempValidation(input.value);
        if (!validation.ok) {
            setTempError(validation.error);
            input.value = oldInput.value;
            return;
        }
    }

    oldInput.value = input.value;
}

function validateChange() {

    if (props.validation) {
        let validation = props.validation(input.value);
        console.log(validation);
        if (!validation.ok) {
            setError(validation.error);
            return;
        }
    }


    if (type.value === 'DNI') {
        let validation = DNIValidation(input.value);
        if (!validation.ok) {
            setError(validation.error);
            return;
        }
    }

    if (type.value === 'NIE') {
        let validation = NIEValidation(input.value);
        if (!validation.ok) {
            setError(validation.error);
            return;
        }
    }

    if (type.value === 'DNINIE') {
        let validation = DNINIEValidation(input.value);
        if (!validation.ok) {
            setError(validation.error);
            return;
        }
    }

    if (type.value === 'IBAN') {
        let validation = IBANValidation(input.value);
        if (!validation.ok) {
            setError(validation.error);
            return;
        }
    }

    if (type.value === 'EMAIL') {
        let validation = EMAILValidation(input.value);
        if (!validation.ok) {
            setError(validation.error);
            return;
        }
    }

    if (type.value === 'PASSWORD' || type.value === 'PASSWORDCONFIRM') {
        let validation = PASSWORDValidation(input.value);
        if (!validation.ok) {
            setError(validation.error);
            return;
        }
    }

    if (input.value.length < minlength.value) {
        console.log('error');
        setError(`El tamaño mínimo es de ${minlength.value}`);
        return;
    }

    stopError();
}

function handleChange(e) {
    validateChange();
}

function handleKeyDown(e) {
    if ((e.key === 'Backspace' || e.key === 'Delete') && (input.value ? input.value.length === 0 : true)) {
        setTempError('No puedes borrar más');
        return;
    }

    stopTempError();

}

function passwordConfirmValidation(value) {
    if (value !== input.value) {
        return { ok: false, error: 'Las contraseñas no coinciden' };
    }

    return { ok: true };
}

</script>

<template>
    <div>
        <div
            class="rounded-[10px] p-[1px]"
            :class="{
                'bg-gradient-to-r from-gray-300 to-gray-500': !error && !input && !focus && !tempError,
                'bg-gradient-to-r from-cyan-500 to-blue-500': !error,
                'bg-gradient-to-r from-red-300 to-red-400': error || tempError,
            }">
            <div v-if="error"
                :class="{ 'scale-out-ver-bottom': closingError, 'fill-red-950 text-red-950': error, 'fill-blue-950 text-blue-950': !error }"
                class="scale-in-ver-bottom font-bold text-sm rounded-t-[10px] px-2 flex gap-1 items-center">
                <svg class="w-4 h-4" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
                    version="1.1" id="Capa_1" x="0px" y="0px" viewBox="0 0 512 512"
                    style="enable-background:new 0 0 512 512;" xml:space="preserve" width="512" height="512">
                    <g>
                        <path
                            d="M256,0C114.615,0,0,114.615,0,256s114.615,256,256,256s256-114.615,256-256C511.847,114.678,397.322,0.153,256,0z M256,448   c-106.039,0-192-85.961-192-192S149.961,64,256,64s192,85.961,192,192C447.882,361.99,361.99,447.882,256,448z" />
                        <path
                            d="M256,321.941c17.673,0,32-14.327,32-32V140.608c0-17.673-14.327-32-32-32s-32,14.327-32,32v149.333   C224,307.614,238.327,321.941,256,321.941z" />
                        <circle cx="256.107" cy="373.333" r="32" />
                    </g>
                </svg>
                {{ error }}
            </div>
            <div class="relative flex">
                <input
                    :type="(type === 'PASSWORD' || type === 'PASSWORDCONFIRM') && !showPassword ? 'password' : 'text'"
                    class="w-full h-full bg-white rounded-[9px] focus:outline-none px-5 py-3"
                    :class="{
                        'pr-[45px]': type === 'PASSWORD' || type === 'PASSWORDCONFIRM',
                    }"
                    @focus="focus = true"
                    @blur="focus = false"
                    @input="handleInput"
                    @change="handleChange"
                    @keydown="handleKeyDown"
                    v-model="input"
                >
                <div
                    v-if="tempError"
                    class="fade-out-delayed absolute w-full h-[100%] bg-white top-0 left-0 text-red-500 flex items-center px-[15px] rounded-[9px] pointer-events-none">
                    <span class="shake-horizontal">{{ tempError }}</span>
                </div>
                <div
                    class="absolute pointer-events-none transition-all duration-200 left-[10px]"
                    :class="{
                        'top-[calc(50%)] transform -translate-y-1/2': !focus && !input && !tempError,
                        'top-[-8px] text-xs bg-white px-2 rounded-[20px]': focus || input || tempError,
                    }">
                        <span
                            class="text-gray-400 select-none"
                            :class="{
                                'bg-gradient-to-r from-cyan-500 to-blue-500 bg-clip-text text-transparent': focus || input,
                                'bg-gradient-to-r from-red-300 to-red-400 bg-clip-text text-transparent': (error && (focus || input)) || tempError
                            }"
                        >
                            {{ placeholder }}
                        </span>
                </div>
                <div v-if="type === 'PASSWORD' || type === 'PASSWORDCONFIRM'" class="absolute right-[10px] top-[50%] transform -translate-y-1/2 h-[30px] w-[30px] p-1 cursor-pointer bg-white" @click="showPassword = !showPassword">
                    <div class="relative select-none">
                        <svg class="w-full h-full" xmlns="http://www.w3.org/2000/svg" id="Outline" viewBox="0 0 24 24" width="512" height="512"><path d="M23.271,9.419C21.72,6.893,18.192,2.655,12,2.655S2.28,6.893.729,9.419a4.908,4.908,0,0,0,0,5.162C2.28,17.107,5.808,21.345,12,21.345s9.72-4.238,11.271-6.764A4.908,4.908,0,0,0,23.271,9.419Zm-1.705,4.115C20.234,15.7,17.219,19.345,12,19.345S3.766,15.7,2.434,13.534a2.918,2.918,0,0,1,0-3.068C3.766,8.3,6.781,4.655,12,4.655s8.234,3.641,9.566,5.811A2.918,2.918,0,0,1,21.566,13.534Z"/><path d="M12,7a5,5,0,1,0,5,5A5.006,5.006,0,0,0,12,7Zm0,8a3,3,0,1,1,3-3A3,3,0,0,1,12,15Z"/></svg>
                        <svg
                            :class="{
                                'scale-in-bl': showPassword,
                                'scale-out-bl': !showPassword
                            }"
                            class="w-full h-full absolute top-0" xmlns="http://www.w3.org/2000/svg" width="100" height="100" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="20" y1="4" x2="4" y2="20"></line></svg>
                    </div>
                </div>
            </div>
        </div>
        <VCInput v-if="type === 'PASSWORDCONFIRM'" class="mt-2" :placeholder="placeholder2" type="password" v-model="inputPasswordConfirm" :validation="passwordConfirmValidation" :error="passwordConfirmError" :originalpassword="input" />
    </div>

</template>

<style scoped>
.scale-in-ver-bottom {
    animation: scale-in-ver-bottom 0.5s cubic-bezier(0.250, 0.460, 0.450, 0.940) both;
}

.scale-out-ver-bottom {
    animation: scale-out-ver-bottom 0.3s cubic-bezier(0.550, 0.085, 0.680, 0.530) both;
}

.shake-horizontal {
	animation: shake-horizontal 0.8s cubic-bezier(0.455, 0.030, 0.515, 0.955) both;
}

.fade-out-delayed {
    animation: fade-out-delayed 1s cubic-bezier(0.250, 0.460, 0.450, 0.940) both;
}

.fade-in {
    animation: fade-in 0.3s cubic-bezier(0.390, 0.575, 0.565, 1.000) both;
}

.fade-out {
    animation: fade-out 0.3s cubic-bezier(0.390, 0.575, 0.565, 1.000) both;
}

.scale-in-bl {
	animation: scale-in-bl 0.1s cubic-bezier(0.250, 0.460, 0.450, 0.940) both;
}

.scale-out-bl {
	animation: scale-out-bl 0.1s cubic-bezier(0.550, 0.085, 0.680, 0.530) both;
}

@keyframes scale-out-bl {
  0% {
    transform: scale(1);
    transform-origin: 0% 100%;
    opacity: 1;
  }
  100% {
    transform: scale(0);
    transform-origin: 0% 100%;
    opacity: 1;
  }
}


@keyframes scale-in-bl {
  0% {
    transform: scale(0);
    transform-origin: 0% 100%;
    opacity: 1;
  }
  100% {
    transform: scale(1);
    transform-origin: 0% 100%;
    opacity: 1;
  }
}

@keyframes fade-in {
    0% {
        opacity: 0;
    }

    100% {
        opacity: 1;
    }
}

@keyframes fade-out {
    0% {
        opacity: 1;
    }

    100% {
        opacity: 0;
    }
}

@keyframes fade-out-delayed {
    0% {
        opacity: 1;
    }

    70% {
        opacity: 1;
    }

    100% {
        opacity: 0;
    }
}

@keyframes scale-in-ver-bottom {
    0% {
        height: 0px;
        transform: scaleY(0);
        transform-origin: 0% 100%;
        padding-block: 0;
        opacity: 1;
    }

    100% {
        height: 36px;
        transform: scaleY(1);
        transform-origin: 0% 100%;
        padding-block: 8px;
        opacity: 1;
    }
}


@keyframes scale-out-ver-bottom {
    0% {
        height: 36px;
        transform: scaleY(1);
        transform-origin: 0% 100%;
        padding-block: 8px;
        opacity: 1;
    }

    100% {
        height: 0px;
        transform: scaleY(0);
        transform-origin: 0% 100%;
        padding-block: 0;
        opacity: 1;
    }
}

@keyframes shake-horizontal {
  0%,
  100% {
    transform: translateX(0);
  }
  10%,
  30%,
  50%,
  70% {
    transform: translateX(-3.5px);
  }
  20%,
  40%,
  60% {
    transform: translateX(3px);
  }
  80% {
    transform: translateX(2px);
  }
  90% {
    transform: translateX(-2px);
  }
}
</style>