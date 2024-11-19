<template>
  <div class="gerenciamento-produtos">
    <h1>Gerenciamento de Produtos</h1>

    <!-- Formulário de cadastro -->
    <div>
      <h3>Cadastrar Novo Produto</h3>
      <form @submit.prevent="cadastrarProduto">
        <div>
          <label for="nome">Nome:</label>
          <input id="nome" v-model="novoProduto.nome" type="text" required />
        </div>
        <div>
          <label for="descricao">Descrição:</label>
          <input id="descricao" v-model="novoProduto.descricao" type="text" />
        </div>
        <div>
          <label for="dataCadastro">Data de Cadastro:</label>
          <input id="dataCadastro" v-model="novoProduto.dataCadastro" type="date" />
        </div>
        <div>
          <label for="dataUltimaCompra">Data/Hora de Última Compra:</label>
          <input id="dataUltimaCompra" v-model="novoProduto.dataUltimaCompra" type="datetime-local" required />
        </div>
        <div>
          <label for="lote">Lote:</label>
          <input id="lote" v-model="novoProduto.lote" type="number" />
        </div>
        <button type="submit">Cadastrar</button>
      </form>
    </div>

    <!-- Campo de busca -->
    <div>
      <h3>Buscar Produto</h3>
      <form @submit.prevent="buscarProduto">
        <div>
          <label for="dataBusca">Data de Cadastro:</label>
          <input id="dataBusca" v-model="busca.data" type="date" />
        </div>
        <div>
          <label for="loteBusca">Lote:</label>
          <input id="loteBusca" v-model="busca.lote" type="number" />
        </div>
        <button type="submit">Buscar</button>
      </form>
    </div>

    <!-- Lista de produtos -->
    <div v-if="produtos.length > 0">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Nome</th>
            <th>Data/Hora de Última Compra</th>
            <th>Situação</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="produto in produtos" :key="produto.id">
            <td>{{ produto.id }}</td>
            <td>{{ produto.nome }}</td>
            <td>{{ produto.dataUltimaCompra }}</td>
            <td>{{ produto.situacao }}</td>
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
import { ref, onMounted } from 'vue';
import axios from 'axios';

// URL da API
const API_URL = 'https://8080-dianemoreno-springtopic-1315dvg8p02.ws-us116.gitpod.io/produto';

// Estados reativos
const produtos = ref<any[]>([]);
const novoProduto = ref({
  nome: '',
  descricao: '',
  dataCadastro: '',
  dataUltimaCompra: '',
  lote: null,
});
const busca = ref({
  data: '',
  lote: '',
});
const mensagemErro = ref('');

// Função para buscar todos os produtos
const buscarTodosProdutos = async () => {
  try {
    const resposta = await axios.get(API_URL);
    if (resposta.data.length === 0) {
      mensagemErro.value = 'Nenhum produto encontrado.';
      produtos.value = [];
    } else {
      mensagemErro.value = '';
      produtos.value = resposta.data.map((produto: any) => ({
        ...produto,
        situacao: produto.lote ? 'Identificado' : 'Suspeito',
      }));
    }
  } catch (error) {
    console.log("Erro ao buscar todos os produtos", error); // Log de erro
    mensagemErro.value = 'Erro ao buscar produtos.';
  }
};

// Função para buscar produto por data e lote
const buscarProduto = async () => {
  try {
    // Monta a URL de busca com base nos parâmetros fornecidos
    const resposta = await axios.get(`https://8080-dianemoreno-springtopic-1315dvg8p02.ws-us116.gitpod.io/produto/buscaLote/${busca.value.data || 'null'}/${busca.value.lote || 'null'}`);

    // Verifica se a resposta contém dados
    if (resposta.data.length === 0) {
      mensagemErro.value = 'Nenhum produto foi encontrado para os critérios fornecidos.';
      produtos.value = [];
    } else {
      mensagemErro.value = '';
      // Mapeia os produtos recebidos, atribuindo a situação com base no lote
      produtos.value = resposta.data.map((produto: any) => ({
        ...produto,
        situacao: produto.lote ? 'Identificado' : 'Suspeito', // Aqui a lógica para a situação
      }));
    }
  } catch (error) {
    mensagemErro.value = 'Erro ao buscar produto.';
    console.log("Erro ao buscar produto", error); // Log para auxiliar no debug
  }
};



const cadastrarProduto = async () => {
  // Verifica se os campos obrigatórios estão preenchidos
  if (!novoProduto.value.nome || !novoProduto.value.dataCadastro || !novoProduto.value.dataHoraUltimaCompra || !novoProduto.value.lote) {
    mensagemErro.value = 'Preencha todos os campos obrigatórios.';
    console.log("Campos obrigatórios não preenchidos", novoProduto.value);
    return;
  }

  // Converte a data de 'dataCadastro' para o formato adequado (LocalDate)
  const dataCadastroFormatada = novoProduto.value.dataCadastro ? new Date(novoProduto.value.dataCadastro) : null;
  const dataHoraUltimaCompraFormatada = novoProduto.value.dataHoraUltimaCompra ? new Date(novoProduto.value.dataHoraUltimaCompra) : null;

  // Cria o objeto de produto com os dados convertidos
  const produtoCadastro = {
    nome: novoProduto.value.nome,
    descricao: novoProduto.value.descricao,
    dataCadastro: dataCadastroFormatada.toISOString().split('T')[0], // Converte para LocalDate
    dataHoraUltimaCompra: dataHoraUltimaCompraFormatada.toISOString(), // Converte para LocalDateTime
    lote: novoProduto.value.lote,
  };

  console.log("Cadastrando produto com os seguintes dados:", produtoCadastro);

  try {
    // Envia o produto para a API
    const resposta = await axios.post('https://8080-dianemoreno-springtopicos20242-dianemoreno-springtopic-1315dvg8p02.ws-us116.gitpod.io/produto', produtoCadastro);
    console.log("Resposta da API após cadastro:", resposta.data);

    if (resposta.status === 200 || resposta.status === 201) {
      mensagemErro.value = ''; // Limpa a mensagem de erro
      buscarTodosProdutos(); // Atualiza a lista de produtos

      // Limpa os campos do formulário após o cadastro
      Object.assign(novoProduto.value, {
        nome: '',
        descricao: '',
        dataCadastro: '',
        dataHoraUltimaCompra: '',
        lote: null,
      });
      console.log("Cadastro bem-sucedido e campos limpos");
    } else {
      mensagemErro.value = 'Erro ao cadastrar produto. Resposta da API não esperada.';
    }
  } catch (error) {
    console.log("Erro ao cadastrar produto", error);
    mensagemErro.value = 'Erro ao cadastrar produto.';
  }
};




// Inicializar a lista de produtos
onMounted(buscarTodosProdutos);
</script>
