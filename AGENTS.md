# AGENTS.md — tarefas-davi

Projeto: material didático de matemática para Davi Galvani (13 anos, 8º ano).
Cada guia: Markdown → pandoc → HTML + postprocess → Chromium → PDF.

**Convenção de nomes:** `<assunto>-YYYY-MM-DD.md` e `<assunto>-YYYY-MM-DD.pdf`.
Use `date +%F` para obter a data. O `.md` fonte e o `.pdf` gerado formam um
par atômico — ambos são commitados juntos.

---

## 1. Pipeline de geração (execute exatamente)

```bash
DATA=$(date +%F)
MD="guia-estudos-matematica-${DATA}.md"
PDF="guia-estudos-matematica-${DATA}.pdf"

# CSS embutido — não altere
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

# 1. Markdown → HTML
pandoc "${MD}" -s --standalone \
  --metadata pagetitle="Guia de Estudos de Matematica - Davi Galvani - 8 Ano" \
  --css /tmp/guia-style.css -o /tmp/guia.html

# 2. Pós-processamento do HTML
python3 << 'PYEOF'
import re
with open('/tmp/guia.html','r') as f: html = f.read()
with open('/tmp/guia-style.css','r') as f: css = f.read()
# Embutir CSS inline
html = html.replace('<link rel="stylesheet" href="/tmp/guia-style.css" />',f'<style>\n{css}\n</style>')
# Remover header de título duplicado que o pandoc injeta
html = re.sub(r'<header id="title-block-header">.*?</header>\s*','',html,flags=re.DOTALL)
with open('/tmp/guia.html','w') as f: f.write(html)
PYEOF

# 3. HTML → PDF
chromium --headless --disable-gpu \
  --print-to-pdf="${PDF}" --no-pdf-header-footer /tmp/guia.html

# 4. Verificar
pdfinfo "${PDF}" | grep -E "Pages|Title"
```

---

## 2. Regras do Markdown fonte

### 2.1 Português

Use **português brasileiro com acentuação completa**. Sem exceções.
ç Ç ã Ã õ Õ á é í ó ú Á É Í Ó Ú â ê î ô û à À.
Nunca escreva "nao", "fracao", "e" (em vez de é), "so" (em vez de só).

### 2.2 Proibido

- **Zero emojis.** Nenhum.
- **Zero HTML.** Sem `<details>`, `<summary>`, `<div>`, `<span>`.
- **Zero quebras de linha forçadas.** Não use double-space no fim de linha.
- **Zero `---` como separador.** Cada `---` vira `<hr>` e polui o PDF.
  Separe seções com títulos (`#`, `##`, `###`), nunca com linhas horizontais.
- **Nunca use `--metadata title=...` no pandoc.** Isso gera um bloco de título
  visível duplicado. Use `--metadata pagetitle=...` (só a tag `<title>`).

### 2.3 Estrutura de cada bloco

```
# Bloco N — Nome do Bloco

## Teoria
(explicação longa, didática, com exemplos passo a passo)

## Exercícios — Bloco N
(enunciados numerados, agrupados por subtópico)

*(Gabarito no final do documento — Seção GN)*

## Vídeos — Bloco N
- [Nome da Aula — Canal](URL completa)
```

### 2.4 Como escrever a teoria

A teoria é para um aluno de 13 anos com dificuldade. Cada seção deve:

- Explicar o **porquê** de cada regra, não apenas a regra.
- Usar **analogias do cotidiano** (dinheiro, pizza, frutas, gavetas).
- Mostrar exemplos resolvidos **passo a passo**, com cada etapa numerada.
- Alertar sobre **erros comuns**: um parágrafo "Cuidado:" ou "Erro comum:".
- Conectar os assuntos entre blocos: "Agora você entende por que fatoração
  é importante — sem ela você não simplifica frações algébricas."

### 2.5 Como escrever exercícios

- Numerados sequencialmente dentro de cada bloco.
- Agrupados por subtópico com subtítulos em negrito.
- De 6 a 16 exercícios por bloco, do fácil ao mais difícil.
- Os 2-3 últimos de cada bloco são **mistos** (identificar qual técnica usar).

### 2.6 Como escrever os gabaritos

- **Todos** os gabaritos concentrados no final do documento:
  `# Gabaritos` → `## GN — Gabarito do Bloco N (Nome)`.
- Cada resposta inclui o **desenvolvimento completo**, não só o resultado.
- Resultado final em **negrito**.
- Indique qual técnica foi usada entre parênteses quando relevante.

### 2.7 Como montar o sumário

Os links do sumário usam os IDs que o pandoc **vai gerar** a partir dos títulos.
Regra de geração de IDs do pandoc: tudo em minúsculas, espaços viram hífens,
acentos e ç são preservados, pontuação é removida.

