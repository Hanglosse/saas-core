<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let session: any = undefined;

  const links = [
    { href: '/app/dashboard', label: 'Dashboard' },
    { href: '/app/produtos', label: 'Produtos' },
    { href: '/app/lotes', label: 'Lotes' },
    { href: '/app/movimentos', label: 'Movimentos' },
    { href: '/app/alertas', label: 'Alertas' },
    { href: '/app/relatorios', label: 'Relatórios' },
    { href: '/app/configuracoes', label: 'Configurações' }
  ];

  onMount(async () => {
    const { data } = await supabase.auth.getSession();
    session = data.session;

    if (!session) {
      window.location.replace('/login');
    }
  });

  async function logout() {
    await supabase.auth.signOut();
    window.location.replace('/login');
  }
</script>

{#if session === undefined}
  <p>A verificar sessão…</p>

{:else if session}
  <nav>
    <p><strong>{session.user.email}</strong></p>

    <ul>
      {#each links as link}
        <li><a href={link.href}>{link.label}</a></li>
      {/each}
      <li><button type="button" on:click={logout}>Logout</button></li>
    </ul>
  </nav>

  <main>
    <slot />
  </main>
{/if}
