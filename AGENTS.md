# AGENTS.md — tarefas-davi

Projeto: material didático de matemática, língua portuguesa e ciências para
Davi Galvani (13 anos, 8º ano).

Cada guia: Markdown → pandoc → HTML + postprocess → Chromium → PDF.

**Convenção de nomes:** `guia-estudos-<assunto>-YYYY-MM-DD.md` e
`guia-estudos-<assunto>-YYYY-MM-DD.pdf`.

Assuntos usados: `matematica`, `portugues`, `ciencias`.

Use `date +%F` para obter a data. O `.md` fonte e o `.pdf` gerado formam um
par atômico — ambos são commitados juntos.

## 1. Pipeline de geração (execute exatamente)

```bash
DATA=$(date +%F)
ASSUNTO="portugues"
TITULO="Guia de Estudos de Lingua Portuguesa - Davi Galvani - 8 Ano"
MD="guia-estudos-${ASSUNTO}-${DATA}.md"
PDF="guia-estudos-${ASSUNTO}-${DATA}.pdf"

# CSS canônico. Copie exatamente este bloco.
# A formatação resultante está documentada na seção 7.
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
ol li::marker{font-weight:bold}
@media print{body{max-width:100%;padding:0}ol{padding-left:2.5em}a{color:#2563eb}}
CSSEOF

# 1. Markdown → HTML
pandoc "${MD}" -s --standalone \
  --metadata pagetitle="${TITULO}" \
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

Use `TITULO` sem acentos no `pagetitle`:

| Assunto | TITULO |
|---|---|
| matematica | `Guia de Estudos de Matematica - Davi Galvani - 8 Ano` |
| portugues | `Guia de Estudos de Lingua Portuguesa - Davi Galvani - 8 Ano` |
| ciencias | `Guia de Estudos de Ciencias - Davi Galvani - 8 Ano` |

O `pagetitle` vira apenas a tag `<title>` do PDF. Nunca use
`--metadata title=...`, porque isso cria um bloco de título visível duplicado.

## 2. Regras do Markdown fonte

### 2.1 Português

Use **português brasileiro com acentuação completa**. Sem exceções.
ç Ç ã Ã õ Õ á é í ó ú Á É Í Ó Ú â ê î ô û à À.
Nunca escreva "nao", "fracao", "e" (em vez de é), "so" (em vez de só).

### 2.2 Proibido

- **Zero emojis decorativos.** Símbolos tipográficos como `✓`, `→`, `×` e
  `÷` são permitidos.
- **Zero HTML.** Sem `<details>`, `<summary>`, `<div>`, `<span>`, `<br>`
  nem comentários `<!-- ... -->`.
- **Zero quebras de linha forçadas.** Não use dois espaços no fim da linha
  nem `\` no fim da linha.
- **Zero `---` como separador.** O CSS esconde `<hr>`, mas o fonte não deve
  depender disso. Separe seções com títulos (`#`, `##`, `###`).
- **Nunca use `--metadata title=...` no pandoc.** Use `--metadata pagetitle=...`.

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
- De 6 a 16 exercícios por bloco; 10 a 12 é o padrão atual.
- Os 2-3 últimos de cada bloco são **mistos** (identificar qual técnica usar).
- Toda alínea usa lista Markdown (`a)`, `b)`, `c)`), nunca texto corrido.
  O marcador gerado pelo pandoc sai em negrito pelo CSS.
- Se a alínea aparecer no meio de uma linha de gabarito, escreva a letra em
  negrito manualmente: `1. **a)** ... **b)** ...`.

### 2.6 Como escrever os gabaritos

- **Todos** os gabaritos concentrados no final do documento:
  `# Gabaritos` → `## GN — Gabarito do Bloco N (Nome)`.
- Cada resposta inclui o **desenvolvimento completo**, não só o resultado.
- Resultado final em **negrito**.
- Indique qual técnica foi usada entre parênteses quando relevante.
- Em listas de correspondência, use a letra em negrito dentro de um item de
  lista (`- **d)** ...`). Não comece a linha com `d)` fora de ordem: o pandoc
  interpreta como lista ordenada e renumera as letras.

### 2.7 Como montar o sumário

Os links do sumário usam os IDs que o pandoc **vai gerar** a partir dos títulos.

