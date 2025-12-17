<script lang="ts">
  import { onMount } from 'svelte'
  import { supabase } from '$lib/supabaseClient'

  let alertas: any[] = []
  let loading = true
  let error = ''

  function diasPara(validade: string) {
    const hoje = new Date()
    const v = new Date(validade)
    return Math.ceil((v.getTime() - hoje.getTime()) / (1000 * 60 * 60 * 24))
  }

  function estado(validade: string) {
    const d = diasPara(validade)
    if (d < 0) return 'expirado'
    if (d <= 30) return 'a-expirar'
    return 'ok'
  }

  async function carregarAlertas() {
    loading = true
    error = ''

    const { data, error: err } = await supabase
      .from('lotes')
      .select(`
        id,
        quantidade,
        validade,
        produtos (
          nome
        )
      `)
      .order('validade', { ascending: true })

    if (err) {
      error = err.message
    } else {
      alertas = (data ?? []).filter(
        (l) => estado(l.validade) !== 'ok'
      )
    }

    loading = false
  }

  onMount(carregarAlertas)
</script>

<!-- CABEÇALHO -->
<header class="page-header">
  <h1>Alertas</h1>
  <p class="subtitle">
    Lotes expirados ou prestes a expirar
  </p>
</header>

{#if loading}
  <p class="loading">A carregar alertas…</p>

{:else if error}
  <p class="error">{error}</p>

{:else if alertas.length === 0}
  <p class="success">
    ✅ Tudo em dia. Nenhum alerta ativo.
  </p>

{:else}
  <ul class="lista">
    {#each alertas as alerta}
      <li class={estado(alerta.validade)}>
        <div class="info">
          <strong>{alerta.produtos.nome}</strong>
          <span>Quantidade: {alerta.quantidade}</span>
          <span>Validade: {alerta.validade}</span>
        </div>

        <span class="badge">
          {#if estado(alerta.validade) === 'expirado'}
            🔴 Expirado ({diasPara(alerta.validade)} dias)
          {:else}
            🟠 A expirar ({diasPara(alerta.validade)} dias)
          {/if}
        </span>
      </li>
    {/each}
  </ul>
{/if}

<style>
  .page-header {
    margin-bottom: 1.5rem;
  }

  .subtitle {
    color: #666;
    margin-top: 0.25rem;
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

  .badge {
    white-space: nowrap;
    font-size: 0.85rem;
  }

  .expirado .badge {
    color: #c62828;
  }

  .a-expirar .badge {
    color: #ef6c00;
  }

  .loading {
    opacity: 0.6;
  }

  .success {
    color: #2e7d32;
  }

  .error {
    color: #c62828;
  }
</style>
