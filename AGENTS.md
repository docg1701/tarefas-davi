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

```
# Guia de Estudos de Matemática — 8º Ano
Organizado para Davi Galvani — 13 anos
Data: <data atual>
```

## Commands

```bash
# Gerar PDF a partir do Markdown

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
# O CSS acima já tem hr{display:none} e h1 sem border-bottom
with open('/tmp/guia.html','w') as f: f.write(html)
PYEOF

# 4. Converter HTML para PDF com Chromium headless
#    --no-pdf-header-footer evita cabeçalho/rodapé automáticos.
#    Links (sumário, YouTube) são preservados no PDF.
chromium --headless --disable-gpu \
  --print-to-pdf=guia-estudos-matematica-${DATA}.pdf \
  --no-pdf-header-footer \
  /tmp/guia.html

# 5. Verificar rápido
pdfinfo guia-estudos-matematica-${DATA}.pdf | grep -E "Pages|Title"

# 6. Abrir para inspeção visual
chromium guia-estudos-matematica-${DATA}.pdf
```

## Project Structure

```
~/dev/tarefas-davi/
  README.md                                    ← capa do projeto
  AGENTS.md                                    ← este arquivo (fonte da verdade)
  guia-estudos-matematica-YYYY-MM-DD.md        ← fonte Markdown (com data)
  guia-estudos-matematica-YYYY-MM-DD.pdf       ← PDF gerado (com data)
```

O `AGENTS.md` é a fonte da verdade para regras de conteúdo, pipeline e
convenções. O `README.md` é a capa do projeto (descrição curta, pré-requisitos,
uso resumido). Ambos devem ser lidos antes de qualquer tarefa.

## Markdown Content Rules

O estilo de Markdown aprovado pelo Galvani. Todo conteúdo novo deve seguir
estas regras exatas.

### Exemplo bom vs. ruim

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

### Português e acentuação

Use **sempre** português brasileiro com acentuação completa:
ç, Ç, ã, Ã, õ, Õ, á, é, í, ó, ú, Á, É, Í, Ó, Ú, â, ê, î, ô, û, à, À.

Jamais escreva sem acentos. Jamais use caracteres ingleses no lugar
(ex: "nao" em vez de "não", "fracao" em vez de "fração").

### Proibições

- **Zero emojis.** Nem ✅ nem 📖 nem 🧮. Substitua por texto ou remova.
- **Zero HTML.** Sem `<details>`, `<summary>`, `<div>`. Markdown puro.
- **Sem quebras de linha forçadas.** Não use double-space no fim de linha
  para `<br>`. Deixe o parágrafo fluir. Use bullet points (`- `) para listas.
- **Sem `---` como separador visual.** `---` vira `<hr>` no HTML e polui o PDF.
  Use quebras de seção naturais (títulos h1, h2) em vez de linhas horizontais.
- **Não repetir o título no metadado e no markdown.** O markdown já tem
  `# Título`. Use `--metadata pagetitle=...` só para a tag `<title>` do HTML.
  Jamais use `--metadata title=...` (gera cabeçalho duplicado).

### Estrutura de conteúdo por bloco

Cada bloco segue esta ordem fixa:

```
# Bloco N — Nome do Bloco

## Teoria
(explicação longa, didática, com exemplos passo a passo)

## Exercícios — Bloco N
(enunciados numerados)

*(Gabarito na página de Gabaritos — Seção GN)*

## Vídeos — Bloco N
(lista de links em bullet points)
```

### Regras para a Teoria

A teoria deve ser **longa e didática**, escrita para um aluno de 13 anos
com dificuldade em matemática:

- Explique o **porquê** de cada regra, não apenas a regra.
- Use **analogias do cotidiano**: dinheiro, pizza, frutas, gavetas, balança.
- Mostre exemplos resolvidos **passo a passo**, com cada etapa numerada.
- Alerte sobre **erros comuns** com parágrafo iniciado por "Cuidado:" ou
  "Erro comum:".
- Conecte os assuntos entre blocos: "Agora você entende por que fatoração
  é importante para simplificar frações...".
- Tabelas de resumo no final de cada seção são bem-vindas.

### Regras para os Exercícios

- Numerados sequencialmente dentro de cada bloco (1., 2., 3., ...).
- Agrupados por subtópico com subtítulos em **negrito**.
- Cada bloco tem de **6 a 16 exercícios**, do mais fácil ao mais difícil.
- Os **2-3 últimos** de cada bloco são "mistos" — exigem identificar qual
  técnica usar entre as aprendidas no bloco.
