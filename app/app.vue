<script setup lang="ts">
import { DrawerContent, DrawerOverlay, DrawerPortal, DrawerRoot, DrawerTrigger } from 'vaul-vue'
</script>

<template>
  <DrawerRoot :default-open="true">
    <DrawerTrigger>Open</DrawerTrigger>
    <DrawerPortal force-mount>
      <DrawerOverlay />
      <DrawerContent>Hello from vaul-vue</DrawerContent>
    </DrawerPortal>
  </DrawerRoot>
</template>
