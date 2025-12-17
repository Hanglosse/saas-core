<script lang="ts">
  import { onMount } from 'svelte'
  import { supabase } from '$lib/supabaseClient'

  let lotes: any[] = []
  let movimentos: any[] = []

  let loteId = ''
  let tipo: 'entrada' | 'saida' = 'entrada'
  let quantidade: number | '' = ''

  let loading = true
  let saving = false
  let error = ''
  let success = ''

  async function carregarLotes() {
    const { data, error: err } = await supabase
      .from('lotes')
      .select(`
        id,
        quantidade,
        validade,
        produtos ( nome )
      `)
      .order('validade')

    if (!err) lotes = data ?? []
  }

  async function carregarMovimentos() {
    const { data, error: err } = await supabase
      .from('movimentos')
      .select(`
        tipo,
        quantidade,
        created_at,
        lotes (
          validade,
          produtos ( nome )
        )
      `)
      .order('created_at', { ascending: false })
      .limit(25)

    if (err) error = err.message
    else movimentos = data ?? []

    loading = false
  }

  async function criarMovimento() {
    if (!loteId || !quantidade) {
      error = 'Preenche todos os campos.'
      return
    }

    const lote = lotes.find((l) => l.id === loteId)
    if (!lote) {
      error = 'Lote inválido.'
      return
    }

    const novaQuantidade =
      tipo === 'entrada'
        ? lote.quantidade + quantidade
        : lote.quantidade - quantidade

    if (novaQuantidade < 0) {
      error = 'Stock insuficiente.'
      return
    }

    saving = true
    error = ''
    success = ''

    // Atualizar lote
    const { error: errLote } = await supabase
      .from('lotes')
      .update({ quantidade: novaQuantidade })
      .eq('id', loteId)

    if (errLote) {
      saving = false
      error = errLote.message
      return
    }

    // Criar movimento
    const { error: errMov } = await supabase
      .from('movimentos')
      .insert({
        lote_id: loteId,
        tipo,
        quantidade
      })

    saving = false

    if (errMov) {
      error = errMov.message
      return
    }

    loteId = ''
    quantidade = ''
    tipo = 'entrada'
    success = 'Movimento registado com sucesso.'

    await carregarLotes()
    await carregarMovimentos()
  }

  onMount(async () => {
    await carregarLotes()
    await carregarMovimentos()
  })
</script>

<!-- CABEÇALHO -->
<header class="page-header">
  <h1>Movimentos</h1>
  <p class="subtitle">
    Registo de entradas e saídas de stock
  </p>
</header>

<!-- REGISTO -->
<section class="card">
  <h2>Novo movimento</h2>

  <div class="form">
    <select bind:value={loteId} disabled={saving}>
      <option value="">Seleciona um lote</option>
      {#each lotes as lote}
        <option value={lote.id}>
          {lote.produtos.nome} — Val: {lote.validade} — Qtd: {lote.quantidade}
        </option>
      {/each}
    </select>

    <select bind:value={tipo} disabled={saving}>
      <option value="entrada">Entrada</option>
      <option value="saida">Saída</option>
    </select>

    <input
      type="number"
      min="1"
      placeholder="Quantidade"
      bind:value={quantidade}
      disabled={saving}
    />

    <button on:click={criarMovimento} disabled={saving}>
      {saving ? 'A guardar…' : 'Registar movimento'}
    </button>
  </div>

  {#if success}
    <p class="success">{success}</p>
  {/if}

  {#if error}
    <p class="error">{error}</p>
  {/if}
</section>

<!-- HISTÓRICO -->
<section class="card">
  <h2>Últimos movimentos</h2>

  {#if loading}
    <p class="loading">A carregar movimentos…</p>

  {:else if movimentos.length === 0}
    <p class="empty">Nenhum movimento registado.</p>

  {:else}
    <ul class="lista">
      {#each movimentos as m}
        <li class={m.tipo}>
          <div>
            <strong>{m.lotes.produtos.nome}</strong>
            <span>
              {new Date(m.created_at).toLocaleString()}
            </span>
          </div>

          <span class="badge">
            {m.tipo === 'entrada' ? '+' : '-'}{m.quantidade}
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
    background: white;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1.5rem;
    border: 1px solid #e5e7eb;
  }

  .form {
    display: grid;
    gap: 0.75rem;
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

  .lista li.entrada .badge {
    color: #2e7d32;
  }

  .lista li.saida .badge {
    color: #c62828;
  }

  .badge {
    font-weight: bold;
  }

  .loading,
  .empty {
    opacity: 0.6;
  }

  .success {
    color: #2e7d32;
    margin-top: 0.75rem;
  }

  .error {
    color: #c62828;
    margin-top: 0.75rem;
  }
</style>
