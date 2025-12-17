<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let alertas: any[] = [];
  let loading = true;
  let error = '';

  function diasPara(validade: string) {
    const hoje = new Date();
    const v = new Date(validade);
    return Math.ceil((v.getTime() - hoje.getTime()) / (1000 * 60 * 60 * 24));
  }

  function estado(validade: string) {
    const d = diasPara(validade);
    if (d < 0) return 'expirado';
    if (d <= 30) return 'a-expirar';
    return 'ok';
  }

  async function carregarAlertas() {
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

    if (err) {
      error = err.message;
    } else {
      alertas = (data ?? []).filter(
        (l) => estado(l.validade) !== 'ok'
      );
    }

    loading = false;
  }

  onMount(async () => {
    await carregarAlertas();
  });
</script>

<h1>Alertas</h1>

<p>
  Aqui estão listados apenas os lotes que
  <strong>expiraram</strong> ou
  <strong>estão prestes a expirar</strong>.
</p>

{#if loading}
  <p>A carregar alertas…</p>

{:else if error}
  <p style="color: red">{error}</p>

{:else if alertas.length === 0}
  <p style="color: green">
    ✅ Tudo em dia. Nenhum alerta ativo.
  </p>

{:else}
  <ul>
    {#each alertas as alerta}
      <li>
        <strong>{alerta.produtos.nome}</strong> —
        Quantidade: {alerta.quantidade} —
        Validade: {alerta.validade} —

        {#if estado(alerta.validade) === 'expirado'}
          <strong style="color: red">
            EXPIRADO ({diasPara(alerta.validade)} dias)
          </strong>
        {:else}
          <strong style="color: orange">
            A expirar ({diasPara(alerta.validade)} dias)
          </strong>
        {/if}
      </li>
    {/each}
  </ul>
{/if}
