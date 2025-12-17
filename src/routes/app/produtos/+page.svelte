<script lang="ts">
  export const ssr = false;

  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let produtos: any[] = [];
  let loading = true;
  let error = '';
  let success = '';

  let nome = '';
  let descricao = '';
  let saving = false;

  async function carregarProdutos() {
    loading = true;
    error = '';

    const { data, error: err } = await supabase
      .from('produtos')
      .select('*')
      .order('created_at', { ascending: false });

    if (err) {
      error = err.message;
    } else {
      produtos = data ?? [];
    }

    loading = false;
  }

  async function criarProduto() {
    if (!nome) {
      error = 'O nome é obrigatório.';
      return;
    }

    saving = true;
    error = '';
    success = '';

    const { error: err } = await supabase.from('produtos').insert({
      nome,
      descricao
    });

    saving = false;

    if (err) {
      error = err.message;
      return;
    }

    nome = '';
    descricao = '';
    success = 'Produto criado com sucesso.';
    await carregarProdutos();
  }

  async function alternarAtivo(produto) {
    const confirmado = confirm(
      `Tens a certeza que queres ${produto.ativo ? 'desativar' : 'ativar'} este produto?`
    );

    if (!confirmado) return;

    error = '';
    success = '';

    const { error: err } = await supabase
      .from('produtos')
      .update({ ativo: !produto.ativo })
      .eq('id', produto.id);

    if (err) {
      error = err.message;
    } else {
      success = 'Estado do produto atualizado.';
      await carregarProdutos();
    }
  }

  onMount(() => {
    carregarProdutos();
  });
</script>

<h1>Produtos</h1>

<section>
  <h2>Novo Produto</h2>

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
    {saving ? 'A guardar…' : 'Criar'}
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
  <h2>Lista de Produtos</h2>

  {#if loading}
    <p>A carregar produtos…</p>

  {:else if produtos.length === 0}
    <p>Nenhum produto criado.</p>

  {:else}
    <ul>
      {#each produtos as produto}
        <li>
          <strong>{produto.nome}</strong>
          {#if !produto.ativo}
            <em> (inativo)</em>
          {/if}

          {#if produto.descricao}
            <div>{produto.descricao}</div>
          {/if}

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
