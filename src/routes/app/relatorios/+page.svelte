<script lang="ts">
  import { supabase } from '$lib/supabaseClient'

  function downloadCSV(nome: string, header: string[], rows: any[][]) {
    const csv =
      [header, ...rows]
        .map((r) => r.map((v) => `"${v ?? ''}"`).join(';'))
        .join('\n')

    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
    const url = URL.createObjectURL(blob)

    const link = document.createElement('a')
    link.href = url
    link.download = nome
    link.click()

    URL.revokeObjectURL(url)
  }

  /* =========================
     PRODUTOS
     ========================= */
  async function exportarProdutos() {
    const { data } = await supabase
      .from('produtos')
      .select('nome, descricao, ativo, created_at')

    if (!data) return

    downloadCSV(
      'produtos.csv',
      ['Nome', 'Descrição', 'Ativo', 'Criado em'],
      data.map((p) => [
        p.nome,
        p.descricao,
        p.ativo ? 'Sim' : 'Não',
        new Date(p.created_at).toLocaleString()
      ])
    )
  }

  /* =========================
     LOTES
     ========================= */
  async function exportarLotes() {
    const { data } = await supabase
      .from('lotes')
      .select(`
        quantidade,
        validade,
        created_at,
        produtos ( nome )
      `)

    if (!data) return

    downloadCSV(
      'lotes.csv',
      ['Produto', 'Quantidade', 'Validade', 'Criado em'],
      data.map((l) => [
        l.produtos.nome,
        l.quantidade,
        l.validade,
        new Date(l.created_at).toLocaleString()
      ])
    )
  }

  /* =========================
     MOVIMENTOS
     ========================= */
  async function exportarMovimentos() {
    const { data } = await supabase
      .from('movimentos')
      .select(`
        tipo,
        quantidade,
        created_at,
        lotes (
          validade,
          produtos ( nome )
        )
      `)
      .order('created_at', { ascending: false })

    if (!data) return

    downloadCSV(
      'movimentos.csv',
      ['Produto', 'Validade', 'Tipo', 'Quantidade', 'Data'],
      data.map((m) => [
        m.lotes.produtos.nome,
        m.lotes.validade,
        m.tipo,
        m.quantidade,
        new Date(m.created_at).toLocaleString()
      ])
    )
  }

  /* =========================
     ALERTAS
     ========================= */
  async function exportarAlertas() {
    const { data } = await supabase
      .from('lotes')
      .select(`
        quantidade,
        validade,
        produtos ( nome )
      `)

    if (!data) return

    const hoje = new Date()
    hoje.setHours(0, 0, 0, 0)

    const alertas = data.filter((l) => {
      const [a, m, d] = l.validade.split('-').map(Number)
      const v = new Date(a, m - 1, d)
      v.setHours(0, 0, 0, 0)
      const diff = (v.getTime() - hoje.getTime()) / 86400000
      return diff < 0 || diff <= 30
    })

    downloadCSV(
      'alertas.csv',
      ['Produto', 'Quantidade', 'Validade'],
      alertas.map((a) => [
        a.produtos.nome,
        a.quantidade,
        a.validade
      ])
    )
  }
</script>

<header class="page-header">
  <h1>Relatórios</h1>
  <p class="subtitle">
    Exportação de dados em CSV
  </p>
</header>

<section class="card">
  <button on:click={exportarProdutos}>📦 Exportar Produtos</button>
  <button on:click={exportarLotes}>🧾 Exportar Lotes</button>
  <button on:click={exportarMovimentos}>🔄 Exportar Movimentos</button>
  <button on:click={exportarAlertas}>🚨 Exportar Alertas</button>
</section>

<style>
  .page-header {
    margin-bottom: 1.5rem;
  }

  .subtitle {
    color: #666;
  }

  .card button {
    display: block;
    width: 100%;
    margin-bottom: 0.5rem;
  }
</style>
