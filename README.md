# Guia de Produção de PDF — Material de Estudos do Davi

Este documento explica como produzir um PDF de material didático de matemática
com a mesma qualidade do último PDF aprovado (`guia-estudos-matematica-2026-06-08.pdf`).
Siga cada passo rigorosamente. Não improvise.

**Convenção de nomes:** todo arquivo final inclui a data (`-YYYY-MM-DD`)
para evitar confusão entre versões de dias diferentes.
Exemplo: `guia-estudos-matematica-2026-06-08.pdf`.

---

## 1. Estrutura de diretórios

```
~/dev/tarefas-davi/
  guia-estudos-matematica-YYYY-MM-DD.md   ← fonte Markdown (com data)
  guia-estudos-matematica-YYYY-MM-DD.pdf  ← PDF gerado (com data)
  README.md                    ← este arquivo
  AGENTS.md                    ← instruções operacionais para agentes
```

---

## 2. Regras do Markdown fonte

### 2.1 Português

Use **sempre** português brasileiro com acentuação completa:
ç, Ç, ã, Ã, õ, Õ, á, é, í, ó, ú, Á, É, Í, Ó, Ú, â, ê, î, ô, û, à, À.

Jamais escreva sem acentos. Jamais use caracteres ingleses no lugar
(ex: "nao" em vez de "não", "fracao" em vez de "fração").

### 2.2 Proibido

- **Nada de emojis.** Zero. Nem ✅ nem 📖 nem 🧮. Substitua por texto ou remova.
- **Nada de HTML.** Zero tags: sem `<details>`, sem `<summary>`, sem `<div>`.
  Markdown puro.
- **Nada de quebras de linha forçadas.** Não use double-space no fim de linha
  para `<br>`. Deixe o parágrafo fluir. Use bullet points (`- `) para listas.
- **Nada de `---` como separador visual dentro do conteúdo.**
  `---` vira `<hr>` no HTML e polui o PDF. Use quebras de seção naturais
  (títulos h1, h2) em vez de linhas horizontais.
- **Não repita o título no metadado do pandoc e no markdown.**
  O markdown já tem `# Título`. Use `--metadata pagetitle=...` só para
  a tag `<title>` do HTML, sem `--metadata title=...`.

### 2.3 Estrutura do conteúdo

Cada bloco segue esta ordem fixa:

```
# Bloco N — Nome do Bloco

## Teoria
(explicação longa, didática, com exemplos passo a passo)

## Exercícios — Bloco N
(enunciados numerados)

*(Gabarito na página de Gabaritos — Seção GN)*

## Vídeos — Bloco N
(lista de links em bullet points, formato:
 - [Nome da Aula — Canal](URL))
```

### 2.4 Teoria

A teoria deve ser **longa e didática**, escrita para um aluno de 13 anos
com dificuldade. Regras:

- Explique o **porquê** de cada regra, não apenas a regra.
- Use **analogias do cotidiano** (dinheiro, pizza, frutas, gavetas).
- Mostre exemplos resolvidos **passo a passo**, com cada etapa numerada.
- Alerte sobre **erros comuns** com um parágrafo "Cuidado:" ou "Erro comum:".
- Conecte os assuntos: "Agora você entende por que fatoração é importante...".
- Tabelas de resumo no final de cada seção são bem-vindas.

### 2.5 Exercícios

- Numerados sequencialmente dentro de cada bloco.
- Agrupados por subtópico com subtítulos em negrito.
- Cada bloco tem de 6 a 16 exercícios, do fácil ao mais difícil.
- Os 2-3 últimos de cada bloco são "mistos" (identificar qual técnica usar).

### 2.6 Gabaritos

- **Todos** os gabaritos ficam concentrados no **final do documento**,
  depois de um `# Gabaritos`.
- Cada seção de gabarito tem ID `## GN — Gabarito do Bloco N (Nome)`.
- Cada resposta inclui o desenvolvimento completo, não só o resultado final.
- Resultado final em **negrito**.

### 2.7 Sumário

O sumário usa links Markdown com os IDs que o pandoc **vai gerar**.
Os IDs do pandoc são gerados assim:
- Tudo em minúsculas
- Espaços viram hífens
- Acentos e ç são preservados
- Pontuação (:, —, etc.) é removida

Exemplo:
- Título: `# Bloco 3 — Produtos Notáveis`
- ID gerado: `bloco-3-produtos-notáveis`
- Link no sumário: `[Bloco 3 — Produtos Notáveis](#bloco-3-produtos-notáveis)`

### 2.8 Vídeos e links externos

