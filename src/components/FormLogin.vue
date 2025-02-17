<template>
  <q-form @submit="onSubmit" class="q-gutter-md">
    <q-card-section>
      <q-input
        dense
        v-model="email"
        label="Correo"
        lazy-rules
        :rules="[
          (val) => (val && val.length >= 0) || 'El campo debe ser llenado.',
          (val) =>
            /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/.test(val) ||
            'El correo no es valido.',
        ]"
        counter
        maxlength="50"
      ></q-input>
      <q-input
        dense
        class="q-mt-md"
        label="Contraseña"
        v-model="password"
        :type="isPwd ? 'password' : 'text'"
        lazy-rules
        :rules="[
          (val) => (val && val.length >= 0) || 'El campo debe ser llenado.',
        ]"
        counter
        maxlength="50"
      >
        <template v-slot:append>
          <q-icon
            :name="isPwd ? 'visibility_off' : 'visibility'"
            class="cursor-pointer"
            color="secondary"
            @click="isPwd = !isPwd"
          /> </template
      ></q-input>
    </q-card-section>
    <q-card-section>
      <q-btn
        type="submit"
        style="border-radius: 8px"
        color="secondary"
        rounded
        size="md"
        label="Ingresar"
        no-caps
        class="full-width"
      ></q-btn>
      <q-btn
        style="border-radius: 8px"
        color="secondary"
        rounded
        size="md"
        label="Registrar"
        no-caps
        class="q-mt-md full-width"
      ></q-btn>
    </q-card-section>
  </q-form>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs } from "vue";

import { useQuasar } from "quasar";
import { useRouter } from "vue-router";
import { api } from "boot/axios";
export default defineComponent({
  name: "FormLogin",
  setup() {
    const $r = useRouter();
    const $q = useQuasar();

    const data = reactive({
      email: null,
      password: null,
      isPwd: true,
    });

    const onSubmit = async () => {
      $q.loading.show({
        message: "Espere por favor...",
      });

      let config = {
        headers: {
          "Content-Type": "application/json",
        },
      };

      let datos = {
        email: data.email,
        password: data.password,
      };

      await api
        .post("auth/signin", datos, config)
        .then(async (response) => {
          $q.localStorage.set("token", response.data.token);
          $q.localStorage.set("user", response.data.user);

          // let config2 = {
          //   headers: {
          //     Authorization: "Bearer " + response.data.access_token,
          //     "Content-Type": "application/json",
          //   },
          // };

          // await api
          //   .get("data", config2)
          //   .then((response) => {
          //     $q.localStorage.set("datos", response.data.datos);
          //     $q.localStorage.set("permisos", response.data.permisos);
          //     $q.localStorage.set("menu", JSON.stringify(response.data.menu));
          //     $q.localStorage.set("submenus", response.data.submenus);
          //     $q.localStorage.set("role", response.data.role);
          //   })
          //   .catch((error) => {
          //     if (error.response.data.message) {
          //       $q.notify({
          //         position: "top",
          //         type: "negative",
          //         message: error.response.data.message,
          //       });
          //     } else {
          //       $q.notify({
          //         position: "top",
          //         type: "negative",
          //         message: "Upps, inténtelo mas tarde.",
          //       });
          //     }
          //   });

          // // $r.push("/register/covid");
          $r.push("/dashboard");
        })
        .catch((error) => {
          if (error.response.data.message) {
            $q.notify({
              position: "top",
              type: "negative",
              message: error.response.data.message,
            });
          } else {
            $q.notify({
              position: "top",
              type: "negative",
              message: "Upps, inténtelo mas tarde.",
            });
          }
        })
        .finally(() => {
          $q.loading.hide();
        });
    };

    return { ...toRefs(data), onSubmit };
  },
});
</script>
