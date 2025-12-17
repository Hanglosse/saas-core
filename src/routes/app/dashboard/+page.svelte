<script lang="ts">
  import { onMount } from 'svelte'
  import { supabase } from '$lib/supabaseClient'

  let loading = true
  let error = ''

  let totalProdutos = 0
  let totalLotes = 0
  let stockTotal = 0
  let expirados = 0
  let aExpirar = 0

  function diasPara(validade: string) {
    const hoje = new Date()
    const v = new Date(validade)
    return Math.ceil((v.getTime() - hoje.getTime()) / (1000 * 60 * 60 * 24))
  }

  async function carregarDashboard() {
    loading = true
    error = ''

    const { count: produtosCount, error: errProdutos } = await supabase
      .from('produtos')
      .select('*', { count: 'exact', head: true })
      .eq('ativo', true)

    if (errProdutos) {
      error = errProdutos.message
      loading = false
      return
    }

    totalProdutos = produtosCount ?? 0

    const { data: lotes, error: errLotes } = await supabase
      .from('lotes')
      .select('quantidade, validade')

    if (errLotes) {
      error = errLotes.message
      loading = false
      return
    }

    totalLotes = lotes?.length ?? 0
    stockTotal = 0
    expirados = 0
    aExpirar = 0

    for (const lote of lotes ?? []) {
      stockTotal += lote.quantidade

      const d = diasPara(lote.validade)
      if (d < 0) expirados++
      else if (d <= 30) aExpirar++
    }

    loading = false
  }

  onMount(carregarDashboard)
</script>

<header class="page-header">
  <h1>Dashboard</h1>
  <p class="subtitle">Visão geral do stock e validade</p>
</header>

{#if loading}
  <p class="loading">A carregar dados…</p>

{:else if error}
  <p class="error">{error}</p>

{:else}

  <!-- KPIs -->
  <section class="kpis">
    <div class="kpi">
      <span>Produtos ativos</span>
      <strong>{totalProdutos}</strong>
    </div>

    <div class="kpi">
      <span>Lotes</span>
      <strong>{totalLotes}</strong>
    </div>

    <div class="kpi">
      <span>Stock total</span>
      <strong>{stockTotal}</strong>
    </div>

    <div class="kpi danger">
      <span>Expirados</span>
      <strong>{expirados}</strong>
    </div>

    <div class="kpi warning">
      <span>A expirar</span>
      <strong>{aExpirar}</strong>
    </div>
  </section>

  <!-- ESTADO GLOBAL -->
  <section class="status">
    {#if expirados > 0}
      <p class="error">
        🔴 Existem <strong>{expirados}</strong> lotes expirados.
        <a href="/app/alertas">Ver alertas</a>
      </p>

    {:else if aExpirar > 0}
      <p class="warning">
        🟠 Existem <strong>{aExpirar}</strong> lotes a expirar em breve.
        <a href="/app/alertas">Ver alertas</a>
      </p>

    {:else}
      <p class="success">
        ✅ Tudo em dia. Nenhuma ação necessária.
      </p>
    {/if}
  </section>

{/if}

<style>
  .page-header {
    margin-bottom: 1.5rem;
  }

  .subtitle {
    color: #666;
    margin-top: 0.25rem;
  }

  .loading {
    opacity: 0.6;
  }

  .kpis {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
    gap: 1rem;
    margin-bottom: 1.5rem;
  }

  .kpi {
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 6px;
  }

  .kpi span {
    display: block;
    color: #666;
    font-size: 0.85rem;
  }

  .kpi strong {
    font-size: 1.6rem;
  }

  .kpi.warning {
    border-color: #f0c36d;
  }

  .kpi.danger {
    border-color: #e57373;
  }

  .status p {
    margin-top: 0.5rem;
  }

  .error {
    color: #c62828;
  }

  .warning {
    color: #ef6c00;
  }

  .success {
    color: #2e7d32;
  }
</style>
