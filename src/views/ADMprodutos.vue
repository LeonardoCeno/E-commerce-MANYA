<template>
  <div class="tudo">
    <!-- Modal de Criação/Edição de Produto -->
    <div v-if="mostraFormulario" class="criacao-form-wrapper">
      <div class="criacao-form">
        <h2>{{ editando ? 'Editar Produto' : 'Criar Produto' }}</h2>
        <form @submit.prevent="editando ? atualizarProduto() : criarProduto()">
          <div>
            <label>Nome:</label>
            <input v-model="nomeForm" required />
          </div>
          <div>
            <label>Descrição:</label>
            <textarea v-model="descricaoForm" required></textarea>
          </div>
          <div class="linha-dupla">
            <div class="campo-metade">
              <label>Preço:</label>
              <input type="number" v-model.number="precoForm" min="0" step="0.01" required />
            </div>
            <div class="campo-metade">
              <label>Estoque:</label>
              <input type="number" v-model.number="estoqueForm" min="0" required />
            </div>
          </div>
          <div>
            <label>Categoria:</label>
            <select v-model="categoriaIdForm" required>
              <option value="" disabled>Selecione</option>
              <option v-for="cat in categorias" :key="cat.id" :value="cat.id">{{ cat.name }}</option>
            </select>
          </div>
          <div>
            <label>Imagem:</label>
            <input type="file" @change="onFileChange" accept="image/*" />
          </div>
          <button type="submit">{{ editando ? 'Salvar' : 'Criar Produto' }}</button>
          <button type="button" @click="editando ? cancelarEdicao() : fecharFormulario()">Cancelar</button>
        </form>
        <p v-if="editando ? mensagemEdicao : mensagem">{{ editando ? mensagemEdicao : mensagem }}</p>
      </div>
    </div>

    <!-- Conteúdo Principal -->
    <div class="produtos">
      <h3 class="titulo-principal">{{ modoDesconto ? 'Descontos' : 'Produtos' }}</h3>
      
      <!-- Header com Filtros -->
    <div class="produtos-header">
        <div class="busca-container">
          <div class="input-busca">
            <input 
              type="text" 
              placeholder="Buscar produtos..." 
              v-model="termoBusca" 
              @input="onInputBusca" 
              @focus="onFocusBusca" 
              @blur="onBlurBusca"
            />
            <img src="../components/img/LupaFinal.png" alt="Buscar" />
          </div>
        </div>
        
      <div class="filtro-categorias">
        <div class="filtro-estoque">
          <label for="filtroEstoque" style="margin-right: 6px; font-size: 1rem;">Estoque:</label>
          <select id="filtroEstoque" v-model="estoqueSelecionado">
            <option value="">Indefinido</option>
            <option value="0">0</option>
            <option value="10-30">10-30</option>
            <option value="30-50">30-50</option>
            <option value="50-100">50-100</option>
            <option value="100+">100 ou mais</option>
          </select>
        </div>
          
        <div class="filtro-categoria">
            <label for="filtroCategoria" style="margin-right: 6px; font-size: 1rem;">Categoria:</label>
          <select id="filtroCategoria" v-model="categoriaSelecionada">
            <option value="">Todas</option>
            <option v-for="cat in categorias" :key="cat.id" :value="cat.id">{{ cat.name }}</option>
          </select>
        </div>
          
          <div v-if="modoDesconto" class="filtro-desconto">
            <label for="filtroDesconto" style="margin-right: 6px; font-size: 1rem;">Desconto:</label>
            <select id="filtroDesconto" v-model="filtroDescontoSelecionado">
              <option value="Todos">Todos</option>
              <option value="Ativo">Ativo</option>
              <option value="Expirado">Expirado</option>
              <option value="Sem desconto">Sem desconto</option>
            </select>
      </div>

          <button v-if="!modoDesconto" class="novo-produto-btn" @click="abrirCriacao">Novo produto</button>
          <button v-if="modoDesconto" class="novo-produto-btn" :class="{ 'cancelar-btn': selecionandoProduto }" @click="selecionandoProduto ? cancelarSelecaoProduto() : abrirCriacaoDesconto()">
            {{ selecionandoProduto ? 'Cancelar' : 'Novo desconto' }}
          </button>
          <button class="alternar-modo-btn" @click="alternarModo">
            {{ modoDesconto ? 'Produtos' : 'Descontos' }}
          </button>
    </div>
      </div>

      <!-- Lista de Produtos -->
    <div v-if="carregandoProdutos">Carregando produtos...</div>
    <div v-else-if="erroProdutos">{{ erroProdutos }}</div>
    <div v-else>
        <div v-if="produtosFiltrados.length === 0">
          <div v-if="termoBusca">Nenhum produto encontrado para "{{ termoBusca }}".</div>
          <div v-else>Nenhum produto cadastrado ainda.</div>
        </div>
      <ul v-else class="lista">
          <li v-for="produto in produtosFiltrados" :key="produto.id" class="produto" :class="{ 
            'produto-selecionado': selecionandoProduto && !temDescontoAtivoProduto(produto), 
            'produto-com-desconto': selecionandoProduto && temDescontoAtivoProduto(produto),
            'produto-com-desconto-modo': modoDesconto && temDescontoAtivoProduto(produto),
            'produto-com-desconto-expirado': modoDesconto && temDescontoExpiradoProduto(produto)
          }" @click="selecionandoProduto && !temDescontoAtivoProduto(produto) ? selecionarProduto(produto) : null">
          <div class="nome-preco-imagem">
            <img v-if="produto.image_path" :src="produto.image_path" alt="Imagem do produto" class="produto-imagem" />
            <h4>{{ produto.name }}</h4>
            <p>R$ {{ produto.price }}</p>
              <div v-if="(selecionandoProduto || modoDesconto) && temDescontoAtivoProduto(produto)" class="desconto-barra ativo">
                <span>DESCONTO ATIVO</span>
          </div>
              <div v-if="modoDesconto && temDescontoExpiradoProduto(produto)" class="desconto-barra expirado">
                <span>DESCONTO EXPIRADO</span>
              </div>
            </div>
            
            <!-- Botões do Modo Produtos -->
            <div class="BTli" @click.stop v-if="!modoDesconto">
            <button @click="editarProduto(produto)">Editar</button>
            <button class="excluir-btn" @click="abrirModalExclusao(produto.id)">Excluir</button>
          </div>
            
            <!-- Botões do Modo Descontos -->
            <div class="BTli" @click.stop v-if="modoDesconto && (temDescontoAtivoProduto(produto) || temDescontoExpiradoProduto(produto))">
              <button class="detalhes-btn" @click="visualizarDesconto(produto)">Detalhes</button>
              <button class="excluir-desconto-btn" @click="excluirDescontoDireto(produto)">Excluir</button>
            </div>
            
          <span style="font-size:12px;color:#555;">Estoque: {{ produto.stock }}</span>
        </li>
      </ul>
    </div>
    </div>
  </div>
  
  <!-- Modal de Confirmação -->
  <div v-if="mostrarModalConfirmacao" class="modal-overlay">
      <div class="modal-confirmacao">
          <h3>Confirmar Exclusão</h3>
          <p>Tem certeza que deseja excluir este produto?</p>
          <div class="modal-botoes">
              <button @click="confirmarExclusao" class="btn-confirmar">Confirmar</button>
              <button @click="fecharModalConfirmacao" class="btn-cancelar">Cancelar</button>
          </div>
      </div>
  </div>

  <!-- Modal de Formulário de Desconto -->
  <div v-if="mostraFormularioDesconto" class="modal-overlay">
    <div class="modal-desconto">
      <h3>Criar Desconto</h3>
      <div v-if="produtoSelecionado" class="produto-selecionado-info">
        <h4>Produto Selecionado:</h4>
        <p><strong>{{ produtoSelecionado.name }}</strong></p>
        <p>Preço: R$ {{ produtoSelecionado.price }}</p>
      </div>
      <form @submit.prevent="criarDesconto">
        <div>
          <label>Descrição do Desconto:</label>
          <input v-model="descricaoDesconto" required />
        </div>
        <div>
          <label>Percentual de Desconto (%):</label>
          <input type="number" v-model.number="percentualDesconto" min="0" max="100" step="0.01" required />
        </div>
        <div class="linha-dupla">
          <div class="campo-metade">
            <label>Data de Início:</label>
            <input type="datetime-local" v-model="dataInicioDesconto" required />
          </div>
          <div class="campo-metade">
            <label>Data de Fim:</label>
            <input type="datetime-local" v-model="dataFimDesconto" required />
          </div>
        </div>
        <div class="modal-botoes">
          <button type="submit" class="btn-confirmar">Criar Desconto</button>
          <button type="button" @click="fecharFormularioDesconto" class="btn-cancelar">Cancelar</button>
        </div>
      </form>
    </div>
  </div>

  <!-- Modal para Visualizar Desconto -->
  <div v-if="mostraModalVisualizarDesconto" class="modal-overlay">
    <div class="modal-desconto">
      <h3>Informações do Desconto</h3>
      <div class="produto-selecionado-info">
        <h4>Produto: {{ produtoVisualizandoDesconto.name }}</h4>
        <p><strong>Preço:</strong> R$ {{ produtoVisualizandoDesconto.price }}</p>
      </div>
      <div v-if="descontoVisualizando" class="desconto-detalhes">
        <div class="desconto-info-completa">
          <h4>Detalhes do Desconto:</h4>
          <div class="info-item">
            <strong>Descrição:</strong>
            <p>{{ descontoVisualizando.description }}</p>
          </div>
          <div class="info-item">
            <strong>Percentual de Desconto:</strong>
            <p>{{ descontoVisualizando.discount_percentage }}%</p>
          </div>
          <div class="info-item">
            <strong>Data de Início:</strong>
            <p>{{ formatarData(descontoVisualizando.start_date) }}</p>
          </div>
          <div class="info-item">
            <strong>Data de Fim:</strong>
            <p>{{ formatarData(descontoVisualizando.end_date) }}</p>
          </div>
          <div class="info-item">
            <strong>Status:</strong>
            <p><span :class="getStatusClass(descontoVisualizando)">{{ getStatusText(descontoVisualizando) }}</span></p>
          </div>
        </div>
      </div>
      <div class="modal-botoes">
        <button type="button" @click="abrirModalEditarDesconto" class="btn-editar">Editar</button>
        <button type="button" @click="fecharModalVisualizarDesconto" class="btn-cancelar">Fechar</button>
      </div>
    </div>
  </div>

  <!-- Modal para Editar Desconto -->
  <div v-if="mostraModalEditarDesconto" class="modal-overlay">
    <div class="modal-desconto">
      <h3>Editar Desconto</h3>
      <div v-if="produtoVisualizandoDesconto" class="produto-selecionado-info">
        <h4>Produto: {{ produtoVisualizandoDesconto.name }}</h4>
        <p><strong>Preço:</strong> R$ {{ produtoVisualizandoDesconto.price }}</p>
      </div>
      <form @submit.prevent="salvarEdicaoDesconto">
        <div>
          <label>Descrição do Desconto:</label>
          <input v-model="descricaoDescontoEditando" required />
        </div>
        <div>
          <label>Percentual de Desconto (%):</label>
          <input type="number" v-model.number="percentualDescontoEditando" min="0" max="100" step="0.01" required />
        </div>
        <div class="linha-dupla">
          <div class="campo-metade">
            <label>Data de Início:</label>
            <input type="datetime-local" v-model="dataInicioDescontoEditando" required />
          </div>
          <div class="campo-metade">
            <label>Data de Fim:</label>
            <input type="datetime-local" v-model="dataFimDescontoEditando" required />
          </div>
        </div>
        <div class="modal-botoes">
          <button type="submit" class="btn-confirmar">Salvar Alterações</button>
          <button type="button" @click="fecharModalEditarDesconto" class="btn-cancelar">Cancelar</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import { useToast } from 'vue-toastification'
