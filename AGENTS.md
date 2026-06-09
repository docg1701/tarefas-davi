# AGENTS.md — tarefas-davi

Projeto: materiais de estudo de matemática para Davi Galvani (13 anos, 8º ano).
Pipeline: Markdown → pandoc → HTML + postprocess → Chromium → PDF.

**Convenção de nomes:** todo arquivo gerado inclui a data no formato
`<assunto>-YYYY-MM-DD.md` / `<assunto>-YYYY-MM-DD.pdf`. Use `date +%F`
para obter a data atual. Exemplo: `guia-estudos-matematica-2026-06-08.pdf`.

## Dados do Davi

- Nome: Davi Galvani
- Idade: 13 anos
- Série: 8º Ano (oitavo ano do Ensino Fundamental)

**Cabeçalho obrigatório no topo de todo Markdown fonte:**

```markdown
# Guia de Estudos de Matemática — 8º Ano
Organizado para Davi Galvani — 13 anos
Data: <data atual>
```

## Pipeline de geração

```bash
# 1. CSS aprovado (fixo — não altere sem aprovação explícita)
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

# 2. Gerar HTML com pandoc
#    --metadata pagetitle define a tag <title> (sem gerar bloco de título visível).
#    NUNCA use --metadata title= (gera cabeçalho duplicado no HTML).
DATA=$(date +%F)
pandoc guia-estudos-matematica-${DATA}.md \
  -s --standalone \
  --metadata pagetitle="Guia de Estudos de Matematica - Davi Galvani - 8 Ano" \
  --css /tmp/guia-style.css \
  -o /tmp/guia.html

# 3. Pós-processar HTML (embutir CSS, remover title-block-header duplicado)
python3 << 'PYEOF'
import re
with open('/tmp/guia.html','r') as f: html = f.read()
with open('/tmp/guia-style.css','r') as f: css = f.read()
html = html.replace('<link rel="stylesheet" href="/tmp/guia-style.css" />', f'<style>\n{css}\n</style>')
html = re.sub(r'<header id="title-block-header">.*?</header>\s*', '', html, flags=re.DOTALL)
with open('/tmp/guia.html','w') as f: f.write(html)
PYEOF

# 4. Converter HTML para PDF com Chromium headless
#    --no-pdf-header-footer evita cabeçalho/rodapé automáticos.
#    Links (sumário, YouTube) são preservados no PDF.
chromium --headless --disable-gpu \
  --print-to-pdf=guia-estudos-matematica-${DATA}.pdf \
  --no-pdf-header-footer \
  /tmp/guia.html
```

## Verificação

Após gerar o PDF, execute:

```bash
DATA=$(date +%F)

# 1. Confirmar número de páginas
pdfinfo guia-estudos-matematica-${DATA}.pdf | grep Pages

# 2. Verificar início limpo (sem título duplicado, sem lixo)
pdftotext guia-estudos-matematica-${DATA}.pdf - | head -20

# 3. Confirmar que links sobreviveram
pdftotext guia-estudos-matematica-${DATA}.pdf - | grep -c "http"
```

Abra o PDF no Chromium e confira manualmente:

- Links do sumário funcionam?
- Links do YouTube abrem no navegador?
- Acentuação está correta?
- Tabelas estão alinhadas?
- Sem linhas horizontais visíveis?
- Sem título repetido na primeira página?

## Project Structure

- `README.md` — capa do projeto
- `AGENTS.md` — este arquivo (fonte da verdade)
- `guia-estudos-matematica-YYYY-MM-DD.md` — fonte Markdown (com data)
- `guia-estudos-matematica-YYYY-MM-DD.pdf` — PDF gerado (com data)

## Markdown Content Rules

Todo conteúdo novo deve seguir estas regras.

### Exemplo bom vs. ruim

**Bom** — parágrafo fluido, acentos, didático:

> A **álgebra** aparece quando usamos letras para representar números
> desconhecidos. Essas letras são chamadas de **variáveis**.
>
> Por que usar letras? Porque nem sempre sabemos o valor exato de algo.
> Se digo "o dobro de um número mais três", escrevo: 2x + 3. O x representa
> "o número que não sei".

**Ruim** — emoji, quebra forçada, sem acentos:

