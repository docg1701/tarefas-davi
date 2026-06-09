# AGENTS.md — tarefas-davi

Projeto: materiais de estudo de matemática para Davi Galvani (13 anos, 8º ano).
Pipelines: Markdown → pandoc → HTML + postprocess → Chromium → PDF.

**Convenção de nomes:** todo arquivo gerado inclui a data no formato
`<assunto>-YYYY-MM-DD.md` / `<assunto>-YYYY-MM-DD.pdf`. Use `date +%F`
para obter a data atual. Exemplo: `guia-estudos-matematica-2026-06-08.pdf`.

## Commands

```bash
# Gerar PDF a partir do Markdown
cat > /tmp/guia-style.css << 'CSSEOF'
body{font-family:"DejaVu Sans",sans-serif;font-size:11pt;line-height:1.65;max-width:800px;margin:0 auto;padding:2em;color:#222}
h1{font-size:1.8em;margin-top:1.5em;margin-bottom:.3em}
h2{font-size:1.4em;color:#1e40af;margin-top:1.8em}
h3{font-size:1.15em;color:#1e3a5f;margin-top:1.3em}
h4{font-size:1.05em;color:#334155;margin-top:1em}
a{color:#2563eb;text-decoration:none}
a:hover{text-decoration:underline}
table{border-collapse:collapse;width:100%;margin:1em 0}
th,td{border:1px solid #cbd5e1;padding:6px 10px;text-align:left}
th{background:#f1f5f9;font-weight:bold}
code{background:#f1f5f9;padding:1px 4px;border-radius:3px;font-size:.95em}
strong{color:#1e3a5f}
hr{display:none}
blockquote{border-left:3px solid #93c5fd;margin:1em 0;padding:.5em 1em;background:#f8fafc}
ul,ol{padding-left:1.5em}
li{margin-bottom:.25em}
@media print{body{max-width:100%;padding:0}a{color:#2563eb}}
CSSEOF

DATA=$(date +%F)
pandoc guia-estudos-matematica-${DATA}.md \
  -s --standalone \
  --metadata pagetitle="Guia de Estudos de Matematica - Davi Galvani - 8 Ano" \
  --css /tmp/guia-style.css \
  -o /tmp/guia.html

python3 << 'PYEOF'
import re
with open('/tmp/guia.html','r') as f: html = f.read()
with open('/tmp/guia-style.css','r') as f: css = f.read()
html = html.replace('<link rel="stylesheet" href="/tmp/guia-style.css" />', f'<style>\n{css}\n</style>')
html = re.sub(r'<header id="title-block-header">.*?</header>\s*', '', html, flags=re.DOTALL)
# O CSS acima já tem hr{display:none} e h1 sem border-bottom
with open('/tmp/guia.html','w') as f: f.write(html)
PYEOF

chromium --headless --disable-gpu \
  --print-to-pdf=guia-estudos-matematica-${DATA}.pdf \
  --no-pdf-header-footer \
  /tmp/guia.html

# Verificar
pdfinfo guia-estudos-matematica-${DATA}.pdf | grep -E "Pages|Title"

# Verificar qualidade pós-geração
chromium guia-estudos-matematica-${DATA}.pdf
```

## Project Structure

```
~/dev/tarefas-davi/
  README.md                       ← Pipeline completo e regras de Markdown
  AGENTS.md                       ← Este arquivo
  guia-estudos-matematica-YYYY-MM-DD.md   ← Fontes com data
  guia-estudos-matematica-YYYY-MM-DD.pdf  ← PDFs gerados com data
```

A fonte da verdade para regras de conteúdo é o `README.md`. O `AGENTS.md`
contém as instruções operacionais. Ambos devem ser lidos antes de qualquer tarefa.

## Code Style (Markdown)

O estilo de Markdown aprovado pelo Galvani. Todo conteúdo novo deve seguir
estas regras exatas. Exemplo bom vs. ruim:

```markdown
### Bom — parágrafo fluido, acentos, didático

A **álgebra** aparece quando usamos letras para representar números
desconhecidos. Essas letras são chamadas de **variáveis**.

Por que usar letras? Porque nem sempre sabemos o valor exato de algo.
Se digo "o dobro de um número mais três", escrevo: 2x + 3. O x representa
"o número que não sei".

```

```markdown
### Ruim — emoji, quebra forçada, sem acentos

🧮 A algebra aparece quando usamos letras.  
Essas letras sao **variaveis**.

2x + 3 representa "o numero que nao sei".
```

Regras:
- Zero emojis. Zero HTML (`<details>`, `<div>`, `<summary>`).
- Português com acentuação completa: ç, ã, õ, á, é, í, ó, ú, â, ê, ô, à.
- Sem quebras de linha forçadas (double-space no fim de linha).
- Sem `---` como separador (vira `<hr>` e polui o PDF).
- Teoria didática e longa: explique o porquê, use analogias, mostre passo a passo.
- Cada bloco: `# Bloco N — Nome` > `## Teoria` > `## Exercícios — Bloco N` > `## Vídeos — Bloco N`.
- Todos os gabaritos concentrados no final em `# Gabaritos`.
- Sumário com links usando IDs do pandoc (minúsculas, espaços=hífens, sem pontuação).

## Testing

Não há testes automatizados. A verificação é manual:

```bash
DATA=$(date +%F)

# 1. Confirmar número de páginas
pdfinfo guia-estudos-matematica-${DATA}.pdf | grep Pages

# 2. Verificar início limpo (sem título duplicado, sem lixo)
pdftotext guia-estudos-matematica-${DATA}.pdf - | head -20

# 3. Confirmar que links sobreviveram
pdftotext guia-estudos-matematica-${DATA}.pdf - | grep -c "http"

# 4. Abrir e testar manualmente
chromium guia-estudos-matematica-${DATA}.pdf
#   - Links do sumário funcionam?
#   - Links do YouTube abrem no navegador?
#   - Acentuação está correta?
#   - Tabelas alinhadas?
#   - Sem linhas horizontais visíveis?
#   - Sem título repetido na primeira página?
```

## Git Workflow

- Branch: `main` somente. Projeto pessoal, sem PRs.
- Commits atômicos: um commit por material concluído.
- Mensagem: `feat: adiciona guia de <assunto>` ou `fix: corrige <problema> no PDF`.
- Nunca commitar arquivos temporários (`/tmp/*`).
- O PDF e o `.md` fonte são commitados juntos, com sufixo de data.
- Arquivos sem data no nome são obsoletos — remova ou renomeie.

## Boundaries

### ✅ Always

- Ler `README.md` e `AGENTS.md` antes de qualquer tarefa neste diretório.
- Usar o pipeline exato documentado (pandoc → postprocess Python → chromium).
- Usar o CSS aprovado (sem bordas em h1, hr escondido).
- Escrever em português brasileiro com acentuação completa.
- Teoria didática e longa, para aluno de 13 anos com dificuldade.
- Gabaritos concentrados no final do documento.
- Verificar o PDF com `pdfinfo` e `pdftotext` após gerar.

### ⚠️ Ask first

- Mudar o CSS ou o pipeline de geração.
- Adicionar novos formatos de conteúdo (ex: imagens, gráficos).
- Alterar a estrutura de diretórios.
- Mudar o estilo de escrita (tom, nível de detalhe).

### 🚫 Never

- Usar emojis no Markdown.
- Usar tags HTML no Markdown (`<details>`, `<summary>`, `<div>`).
- Usar `---` como separador visual dentro do conteúdo.
- Escrever sem acentos ou com caracteres ingleses no lugar (ex: "nao", "fracao").
- Usar `--metadata title=...` no pandoc (gera标题 duplicado). Use `--metadata pagetitle=...`.
- Gerar PDF com pandoc + LaTeX (xelatex/pdflatex). Sempre use pandoc → HTML → Chromium.
- Deixar `<header id="title-block-header">` no HTML final.
- Commitar arquivos de `/tmp/`.
- Gerar arquivos sem data no nome (ex: `guia-estudos-matematica.pdf`). Sempre use `-YYYY-MM-DD`.