import api from '../services/api'

const toast = useToast()
const produtos = ref([])
const categorias = ref([])
const carregandoProdutos = ref(true)
const erroProdutos = ref('')

// Estados de formulário
const mostraFormulario = ref(false)
const editando = ref(false)
const editId = ref(null)
const mensagem = ref('')
const mensagemEdicao = ref('')

// Formulário de produto
const nomeForm = ref('')
const descricaoForm = ref('')
const precoForm = ref(0)
const estoqueForm = ref(0)
const categoriaIdForm = ref('')
const imagemForm = ref(null)
const imagem = ref(null)

// Estados de desconto
const modoDesconto = ref(false)
const mostraFormularioDesconto = ref(false)
const produtoSelecionado = ref(null)
const selecionandoProduto = ref(false)
const descricaoDesconto = ref('')
const percentualDesconto = ref(0)
const dataInicioDesconto = ref('')
const dataFimDesconto = ref('')

// Visualização de desconto
const mostraModalVisualizarDesconto = ref(false)
const produtoVisualizandoDesconto = ref(null)
const descontoVisualizando = ref(null)

// Edição de desconto
const mostraModalEditarDesconto = ref(false)
const descontoEditando = ref(null)
const descricaoDescontoEditando = ref('')
const percentualDescontoEditando = ref(0)
const dataInicioDescontoEditando = ref('')
const dataFimDescontoEditando = ref('')

