<script lang="ts">
  import { supabase } from '$lib/supabaseClient'
  import { goto } from '$app/navigation'

  let email = ''
  let password = ''
  let loading = false
  let error = ''
  let success = ''

  async function registar() {
    if (!email || !password) {
      error = 'Preenche email e password.'
      return
    }

    if (password.length < 6) {
      error = 'A password deve ter pelo menos 6 caracteres.'
      return
    }

    loading = true
    error = ''
    success = ''

    const { error: err } = await supabase.auth.signUp({
      email,
      password
    })

    loading = false

    if (err) {
      error = err.message
      return
    }

    success = 'Conta criada. Verifica o teu email para confirmar.'
  }
</script>

<div class="login-page">
  <div class="card">
    <h1>Criar conta</h1>
    <p class="subtitle">Regista-te para começar</p>

    <input
      type="email"
      placeholder="Email"
      bind:value={email}
      disabled={loading}
    />

    <input
      type="password"
      placeholder="Password (mín. 6 caracteres)"
      bind:value={password}
      disabled={loading}
    />

    <button on:click={registar} disabled={loading}>
      {loading ? 'A criar conta…' : 'Criar conta'}
    </button>

    {#if error}
      <p class="error">{error}</p>
    {/if}

    {#if success}
      <p class="success">{success}</p>
    {/if}

    <div class="links">
      <a href="/login">Já tenho conta</a>
    </div>
  </div>
</div>

<style>
  .login-page {
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #f2f4f7;
  }

  .card {
    background: white;
    padding: 2rem;
    border-radius: 10px;
    width: 100%;
    max-width: 360px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.08);
    display: grid;
    gap: 0.75rem;
  }

  h1 {
    margin: 0;
  }

  .subtitle {
    color: #666;
    margin-bottom: 0.5rem;
  }

  input {
    padding: 0.6rem;
    border-radius: 6px;
    border: 1px solid #ccc;
  }

  button {
    padding: 0.6rem;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    background: #111827;
    color: white;
  }

  .error {
    color: #c62828;
    font-size: 0.9rem;
  }

  .success {
    color: #2e7d32;
    font-size: 0.9rem;
  }

  .links {
    text-align: center;
    font-size: 0.85rem;
    margin-top: 0.5rem;
  }

  a {
    color: #2563eb;
    text-decoration: none;
  }
</style>
