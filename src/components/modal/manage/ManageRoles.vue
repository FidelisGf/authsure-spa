<template>
  <ModalBase :isOpen="dialog" :title="'Registro de Reino'" :needsClose="false">
    <v-card>
      <v-card-title>
        <span class="text-h6"> Registro de Cargo </span>
      </v-card-title>
      <v-card-subtitle>
        <span>Realize o registro de cargo.</span>
      </v-card-subtitle>
      <v-card-text>
        <v-form ref="form" @submit.prevent="save">
          <v-row>
            <v-col :cols="'12'">
              <v-text-field
                :rules="rolesRules.required"
                :placeholder="'Nome'"
                :label="'Nome'"
                required
                variant="underlined"
                v-model="role.name"
              >
              </v-text-field>
            </v-col>
          </v-row>
          <v-card-actions>
            <v-row>
              <v-col cols="12" class="d-flex justify-end">
                <v-btn variant="text" type="submit"> Salvar </v-btn>
                <v-btn variant="text" @click="closeDialog"> Fechar </v-btn>
              </v-col>
            </v-row>
          </v-card-actions>
        </v-form>
      </v-card-text>
    </v-card>
  </ModalBase>
</template>

<script setup>
import ModalBase from "@/components/modal/ModalBase.vue";
import roleComp from "@/compositionAPI/roleComp";

const { role, sendPayload, appStore } = roleComp();
import { ref, onMounted } from "vue";
const form = ref(null);

const props = defineProps({
  dialog: Boolean,
  object: Object,
});

const rolesRules = {
  required: [(v) => !!v || "Campo obrigatorio"],
};

onMounted(() => {
  if (props.object) {
    role.value = { ...props.object };
  }
});

const emit = defineEmits("close");

function closeDialog(e) {
  emit("close");
}
async function save() {
  try {
    const validated = await form.value.validate();
    if (validated.valid) {
      sendPayload(props.object ? true : false);
      closeDialog();
      const action = props.object ? "alterado" : "registrado";
      appStore.changeDialog({
        color: "green",
        message: `Cargo ${action} com sucesso!`,
        show: true,
      });
    } else {
      appStore.changeDialog({
        color: "red",
        message: "Preencha os campos!",
        show: true,
      });
    }
  } catch (error) {
    appStore.changeDialog({
      color: "red",
      message: error,
      show: true,
    });
    console.error(error);
  }
}
</script>

<style lang="scss" scoped></style>
