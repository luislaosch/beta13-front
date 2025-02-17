<template>
  <q-layout view="hHh LpR lFf">
    <q-header elevated>
      <q-toolbar style="color: #000">
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />
        <q-img src="~/assets/idat.png" width="50px" />
        <q-toolbar-title></q-toolbar-title>

        <q-icon name="account_circle"></q-icon>
        <div class="q-ml-sm">{{ user.username }}</div>
        <q-btn flat round dense icon="more_vert">
          <q-menu
            transition-show="scale"
            transition-hide="scale"
            auto-close
            :offset="[10, 0]"
          >
            <q-list style="min-width: 150px">
              <q-item @click="logout" clickable>
                <q-item-section>Salir</q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
      </q-toolbar>
    </q-header>

    <q-drawer
      elevated
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      content-class="bg-grey-1"
    >
      <q-scroll-area style="height: calc(100% - 150px); margin-top: 150px">
        <NavBar />
      </q-scroll-area>
      <q-img
        v-if="datosLoader"
        class="q-mt-sm absolute-top"
        style="height: 160px"
      >
        <div class="absolute-bottom bg-transparent text-center">
          <q-img width="80px" src="~/assets/idat.png" />

          <div style="color: #000000" class="q-mt-sm text-weight-bold">
            {{ user.username }}
          </div>
          <div style="color: #000000" class="text-weight-bold">
            {{ user.email }}
          </div>

          <q-separator class="q-mt-xs" size="1px" />
        </div>
      </q-img>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs } from "vue";

import { useQuasar } from "quasar";

import NavBar from "src/components/NavBar.vue";
import { api } from "boot/axios";
import { useRouter } from "vue-router";

export default defineComponent({
  name: "MainLayout",

  components: {
    NavBar,
  },

  setup() {
    const $q = useQuasar();
    const $r = useRouter();

    const data = reactive({
      leftDrawerOpen: false,
      datosLoader: false,
      user: null,
    });

    data.user = $q.localStorage.getItem("user");

    data.datosLoader = true;

    const logout = async () => {
      $q.loading.show({
        message: "Cerrando sesión...",
      });
      let config = {
        headers: {
          "x-access-token": $q.localStorage.getItem("token") || "",
          "Content-Type": "application/json",
        },
      };
      await api
        .get("auth/logout", config)
        .then(() => {
          $q.localStorage.clear();
          $r.push("/");
        })
        .catch(() => {
          $q.notify({
            type: "negative",
            message: "Error, inténtelo mas tarde.",
          });
          $r.push("/dashboard");
        })
        .finally(() => {
          $q.loading.hide();
        });
    };

    return { ...toRefs(data), logout };
  },
});
</script>