Exemplos:
- `# Bloco 3 — Produtos Notáveis` → id = `bloco-3-produtos-notáveis`
- `# Gabaritos` → id = `gabaritos`
- `## G3 — Gabarito do Bloco 3 (Produtos Notáveis)` → id = `g3-gabarito-do-bloco-3-produtos-notáveis`

Depois de gerar o HTML, confira com `grep -oP 'id="[^"]*"' /tmp/guia.html`
e corrija o sumário se necessário.

### 2.8 Exemplo de Markdown bom vs. ruim

Bom:
```markdown
A **álgebra** aparece quando usamos letras para representar números
desconhecidos. Essas letras são chamadas de **variáveis**.

Por que usar letras? Porque nem sempre sabemos o valor exato de algo.
Se digo "o dobro de um número mais três", escrevo: 2x + 3. O x representa
"o número que não sei".
```

Ruim:
```markdown
🧮 A algebra aparece quando usamos letras.  
Essas letras sao **variaveis**. 2x + 3 = "o numero que nao sei".
```

### 2.9 Cabeçalho do documento

```markdown
# Guia de Estudos de Matemática — 8º Ano

Organizado para Davi Galvani — 13 anos

Data: YYYY-MM-DD
```

### 2.10 Links de vídeos

Formato: `- [texto com nome do canal](URL completa com https://)`
Canais preferenciais: Gis com Giz, Ferretto, Sandro Curió, Brasil Escola,
Toda Matéria, Khan Academy BR, Portal OBMEP.

---

## 3. Estrutura do projeto

```
~/dev/tarefas-davi/
  AGENTS.md                               ← este arquivo (instruções operacionais)
  README.md                               ← resumo para humanos
  guia-estudos-matematica-YYYY-MM-DD.md   ← fonte Markdown
  guia-estudos-matematica-YYYY-MM-DD.pdf  ← PDF gerado
```

---

## 4. Verificação de qualidade

Após gerar o PDF, execute e confira:

```bash
DATA=$(date +%F)

# Páginas
pdfinfo "guia-estudos-matematica-${DATA}.pdf" | grep Pages

# Início limpo (sem título duplicado, sem lixo)
pdftotext "guia-estudos-matematica-${DATA}.pdf" - | head -20

# Links presentes
pdftotext "guia-estudos-matematica-${DATA}.pdf" - | grep -c "http"
```

Depois abra com `chromium guia-estudos-matematica-${DATA}.pdf` e teste manualmente:
- Links do sumário funcionam?
- Links do YouTube abrem no navegador?
- Acentuação está correta?
- Tabelas alinhadas?
- Sem linhas horizontais visíveis?
- Sem título repetido na primeira página?

---

## 5. Git

```bash
git add "guia-estudos-matematica-${DATA}.md" "guia-estudos-matematica-${DATA}.pdf"
git commit -m "feat: guia de matematica ${DATA}"
git push origin main
```

- Branch única: `main`.
- Commits atômicos: um commit por material concluído.
- `.md` e `.pdf` sempre commitados juntos.
- Nunca commitar arquivos temporários de `/tmp/`.
- Arquivos sem data no nome são obsoletos — remova ou renomeie.
- Remote: `git@github.com:docg1701/tarefas-davi.git` (SSH).

---

## 6. Boundaries

### ✅ Always

- Ler este AGENTS.md inteiro antes de qualquer tarefa.
- Pipeline exato: pandoc → postprocess Python → Chromium.
- CSS aprovado embutido neste arquivo (sem bordas em h1, hr escondido).
- Português brasileiro com acentuação completa.
- Teoria longa e didática, com exemplos passo a passo.
- Gabaritos concentrados no final do documento.
- Verificar PDF gerado com `pdfinfo` e `pdftotext`.
- Arquivos com sufixo de data (`-YYYY-MM-DD`).

### ⚠️ Ask first

- Mudar o CSS.
- Alterar o pipeline (pandoc, postprocess, chromium).
- Adicionar imagens, gráficos ou diagramas.
- Mudar o tom ou nível de detalhe da teoria.

### 🚫 Never

- Emojis no Markdown.
- Tags HTML no Markdown (`<details>`, `<summary>`, `<div>`).
- `---` como separador visual no conteúdo.
- Escrever sem acentos (ex: "nao", "fracao").
- `--metadata title=...` no pandoc (use `--metadata pagetitle=...`).
- PDF via pandoc + LaTeX (xelatex, pdflatex). Sempre pandoc → HTML → Chromium.
- Deixar `<header id="title-block-header">` no HTML.
- Commitar arquivos de `/tmp/`.
- Arquivos sem data no nome (ex: `guia-estudos-matematica.pdf`).
