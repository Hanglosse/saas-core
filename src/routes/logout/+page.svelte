<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  onMount(async () => {
    await supabase.auth.signOut();
    window.location.replace('/login');
  });
</script>

<p>A terminar sessão…</p>
