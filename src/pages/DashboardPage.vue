<template>
  <q-page>
    <div class="row">
      <!-- Lista de Productos -->
      <div class="col-12 col-md-8 q-pa-md order-first">
        <h4 class="text-center q-mb-md">Productos Disponibles</h4>
        <div class="row q-gutter-md justify-center">
          <q-card
            v-for="product in products"
            :key="product._id"
            class="my-card col-10 col-sm-6 col-md-4 col-lg-3"
          >
            <img :src="product.imgURL" class="responsive-img" />
            <q-card-section>
              <div class="text-h6">{{ product.name }}</div>
              <div class="text-subtitle2">
                S/.{{ product.price.toLocaleString() }}
              </div>
            </q-card-section>
            <q-card-actions>
              <q-btn color="secondary" flat @click="addToCart(product)">
                Añadir al carrito
              </q-btn>
            </q-card-actions>
          </q-card>
        </div>
      </div>

      <!-- Carrito de Compras -->
      <div class="col-12 col-md-4 q-pa-md bg-grey-3 order-md-last">
        <h4 class="text-center q-mb-md">
          Tu Carrito ({{ cart.items.length }})
        </h4>

        <div v-if="cart.items.length === 0" class="text-grey-6">
          El carrito está vacío
        </div>

        <div v-else>
          <q-list bordered>
            <q-item v-for="(item, index) in cart.items" :key="index">
              <q-item-section>
                <q-item-label>{{ item.product.name }}</q-item-label>
                <q-item-label caption>
                  {{ item.quantity }} x S/.{{
                    item.product.price.toLocaleString()
                  }}
                </q-item-label>
              </q-item-section>

              <q-item-section side>
                <div class="text-h6">
                  S/.{{ (item.quantity * item.product.price).toLocaleString() }}
                </div>
                <q-btn
                  flat
                  round
                  color="secondary"
                  icon="remove"
                  size="sm"
                  class="q-ml-sm"
                  @click="removeFromCart(item)"
                />
              </q-item-section>
            </q-item>
          </q-list>

          <div class="q-mt-md text-h6">
            Total: S/.{{ cart.total.toLocaleString() }}
          </div>

          <q-btn color="secondary" class="q-mt-md full-width" @click="checkout">
            Finalizar Compra
          </q-btn>

          <q-btn
            color="negative"
            class="q-mt-md q-ml-sm full-width"
            @click="clearCart"
          >
            Vaciar Carrito
          </q-btn>
        </div>
      </div>
    </div>

    <!-- Diálogo de pago -->
    <q-dialog v-model="showCheckoutDialog" persistent>
      <q-card style="min-width: 400px">
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h6">Datos de pago</div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup />
        </q-card-section>

        <q-card-section>
          <q-form @submit="processPayment" class="q-gutter-md">
            <q-input
              v-model="paymentData.card_number"
              label="Número de tarjeta"
              mask="#### - #### - #### - ####"
              unmasked-value
              :rules="[(val) => val?.length === 16 || 'Debe tener 16 dígitos']"
              required
            />

            <div class="row q-gutter-sm">
              <q-input
                v-model="paymentData.expiration_month"
                label="Mes Exp."
                mask="##"
                unmasked-value
                class="col"
                :rules="[(val) => (val >= 1 && val <= 12) || 'Mes inválido']"
                required
              />

              <q-input
                v-model="paymentData.expiration_year"
                label="Año Exp."
                mask="####"
                unmasked-value
                class="col"
                :rules="[
                  (val) => val >= new Date().getFullYear() || 'Año inválido',
                ]"
                required
              />

              <q-input
                v-model="paymentData.cvv"
                label="CVV"
                mask="####"
                unmasked-value
                class="col"
                :rules="[
                  (val) =>
                    (val.length >= 3 && val.length <= 4) || 'CVV inválido',
                ]"
                required
              />
            </div>

            <q-input
              v-model="paymentData.email"
              label="Email"
              type="email"
              :rules="[
                (val) => (!!val && /.+@.+\..+/.test(val)) || 'Email inválido',
              ]"
              required
            />

            <q-card-actions align="right">
              <q-btn label="Cancelar" color="negative" v-close-popup />
              <q-btn label="Pagar" type="submit" color="primary" />
            </q-card-actions>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, ref } from "vue";
