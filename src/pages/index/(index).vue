<template>
  <section class="form__tela">
    <div class="form__content">
      <h1 class="form__title">Formulário de cadastrado</h1>
      <form @submit.prevent="adicionarNovoUsuario" class="form">
        <q-input
          type="text"
          placeholder="nome"
          v-model="formulario.nome"
          outlined request
        />
        <q-input
          type="email"
          placeholder="e-mail"
          v-model="formulario.email"
          outlined request
        />
        <q-input
          type="password"
          placeholder="senha"
          v-model="formulario.senha"
          outlined request
        />
        <q-btn color="primary" type="submit" label="adicionar" />
      </form>
    </div>
    <ul class="cards">
      <li v-for="dado in dados" :key="dado.id" class="card">
        <div class="card__body">
          <p class="line__nome">{{ dado.nome }}</p>
          <p class="line__email">{{ dado.email }}</p>
        </div>
        <div class="actions">
          <q-btn @click="removerUsuario(dado.id)" icon="delete" color="red" />
          <q-btn @click="editarUsuario(dado.id)" icon="edit" color="orange" />
        </div>
      </li>
    </ul>

    <q-dialog v-model="dialogEdicao">
      <q-card style="min-width: 360px">
        <q-card-section>
          <div class="text-h6">Editar usuário</div>
        </q-card-section>

        <q-card-section>
          <form @submit.prevent="salvarEdicao" class="form">
            <q-input
              type="text"
              placeholder="nome"
              v-model="formEditar.nome"
              outlined request
            />
            <q-input
              type="email"
              placeholder="e-mail"
              v-model="formEditar.email"
              outlined request
            />
            <q-input
              type="password"
              placeholder="senha (deixe em branco para manter)"
              v-model="formEditar.senha"
              outlined
            />

            <div class="actions">
              <q-btn color="green" type="submit" label="salvar edição" />
              <q-btn color="orange" label="cancelar" @click="cancelarEdicao" />
            </div>
          </form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </section>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useQuasar } from "quasar";
const $q = useQuasar();
import axios from "axios";
const dados = ref<any[]>([]);
const dialogEdicao = ref(false);

const formulario = ref({
  nome: "",
  email: "",
  senha: ""
});

const formEditar = ref({
  nome: "",
  email: "",
  senha: ""
});

const url =
  "https://special-spork-6vg5xxpg7x553557r-3000.app.github.dev/usuarios";

async function adicionarNovoUsuario() {
  if (
    !formulario.value.nome.trim() ||
    !formulario.value.email.trim() ||
    !formulario.value.senha.trim()
  ) {
    $q.notify({
      type: "negative",
      message: "Preencha todos os campos!",
      position: "top"
    });
    return;
  }

  const response = await axios.post(url, formulario.value);
  await listarTodosUsuarios();
}

async function listarTodosUsuarios() {
  try {
    const response = await axios.get(url);
    dados.value = response.data;
  } catch (error) {
    console.log("Erro na tentativa de busca", error);
    //alert("houve erro na aplicação, consulte o servidor");
  }
}

async function removerUsuario(id: number) {
  try {
    if (confirm("Deseja apagar?")) {
      dados.value = dados.value.filter(u => u.id !== id);
      await axios.delete(`${url}/${id}`);
    }
  } catch (error) {
    console.log("Erro na tentativa de apagar dados! Veja o servidor!");
    //alert("houve erro na aplicação, consulte o servidor");
  }
}

const editarId = ref<number | null>(null);

async function editarUsuario(id: number) {
  const usuarioExiste = dados.value.find(d => d.id === id);
  if (!usuarioExiste) return;

  editarId.value = id;
  formEditar.value = {
    nome: usuarioExiste.nome,
    email: usuarioExiste.email,
    senha: "" // não copia a senha do backend
  };

  dialogEdicao.value = true;
}

const cancelarEdicao = () => {
  formEditar.value = { nome: "", email: "", senha: "" };
  editarId.value = null;
  dialogEdicao.value = false;
};

async function salvarEdicao() {
  if (editarId.value === null) return;

  const indice = dados.value.findIndex(u => u.id === editarId.value);
  if (indice === -1) return;

  const payload: any = {
    nome: formEditar.value.nome,
    email: formEditar.value.email
  };
  if (formEditar.value.senha.trim()) {
    payload.senha = formEditar.value.senha;
  }

  try {
    await axios.put(`${url}/${editarId.value}`, payload);

    dados.value[indice] = { ...dados.value[indice], ...payload };
    cancelarEdicao(); // já fecha o dialog e limpa o estado
    $q.notify({ type: "positive", message: "Usuário atualizado." });
  } catch (error: any) {
    console.log(
      "Erro ao salvar",
      error?.response?.status,
      error?.response?.data
    );
    $q.notify({ type: "negative", message: "Erro ao salvar edição." });
  }
}

onMounted(() => {
  listarTodosUsuarios();
});
</script>

<style>
.form__tela {
  margin: 0 auto;
  max-width: 768px;
  width: 100%;
  padding: 2rem 1rem;
}
.form {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
  padding: 2rem;
}

.form__content {
  padding: 0rem;
  border-radius: 8px;
  box-shadow: 0 0 5px #ccc;
}

.form__title {
  font-size: 2rem;
  padding: 2rem 0 0 0;
  margin: 0;
  text-align: center;
  font-weight: bold;
}

.cards {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 1rem;
  padding: 10px;

  @media (max-width:768px){
    grid-template-columns:1fr;
  }
}

.card {
  display: flex;
  flex-direction: column;
  flex: 1 0 200px;
  box-shadow: 0 0 5px #ccc;
  padding: 0.8rem;
  height: 100%;
}

.line__nome {
  font-size: 1.5rem;
  font-weight: 500;
  text-transform: uppercase;
}

.line__email {
  text-transform: lowercase;
}

.actions {
  display: flex;
  gap: 0.8rem;
  margin-top: auto;
}

.form__edicao {
  background: #f2f2f2;
}
</style>
