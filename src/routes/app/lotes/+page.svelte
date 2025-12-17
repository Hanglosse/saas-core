<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let produtos: any[] = [];
  let lotes: any[] = [];
  let lotesFiltrados: any[] = [];

  let produtoId = '';
  let quantidade: number | '' = '';
  let validade = '';

  let loading = true;
  let saving = false;
  let error = '';
  let success = '';

  let filtro: 'todos' | 'a-expirar' | 'expirados' = 'todos';

  async function carregarProdutos() {
    const { data, error: err } = await supabase
      .from('produtos')
      .select('id, nome')
      .eq('ativo', true)
      .order('nome');

    if (err) error = err.message;
    else produtos = data ?? [];
  }

  async function carregarLotes() {
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
      .order('validade', { ascending: true });

    if (err) error = err.message;
    else {
      lotes = data ?? [];
      aplicarFiltro();
    }

    loading = false;
  }

  async function criarLote() {
    if (!produtoId || !quantidade || !validade) {
      error = 'Preenche todos os campos.';
      return;
    }

    saving = true;
    error = '';
    success = '';

    const { error: err } = await supabase.from('lotes').insert({
      produto_id: produtoId,
      quantidade,
      validade
    });

    saving = false;

    if (err) {
      error = err.message;
      return;
    }

    produtoId = '';
    quantidade = '';
    validade = '';
    success = 'Lote criado com sucesso.';
    await carregarLotes();
  }

  function diasPara(validade: string) {
    const hoje = new Date();
    const v = new Date(validade);
    return Math.ceil((v.getTime() - hoje.getTime()) / (1000 * 60 * 60 * 24));
  }

  function estadoValidade(validade: string) {
    const d = diasPara(validade);
    if (d < 0) return 'expirado';
    if (d <= 30) return 'a-expirar';
    return 'ok';
  }

  function aplicarFiltro() {
    if (filtro === 'todos') {
      lotesFiltrados = lotes;
    } else {
      lotesFiltrados = lotes.filter(
        (l) => estadoValidade(l.validade) === filtro
      );
    }
  }

  function contar(estado: 'expirado' | 'a-expirar') {
    return lotes.filter((l) => estadoValidade(l.validade) === estado).length;
  }

  onMount(async () => {
    await carregarProdutos();
    await carregarLotes();
  });
</script>

<h1>Lotes</h1>

<section>
  <h2>Novo Lote</h2>

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

  <input type="date" bind:value={validade} disabled={saving} />

  <button type="button" on:click={criarLote} disabled={saving}>
    {saving ? 'A guardar…' : 'Criar Lote'}
  </button>
</section>

{#if success}
  <p style="color: green">{success}</p>
{/if}

{#if error}
  <p style="color: red">{error}</p>
{/if}

<hr />

<section>
  <h2>Filtros</h2>

  <button type="button" on:click={() => { filtro = 'todos'; aplicarFiltro(); }}>
    Todos ({lotes.length})
  </button>

  <button type="button" on:click={() => { filtro = 'a-expirar'; aplicarFiltro(); }}>
    A expirar ({contar('a-expirar')})
  </button>

  <button type="button" on:click={() => { filtro = 'expirados'; aplicarFiltro(); }}>
    Expirados ({contar('expirado')})
  </button>
</section>

<hr />

<section>
  <h2>Lista de Lotes</h2>

  {#if loading}
    <p>A carregar lotes…</p>

  {:else if lotesFiltrados.length === 0}
    <p>Nenhum lote para este filtro.</p>

  {:else}
    <ul>
      {#each lotesFiltrados as lote}
        <li>
          <strong>{lote.produtos.nome}</strong> —
          Quantidade: {lote.quantidade} —
          Validade: {lote.validade} —
          {#if estadoValidade(lote.validade) === 'expirado'}
            <strong style="color: red">
              EXPIRADO ({diasPara(lote.validade)} dias)
            </strong>
          {:else if estadoValidade(lote.validade) === 'a-expirar'}
            <strong style="color: orange">
              A expirar ({diasPara(lote.validade)} dias)
            </strong>
          {:else}
            <span style="color: green">
              OK ({diasPara(lote.validade)} dias)
            </span>
          {/if}
        </li>
      {/each}
    </ul>
  {/if}
</section>
