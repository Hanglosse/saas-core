<script lang="ts">
  import { onMount } from 'svelte'
  import { supabase } from '$lib/supabaseClient'

  let produtos: any[] = []
  let lotes: any[] = []
  let lotesFiltrados: any[] = []

  let produtoId = ''
  let quantidade: number | '' = ''
  let validade = ''

  let loading = true
  let saving = false
  let error = ''
  let success = ''

  let filtro: 'todos' | 'a-expirar' | 'expirados' = 'todos'

  async function carregarProdutos() {
    const { data, error: err } = await supabase
      .from('produtos')
      .select('id, nome')
      .eq('ativo', true)
      .order('nome')

    if (err) error = err.message
    else produtos = data ?? []
  }

  async function carregarLotes() {
    loading = true

    const { data, error: err } = await supabase
      .from('lotes')
      .select(`
        id,
        quantidade,
        validade,
        created_at,
        produtos (
          nome
        )
      `)
      .order('validade', { ascending: true })

    if (err) {
      error = err.message
    } else {
      lotes = data ?? []
      aplicarFiltro()
    }

    loading = false
  }

  async function criarLote() {
    if (!produtoId || !quantidade || !validade) {
      error = 'Preenche todos os campos.'
      return
    }

    saving = true
    error = ''
    success = ''

    const { error: err } = await supabase.from('lotes').insert({
      produto_id: produtoId,
      quantidade,
      validade
    })

    saving = false

    if (err) {
      error = err.message
      return
    }

    produtoId = ''
    quantidade = ''
    validade = ''
    success = 'Lote criado com sucesso.'
    await carregarLotes()
  }

  function diasPara(validade: string) {
    const hoje = new Date()
    const v = new Date(validade)
    return Math.ceil((v.getTime() - hoje.getTime()) / (1000 * 60 * 60 * 24))
  }

  function estadoValidade(validade: string) {
    const d = diasPara(validade)
    if (d < 0) return 'expirado'
    if (d <= 30) return 'a-expirar'
    return 'ok'
  }

  function aplicarFiltro() {
    if (filtro === 'todos') {
      lotesFiltrados = lotes
    } else {
      lotesFiltrados = lotes.filter(
        (l) => estadoValidade(l.validade) === filtro
      )
    }
  }

  function contar(estado: 'expirado' | 'a-expirar') {
    return lotes.filter((l) => estadoValidade(l.validade) === estado).length
  }

  onMount(async () => {
    await carregarProdutos()
    await carregarLotes()
  })
</script>

<!-- CABEÇALHO -->
<header class="page-header">
  <h1>Lotes</h1>
  <p class="subtitle">
    Gestão de stock por validade
  </p>
</header>

<!-- CRIAR LOTE -->
<section class="card">
  <h2>Novo lote</h2>

  <div class="form">
    <select bind:value={produtoId} disabled={saving}>
      <option value="">Seleciona um produto</option>
      {#each produtos as produto}
        <option value={produto.id}>{produto.nome}</option>
      {/each}
    </select>

    <input
      type="number"
      placeholder="Quantidade"
      bind:value={quantidade}
      min="0"
      disabled={saving}
    />

    <input
      type="date"
      bind:value={validade}
      disabled={saving}
    />

    <button type="button" on:click={criarLote} disabled={saving}>
      {saving ? 'A guardar…' : 'Criar lote'}
    </button>
  </div>
</section>

<!-- FEEDBACK -->
{#if success}
  <p class="feedback success">{success}</p>
{/if}

{#if error}
  <p class="feedback error">{error}</p>
{/if}

<!-- FILTROS -->
<section class="filters">
  <button
    class:active={filtro === 'todos'}
    on:click={() => { filtro = 'todos'; aplicarFiltro() }}
  >
    Todos ({lotes.length})
  </button>

  <button
    class:active={filtro === 'a-expirar'}
    on:click={() => { filtro = 'a-expirar'; aplicarFiltro() }}
  >
    A expirar ({contar('a-expirar')})
  </button>

  <button
    class:active={filtro === 'expirados'}
    on:click={() => { filtro = 'expirados'; aplicarFiltro() }}
  >
    Expirados ({contar('expirado')})
  </button>
</section>
  
<!-- LISTA -->
<section class="card">
  <h2>Lista de lotes</h2>

  {#if loading}
    <p class="loading">A carregar lotes…</p>

  {:else if lotesFiltrados.length === 0}
    <p class="empty">
      Nenhum lote para este filtro.
    </p>


  {:else}
    <ul class="lista">
      {#each lotesFiltrados as lote}
        <li class={estadoValidade(lote.validade)}>
          <div class="info">
            <strong>{lote.produtos.nome}</strong>
            <span>
              Quantidade: {lote.quantidade}
            </span>
            <span>
              Validade: {lote.validade}
            </span>
          </div>

          <span class="badge">
            {#if estadoValidade(lote.validade) === 'expirado'}
              🔴 Expirado ({diasPara(lote.validade)} dias)
            {:else if estadoValidade(lote.validade) === 'a-expirar'}
              🟠 A expirar ({diasPara(lote.validade)} dias)
            {:else}
              🟢 OK ({diasPara(lote.validade)} dias)
            {/if}
          </span>
        </li>
      {/each}
    </ul>
  {/if}
</section>

<style>
  .page-header {
    margin-bottom: 1.5rem;
  }

  .subtitle {
    color: #666;
    margin-top: 0.25rem;
  }

  .card {
    border: 1px solid #ddd;
    border-radius: 6px;
    padding: 1rem;
    margin-bottom: 1.5rem;
  }

  .form {
    display: grid;
    gap: 0.75rem;
  }

  .filters {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  .filters button.active {
    font-weight: bold;
  }

  .lista {
    list-style: none;
    padding: 0;
    margin: 0;
  }

  .lista li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.75rem 0;
    border-bottom: 1px solid #eee;
  }

  .lista li:last-child {
    border-bottom: none;
  }

  .badge {
    font-size: 0.85rem;
    white-space: nowrap;
  }

  .expirado .badge {
    color: #c62828;
  }

  .a-expirar .badge {
    color: #ef6c00;
  }

  .ok .badge {
    color: #2e7d32;
  }

  .feedback {
    margin-bottom: 1rem;
  }

  .success {
    color: #2e7d32;
  }

  .error {
    color: #c62828;
  }

  .loading,
  .empty {
    opacity: 0.6;
  }
</style>
