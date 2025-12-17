<script lang="ts">
  import { supabase } from '$lib/supabaseClient'

  let email = ''
  let loading = false
  let error = ''
  let success = ''

  async function recuperar() {
    if (!email) {
      error = 'Indica o teu email.'
      return
    }

    loading = true
    error = ''
    success = ''

    const { error: err } = await supabase.auth.resetPasswordForEmail(email, {
      redirectTo: `${window.location.origin}/login`
    })

    loading = false

    if (err) {
      error = err.message
      return
    }

    success = 'Email enviado. Verifica a tua caixa de correio.'
  }
</script>

<div class="login-page">
  <div class="card">
    <h1>Recuperar password</h1>
    <p class="subtitle">Vamos enviar-te um email para redefinir</p>

    <input
      type="email"
      placeholder="Email"
      bind:value={email}
      disabled={loading}
    />

    <button on:click={recuperar} disabled={loading}>
      {loading ? 'A enviar…' : 'Enviar email'}
    </button>

    {#if error}
      <p class="error">{error}</p>
    {/if}

    {#if success}
      <p class="success">{success}</p>
    {/if}

    <div class="links">
      <a href="/login">Voltar ao login</a>
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