- Use bullet points: `- [texto descritivo](URL completa)`
- Sempre inclua o nome do canal no texto do link.
- URLs devem ser completas (com `https://`).
- Prefira canais brasileiros: Gis com Giz, Ferretto, Sandro Curió.

---

## 3. Dados do Davi

- Nome: Davi Galvani
- Idade: 13 anos
- Série: 8º Ano (oitavo ano do Ensino Fundamental)

Sempre incluir no cabeçalho do documento:
```
# Guia de Estudos de Matemática — 8º Ano
Organizado para Davi Galvani — 13 anos
Data: <data atual>
```

---

## 4. CSS de estilo (fixo, não mude)

Este CSS deve ser embutido no HTML. Ele foi aprovado pelo Galvani.
Não adicione bordas, não mude fontes, não invente.

```css
body {
  font-family: "DejaVu Sans", sans-serif;
  font-size: 11pt;
  line-height: 1.65;
  max-width: 800px;
  margin: 0 auto;
  padding: 2em;
  color: #222;
}
h1 { font-size: 1.8em; margin-top: 1.5em; margin-bottom: 0.3em; }
h2 { font-size: 1.4em; color: #1e40af; margin-top: 1.8em; }
h3 { font-size: 1.15em; color: #1e3a5f; margin-top: 1.3em; }
h4 { font-size: 1.05em; color: #334155; margin-top: 1em; }
a { color: #2563eb; text-decoration: none; }
a:hover { text-decoration: underline; }
table { border-collapse: collapse; width: 100%; margin: 1em 0; }
th, td { border: 1px solid #cbd5e1; padding: 6px 10px; text-align: left; }
th { background: #f1f5f9; font-weight: bold; }
code { background: #f1f5f9; padding: 1px 4px; border-radius: 3px; font-size: 0.95em; }
strong { color: #1e3a5f; }
hr { display: none; }
blockquote { border-left: 3px solid #93c5fd; margin: 1em 0; padding: 0.5em 1em; background: #f8fafc; }
ul, ol { padding-left: 1.5em; }
li { margin-bottom: 0.25em; }
@media print {
  body { max-width: 100%; padding: 0; }
  a { color: #2563eb; }
}
```

---

## 5. Pipeline de geração do PDF

### 5.1 Pré-requisitos do sistema

```bash
pandoc  >= 3.0
chromium (headless)
python3
```

Verifique com:
```bash
which pandoc && pandoc --version | head -1
which chromium
```

### 5.2 Comandos exatos (copie e cole)

```bash
# 1. Salve o CSS em um arquivo temporário
cat > /tmp/guia-style.css << 'CSSEOF'
body {
  font-family: "DejaVu Sans", sans-serif;
  font-size: 11pt;
  line-height: 1.65;
  max-width: 800px;
  margin: 0 auto;
  padding: 2em;
  color: #222;
}
h1 { font-size: 1.8em; margin-top: 1.5em; margin-bottom: 0.3em; }
h2 { font-size: 1.4em; color: #1e40af; margin-top: 1.8em; }
h3 { font-size: 1.15em; color: #1e3a5f; margin-top: 1.3em; }
h4 { font-size: 1.05em; color: #334155; margin-top: 1em; }
a { color: #2563eb; text-decoration: none; }
a:hover { text-decoration: underline; }
table { border-collapse: collapse; width: 100%; margin: 1em 0; }
th, td { border: 1px solid #cbd5e1; padding: 6px 10px; text-align: left; }
th { background: #f1f5f9; font-weight: bold; }
code { background: #f1f5f9; padding: 1px 4px; border-radius: 3px; font-size: 0.95em; }
strong { color: #1e3a5f; }
hr { display: none; }
blockquote { border-left: 3px solid #93c5fd; margin: 1em 0; padding: 0.5em 1em; background: #f8fafc; }
ul, ol { padding-left: 1.5em; }
li { margin-bottom: 0.25em; }
@media print {
  body { max-width: 100%; padding: 0; }
  a { color: #2563eb; }
}
CSSEOF

# 2. Gere o HTML a partir do Markdown
#    Use --metadata pagetitle (não --metadata title) para evitar
#    que o pandoc gere um bloco de título visível duplicado.
pandoc ~/dev/tarefas-davi/guia-estudos-matematica-${DATA}.md \
  -s --standalone \
  --metadata pagetitle="Guia de Estudos de Matematica - Davi Galvani - 8 Ano" \
  --css /tmp/guia-style.css \
  -o /tmp/guia.html

# 3. Pós-processamento do HTML (corrigir defeitos do pandoc)
python3 << 'PYEOF'
import re

with open('/tmp/guia.html', 'r') as f:
    html = f.read()

# 3a. Embutir o CSS dentro do HTML (inline) para o Chromium renderizar
with open('/tmp/guia-style.css', 'r') as f:
    css = f.read()
html = html.replace(
    '<link rel="stylesheet" href="/tmp/guia-style.css" />',
    f'<style>\n{css}\n</style>'
)

# 3b. Remover o header de título duplicado que o pandoc gera
html = re.sub(
    r'<header id="title-block-header">.*?</header>\s*',
    '', html, flags=re.DOTALL
)

# 3c. Remover borda de h1 que o pandoc injeta no CSS padrão
#     (o pandoc gera um border-bottom inline para .title)
html = html.replace(
    'h1 { font-size: 1.8em; border-bottom: 2px solid #2563eb; padding-bottom: 0.3em; margin-top: 1.5em; }',
    'h1 { font-size: 1.8em; margin-top: 1.5em; margin-bottom: 0.3em; }'
)

# 3d. Esconder todos os <hr> (vêm dos --- no markdown)
html = html.replace(
    'hr { border: none; border-top: 1px solid #e2e8f0; margin: 2em 0; }',
    'hr { display: none; }'
)

with open('/tmp/guia.html', 'w') as f:
    f.write(html)

print("HTML pós-processado com sucesso")
PYEOF

# 4. Converter HTML para PDF com Chromium headless
#    --no-pdf-header-footer evita cabeçalho/rodapé automáticos
#    O Chromium preserva hyperlinks no PDF (links são clicáveis)
chromium --headless --disable-gpu \
  --print-to-pdf=~/dev/tarefas-davi/guia-estudos-matematica-${DATA}.pdf \
  --no-pdf-header-footer \
  /tmp/guia.html

# 5. Verificar o resultado
ls -lh ~/dev/tarefas-davi/guia-estudos-matematica-${DATA}.pdf
pdfinfo ~/dev/tarefas-davi/guia-estudos-matematica-${DATA}.pdf | grep -E "Pages|Title"
```

