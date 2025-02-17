<template>
  <q-list class="margin-top: 150px;">
    <q-item clickable to="/dashboard">
      <q-item-section avatar>
        <q-icon name="home" class="text-black" />
      </q-item-section>

      <q-item-section>
        <q-item-label class="text-black">Inicio</q-item-label>
      </q-item-section>
    </q-item>
  </q-list>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs } from "vue";
import { useQuasar } from "quasar";

export default defineComponent({
  name: "NavBar",

  setup() {
    const $q = useQuasar();

    const data = reactive({});

    return { ...toRefs(data) };
  },
});
</script>
