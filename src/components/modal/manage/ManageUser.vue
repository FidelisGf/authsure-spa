<template>
  <ModalBase :isOpen="dialog" :title="'Registro de Reino'" :needsClose="false">
    <v-card>
      <v-card-title>
        <span class="text-h6"> Registro de Usuario </span>
      </v-card-title>
      <v-card-subtitle>
        <span>Realize o registro de usuario.</span>
      </v-card-subtitle>
      <v-card-text>
        <v-form ref="form" @submit.prevent="save">
          <v-row>
            <v-col :cols="'6'">
              <v-text-field
                :rules="userRules.required"
                :placeholder="'Nome'"
                :label="'Nome'"
                required
                variant="underlined"
                v-model="user.username"
              >
              </v-text-field>
            </v-col>
            <v-col :cols="'6'">
              <v-text-field
                :rules="userRules.password"
                :placeholder="'Senha'"
                :label="'Senha'"
                :type="'password'"
                required
                variant="underlined"
                counter
                :min="'8'"
                :max="'255'"
                v-model="user.password"
              >
              </v-text-field>
            </v-col>
            <v-col :cols="'6'">
              <v-text-field
                :rules="userRules.emailRules"
                :placeholder="'Email'"
                :label="'Email'"
                required
                type="email"
                variant="underlined"
                v-model="user.email"
              >
              </v-text-field>
            </v-col>
            <v-col cols="6">
              <v-select
                item-title="name"
                item-value="id"
                return-object
                :rules="userRules.required"
                v-model="user.realm_id"
                :label="'Id reino'"
                :items="realms"
                variant="underlined"
              >
              </v-select>
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
import userComp from "@/compositionAPI/userComp";

const { user, sendPayload, appStore, realms, fetchRealms } = userComp();
import { ref, onMounted, watch } from "vue";
const form = ref(null);

const props = defineProps({
  dialog: Boolean,
  object: Object,
});

watch(realms, (nw, old) => {
  if (nw && nw.length > 0) {
    if (user.value.realm_id)
      user.value.realm_id = realms.value.find(
        (i) => i.id == user.value.realm_id
      );
  }
});

const userRules = {
  required: [(v) => !!v || "Campo obrigatorio"],
  password: [
    (v) => !!v || "Campo obrigatorio",
    (v) => v.length >= 8 || "Senha mínima deve possuir 8 caracteres",
  ],
  emailRules: [
    (v) => !!v || "Campo obrigatorio",
    (v) =>
      !v ||
      /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) ||
      "E-mail inválido",
  ],
};

onMounted(() => {
  fetchRealms();
  if (props.object) {
    user.value = { ...props.object };
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
