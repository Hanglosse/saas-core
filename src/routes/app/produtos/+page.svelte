<script lang="ts">
  import { onMount } from 'svelte'
  import { supabase } from '$lib/supabaseClient'

  let produtos: any[] = []
  let loading = true
  let error = ''
  let success = ''

  let nome = ''
  let descricao = ''
  let saving = false

  async function carregarProdutos() {
    loading = true
    error = ''

    const { data, error: err } = await supabase
      .from('produtos')
      .select('*')
      .order('created_at', { ascending: false })

    if (err) {
      error = err.message
    } else {
      produtos = data ?? []
    }

    loading = false
  }

  async function criarProduto() {
    if (!nome) {
      error = 'O nome do produto é obrigatório.'
      return
    }

    saving = true
    error = ''
    success = ''

    const { error: err } = await supabase.from('produtos').insert({
      nome,
      descricao
    })

    saving = false

    if (err) {
      error = err.message
      return
    }

    nome = ''
    descricao = ''
    success = 'Produto criado com sucesso.'
    await carregarProdutos()
  }

  async function alternarAtivo(produto) {
    const confirmado = confirm(
      `Tens a certeza que queres ${produto.ativo ? 'desativar' : 'ativar'} este produto?`
    )

    if (!confirmado) return

    error = ''
    success = ''

    const { error: err } = await supabase
      .from('produtos')
      .update({ ativo: !produto.ativo })
      .eq('id', produto.id)

    if (err) {
      error = err.message
    } else {
      success = 'Estado do produto atualizado.'
      await carregarProdutos()
    }
  }

  onMount(carregarProdutos)
</script>

<!-- CABEÇALHO -->
<header class="page-header">
  <h1>Produtos</h1>
  <p class="subtitle">
    Gestão de produtos ativos e inativos
  </p>
</header>

<!-- FORMULÁRIO -->
<section class="card">
  <h2>Novo produto</h2>

  <div class="form">
    <input
      placeholder="Nome do produto"
      bind:value={nome}
      disabled={saving}
    />

    <textarea
      placeholder="Descrição (opcional)"
      bind:value={descricao}
      disabled={saving}
    ></textarea>

    <button type="button" on:click={criarProduto} disabled={saving}>
      {saving ? 'A guardar…' : 'Criar produto'}
    </button>
  </div>
</section>

<!-- FEEDBACK -->
{#if success}
  <p class="success">{success}</p>
{/if}

{#if error}
  <p class="error">{error}</p>
{/if}

<!-- LISTA -->
<section class="card">
  <h2>Lista de produtos</h2>

  {#if loading}
    <p class="loading">A carregar produtos…</p>

  {:else if produtos.length === 0}
    <p class="empty">
      Ainda não existem produtos.<br />
      Cria o primeiro produto acima.
    </p>

  {:else}
    <ul class="lista">
      {#each produtos as produto}
        <li class:inativo={!produto.ativo}>
          <div class="info">
            <strong>{produto.nome}</strong>
            {#if produto.descricao}
              <span>{produto.descricao}</span>
            {/if}
          </div>

          <button
            type="button"
            on:click={() => alternarAtivo(produto)}
          >
            {produto.ativo ? 'Desativar' : 'Ativar'}
          </button>
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

  .loading {
    opacity: 0.6;
  }

  .empty {
    opacity: 0.6;
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

  .info span {
    display: block;
    color: #666;
    font-size: 0.85rem;
  }

  .inativo strong {
    opacity: 0.6;
  }

  .success {
    color: #2e7d32;
    margin-bottom: 1rem;
  }

  .error {
    color: #c62828;
    margin-bottom: 1rem;
  }
</style>
