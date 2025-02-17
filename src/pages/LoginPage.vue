<template>
  <q-page class="flex flex-center bg-grey-4">
    <q-card class="q-pa-md shadow-2" bordered>
      <q-card-section class="text-center">
        <q-img src="~assets/idat.png" width="300px"></q-img>
      </q-card-section>
      <FormLogin />
    </q-card>
  </q-page>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { useQuasar } from "quasar";
import { useRouter } from "vue-router";
import FormLogin from "components/FormLogin.vue";
export default defineComponent({
  name: "LoginPage",
  components: { FormLogin },
  setup() {
    const $r = useRouter();
    const $q = useQuasar();

    // if ($q.localStorage.getItem('token')) {
    //   $r.push('/dashboard');
    // }

    return {};
  },
});
</script>
