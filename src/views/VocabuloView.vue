<template>
  <div class="vocabulario">
    <h1>Gerencie Vocábulos</h1>

    <!-- Formulário de cadastro -->
    <div>
      <h3>Cadastrar Novo Vocábulo</h3>
      <form @submit.prevent="cadastrarVocabulo">
        <div>
          <label for="termo">Termo:</label>
          <input id="termo" v-model="novoVocabulo.termo" type="text" required />
        </div>
        <div>
          <label for="significado">Significado:</label>
          <input id="significado" v-model="novoVocabulo.significado" type="text" required />
        </div>
        <div>
          <label for="versao">Versão:</label>
          <input id="versao" v-model="novoVocabulo.versao" type="number" required />
        </div>
        <div>
          <label for="observacoes">Observações:</label>
          <input id="observacoes" v-model="novoVocabulo.observacoes" type="text" />
        </div>
        <div>
          <label for="dataHoraDesativacao">Data/Hora de Desativação:</label>
          <input id="dataHoraDesativacao" v-model="novoVocabulo.dataHoraDesativacao" type="datetime-local" />
        </div>
        <button type="submit">Cadastrar</button>
      </form>
    </div>

    <!-- Campo de busca -->
    <div>
      <h3>Buscar Vocábulo</h3>
      <form @submit.prevent="buscarVocabulo">
        <div>
          <label for="termoBusca">Termo:</label>
          <input id="termoBusca" v-model="busca.termo" type="text" />
        </div>
        <div>
          <label for="versaoBusca">Versão:</label>
          <input id="versaoBusca" v-model="busca.versao" type="number" />
        </div>
        <button type="submit">Buscar</button>
      </form>
    </div>

    <!-- Lista de vocábulos -->
    <div v-if="vocabulos.length > 0">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Termo</th>
            <th>Significado</th>
            <th>Versão</th>
            <th>Situação</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="vocabulo in vocabulos" :key="vocabulo.id">
            <td>{{ vocabulo.id }}</td>
            <td>{{ vocabulo.termo }}</td>
            <td>{{ vocabulo.significado }}</td>
            <td>{{ vocabulo.versao }}</td>
            <td>{{ vocabulo.situacao }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-else>
      <p>{{ mensagemErro }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'

const API_URL = 'https://8080-dianemoreno-springboot3-403ehzjxqfd.ws-us116.gitpod.io/vocabulo'

const vocabulos = ref<any[]>([])
const novoVocabulo = ref({
  termo: '',
  significado: '',
  versao: null,
  observacoes: '',
  dataHoraDesativacao: null,
})
const busca = ref({
  termo: '',
  versao: null,
})
const mensagemErro = ref('')

// Funções
// Função para buscar todos os vocábulos cadastrados no banco
const buscarTodosVocabulos = async () => {
  try {
    const resposta = await axios.get('https://8080-dianemoreno-springboot3-403ehzjxqfd.ws-us116.gitpod.io/vocabulo')
    if (resposta.data.length === 0) {
      mensagemErro.value = 'Nenhum vocábulo encontrado.'
      vocabulos.value = []
    } else {
      mensagemErro.value = ''
      vocabulos.value = resposta.data.map((vocabulo: any) => ({
        ...vocabulo,
        situacao: vocabulo.voc_data_hora_desativacao ? 'Desativado' : 'Ativo',
      }))
    }
  } catch (error) {
    mensagemErro.value = 'Erro ao buscar vocábulos.'
  }
}

// Função para buscar vocábulo por termo e versão
const buscarVocabulo = async () => {
  try {
    const resposta = await axios.get(`https://8080-dianemoreno-springboot3-403ehzjxqfd.ws-us116.gitpod.io/vocabulo/${busca.value.termo}/${busca.value.versao}`)
    if (resposta.data.length === 0) {
      mensagemErro.value = 'Nenhum vocábulo foi encontrado para os critérios fornecidos.'
      vocabulos.value = []
    } else {
      mensagemErro.value = ''
      vocabulos.value = resposta.data.map((vocabulo: any) => ({
        ...vocabulo,
        situacao: vocabulo.voc_data_hora_desativacao ? 'Desativado' : 'Ativo',
      }))
    }
  } catch (error) {
    mensagemErro.value = 'Erro ao buscar vocábulo.'
  }
}


const cadastrarVocabulo = async () => {
  if (!novoVocabulo.value.termo || !novoVocabulo.value.significado || !novoVocabulo.value.versao) {
    mensagemErro.value = 'Preencha todos os campos obrigatórios.'
    return
  }
  try {
    await axios.post(API_URL, novoVocabulo.value)
    mensagemErro.value = ''
    buscarTodosVocabulos()
    Object.assign(novoVocabulo.value, {
      termo: '',
      significado: '',
      versao: null,
      observacoes: '',
      dataHoraDesativacao: null,
    })
  } catch {
    mensagemErro.value = 'Erro ao cadastrar vocábulo.'
  }
}

onMounted(buscarTodosVocabulos)
</script>