// import { useChecktokenStore } from 'stores/checktoken';
import { useQuasar } from "quasar";

import { api } from "boot/axios";

export default defineComponent({
  name: "DashboardPage",
  setup() {
    const $q = useQuasar();
    const data = reactive({
      products: [],
      cart: {
        items: [],
        total: 0,
      },
    });

    const paymentData = reactive({
      card_number: "",
      cvv: "",
      expiration_month: "",
      expiration_year: "",
      email: "",
    });

    const showCheckoutDialog = ref(false);

    // const store = useChecktokenStore();

    // store.checkToken();

    const getProducts = async () => {
      $q.loading.show({
        message: "Espere por favor...",
      });

      let config = {
        headers: {
          Authorization: "Bearer " + $q.localStorage.getItem("token"),
          "Content-Type": "application/json",
        },
      };

      await api
        .get("products", config)
        .then((response) => {
          console.log(response);
          data.products = response.data;
        })
        .catch((error) => {
          if (error.response.data.message) {
            $q.notify({
              type: "negative",
              message: error.response.data.message,
            });
          } else {
            $q.notify({
              type: "negative",
              message: "Upps, inténtelo mas tarde.",
            });
          }
        })
        .finally(() => {
          $q.loading.hide();
        });
    };

    const addToCart = (item) => {
      const existingItem = data.cart.items.find(
        (cartItem) => cartItem.product._id === item._id
      );
      if (existingItem) {
        existingItem.quantity++;
      } else {
        data.cart.items.push({ product: item, quantity: 1 });
      }
      calculateTotal();
    };

    const removeFromCart = (item) => {
      const index = data.cart.items.findIndex(
        (cartItem) => cartItem.product._id === item.product._id
      );
      if (index !== -1) {
        if (data.cart.items[index].quantity > 1) {
          data.cart.items[index].quantity--;
        } else {
          data.cart.items.splice(index, 1);
        }
      }
      calculateTotal();
    };

    const calculateTotal = () => {
      data.cart.total = data.cart.items.reduce((acc, item) => {
        return acc + item.product.price * item.quantity;
      }, 0);
    };

    const clearCart = () => {
      data.cart.items = [];
      data.cart.total = 0;
    };

    const checkout = () => {
      if (data.cart.items.length === 0) {
        $q.notify({
          type: "warning",
          message: "El carrito está vacío",
        });
        return;
      }
      showCheckoutDialog.value = true;
    };

    const processPayment = async () => {
      $q.loading.show();

      try {
        const config = {
          headers: {
            Authorization: "Bearer " + $q.localStorage.getItem("token"),
            "Content-Type": "application/json",
          },
        };

        const payload = {
          ...paymentData,
          amount: data.cart.total,
          cart: data.cart.items,
        };

        const response = await api.post("/process/pay", payload, config);

        $q.notify({
          type: "positive",
          message: "Pago procesado exitosamente",
        });

        // Limpiar carrito y formulario
        clearCart();
        Object.assign(paymentData, {
          card_number: "",
          cvv: "",
          expiration_month: "",
          expiration_year: "",
          email: "",
        });
        showCheckoutDialog.value = false;
      } catch (error) {
        let message = "Error procesando el pago";
        if (error.response?.data?.message) {
          message = error.response.data.message;
        }
        $q.notify({ type: "negative", message });
      } finally {
        $q.loading.hide();
      }
    };

    getProducts();

    return {
      ...toRefs(data),
      addToCart,
      removeFromCart,
      clearCart,
      checkout,
      processPayment,
      paymentData,
      showCheckoutDialog,
    };
  },
});
</script>

<style>
.my-card {
  min-width: auto;
  width: 100%;
  margin: 8px;
  transition: transform 0.2s;
}

.responsive-img {
  height: 150px;
  object-fit: cover;
  width: 100%;
}

@media (max-width: 600px) {
  .q-item__section--side {
    flex-direction: row !important;
    align-items: center;
  }

  .q-card__actions {
    flex-wrap: wrap;
  }

  .q-btn {
    margin: 4px 0;
  }
}
</style>
