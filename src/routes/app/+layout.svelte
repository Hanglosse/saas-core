<script lang="ts">
  import { page } from '$app/stores'
  import { supabase } from '$lib/supabaseClient'
  import { onMount } from 'svelte'

  let alertasCount = 0

  async function logout() {
    await supabase.auth.signOut()
    window.location.href = '/login'
  }

  $: path = $page.url.pathname

  /* =========================
     ALERTAS — CONTADOR GLOBAL
     ========================= */

  function diasPara(validade: string) {
    const hoje = new Date()
    hoje.setHours(0, 0, 0, 0)

    const [ano, mes, dia] = validade.split('-').map(Number)
    const v = new Date(ano, mes - 1, dia)
    v.setHours(0, 0, 0, 0)

    return Math.floor((v.getTime() - hoje.getTime()) / 86400000)
  }

  function estado(validade: string) {
    const d = diasPara(validade)
    if (d < 0) return 'expirado'
    if (d <= 30) return 'a-expirar'
    return 'ok'
  }

  async function carregarAlertas() {
    const { data, error } = await supabase
      .from('lotes')
      .select('validade')

    if (!error && data) {
      alertasCount = data.filter(
        (l) => estado(l.validade) !== 'ok'
      ).length
    }
  }

  onMount(carregarAlertas)
</script>

<div class="app">
  <!-- SIDEBAR -->
  <aside class="sidebar">
    <h2 class="logo">SaaS</h2>

    <nav>
      <a href="/app/dashboard" class:active={path === '/app/dashboard'}>
        📊 Dashboard
      </a>

      <a href="/app/produtos" class:active={path.startsWith('/app/produtos')}>
        📦 Produtos
      </a>

      <a href="/app/lotes" class:active={path.startsWith('/app/lotes')}>
        🧾 Lotes
      </a>

      <a href="/app/movimentos" class:active={path.startsWith('/app/movimentos')}>
        🔄 Movimentos
      </a>
      <a href="/app/relatorios" class:active={path.startsWith('/app/relatorios')}>
        📊 Relatórios
      </a>

      <a href="/app/alertas" class:active={path.startsWith('/app/alertas')}>
        🚨 Alertas
        {#if alertasCount > 0}
          <span class="badge-alerta">
            {alertasCount}
          </span>
        {/if}
      </a>
    </nav>

    <button class="logout" on:click={logout}>
      Sair
    </button>
  </aside>

  <!-- CONTEÚDO -->
  <main class="content">
    <slot />
  </main>
</div>

<style>
  /* =========================
     BASE
     ========================= */

  .app {
    display: grid;
    grid-template-columns: 220px 1fr;
    min-height: 100vh;
    background: #f4f6f8;
  }

  /* =========================
     SIDEBAR
     ========================= */

  .sidebar {
    background: #ffffff;
    border-right: 1px solid #e0e0e0;
    padding: 1rem;

    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .logo {
    margin: 0;
    font-size: 1.2rem;
  }

  nav {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  nav a {
    display: flex;
    justify-content: space-between;
    align-items: center;

    text-decoration: none;
    padding: 0.5rem 0.75rem;
    border-radius: 6px;
    color: #333;
    opacity: 0.75;
  }

  nav a:hover {
    background: #f0f0f0;
    opacity: 1;
  }

  nav a.active {
    background: #e3f2fd;
    color: #0d47a1;
    font-weight: 600;
    opacity: 1;
  }

  .badge-alerta {
    background: #c62828;
    color: #fff;
    font-size: 0.75rem;
    font-weight: 600;
    padding: 2px 6px;
    border-radius: 999px;
  }

  .logout {
    margin-top: auto;
    background: none;
    border: none;
    color: #c62828;
    cursor: pointer;
    text-align: left;
    padding: 0.5rem 0;
  }

  /* =========================
     CONTEÚDO
     ========================= */

  .content {
    padding: 1.5rem;
  }

  /* =========================
     CARDS
     ========================= */

  :global(.card) {
    background: #ffffff;
    border-radius: 8px;
    padding: 1rem;
    margin-bottom: 1.5rem;
    box-shadow:
      0 1px 2px rgba(0, 0, 0, 0.04),
      0 4px 12px rgba(0, 0, 0, 0.06);
  }
</style>
