<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let loading = true;
  let error = '';

  let totalProdutos = 0;
  let totalLotes = 0;
  let stockTotal = 0;
  let expirados = 0;
  let aExpirar = 0;

  function diasPara(validade: string) {
    const hoje = new Date();
    const v = new Date(validade);
    return Math.ceil((v.getTime() - hoje.getTime()) / (1000 * 60 * 60 * 24));
  }

  async function carregarDashboard() {
    // Produtos ativos
    const { count: produtosCount, error: errProdutos } = await supabase
      .from('produtos')
      .select('*', { count: 'exact', head: true })
      .eq('ativo', true);

    if (errProdutos) {
      error = errProdutos.message;
      return;
    }

    totalProdutos = produtosCount ?? 0;

    // Lotes
    const { data: lotes, error: errLotes } = await supabase
      .from('lotes')
      .select('quantidade, validade');

    if (errLotes) {
      error = errLotes.message;
      return;
    }

    totalLotes = lotes?.length ?? 0;
    stockTotal = 0;
    expirados = 0;
    aExpirar = 0;

    for (const lote of lotes ?? []) {
      stockTotal += lote.quantidade;

      const d = diasPara(lote.validade);
      if (d < 0) expirados++;
      else if (d <= 30) aExpirar++;
    }

    loading = false;
  }

  onMount(async () => {
    await carregarDashboard();
  });
</script>

<h1>Dashboard</h1>

{#if loading}
  <p>A carregar dados…</p>

{:else if error}
  <p class="error">{error}</p>

{:else}
  <section
    style="
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 1rem;
    "
  >
    <div>
      <h3>Produtos ativos</h3>
      <p><strong>{totalProdutos}</strong></p>
    </div>

    <div>
      <h3>Lotes</h3>
      <p><strong>{totalLotes}</strong></p>
    </div>

    <div>
      <h3>Stock total</h3>
      <p><strong>{stockTotal}</strong></p>
    </div>

    <div>
      <h3 style="color: red">Expirados</h3>
      <p><strong>{expirados}</strong></p>
    </div>

    <div>
      <h3 style="color: orange">A expirar</h3>
      <p><strong>{aExpirar}</strong></p>
    </div>
  </section>

  <section>
    {#if expirados > 0}
      <p class="error">
        🔴 Existem <strong>{expirados}</strong> lotes expirados.
        <a href="/app/alertas">Ver alertas</a>
      </p>

    {:else if aExpirar > 0}
      <p style="color: orange">
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