### 5.3 Ajuste de caminho

O comando acima salva o PDF em `~/dev/tarefas-davi/guia-estudos-matematica-$(date +%F).pdf`.
A variável `DATA` é preenchida automaticamente com a data atual.

---

## 6. Verificação de qualidade

Antes de entregar o PDF, verifique:

1. `pdfinfo guia-estudos-matematica-$(date +%F).pdf | grep Pages` — confirme o número de páginas.
2. `pdftotext guia-estudos-matematica-$(date +%F).pdf - | head -20` — veja se o início está limpo
   (sem título duplicado, sem linhas estranhas).
3. `pdftotext guia-estudos-matematica-$(date +%F).pdf - | grep -c "http"` — confirme que os links
   estão presentes no texto extraído.
4. Abra o PDF em um visualizador e teste:
   - Links do sumário funcionam?
   - Links do YouTube abrem no navegador?
   - Acentuação está correta?
   - Tabelas estão alinhadas?
   - Não há linhas horizontais estranhas?
   - Não há título repetido na primeira página?

---

## 7. Lista de verificação para novos materiais

Ao criar um novo guia de estudos para o Davi:

- [ ] Markdown sem emojis, sem HTML, sem `---` separadores
- [ ] Português com acentuação completa (ç, ã, õ, á, é, í, ó, ú, â, ê, ô, à)
- [ ] Cabeçalho: "Guia de Estudos de Matemática — 8º Ano" e "Organizado para Davi Galvani — 13 anos"
- [ ] Teoria didática e longa (não resumida), com exemplos passo a passo
- [ ] Exercícios por bloco, com indicação "*(Gabarito na página de Gabaritos — Seção GN)*"
- [ ] Todos os gabaritos no final do documento, seção `# Gabaritos`
- [ ] Sumário com links usando os IDs corretos que o pandoc gera
- [ ] Vídeos com links completos e nome do canal
- [ ] PDF gerado pelo pipeline acima (pandoc → postprocess → chromium)
- [ ] Verificação de qualidade executada

---

## 8. Exemplo mínimo funcional

Para testar o pipeline, crie um arquivo mínimo `teste.md`:

```markdown
# Teste

## Seção 1

Conteúdo com acentuação: não, fração, polinômios, é, á, ç.

[Link para Seção 2](#seção-2)

## Seção 2

Mais conteúdo.

- [YouTube — Gis com Giz](https://www.youtube.com/watch?v=WNG0UkUSoWY)
```

Execute o pipeline e verifique se o PDF gerado tem:
- Título "Teste" uma única vez
- Sem linhas horizontais
- Link do sumário funcional
- Link do YouTube clicável

---

*Guia criado em 08/06/2026. Atualize sempre que o processo mudar.*