- Após o último exercício, incluir a indicação:
  `*(Gabarito na página de Gabaritos — Seção GN)*` onde GN é o número do bloco.

### Regras para os Gabaritos

- **Todos** os gabaritos concentrados no **final do documento**, depois de
  um `# Gabaritos`.
- Cada seção de gabarito: `## GN — Gabarito do Bloco N (Nome)`.
  Exemplo: `## G3 — Gabarito do Bloco 3 (Produtos Notáveis)`.
- Cada resposta inclui o **desenvolvimento completo**, não só o resultado final.
- Resultado final em **negrito**.

### Regras para o Sumário

O sumário usa links Markdown com os IDs que o pandoc **vai gerar**.
Os IDs do pandoc seguem estas regras:

- Tudo em **minúsculas**.
- Espaços viram **hífens**.
- Acentos e ç são **preservados**.
- Pontuação (:, —, ?, !, etc.) é **removida**.

Exemplo de mapeamento:

| Título no Markdown | ID gerado pelo pandoc |
|---|---|
| `# Bloco 3 — Produtos Notáveis` | `bloco-3-produtos-notáveis` |
| `## Teoria` | `teoria` |
| `## Exercícios — Bloco 3` | `exercícios-bloco-3` |
| `# Gabaritos` | `gabaritos` |
| `## G3 — Gabarito do Bloco 3 (Produtos Notáveis)` | `g3-gabarito-do-bloco-3-produtos-notáveis` |

Link no sumário:
```markdown
- [Bloco 3 — Produtos Notáveis](#bloco-3-produtos-notáveis)
  - [Teoria](#teoria)
  - [Exercícios](#exercícios-bloco-3)
  - [Vídeos](#vídeos-bloco-3)
```

### Regras para Vídeos e links externos

- Use bullet points: `- [texto descritivo](URL completa)`
- Sempre inclua o **nome do canal** no texto do link.
  Exemplo: `- [Nome da Aula — Canal](https://...)`
- URLs devem ser completas (com `https://`).
- Prefira canais brasileiros de matemática: **Gis com Giz**, **Ferretto**,
  **Sandro Curió**, **Marcos Aba**.

## Checklist para novos materiais

Antes de gerar o PDF de um novo guia:

- [ ] Cabeçalho: "# Guia de Estudos de Matemática — 8º Ano" e "Organizado para Davi Galvani — 13 anos"
- [ ] Markdown sem emojis, sem HTML, sem `---` separadores
- [ ] Português com acentuação completa (ç, ã, õ, á, é, í, ó, ú, â, ê, ô, à)
- [ ] Teoria didática e longa, com exemplos passo a passo, analogias e alertas de erros comuns
- [ ] Cada bloco com `# Bloco N`, `## Teoria`, `## Exercícios — Bloco N`, `## Vídeos — Bloco N`
- [ ] Exercícios: 6 a 16 por bloco, numerados, últimos 2-3 mistos
- [ ] Indicação `*(Gabarito na página de Gabaritos — Seção GN)*` após cada bloco de exercícios
- [ ] Todos os gabaritos no final, seção `# Gabaritos`, com desenvolvimento completo
- [ ] Sumário com links usando IDs corretos do pandoc (minúsculas, hífens, sem pontuação)
- [ ] Vídeos com links completos e nome do canal (preferir Gis com Giz, Ferretto, Sandro Curió)
- [ ] PDF gerado pelo pipeline acima (pandoc → postprocess → chromium)
- [ ] Verificação de qualidade executada (pdfinfo, pdftotext, inspeção visual)

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

- Seguir o pipeline exato documentado (pandoc → postprocess Python → chromium).
- Usar o CSS aprovado na seção Commands (sem bordas em h1, hr escondido).
- Escrever em português brasileiro com acentuação completa.
- Teoria didática e longa, para aluno de 13 anos com dificuldade.
- Gabaritos concentrados no final do documento.
- Verificar o PDF com `pdfinfo` e `pdftotext` após gerar.
- Passar pelo checklist de novos materiais antes de finalizar.

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
- Usar `--metadata title=...` no pandoc (gera cabeçalho duplicado). Use `--metadata pagetitle=...`.
- Gerar PDF com pandoc + LaTeX (xelatex/pdflatex). Sempre use pandoc → HTML → Chromium.
- Deixar `<header id="title-block-header">` no HTML final.
- Commitar arquivos de `/tmp/`.
- Gerar arquivos sem data no nome (ex: `guia-estudos-matematica.pdf`). Sempre use `-YYYY-MM-DD`.