> 🧮 A algebra aparece quando usamos letras.  
> Essas letras sao **variaveis**.
>
> 2x + 3 representa "o numero que nao sei".

### Proibições

- **Zero emojis.** Nem ✅ nem 📖 nem 🧮.
- **Zero HTML.** Sem `<details>`, `<summary>`, `<div>`. Markdown puro.
- **Sem quebras de linha forçadas.** Não use double-space para `<br>`. Parágrafo fluido.
- **Sem `---` como separador.** Vira `<hr>` no HTML e polui o PDF.
- **Nunca use `--metadata title=...`** (gera cabeçalho duplicado). Use `--metadata pagetitle=...`.

### Português

Use **sempre** português brasileiro com acentuação completa:
ç, Ç, ã, Ã, õ, Õ, á, é, í, ó, ú, Á, É, Í, Ó, Ú, â, ê, î, ô, û, à, À.
Jamais escreva "nao", "fracao", "polinomios".

### Estrutura por bloco

Cada bloco segue esta ordem fixa:

1. `# Bloco N — Nome do Bloco`
2. `## Teoria`
3. `## Exercícios — Bloco N`
4. `*(Gabarito na página de Gabaritos — Seção GN)*`
5. `## Vídeos — Bloco N`

### Teoria

Longa e didática, para aluno de 13 anos com dificuldade:

- Explique o **porquê** de cada regra, não apenas a regra.
- Use **analogias do cotidiano**: dinheiro, pizza, frutas, gavetas, balança.
- Exemplos resolvidos **passo a passo**, com cada etapa numerada.
- Alerte **erros comuns** com "Cuidado:" ou "Erro comum:".
- Conecte assuntos entre blocos.

### Exercícios

- 6 a 16 por bloco, numerados sequencialmente, do fácil ao difícil.
- Agrupados por subtópico com subtítulos em **negrito**.
- Os 2-3 últimos são **mistos** (identificar qual técnica usar).
- Incluir após o último: `*(Gabarito na página de Gabaritos — Seção GN)*`.

### Gabaritos

- Todos concentrados no final, sob `# Gabaritos`.
- Cada seção: `## GN — Gabarito do Bloco N (Nome)`.
- Desenvolvimento completo, resultado final em **negrito**.

### Sumário

Links com IDs que o pandoc **vai gerar**:

| Regra | Exemplo |
|---|---|
| Minúsculas, espaços=hífens, sem pontuação | `# Bloco 3 — Produtos Notáveis` → `bloco-3-produtos-notáveis` |
| Acentos e ç preservados | `# Frações` → `frações` |
| Link no sumário | `[Bloco 3 — Produtos Notáveis](#bloco-3-produtos-notáveis)` |

### Vídeos

- Bullet points: `- [texto descritivo](URL completa)`.
- Incluir nome do canal: `- [Nome da Aula — Canal](https://...)`.
- Preferir canais brasileiros: **Gis com Giz**, **Ferretto**, **Sandro Curió**.

## Git Workflow

- Branch: `main`. Projeto pessoal, sem PRs.
- Commits atômicos: um por material concluído.
- Mensagem: `feat: adiciona guia de <assunto>` ou `fix: corrige <problema> no PDF`.
- Nunca commitar `/tmp/*`.
- PDF e `.md` fonte commitados juntos, com sufixo de data.

## Boundaries

### ✅ Always

- Pipeline exato: pandoc → postprocess Python → chromium.
- CSS aprovado (sem bordas em h1, hr escondido).
- Português brasileiro com acentuação completa.
- Teoria didática e longa.
- Gabaritos no final do documento.
- Verificar PDF com `pdfinfo` e `pdftotext` após gerar.

### ⚠️ Ask first

- Mudar CSS ou pipeline.
- Adicionar novos formatos (imagens, gráficos).
- Alterar estrutura de diretórios.
- Mudar tom ou nível de detalhe da escrita.

### 🚫 Never

- Emojis no Markdown.
- Tags HTML (`<details>`, `<summary>`, `<div>`).
- `---` como separador visual.
- Escrever sem acentos ("nao", "fracao").
- `--metadata title=...` no pandoc.
- Pandoc + LaTeX (xelatex/pdflatex).
- Deixar `<header id="title-block-header">` no HTML final.
- Commitar arquivos de `/tmp/`.
- Gerar arquivos sem data no nome.