Regra de geração de IDs do pandoc: tudo em minúsculas, espaços viram hífens,
acentos e ç são preservados, pontuação é removida. O pandoc também remove
tudo antes da primeira letra; por isso `### 1.1 Puberdade` gera
`puberdade-...`, não `1-1-puberdade-...`.

Exemplos:
- `# Bloco 3 — Produtos Notáveis` → id = `bloco-3-produtos-notáveis`
- `# Gabaritos` → id = `gabaritos`
- `## G3 — Gabarito do Bloco 3 (Produtos Notáveis)` → id = `g3-gabarito-do-bloco-3-produtos-notáveis`

Depois de gerar o HTML, confira com:

```bash
python3 - << 'PY'
import re
html = open('/tmp/guia.html', encoding='utf-8').read()
ids = set(re.findall(r'id="([^"]+)"', html))
hrefs = re.findall(r'href="#([^"]+)"', html)
print('links internos quebrados:', [h for h in hrefs if h not in ids])
PY
```

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
# Guia de Estudos de <Assunto> — 8º Ano

Organizado para Davi Galvani — 13 anos

Data: YYYY-MM-DD
```

### 2.10 Links de vídeos

Formato: `- [texto com nome do canal](URL completa com https://)`

Canais preferenciais por área:

- **Matemática:** Gis com Giz, Ferretto, Sandro Curió, Portal OBMEP.
- **Português:** Brasil Escola, Toda Matéria, Prof. Noslen, Khan Academy BR.
- **Ciências:** Brasil Escola, Toda Matéria, Canal Futura, Prof. Jubilut,
  Khan Academy BR.

## 3. Estrutura do projeto

```
~/dev/tarefas-davi/
  AGENTS.md
  README.md
  guia-estudos-matematica-YYYY-MM-DD.md
  guia-estudos-matematica-YYYY-MM-DD.pdf
  guia-estudos-portugues-YYYY-MM-DD.md
  guia-estudos-portugues-YYYY-MM-DD.pdf
  guia-estudos-ciencias-YYYY-MM-DD.md
  guia-estudos-ciencias-YYYY-MM-DD.pdf
```

## 4. Verificação de qualidade

Após gerar o PDF, execute:

```bash
DATA=$(date +%F)
ASSUNTO="portugues"
PDF="guia-estudos-${ASSUNTO}-${DATA}.pdf"

# Páginas e título
pdfinfo "${PDF}" | grep -E "Pages|Title"

# Links externos (as URLs ficam como anotações, não como texto)
pdfinfo -url "${PDF}" 2>/dev/null | grep -c Annotation

# Início limpo (sem título duplicado, sem lixo)
pdftotext "${PDF}" - | head -20

# Não deve haver separador --- no fonte
grep -c '^---$' "guia-estudos-${ASSUNTO}-${DATA}.md"
```

Não use `pdftotext | grep -c "http"` como teste de links: o Chromium grava as
URLs como anotações, e o texto extraído pode retornar 0 mesmo com os links
funcionando. Use `pdfinfo -url`.

Confira também no HTML:

```bash
python3 - << 'PY'
import re
html = open('/tmp/guia.html', encoding='utf-8').read()
ids = set(re.findall(r'id="([^"]+)"', html))
hrefs = re.findall(r'href="#([^"]+)"', html)
print('links internos quebrados:', [h for h in hrefs if h not in ids])
PY
```

Depois abra com `chromium "guia-estudos-${ASSUNTO}-${DATA}.pdf"` e teste:

- Links do sumário funcionam?
- Links do YouTube abrem no navegador?
- Acentuação está correta?
- Tabelas alinhadas?
- Sem linhas horizontais visíveis?
- Sem título repetido na primeira página?
- Números de lista com dois dígitos (`10.`, `11.`, `12.`) aparecem completos?
- Números e letras das alíneas estão em negrito?
- Nenhum marcador de lista foi cortado na borda esquerda?

## 5. Git

```bash
DATA=$(date +%F)
ASSUNTO="portugues"

git add "guia-estudos-${ASSUNTO}-${DATA}.md" "guia-estudos-${ASSUNTO}-${DATA}.pdf"
git commit -m "feat: guia de ${ASSUNTO} ${DATA}"
git push origin main
```

- Branch única: `main`.
- Commits atômicos: um commit por material concluído.
- `.md` e `.pdf` sempre commitados juntos.
- Nunca commitar arquivos temporários de `/tmp/`.
- Guias sem data no nome são obsoletos — remova ou renomeie.
- Remote: `git@github.com:docg1701/tarefas-davi.git` (SSH).

## 6. Boundaries

### Sempre

- Ler este AGENTS.md inteiro antes de qualquer tarefa.
- Pipeline exato: pandoc → postprocess Python → Chromium.
- CSS canônico copiado deste arquivo; a formatação está na seção 7.
- Português brasileiro com acentuação completa.
- Teoria longa e didática, com exemplos passo a passo.
- Gabaritos concentrados no final do documento.
- Verificar o PDF com `pdfinfo`, `pdfinfo -url` e `pdftotext`.
- Arquivos com sufixo de data (`-YYYY-MM-DD`).

### Perguntar antes

- Mudar o CSS fora de correção de bug de formatação.
- Alterar o pipeline (pandoc, postprocess, chromium).
- Adicionar imagens, gráficos ou diagramas.
- Mudar o tom ou nível de detalhe da teoria.

### Nunca

- Emojis no Markdown.
- Tags HTML no Markdown, inclusive comentários `<!-- ... -->`.
- `---` como separador visual no conteúdo.
- Escrever sem acentos (ex: "nao", "fracao").
- `--metadata title=...` no pandoc (use `--metadata pagetitle=...`).
- PDF via pandoc + LaTeX (xelatex, pdflatex). Sempre pandoc → HTML → Chromium.
- Deixar `<header id="title-block-header">` no HTML.
- Commitar arquivos de `/tmp/`.
- Guias sem data no nome (ex: `guia-estudos-matematica.pdf`).
- Deixar marcador de lista cortado ou sem negrito.

## 7. Formatação canônica do PDF

O CSS do item 1 é a fonte única da formatação. Não crie CSS paralelo.

### 7.1 Página e texto

- Papel carta, sem cabeçalho e sem rodapé (`--no-pdf-header-footer`).
- Fonte DejaVu Sans, 11pt, altura de linha 1,65, largura máxima 800px.
- Cor do texto `#222`.
- Títulos: h1 1,8em; h2 1,4em em `#1e40af`; h3 1,15em em `#1e3a5f`;
  h4 1,05em em `#334155`. Sem bordas.
- Links em `#2563eb`, sem sublinhado; sublinhado apenas no hover.
- Negrito em `#1e3a5f`.

### 7.2 Listas e numeração

- `ol li::marker{font-weight:bold}`: **todos** os marcadores de lista
  ordenada saem em negrito — números (`1.`, `2.`, `10.`) e letras
  (`a.`, `b.`). Isso vale para sumário, teoria, exercícios e gabaritos.
- `@media print ... ol{padding-left:2.5em}`: recuo extra na impressão para
  o marcador de dois dígitos não ser cortado na borda esquerda.
- `ul` mantém marcadores normais (bolinhas), sem negrito.
- Alíneas que não são marcadores de lista (ex.: `1. a) ... b) ...`) precisam
  de negrito manual no Markdown: `1. **a)** ... **b)** ...`.
- Não use `---`; o CSS esconde `<hr>`, mas o fonte não deve ter separadores.

### 7.3 Tabelas

- Largura 100%, `border-collapse: collapse`.
- Bordas de 1px em `#cbd5e1`.
- Células com `padding: 6px 10px`, alinhadas à esquerda.
- Cabeçalho com fundo `#f1f5f9` e negrito.
- Tabela não deve ser usada como layout; apenas para dados.

### 7.4 Blocos e elementos

- `hr{display:none}`: nenhuma linha horizontal visível.
- `blockquote`: borda esquerda azul, fundo `#f8fafc`.
- `code`: fundo `#f1f5f9`, padding 1px 4px, raio 3px.
- `strong`: cor `#1e3a5f`.
- Sem imagens, gráficos ou diagramas sem autorização.

### 7.5 Checklist de formatação

1. `pdfinfo` mostra o título correto e a contagem de páginas.
2. Primeiras 20 linhas não têm título duplicado nem lixo.
3. Links internos do sumário não estão quebrados.
4. `pdfinfo -url` lista os links externos esperados.
5. Números `10.`, `11.`, `12.` aparecem inteiros, sem corte.
6. Números e letras de alínea estão em negrito.
7. Tabelas alinhadas e com cabeçalho.
8. Nenhuma linha horizontal visível.
9. Acentuação completa.
10. Nenhum `---` no Markdown fonte.
