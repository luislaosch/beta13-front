<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar style="color: #000000">
        <q-img src="~/assets/idat.png" width="50px" />
        <q-toolbar-title>Beta13</q-toolbar-title>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "LoginLayout",
  components: {},
  setup() {
    const reload = () => {
      location.reload(true);
    };
    return { reload };
  },
});
</script>
