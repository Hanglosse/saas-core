<script lang="ts">
  import { supabase } from '$lib/supabaseClient';

  let email = '';
  let password = '';
  let error = '';
  let loading = false;

  async function login() {
    error = '';
    loading = true;

    const { error: loginError } = await supabase.auth.signInWithPassword({
      email,
      password
    });

    loading = false;

    if (loginError) {
      error = loginError.message;
    } else {
      window.location.href = '/app/dashboard';
    }
  }
</script>

<h1>Login</h1>

<form on:submit|preventDefault={login}>
  <div>
    <input
      type="email"
      placeholder="Email"
      bind:value={email}
      required
    />
  </div>

  <div>
    <input
      type="password"
      placeholder="Password"
      bind:value={password}
      required
    />
  </div>

  <button type="submit" disabled={loading}>
    {loading ? 'A entrar…' : 'Entrar'}
  </button>
</form>

{#if error}
  <p style="color: red">{error}</p>
{/if}
