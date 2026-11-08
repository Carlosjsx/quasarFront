<template>
  <q-page class="flex flex-center">
  <div class="form_content">
    <form @submit.prevent="adicionarNovoUsuario" class="form">
      <q-input type="text" placeholder="nome" v-model="formulario.nome" outlined/>
      <q-input type="text" placeholder="e-mail" v-model="formulario.email" outlined/>
      <q-input type="text" placeholder="senha" v-model="formulario.senha" outlined/>
      <q-btn color="primary" type="submit" label="adicionar" />
    </form>
</div>
    <ul>
      <li v-for="dado in dados" :key="dado.id">
        <p>{{ dado.nome }}</p>
        <button @click="removerUsuario(dado.id)">remover</button>
      </li>
    </ul>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";
const dados = ref<any[]>([]);

const formulario = ref({
  nome: "",
  email: "",
  senha: ""
});

const url = "https://special-spork-6vg5xxpg7x553557r-3000.app.github.dev/usuarios";

async function adicionarNovoUsuario() {
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

onMounted(() => {
  listarTodosUsuarios();
});
</script>


<style>

  .form{
    display:flex;
    flex-direction:column;
    gap:.8rem;
    box-shadow: 0 0 5px;
    padding:2rem;
   }

  .form__content{
    padding:0rem;
  }
</style>
