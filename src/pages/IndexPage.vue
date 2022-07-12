<template>
  <q-page padding class="fondo">
    <!-- Sección donde se muestra la selección de la plataforma de juegos -->
    <!-- Nota: en html5 <section></section> es equivalente a un <div> (https://developer.mozilla.org/es/docs/Web/HTML/Element/section) -->
    <section class="q-pa-md cabecera">
      <p class="text-h6">
        Selecciona una plataforma para mostrar los juegos gratuitos disponibles:
      </p>
      <!-- Componente predefinido en Quasar que muestra las propiedades establecidas en la variable "platforms". El valor seleccionado se guarda en la variable "platform" -->
      <q-select
        color="$primary"
        square
        filled
        v-model="platform"
        :options="platforms"
        option-value="id"
        option-label="desc"
        class="juegos"
      >
        <template v-slot:prepend>
          <q-icon name="search" class="q-pl-sm" />
        </template>
      </q-select>
    </section>

    <!-- Sección que muestra el número de juegos encontrados -->
    <section class="juegos-encontrados">
      <p v-if="data.length > 0" class="q-pt-xs text-center">
        {{ data.length }} juegos encontrados.
      </p>
      <p v-else class="q-pt-xs text-center">
        Selecciona una plataforma para buscar los juegos.
      </p>
    </section>

    <!-- Sección donde se carga el listado de juegos disponibles -->
    <section v-if="data.length > 0" class="q-mt-xl section-style row">
      <!-- Nota 1: <article></article> es equivalente a un <div> https://developer.mozilla.org/es/docs/Web/HTML/Element/article -->
      <!-- Nota 2: Bucle FOR directamente en HTML con Vue.js (https://es.vuejs.org/v2/guide/list.html) -->
      <!-- Ejemplo del contenido de un item que devuelve el API:<template>
        {
          "id": 521,
          "title": "Diablo Immortal",
          "thumbnail": "https://www.freetogame.com/g/521/thumbnail.jpg",
          "short_description": "Built for mobile and also released on PC, Diablo Immortal fills in the gaps between Diablo II and III in an MMOARPG environment.",
          "game_url": "https://www.freetogame.com/open/diablo-immortal",
          "genre": "MMOARPG",
          "platform": "PC (Windows)",
          "publisher": "Blizzard Entertainment",
          "developer": "Blizzard Entertainment",
          "release_date": "2022-06-02",
          "freetogame_profile_url": "https://www.freetogame.com/diablo-immortal"
        }
      -->
      <div
        class="card-wrapper col-xs-12 col-sm-6 col-md-4 col-xl-3"
        v-for="item in data"
        :key="item.id"
      >
        <q-card padding class="my-card card">
          <img :src="item.thumbnail" class="imagen-card" />
          <q-card-section>
            <div class="text-h6 q-mb-xs text-white">{{ item.title }}</div>
            <div class="text-caption text-grey">
              {{ item.short_description }}
            </div>
            <div class="q-pt-md row no-wrap items-center q-gutter-sm">
              <q-badge outline color="color-font">{{ item.genre }}</q-badge>
              <q-badge outline color="accent">{{ item.platform }}</q-badge>
            </div>
          </q-card-section>
        </q-card>
      </div>

      <q-inner-loading :showing="loading">
        <q-spinner-gears size="50px" color="primary" />
      </q-inner-loading>
    </section>
  </q-page>
</template>

<script>
import { defineComponent, ref, watch } from "vue";
import { api } from "boot/axios";
import { useQuasar } from "quasar";
export default defineComponent({
  name: "IndexPage",
  setup() {
    const $q = useQuasar();
    const platform = ref(null);
    const data = ref([]);
    const loading = ref(false);
    const loadData = () => {
      loading.value = true;
      const params = new URLSearchParams([
        ["platform", platform.value.id],
        ["sort-by", "alphabetical"],
      ]);
      api
        .get("/api/games", {
          params,
          headers: {
            "X-RapidAPI-Key":
              "67d9436ce8msh5a7f6cb557501a3p1efcadjsn7e754104fc3a",
            "X-RapidAPI-Host": "free-to-play-games-database.p.rapidapi.com",
          },
        })
        .then((response) => {
          data.value = response.data;
          loading.value = false;
        })
        .catch(() => {
          $q.notify({
            color: "negative",
            position: "top",
            message: "Error al recuperar datos",
            icon: "report_problem",
          });
          loading.value = false;
        });
      console.log(data);
    };
    watch(platform, (newvalue, oldvalue) => {
      //console.log('filter by ', newvalue)
      loadData();
    });
    return {
      data,
      platform,
      platforms: [
        { id: "all", desc: "Mostrar todos" },
        { id: "pc", desc: "Ver juegos para PC" },
        { id: "browser", desc: "Ver juegos para Navegador web" },
      ],
      loading,
    };
  },
});
</script>

<style lang="scss">
.cabecera {
  border: 1px solid #dedede;
  background-color: #20202e;
  color: #eff2f3;
}

.fondo {
  color: #eff2f3;
}

.juegos-encontrados {
  margin-top: 8px;
}

.juegos {
  background-color: #38384e;
}

.card-wrapper {
  padding: 0.5rem;
}
.card {
  background-color: #92888831;
  border-radius: 0px;
}

.imagen-card {
  box-shadow: 0 0 #ecb073;
  transition: 0.25s;
  max-width: 100vw;
}

.imagen-card:hover {
  box-shadow: -5px 5px #ecb073, -4.5px 4.5px #ecb073, -4px 4px #ecb073,
    -3px 3px #ecb073, -2.5px 2.5px #ecb073, -1px 1px #ecb073,
    -0.5px 0.5px #ecb073;
  transform: translate(5px, -5px);
}
</style>