// Estados de exclusão
const mostrarModalConfirmacao = ref(false)
const produtoParaExcluir = ref(null)

// Filtros
const termoBusca = ref('')
const categoriaSelecionada = ref('')
const estoqueSelecionado = ref('')
const filtroDescontoSelecionado = ref('Todos')

// Debounce para busca
let timeoutBusca = null

// Funções utilitárias
function formatarData(data) {
  return new Date(data).toLocaleDateString('pt-BR', { 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

function temDescontoAtivoProduto(produto) {
  if (!produto.discounts || produto.discounts.length === 0) return false
  
  const hoje = new Date()
  return produto.discounts.some(desconto => {
    const inicio = new Date(desconto.start_date)
    const fim = new Date(desconto.end_date)
    return hoje >= inicio && hoje <= fim
  })
}

function temDescontoExpiradoProduto(produto) {
  if (!produto.discounts || produto.discounts.length === 0) return false
  
  const hoje = new Date()
  return produto.discounts.some(desconto => {
    const fim = new Date(desconto.end_date)
    return hoje > fim
  })
}

function getStatusText(desconto) {
  const hoje = new Date()
  const inicio = new Date(desconto.start_date)
  const fim = new Date(desconto.end_date)
  
  if (hoje >= inicio && hoje <= fim) return 'ATIVO'
  if (hoje > fim) return 'EXPIRADO'
  return 'FUTURO'
}

function getStatusClass(desconto) {
  const hoje = new Date()
  const inicio = new Date(desconto.start_date)
  const fim = new Date(desconto.end_date)
  
  if (hoje >= inicio && hoje <= fim) return 'status-ativo'
  if (hoje > fim) return 'status-expirado'
  return 'status-futuro'
}

// Computed para filtros
const produtosFiltrados = computed(() => {
  let filtrados = produtos.value

  // Filtro por busca
  if (termoBusca.value.trim()) {
    const termo = termoBusca.value.toLowerCase().trim()
    filtrados = filtrados.filter(produto => 
      produto.name && produto.name.toLowerCase().includes(termo)
    )
  }

  // Filtro por categoria
  if (categoriaSelecionada.value) {
    filtrados = filtrados.filter(produto => 
      String(produto.category_id) === String(categoriaSelecionada.value)
    )
  }

  // Filtro por estoque
  if (estoqueSelecionado.value) {
    filtrados = filtrados.filter(produto => {
      const estoque = produto.stock
      switch (estoqueSelecionado.value) {
        case '0': return estoque === 0
        case '10-30': return estoque >= 10 && estoque <= 30
        case '30-50': return estoque >= 30 && estoque <= 50
        case '50-100': return estoque >= 50 && estoque <= 100
        case '100+': return estoque >= 100
        default: return true
      }
    })
  }

  // Filtro por desconto (apenas no modo desconto)
  if (modoDesconto.value && filtroDescontoSelecionado.value) {
    filtrados = filtrados.filter(produto => {
      if (!produto.discounts || produto.discounts.length === 0) {
        return filtroDescontoSelecionado.value === 'Sem desconto'
      }
      
      const hoje = new Date()
      const temAtivo = produto.discounts.some(desconto => {
        const inicio = new Date(desconto.start_date)
        const fim = new Date(desconto.end_date)
        return hoje >= inicio && hoje <= fim
      })
      
      const temExpirado = produto.discounts.some(desconto => {
        const fim = new Date(desconto.end_date)
        return hoje > fim
      })
      
      switch (filtroDescontoSelecionado.value) {
        case 'Todos': return temAtivo || temExpirado
        case 'Ativo': return temAtivo
        case 'Expirado': return temExpirado
        case 'Sem desconto': return !temAtivo && !temExpirado
        default: return true
      }
    })
    
    // Ordenar: produtos com desconto ativo primeiro
    filtrados.sort((a, b) => {
      const hoje = new Date()
      const aTemAtivo = a.discounts.some(desconto => {
        const inicio = new Date(desconto.start_date)
        const fim = new Date(desconto.end_date)
        return hoje >= inicio && hoje <= fim
      })
      
      const bTemAtivo = b.discounts.some(desconto => {
        const inicio = new Date(desconto.start_date)
        const fim = new Date(desconto.end_date)
        return hoje >= inicio && hoje <= fim
      })
      
      if (aTemAtivo && !bTemAtivo) return -1
      if (bTemAtivo && !aTemAtivo) return 1
      return 0
    })
  }

  return filtrados
})

// ===== FUNÇÕES COMPARTILHADAS (AMBOS OS MODOS) =====

// Funções de busca
function onInputBusca() {
  clearTimeout(timeoutBusca)
  timeoutBusca = setTimeout(() => {}, 100)
}

function onFocusBusca() {}
function onBlurBusca() {}

// Funções de modo
function alternarModo() {
  modoDesconto.value = !modoDesconto.value
  selecionandoProduto.value = false
  produtoSelecionado.value = null
  mostraFormularioDesconto.value = false
  termoBusca.value = ''
  categoriaSelecionada.value = ''
  estoqueSelecionado.value = ''
  filtroDescontoSelecionado.value = 'Todos'
}

// Carregamento de dados
async function carregarProdutos() {
  carregandoProdutos.value = true
  erroProdutos.value = ''
  try {
    const { data } = await api.get('/products/user/228')
    
    const produtosComDescontos = await Promise.all(data.map(async (produto) => {
      try {
        const { data: descontos } = await api.get(`/discounts/`)
        const descontosDoProduto = descontos.filter(desconto => desconto.product_id === produto.id)
        return {
          ...produto,
          image_path: produto.image_path 
            ? `http://35.196.79.227:8000${produto.image_path}` 
            : '/placeholder-image.jpg',
          discounts: descontosDoProduto || []
        }
  } catch (e) {
        return {
          ...produto,
          image_path: produto.image_path 
            ? `http://35.196.79.227:8000${produto.image_path}` 
            : '/placeholder-image.jpg',
          discounts: []
        }
      }
    }))
    
    produtos.value = produtosComDescontos
  } catch (e) {
    erroProdutos.value = 'Erro ao carregar produtos.'
  } finally {
    carregandoProdutos.value = false
  }
}

// ===== FUNÇÕES DE PRODUTOS =====

function onFileChange(e) {
  imagem.value = e.target.files[0]
  imagemForm.value = e.target.files[0]
}

function abrirCriacao() {
  editando.value = false
  mostraFormulario.value = true
  limparFormulario()
}

function fecharFormulario() {
  mostraFormulario.value = false
  editando.value = false
  limparFormulario()
}

function limparFormulario() {
  nomeForm.value = ''
  descricaoForm.value = ''
  precoForm.value = 0
  estoqueForm.value = 0
  categoriaIdForm.value = ''
  imagemForm.value = null
  mensagem.value = ''
  mensagemEdicao.value = ''
}

async function criarProduto() {
  mensagem.value = ''
  try {
    const formData = new FormData()
    formData.append('name', nomeForm.value)
    formData.append('description', descricaoForm.value)
    formData.append('price', precoForm.value)
    formData.append('stock', estoqueForm.value)
    formData.append('category_id', categoriaIdForm.value)
    if (imagemForm.value) formData.append('image', imagemForm.value)
    
    await api.post('/products/', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
    toast.success('Produto criado com sucesso!')
    await carregarProdutos()
    fecharFormulario()
  } catch (e) {
    toast.error('Erro ao criar produto.')
  }
}

function editarProduto(produto) {
  editando.value = true
  editId.value = produto.id
  nomeForm.value = produto.name
  descricaoForm.value = produto.description
  precoForm.value = produto.price
  estoqueForm.value = produto.stock
  categoriaIdForm.value = produto.category_id
  imagemForm.value = null
  mensagemEdicao.value = ''
  mostraFormulario.value = true
}

function cancelarEdicao() {
  editando.value = false
  editId.value = null
  limparFormulario()
  mostraFormulario.value = false
}

async function atualizarProduto() {
  mensagemEdicao.value = ''
  try {
    await api.put(`/products/${editId.value}`, {
      name: nomeForm.value,
      description: descricaoForm.value,
      price: precoForm.value,
      category_id: categoriaIdForm.value
    })
    await api.put(`/products/${editId.value}/stock`, {
      stock: Number(estoqueForm.value)
    })
    if (imagemForm.value) {
      const formData = new FormData()
      formData.append('image', imagemForm.value)
      await api.put(`/products/${editId.value}/image`, formData, {
        headers: { 'Content-Type': 'multipart/form-data' }
      })
    }
    toast.success('Produto atualizado com sucesso!')
    await carregarProdutos()
    cancelarEdicao()
  } catch (e) {
    toast.error('Erro ao atualizar produto.')
  }
}

function abrirModalExclusao(id) {
  produtoParaExcluir.value = id
  mostrarModalConfirmacao.value = true
}

async function confirmarExclusao() {
  try {
    await api.delete(`/products/${produtoParaExcluir.value}`)
    toast.success('Produto excluído com sucesso!')
    await carregarProdutos()
    fecharModalConfirmacao()
  } catch (e) {
    toast.error('Erro ao excluir produto.')
    fecharModalConfirmacao()
  }
}

function fecharModalConfirmacao() {
  mostrarModalConfirmacao.value = false
  produtoParaExcluir.value = null
}

// ===== FUNÇÕES DE DESCONTOS =====

function abrirCriacaoDesconto() {
  selecionandoProduto.value = true
  filtroDescontoSelecionado.value = 'Sem desconto'
  toast.info('Selecione o produto')
}

function selecionarProduto(produto) {
  const hoje = new Date()
  const temDescontoAtivo = produto.discounts && produto.discounts.some(desconto => {
    const inicio = new Date(desconto.start_date)
    const fim = new Date(desconto.end_date)
    return hoje >= inicio && hoje <= fim
  })
  
  if (temDescontoAtivo) {
    toast.error('Este produto já possui um desconto ativo!')
    return
  }
  
  produtoSelecionado.value = produto
  selecionandoProduto.value = false
  mostraFormularioDesconto.value = true
}

function cancelarSelecaoProduto() {
  selecionandoProduto.value = false
  produtoSelecionado.value = null
}

function fecharFormularioDesconto() {
  mostraFormularioDesconto.value = false
  produtoSelecionado.value = null
  descricaoDesconto.value = ''
  percentualDesconto.value = 0
  dataInicioDesconto.value = ''
  dataFimDesconto.value = ''
}

async function criarDesconto() {
  try {
    const descontoData = {
      description: descricaoDesconto.value,
      discount_percentage: percentualDesconto.value,
      start_date: dataInicioDesconto.value,
      end_date: dataFimDesconto.value,
      product_id: produtoSelecionado.value.id
    }
    
    await api.post('/discounts/', descontoData)
    toast.success('Desconto criado com sucesso!')
    fecharFormularioDesconto()
    await carregarProdutos()
  } catch (e) {
    toast.error('Erro ao criar desconto.')
  }
}

// Funções de visualização de desconto
function visualizarDesconto(produto) {
  produtoVisualizandoDesconto.value = produto
  
  if (produto.discounts && produto.discounts.length === 1) {
    descontoVisualizando.value = produto.discounts[0]
    mostraModalVisualizarDesconto.value = true
  }
}

function fecharModalVisualizarDesconto() {
  mostraModalVisualizarDesconto.value = false
  produtoVisualizandoDesconto.value = null
  descontoVisualizando.value = null
}

function abrirModalEditarDesconto() {
  descontoEditando.value = descontoVisualizando.value
  descricaoDescontoEditando.value = descontoVisualizando.value.description
  percentualDescontoEditando.value = descontoVisualizando.value.discount_percentage
  dataInicioDescontoEditando.value = descontoVisualizando.value.start_date.slice(0, 16)
  dataFimDescontoEditando.value = descontoVisualizando.value.end_date.slice(0, 16)
  mostraModalVisualizarDesconto.value = false
  mostraModalEditarDesconto.value = true
}

async function salvarEdicaoDesconto() {
  try {
    const descontoData = {
      description: descricaoDescontoEditando.value,
      discount_percentage: percentualDescontoEditando.value,
      start_date: dataInicioDescontoEditando.value,
      end_date: dataFimDescontoEditando.value
    }
    
    await api.put(`/discounts/${descontoEditando.value.id}`, descontoData)
    toast.success('Desconto atualizado com sucesso!')
    fecharModalEditarDesconto()
    await carregarProdutos()
  } catch (e) {
    toast.error('Erro ao atualizar desconto.')
  }
}

function fecharModalEditarDesconto() {
  mostraModalEditarDesconto.value = false
  descontoEditando.value = null
  descricaoDescontoEditando.value = ''
  percentualDescontoEditando.value = 0
  dataInicioDescontoEditando.value = ''
  dataFimDescontoEditando.value = ''
}

async function excluirDescontoDireto(produto) {
  try {
    if (!produto.discounts || produto.discounts.length === 0) {
      toast.error('Este produto não possui descontos.')
      return
    }
    
    if (produto.discounts.length === 1) {
      await api.delete(`/discounts/${produto.discounts[0].id}`)
      toast.success('Desconto excluído com sucesso!')
    }
    
    await carregarProdutos()
  } catch (e) {
    toast.error('Erro ao excluir desconto.')
  }
}

// Watchers
watch(editando, (novo) => {
  if (novo) {
    nomeForm.value = editId.value ? nomeForm.value : ''
    descricaoForm.value = editId.value ? descricaoForm.value : ''
    precoForm.value = editId.value ? precoForm.value : 0
    estoqueForm.value = editId.value ? estoqueForm.value : 0
    categoriaIdForm.value = editId.value ? categoriaIdForm.value : ''
    imagemForm.value = null
  } else {
    limparFormulario()
  }
})

// Inicialização
onMounted(async () => {
  try {
    const { data } = await api.get('/categories/user/228')
    categorias.value = data
  } catch (e) {
    mensagem.value = 'Erro ao carregar categorias.'
  }
  await carregarProdutos()
})
</script>

<style scoped>
/* ===== LAYOUT PRINCIPAL ===== */
.tudo {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  background-color: #ffffff;
  padding: 30px 0px 0px 70px;
}

.produtos {
  width: 95%;
  height: 30%;
}

/* ===== FORMULÁRIOS ===== */
.criacao-form-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  overflow: auto;
}

.criacao-form {
  background-color: #fff;
  padding: 32px 24px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  width: 98vw;
  max-width: 540px;
  max-height: 90vh;
  overflow-y: auto;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
}

.criacao-form h2 {
  font-size: 2.5rem;
  font-family: helvetica;
  margin-bottom: 20px;
}

.criacao-form label {
  display: block;
  margin-bottom: 10px;
  font-size: 1.1rem;
  font-weight: bold;
}

.criacao-form input[type="text"],
.criacao-form textarea,
.criacao-form input[type="file"],
.criacao-form input[type="number"],
.criacao-form select {
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1rem;
}

.criacao-form button {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1.1rem;
  margin-right: 10px;
}

.criacao-form button:last-child {
  background-color: #f44336;
}

.criacao-form button:last-child:hover {
  background-color: #da190b;
}

.criacao-form form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.criacao-form form button {
  margin-right: 10px;
  margin-bottom: 0;
}

/* ===== HEADER E FILTROS ===== */
.produtos-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 25px;
  border-bottom: 1px solid rgb(167, 167, 167);
  padding-bottom: 20px;
}

.produtos-header h3 {
  font-size: 2.5rem;
  font-family: helvetica;
  margin: 0;
}

.titulo-principal {
  margin: 0 0 20px 0;
  color: #333;
  font-size: 1.8rem;
  font-weight: bold;
  text-align: center;
  padding: 20px 0;
  border-bottom: 2px solid #e0e0e0;
}

/* ===== BUSCA ===== */
.busca-container {
  display: flex;
  justify-content: center;
  width: 100%;
}

.input-busca {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #ffffff;
  border-radius: 15px;
  padding: 0 20px;
  width: 80%;
  max-width: 400px;
  height: 40px;
  border: 1px solid #6d6d6d;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
  position: relative;
}

.input-busca:focus-within {
  border: 1.8px solid #03040cf5;
}

.input-busca input {
  width: 100%;
  border: none;
  outline: none;
  background-color: transparent;
  font-size: 16px;
  color: #2c3e50;
  font-weight: 500;
  padding-right: 40px;
}

.input-busca input::placeholder {
  color: #95a5a6;
  font-style: italic;
  font-weight: 400;
}

.input-busca img {
  width: auto;
  height: 20px;
  filter: brightness(20%);
  position: absolute;
  right: 15px;
}

/* ===== FILTROS ===== */
.filtro-categorias {
  display: flex;
  align-items: center;
  gap: 12px;
}

.filtro-estoque,
.filtro-categoria,
.filtro-desconto {
  display: flex;
  align-items: center;
  gap: 6px;
}

.filtro-categorias select,
.filtro-desconto select {
  padding: 6px 10px;
  border-radius: 5px;
  border: 1px solid #bdbdbd;
  font-size: 1rem;
}

.filtro-desconto select {
  min-width: 150px;
}

.filtro-desconto select:focus {
  outline: none;
  border-color: #2196F3;
  box-shadow: 0 0 0 2px rgba(33, 150, 243, 0.2);
}

/* ===== BOTÕES ===== */
.novo-produto-btn,
.alternar-modo-btn {
  padding: 10px 20px;
  font-size: 1.2rem;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  white-space: nowrap;
  display: inline-block;
}

.novo-produto-btn {
  background-color: #4CAF50;
}

.novo-produto-btn:hover {
  background-color: #45a049;
}

.alternar-modo-btn {
  background-color: #06080afa;
}

.alternar-modo-btn:hover {
  background-color: #0a0d0f;
}

.cancelar-btn {
  background-color: #6c757d !important;
}

.cancelar-btn:hover {
  background-color: #5a6268 !important;
}

/* ===== LISTA DE PRODUTOS ===== */
.lista {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  justify-items: center;
  padding: 4px;
  gap: 5px;
  position: relative;
}

ul {
  margin-top: 1rem;
  padding-left: 0;
  list-style: none;
  max-height: 70vh;
  overflow-y: auto;
  scrollbar-color: rgb(100, 100, 100) rgba(241, 241, 241, 0.527);
}

li {
  background-color: #f0f0f0;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: 7px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 1.1rem;
  border: 1px solid #a9b5b6;
}

/* ===== PRODUTO ===== */
.produto {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  text-align: center;
  width: 230px;
  height: 92%;
  margin-top: 3vh;
  padding-left: 10px;
  padding-right: 10px;
  transition: all 0.3s ease;
  position: relative;
}

.produto h4 {
  font-family: 'Roboto', sans-serif;
  font-size: 15px;
  color: rgb(65, 65, 65);
  margin-top: 10px;
  height: 40px;
}

.nome-preco-imagem {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

.nome-preco-imagem p {
  font-size: 22px;
  color: rgb(49, 49, 49);
  font-weight: bold;
}

.nome-preco-imagem img {
  margin-top: 10px;
  height: 225px;
  width: 160px;
  border: 0.1px solid rgb(212, 212, 212);
  filter: contrast(100%);
}

/* ===== BOTÕES DE AÇÃO ===== */
.BTli {
  display: flex;
  gap: 1vw;
}

.BTli button,
.excluir-btn,
.detalhes-btn,
.excluir-desconto-btn {
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9rem;
  margin-left: 10px;
  color: white;
}

.BTli button {
  background-color: #6c757d;
}

.BTli button:hover {
  background-color: #5a6268;
}

.excluir-btn,
.excluir-desconto-btn {
  background-color: #dc3545 !important;
}

.excluir-btn:hover,
.excluir-desconto-btn:hover {
  background-color: #b71c1c !important;
}

.detalhes-btn {
  background-color: #06080afa !important;
}

.detalhes-btn:hover {
  background-color: #050709fa !important;
}

/* ===== ESTADOS DE PRODUTO ===== */
.produto-selecionado {
  border: 2px solid #2196F3 !important;
  background-color: #E3F2FD !important;
  transform: scale(1.02);
  transition: all 0.3s ease;
  cursor: pointer;
}

.produto-selecionado:hover {
  transform: scale(1.03);
  box-shadow: 0 4px 12px rgba(33, 150, 243, 0.3);
}

.produto-com-desconto {
  border: 2px solid #6c757d !important;
  background-color: #f8f9fa !important;
  opacity: 0.8;
}

.produto-com-desconto:hover {
  opacity: 1;
}

.produto-com-desconto-modo {
  border: 2px solid #90EE90 !important;
  background-color: #f8f9fa !important;
  opacity: 0.9;
}

.produto-com-desconto-modo:hover {
  opacity: 1;
}

.produto-com-desconto-expirado {
  border: 2px solid #dc3545 !important;
  background-color: #f8f9fa !important;
  opacity: 0.9;
}

.produto-com-desconto-expirado:hover {
  opacity: 1;
}

/* ===== BARRAS DE DESCONTO ===== */
.desconto-barra {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  padding: 6px 0;
  font-size: 0.7rem;
  font-weight: bold;
  text-align: center;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 160px;
  border-radius: 8px 8px 0 0;
}

.desconto-barra span {
  font-size: 0.6rem;
  letter-spacing: 0.5px;
  white-space: nowrap;
}

.desconto-barra.ativo {
  background-color: #90EE90;
  color: black;
}

.desconto-barra.expirado {
  background-color: #dc3545;
  color: white;
}

/* ===== MODAIS ===== */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

.modal-confirmacao,
.modal-desconto {
  background: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  text-align: center;
}

.modal-confirmacao {
  max-width: 400px;
  width: 90%;
}

.modal-desconto {
  max-width: 600px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-confirmacao h3,
.modal-desconto h3 {
  margin: 0 0 15px 0;
  color: #333;
  font-size: 1.3rem;
}

.modal-desconto h3 {
  font-size: 1.5rem;
  margin-bottom: 20px;
}

.modal-confirmacao p {
  margin: 0 0 25px 0;
  color: #666;
  font-size: 1rem;
}

.modal-botoes {
  display: flex;
  gap: 15px;
  justify-content: center;
}

.btn-confirmar,
.btn-cancelar,
.btn-editar {
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.2s;
  color: white;
}

.btn-confirmar {
  background-color: #4CAF50;
}

.btn-confirmar:hover {
  background-color: #b71c1c;
}

.btn-cancelar {
  background-color: #6c757d;
}

.btn-cancelar:hover {
  background-color: #545b62;
}

.btn-editar {
  background-color: #2196F3 !important;
  padding: 8px 16px;
    font-size: 0.9rem;
  margin-right: 10px;
}

.btn-editar:hover {
  background-color: #1976D2 !important;
}

/* ===== FORMULÁRIOS DE DESCONTO ===== */
.produto-selecionado-info {
  background: #E3F2FD;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  border-left: 4px solid #2196F3;
}

.produto-selecionado-info h4 {
  margin: 0 0 10px 0;
  color: #1976D2;
  font-size: 1.1rem;
}

.produto-selecionado-info p {
  margin: 5px 0;
  color: #333;
}

.modal-desconto form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.modal-desconto label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

.modal-desconto input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1rem;
}

/* ===== LAYOUTS ===== */
.linha-dupla {
  display: flex;
  gap: 16px;
}

.campo-metade {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.modal-desconto .linha-dupla {
  gap: 15px;
}

.modal-desconto .campo-metade {
  flex: 1;
}

/* ===== DETALHES DE DESCONTO ===== */
.desconto-detalhes {
  margin: 20px 0;
}

.desconto-info-completa {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
  border-left: 4px solid #2196F3;
}

.desconto-info-completa h4 {
  margin: 0 0 15px 0;
  color: #1976D2;
  font-size: 1.1rem;
}

.info-item {
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 1px solid #e9ecef;
}

.info-item:last-child {
  border-bottom: none;
  margin-bottom: 0;
}

.info-item strong {
  display: block;
  color: #333;
  font-size: 0.9rem;
  margin-bottom: 5px;
}

.info-item p {
  margin: 0;
  color: #666;
  font-size: 1rem;
}

/* ===== STATUS ===== */
.status-ativo {
  color: #28a745;
  font-weight: bold;
}

.status-expirado {
  color: #dc3545;
  font-weight: bold;
}

.status-futuro {
  color: #ffc107;
  font-weight: bold;
}

/* ===== RESPONSIVIDADE ===== */
@media (max-width: 768px) {
  .tudo {
    padding: 20px 15px 0px 15px;
  }
  
  .produtos {
    width: 100%;
    height: auto;
  }
  
  .produtos-header {
    flex-direction: column;
    gap: 15px;
    align-items: stretch;
  }
  
  .produtos-header h3 {
    font-size: 2rem;
  }
  
  .criacao-form {
    width: 95%;
    padding: 20px;
    margin: 10px;
  }
  
  .criacao-form h2 {
    font-size: 2rem;
    margin-bottom: 15px;
  }
  
  .criacao-form label {
    font-size: 1rem;
    margin-bottom: 8px;
  }
  
  .criacao-form input[type="text"],
  .criacao-form textarea,
  .criacao-form input[type="file"],
  .criacao-form input[type="number"],
  .criacao-form select {
    padding: 8px;
    margin-bottom: 15px;
    font-size: 0.9rem;
  }
  
  .criacao-form button {
    padding: 8px 16px;
    font-size: 1rem;
  }
  
  .novo-produto-btn,
  .alternar-modo-btn {
    padding: 8px 16px;
    font-size: 1rem;
  }
  
  .alternar-modo-btn {
    margin-left: 5px;
  }
  
  .input-busca {
    max-width: 100%;
    height: 45px;
  }
  
  .input-busca input {
    font-size: 14px;
  }
  
  .input-busca img {
    height: 18px;
  }
  
  .filtro-categorias {
    flex-direction: column;
    align-items: stretch;
    gap: 6px;
  }
  
  .filtro-estoque,
  .filtro-categoria {
    flex-direction: column;
    align-items: stretch;
    gap: 4px;
  }
  
  .filtro-categorias select {
    padding: 4px 8px;
    font-size: 0.9rem;
  }
  
  .linha-dupla {
    flex-direction: column;
    gap: 12px;
  }
  
  .campo-metade {
    flex: none;
  }
  
  .nome-preco-imagem {
    gap: 8px;
  }
  
  .nome-preco-imagem p {
    font-size: 18px;
  }
  
  .nome-preco-imagem img {
    height: 180px;
    width: 130px;
  }
  
  .produto {
    width: 200px;
    height: auto;
    margin-top: 2vh;
    padding: 8px;
  }
  
  .produto h4 {
    font-size: 14px;
    height: 35px;
  }
  
  .lista {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .BTli {
    gap: 8px;
  }
  
  .BTli button {
    padding: 4px 8px;
    font-size: 0.8rem;
    margin-left: 5px;
  }
  
  .desconto-barra {
    width: 130px;
  }
  
  .modal-desconto .linha-dupla {
    flex-direction: column;
    gap: 10px;
  }
  
  .modal-desconto {
    padding: 20px;
    margin: 10px;
  }
}

@media (max-width: 480px) {
  .tudo {
    padding: 15px 10px 0px 10px;
  }
  
  .produtos-header {
    gap: 10px;
  }
  
  .produtos-header h3 {
    font-size: 1.5rem;
  }
  
  .criacao-form {
    width: 98%;
    padding: 15px;
    margin: 5px;
  }
  
  .criacao-form h2 {
    font-size: 1.5rem;
    margin-bottom: 10px;
  }
  
  .criacao-form label {
    font-size: 0.9rem;
    margin-bottom: 6px;
  }
  
  .criacao-form input[type="text"],
  .criacao-form textarea,
  .criacao-form input[type="file"],
  .criacao-form input[type="number"],
  .criacao-form select {
    padding: 6px;
    margin-bottom: 10px;
    font-size: 0.8rem;
  }
  
  .criacao-form button {
    padding: 6px 12px;
    font-size: 0.9rem;
    margin-bottom: 5px;
  }
  
  .novo-produto-btn,
  .alternar-modo-btn {
    padding: 6px 12px;
    font-size: 0.9rem;
  }
  
  .alternar-modo-btn {
    margin-left: 3px;
  }
  
  .input-busca {
    height: 40px;
  }
  
  .input-busca input {
    font-size: 13px;
  }
  
  .input-busca img {
    height: 16px;
  }
  
  .filtro-categorias {
    gap: 4px;
  }
  
  .filtro-estoque,
  .filtro-categoria {
    gap: 2px;
  }
  
  .filtro-categorias select {
    padding: 3px 6px;
    font-size: 0.8rem;
  }
  
  .linha-dupla {
    gap: 8px;
  }
  
  .nome-preco-imagem p {
    font-size: 16px;
  }
  
  .nome-preco-imagem img {
    height: 150px;
    width: 110px;
  }
  
  .produto {
    width: 180px;
    margin-top: 1vh;
    padding: 6px;
  }
  
  .produto h4 {
    font-size: 13px;
    height: 30px;
  }
  
  .lista {
    grid-template-columns: repeat(1, 1fr);
  }
  
  .BTli {
    gap: 5px;
  }
  
  .BTli button {
    padding: 3px 6px;
    font-size: 0.75rem;
    margin-left: 3px;
  }
  
  .desconto-barra {
    width: 110px;
  }
  
    .modal-confirmacao {
        padding: 20px;
        margin: 20px;
    }
    
    .modal-botoes {
        flex-direction: column;
        gap: 10px;
    }
    
    .btn-confirmar,
    .btn-cancelar {
        padding: 12px 20px;
    }
  
  .modal-desconto {
    padding: 15px;
    margin: 5px;
  }
  
  .modal-desconto h3 {
    font-size: 1.3rem;
  }
  
  .produto-selecionado-info {
    padding: 10px;
  }
}

@media (max-width: 800px) {
  .lista {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 550px) {
  .lista {
    grid-template-columns: repeat(1, 1fr);
  }
}
</style>