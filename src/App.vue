<script setup>
import ComponentShowcase from '@/components/ComponentShowcase.vue'
import VCInput from '@/components/VCInput.vue'
import { ref, watch } from 'vue'


const error = ref(null);
const input = ref(null);

</script>

<template>
  <div class="h-screen bg-teal-300 flex flex-col p-10 gap-5">
    <h1 class="text-teal-900 font-bold text-3xl text-center">Vue Components collection</h1>
    {{ input }}
    <ComponentShowcase title="VCInput confirmar contraseña">
      <VCInput :error="error" v-model="input" placeholder="Contraseña" placeholder2="Confirmar contraseña" type="passwordconfirm" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput contraseña">
      <VCInput :error="error" v-model="input" placeholder="Contraseña" type="password" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput E-Mail">
      <VCInput :error="error" v-model="input" placeholder="Correo electrónico" type="email" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput IBAN">
      <VCInput :error="error" v-model="input" placeholder="IBAN" type="iban" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput DNI/NIE">
      <VCInput :error="error" v-model="input" placeholder="Número de identidad (DNI/NIE)" type="dninie" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput NIE">
      <VCInput :error="error" v-model="input" placeholder="Documento de identidad (NIE)" type="nie" />
    </ComponentShowcase>
    <ComponentShowcase title="VCInput DNI">
      <VCInput :error="error" v-model="input" placeholder="Documento de identidad (DNI)" type="dni" />
    </ComponentShowcase>
  </div>

</template>