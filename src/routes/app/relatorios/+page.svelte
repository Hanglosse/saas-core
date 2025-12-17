<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let lotes: any[] = [];
  let loading = true;
  let error = '';

  function diasPara(validade: string) {
    const hoje = new Date();
    const v = new Date(validade);
    return Math.ceil((v.getTime() - hoje.getTime()) / (1000 * 60 * 60 * 24));
  }

  function estado(validade: string) {
    const d = diasPara(validade);
    if (d < 0) return 'EXPIRADO';
    if (d <= 30) return 'A EXPIRAR';
    return 'OK';
  }

  async function carregarRelatorio() {
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
      .order('validade', { ascending: true });

    if (err) error = err.message;
    else lotes = data ?? [];

    loading = false;
  }

  function exportarCSV() {
    const headers = [
      'Produto',
      'Quantidade',
      'Validade',
      'Estado'
    ];

    const linhas = lotes.map((l) => [
      l.produtos.nome,
      l.quantidade,
      l.validade,
      estado(l.validade)
    ]);

    const csv =
      [headers, ...linhas]
        .map((row) => row.join(';'))
        .join('\n');

    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);

    const a = document.createElement('a');
    a.href = url;
    a.download = 'relatorio_lotes.csv';
    a.click();

    URL.revokeObjectURL(url);
  }

  onMount(async () => {
    await carregarRelatorio();
  });
</script>

<h1>Relatórios</h1>

<p>
  Relatório atual de lotes, quantidades e validade.
</p>

<button type="button" on:click={exportarCSV} disabled={loading || lotes.length === 0}>
  Exportar CSV
</button>

<hr />

{#if loading}
  <p>A carregar relatório…</p>

{:else if error}
  <p style="color: red">{error}</p>

{:else if lotes.length === 0}
  <p>Nenhum dado disponível.</p>

{:else}
  <table border="1" cellpadding="6">
    <thead>
      <tr>
        <th>Produto</th>
        <th>Quantidade</th>
        <th>Validade</th>
        <th>Estado</th>
      </tr>
    </thead>
    <tbody>
      {#each lotes as l}
        <tr>
          <td>{l.produtos.nome}</td>
          <td>{l.quantidade}</td>
          <td>{l.validade}</td>
          <td>
            {#if estado(l.validade) === 'EXPIRADO'}
              <strong style="color: red">EXPIRADO</strong>
            {:else if estado(l.validade) === 'A EXPIRAR'}
              <strong style="color: orange">A EXPIRAR</strong>
            {:else}
              <span style="color: green">OK</span>
            {/if}
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
{/if}
