<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let lotes: any[] = [];
  let movimentos: any[] = [];

  let loteId = '';
  let tipo: 'entrada' | 'saida' = 'entrada';
  let quantidade: number | '' = '';

  let loading = true;
  let saving = false;
  let error = '';
  let success = '';

  async function carregarLotes() {
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
  }

  async function carregarMovimentos() {
    const { data, error: err } = await supabase
      .from('movimentos')
      .select(`
        id,
        tipo,
        quantidade,
        created_at,
        lotes (
          validade,
          produtos (
            nome
          )
        )
      `)
      .order('created_at', { ascending: false })
      .limit(20);

    if (err) error = err.message;
    else movimentos = data ?? [];

    loading = false;
  }

  async function criarMovimento() {
    if (!loteId || !quantidade) {
      error = 'Preenche todos os campos.';
      return;
    }

    const lote = lotes.find((l) => l.id === loteId);
    if (!lote) {
      error = 'Lote inválido.';
      return;
    }

    const novaQuantidade =
      tipo === 'entrada'
        ? lote.quantidade + quantidade
        : lote.quantidade - quantidade;

    if (novaQuantidade < 0) {
      error = 'Quantidade insuficiente no lote.';
      return;
    }

    saving = true;
    error = '';
    success = '';

    // 1️⃣ Atualizar lote
    const { error: errLote } = await supabase
      .from('lotes')
      .update({ quantidade: novaQuantidade })
      .eq('id', loteId);

    if (errLote) {
      saving = false;
      error = errLote.message;
      return;
    }

    // 2️⃣ Criar movimento
    const { error: errMov } = await supabase
      .from('movimentos')
      .insert({
        lote_id: loteId,
        tipo,
        quantidade
      });

    saving = false;

    if (errMov) {
      error = errMov.message;
      return;
    }

    loteId = '';
    quantidade = '';
    tipo = 'entrada';

    success = 'Movimento registado com sucesso.';

    await carregarLotes();
    await carregarMovimentos();
  }

  onMount(async () => {
    await carregarLotes();
    await carregarMovimentos();
  });
</script>

<h1>Movimentos de Stock</h1>

<section>
  <h2>Novo Movimento</h2>

  <select bind:value={loteId} disabled={saving}>
    <option value="">Seleciona um lote</option>
    {#each lotes as lote}
      <option value={lote.id}>
        {lote.produtos.nome} —
        Val: {lote.validade} —
        Qtd: {lote.quantidade}
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

  <button type="button" on:click={criarMovimento} disabled={saving}>
    {saving ? 'A guardar…' : 'Registar movimento'}
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
  <h2>Últimos Movimentos</h2>

  {#if loading}
    <p>A carregar movimentos…</p>

  {:else if movimentos.length === 0}
    <p>Nenhum movimento registado.</p>

  {:else}
    <ul>
      {#each movimentos as m}
        <li>
          <strong>{m.lotes.produtos.nome}</strong> —
          {m.tipo.toUpperCase()} —
          Qtd: {m.quantidade} —
          {new Date(m.created_at).toLocaleString()}
        </li>
      {/each}
    </ul>
  {/if}
</section>
