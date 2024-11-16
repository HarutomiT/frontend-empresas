<template>
  <div>
    <view-document
      v-if="showDocumentMenu"
      :show="showDocumentMenu"
      :document="doc"
      @handleCloseDocumentMenu="handleCloseDocumentMenu"
    />
    <div v-if="!isLoading" class="q-mx-auto" style="max-width: 1000px">
      <q-img
        :src="api_base_backend + empresa.image"
        alt="Esta empresa no posee imagen"
        style="height: 350px"
        :fit="cover"
      >
        <template v-slot:error>
          <div class="absolute-full text-subtitle2 flex flex-center">
            <h4 class="text-h4">
              {{ empresa.nombre }}
            </h4>
          </div>
        </template>
        <div class="absolute-full text-subtitle2 flex flex-center">
          <h4 class="text-h4">
            {{ empresa.nombre }}
          </h4>
        </div>
      </q-img>
      <div class="row justify-between">
        <div class="text-caption">
          <div class="text-grey row items-center">
            {{ empresa.is_valid ? "Verificado" : "No verificado" }}
            <q-icon
              :name="empresa.is_valid ? 'check_circle' : 'cancel'"
              :color="empresa.is_valid ? 'green' : 'red'"
              size="30px"
            />
            <q-card-section class="q-pt-none">
              <div class="flex justify-left">
                <q-btn
                  label="Editar"
                  class="q-mt-md q-mr-sm"
                  type="button"
                  color="primary"
                  @click="handleOpenMenuEmpresa"
                />
                <menu-edit-empresa
                  v-if="menuEmpresa"
                  :empresa="empresa"
                  :show="menuEmpresa"
                  @handleCloseMenuEmpresa="handleCloseMenuEmpresa"
                />
              </div>
            </q-card-section>
          </div>
        </div>
        <div style="width: 400px" v-if="empresa.user">
          <h4 class="text-h5 q-my-none">ENCARGADO</h4>
          <q-card>
            <q-card-section> Nombre: {{ empresa.user.name }} </q-card-section>
            <q-card-section> Email: {{ empresa.user.email }} </q-card-section>
          </q-card>
        </div>
      </div>
      <h4 class="text-h5 q-my-none">OPERADORES</h4>
      <div class="">
        <q-table
          class="my-sticky-column-table"
          style="height: 400px; width: 100%"
          flat
          bordered
          :rows="operators"
          :columns="columnOperators"
          row-key="id"
          @row-click="onRowClick"
          virtual-scroll
        />
      </div>

      <div style="width: 100%; height: 100vh" class="q-mt-lg">
        <h4 class="text-h4 q-my-none">Documentos</h4>
        <q-markup-table flat bordered>
          <thead class="bg-teal text-white">
            <tr>
              <th class="text-left">Título</th>
              <th class="text-left">Vencimiento</th>
              <th class="text-left">Autorización</th>
              <th class="text-left">empresa</th>
            </tr>
          </thead>
          <tbody :class="$q.dark.isActive ? 'bg-grey-9' : 'bg-grey-3'">
            <tr
              v-for="(document, index) in documents"
              :key="index"
              class="cursor-pointer"
              @click="() => handleOpenDocumentMenu(document.id)"
            >
              <td class="text-left">
                {{ document.title }}
              </td>
              <td class="text-left">
                {{ document.expire }}
              </td>
              <td class="text-left">
                <p :class="document.is_valid ? 'text-green' : 'text-red'">
                  {{ document.is_valid ? "Autorizado" : "No Autorizado" }}
                </p>
              </td>
              <td class="text-left">
                {{ document.enterprise }}
              </td>
            </tr>
          </tbody>
        </q-markup-table>
      </div>
    </div>

    <div v-if="isLoading" class="text-center">Cargando...</div>
    <div v-if="empresaNoExiste" class="row justify-center">
      <h4 class="text-h4 column">
        No existe esta empresa
        <q-icon name="warning" size="50px" color="warning" />
      </h4>
    </div>
  </div>
</template>

<script>
import MenuEditEmpresa from "src/components/MenuEditEmpresa.vue";
import { useRouter } from "vue-router";
import { api } from "src/boot/axios";
import { ref } from "vue";
import { api_base_backend } from "../helpers.js";
import ViewDocument from "../components/ViewDocument.vue";
import { useUserStore } from "src/store/user.store.js";

export default {
  components: {
    MenuEditEmpresa,
    ViewDocument,
  },
  setup() {
   
    const router = useRouter();

    const userStore = useUserStore();
    const user = userStore.getUser
    console.log(user)
    const { slug } = user.enterprise

    const isLoading = ref(true);
    const empresa = ref(null);
    const operators = ref(null);
    const documents = ref(null);

    const empresaNoExiste = ref(false);

    const menuEmpresa = ref(false);

    const handleValidEnterprise = () => {
      api
        .patch(`enterprises/${user.enterprise.slug}`, {
          is_valid: true,
        })
        .then((response) => {
          if (response.status === 200) {
            router.push("/home");
          }
        });
    };

    const handleDesvalidEnterprise = () => {
      api.delete(`enterprises/${user.enterprise.slug}`).then((response) => {
        if (response.status === 200) {

          router.push("/home");
        }
      });
    };

    const columnOperators = [
      { name: "ci", label: "C.I.", field: "ci", align: "left" },
      { name: "name", label: "Nombre", field: "name", align: "left" },
      {
        name: "is_valid",
        label: "Autorización",
        field: "is_valid",
        align: "left",
      },
      {
        name: "cargo",
        label: "Cargo",
        field: "cargo",
        align: "left",
      },
    ];

    api
      .get(`enterprises/${user.enterprise.slug}`)
      .then(async (response) => {
        empresa.value = response.data.enterprise;

        if (response.status === 200) {
          await api
            .get(`enterprises/${slug}/operators`)
            .then((response) => {
              operators.value = response.data.operators;
            });
          await api
            .get(`/${params.slug}/documents`)
            .then((response) => {
              documents.value = response.data.documents;
            });
        }
      })
      .catch((err) => {
        isLoading.value = false;
        if (err?.response?.status === 404) {
          empresaNoExiste.value = true;
        }
      })
      .finally(() => {
        isLoading.value = false;
      });

    const handleOpenMenuEmpresa = () => (menuEmpresa.value = true);
    const handleCloseMenuEmpresa = () => {
      menuEmpresa.value = false;
    };

    const onRowClick = (e, item) => {
      router.push({
        name: "operators-detail",
        params: {
          pk: item.id,
          enterprise: params.slug,
        },
      });
    };

    const doc = ref(null);
    const showDocumentMenu = ref(false);

    const handleOpenDocumentMenu = (pk) => {
      doc.value = pk;
      showDocumentMenu.value = true;
    };
    const handleCloseDocumentMenu = () => {
      showDocumentMenu.value = false;
    };

    return {
      doc,
      showDocumentMenu,
      handleOpenDocumentMenu,
      handleCloseDocumentMenu,
      isLoading,
      empresa,
      empresaNoExiste,
      api_base_backend,
      columnOperators,
      operators,
      handleValidEnterprise,
      onRowClick,
      handleCloseMenuEmpresa,
      handleOpenMenuEmpresa,
      handleDesvalidEnterprise,
      menuEmpresa,
      documents,
      pagination: ref({
        rowsPerPage: 0,
      }),
    };
  },
};
</script>
