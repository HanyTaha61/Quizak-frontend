<template>
    <StickyAd />

    <v-app>
      <v-main class="pa-0 ">
        <slot />
      </v-main>
    </v-app>
</template>

<script setup>
import StickyAd from "@/components/StickyAd.vue";
</script>