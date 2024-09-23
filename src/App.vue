<script setup>
import ComponentShowcase from '@/components/ComponentShowcase.vue'
import VCInput from '@/components/VCInput.vue'
import VCSelect from '@/components/VCSelect.vue'
import VCTelf from '@/components/VCTelf.vue'
import VCTime from '@/components/VCTime.vue'
import VCDate from './components/VCDate.vue'
import { ref, watch } from 'vue'


const error = ref(null);
const input1 = ref("");
const input2 = ref("");
const input3 = ref("");
const input4 = ref("");
const input5 = ref("");
const input6 = ref("");
const input7 = ref("");
const input8 = ref("");
const input9 = ref("");
const input10 = ref("");
const input11 = ref("");

</script>

<template>
  <div class="bg-teal-300 flex flex-col p-10 gap-5 w-screen">
    <h1 class="text-teal-900 font-bold text-3xl text-center">Vue Components collection</h1>
    <ComponentShowcase title="VCDate" :input="input11">
      <VCDate v-model="input11" />
    </ComponentShowcase>
    <ComponentShowcase title="VCTime" :input="input10">
      <VCTime v-model="input10" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput confirmar contraseña" :input="input1">
      <VCInput :error="error" v-model="input1" placeholder="Contraseña" placeholder2="Confirmar contraseña" type="passwordconfirm" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput contraseña" :input="input2">
      <VCInput :error="error" v-model="input2" placeholder="Contraseña" type="password" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput E-Mail" :input="input3">
      <VCInput :error="error" v-model="input3" placeholder="Correo electrónico" type="email" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput teléfono" :input="input9">
      <VCTelf v-model="input9" placeholder="Teléfono" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput IBAN" :input="input4">
      <VCInput :error="error" v-model="input4" placeholder="IBAN" type="iban" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput DNI/NIE" :input="input5">
      <VCInput :error="error" v-model="input5" placeholder="Número de identidad (DNI/NIE)" type="dninie" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput NIE" :input="input6">
      <VCInput :error="error" v-model="inpu6t" placeholder="Documento de identidad (NIE)" type="nie" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput DNI" :input="input7">
      <VCInput :error="error" v-model="input7" placeholder="Documento de identidad (DNI)" type="dni" />
    </ComponentShowcase>
    <ComponentShowcase title="VCSelect" :input="input8">
      <VCSelect v-model="input8" :options="{ 1: 'Opción 1', 2: 'Opción 2', 3: 'Opción 3' }" placeholder="Selecciona una opción" />
    </ComponentShowcase>
  </div>

</template>