<template>
  <div class="q-form-container">
    <!-- Header with Actions -->
    <div class="row items-center justify-between q-mb-lg">
      <div class="text-subtitle1 text-weight-bold text-slate-700">
        {{ formData?.id ? 'Visualização de Registro' : 'Novo Registro' }}
      </div>

      <div class="row items-center q-gutter-x-sm">
        <q-btn v-if="formData?.id" color="slate-400" flat round icon="more_vert" size="sm">
          <q-menu anchor="bottom right" self="top right" auto-close class="glass-card shadow-lg">
            <q-list min-width="150px">
              <q-item v-if="formStore.isDisable" clickable @click="setEdit()">
                <q-item-section avatar>
                  <q-icon name="edit_note" size="xs" />
                </q-item-section>
                <q-item-section>Editar</q-item-section>
              </q-item>
              <q-item clickable @click="showConfirmMessage = true" class="text-rose-600">
                <q-item-section avatar>
                  <q-icon name="delete_outline" size="xs" />
                </q-item-section>
                <q-item-section>Excluir</q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
      </div>
    </div>

    <!-- Main Form Component -->
    <q-form class="column q-gutter-y-md" @submit.prevent="onSubmit()">
      <div class="form-content">
        <component :is="dynamicComponent" :model="formData"></component>
      </div>

      <!-- Action Footer -->
      <div class="row items-center justify-end q-mt-xl q-gutter-x-md sticky-footer">
        <template v-if="!formStore.isDisable">
          <q-btn label="Cancelar" flat color="slate-500" @click="btnCancel()" class="rounded-lg" />
          <q-btn label="Salvar Alterações" type="submit" unelevated color="primary" class="rounded-lg q-px-lg" />
        </template>
      </div>
    </q-form>

    <!-- Information Accordion (Optional Meta Data) -->
    <div class="q-mt-xl opacity-80">
      <information-expanse :model="formData"></information-expanse>
    </div>

    <!-- Confirm Delete -->
    <q-dialog v-model="showConfirmMessage" persistent>
      <q-card flat class="glass-card q-pa-md border-slate-200 shadow-2xl overflow-hidden" style="width: 400px">
        <q-card-section class="column items-center text-center q-py-lg">
          <div class="bg-rose-50 q-pa-md rounded-full q-mb-md">
            <q-icon name="warning" color="rose-600" size="md" />
          </div>
          <div class="text-h6 text-weight-bold text-slate-800">Confirmar Exclusão</div>
          <div class="text-body2 text-slate-500 q-mt-sm">
            Tem certeza que deseja excluir este registro? Esta ação não pode ser desfeita.
          </div>
        </q-card-section>

        <q-card-actions
        align="center"
        class="q-gutter-x-md q-pb-md"
        >
          <q-btn flat label="Cancelar" color="slate-600" class="rounded-lg" v-close-popup />
          <q-btn unelevated label="Sim, Excluir" color="rose-600" class="rounded-lg q-px-md" @click="requestDelete"
            v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script lang="ts">
import defaultService from 'src/api/default';
import { useFormStore } from 'stores/form';
import { computed, ref, defineComponent, defineAsyncComponent } from 'vue';
import InformationExpanse from 'components/forms/informationExpanse.vue';

export default defineComponent({
  name: 'defaultForm',
  components: { InformationExpanse },
  props: {
    model: { type: Object, required: true },
    isEdit: { type: Boolean, default: false },
    routeApi: { type: String, required: true },
    component: { type: String, required: true }
  },
  emits: ['saved', 'deleted', 'cancelled'],
  setup(props, { emit }) {
    const formStore = useFormStore();
    const { update, create, remove } = defaultService();

    // Maintain full reactivity with the parent model
    const formData = computed(() => props.model);

    // Dynamic component loading based on props.component
    const dynamicComponent = computed(() => {
      return defineAsyncComponent(() => import(/* @vite-ignore */  `./${props.component}.vue`));
    });

    const showConfirmMessage = ref(false);

    const closeSideBar = () => {
      emit('cancelled');
    };

    const setEdit = () => {
      formStore.setIsDisable(false);
    };

    const btnCancel = () => {
      formStore.setIsDisable(true);
      emit('cancelled');
    };

    const handleApi = () => {
      if (formData.value && formData.value.id != undefined) {
        update(
          props.routeApi,
          formData.value.id,
          formData.value,
          (response) => {
            if (response.success) {
              formStore.setIsDisable(true);
              emit('saved', response.data);
            }
          }
        );
      } else {
        create(
          props.routeApi,
          formData.value,
          (response) => {
            if (response.success) {
              formStore.setIsDisable(true);
              emit('saved', response.data);
            }
          });
      }
    };

    const onSubmit = () => {
      handleApi();
    };

    const requestDelete = () => {
      if (!formData.value?.id) return;
      remove(
        props.routeApi,
        formData.value.id,
        (response) => {
          if (response.success) {
            emit('deleted', response.data);
            emit('saved', response.data);
          }
        });
    };

    return {
      showConfirmMessage,
      formStore,
      formData,
      dynamicComponent,
      closeSideBar,
      setEdit,
      btnCancel,
      onSubmit,
      requestDelete
    };
  }
});
</script>

<style lang="scss" scoped>
.q-form-container {
  padding: 8px;
}

.sticky-footer {
  position: sticky;
  bottom: 0;
  background: transparent;
  backdrop-filter: blur(8px);
  padding: 16px 0;
  border-top: 1px solid rgba(0, 0, 0, 0.05);
}

.form-content {
  min-height: 200px;
}
</style>