# tarefas-davi

Material didático de matemática para o Davi Galvani (13 anos, 8º ano).
Markdown → pandoc → HTML pós-processado → Chromium headless → PDF.

## O que é

Geração automatizada de guias de estudo de matemática em PDF, escritos em
português brasileiro com teoria didática longa, exercícios por bloco e
gabaritos completos. Cada PDF é pensado para um aluno de 13 anos com
dificuldade — passo a passo, com analogias do cotidiano e alertas de erros
comuns.

O pipeline é determinístico: um arquivo Markdown fonte é convertido em PDF
via pandoc + HTML pós-processado + Chromium headless. Os PDFs finais têm
sumário com links clicáveis, vídeos do YouTube referenciados e tipografia
limpa (DejaVu Sans, sem bordas, sem linhas horizontais).

## Pré-requisitos

- [pandoc](https://pandoc.org/) ≥ 3.0
- [Chromium](https://www.chromium.org/) (headless)
- Python 3

```bash
which pandoc && pandoc --version | head -1
which chromium
```

## Uso

```bash
# Gerar o PDF a partir do Markdown fonte com data
DATA=$(date +%F)
pandoc guia-estudos-matematica-${DATA}.md \
  -s --standalone \
  --metadata pagetitle="Guia de Estudos de Matematica - Davi Galvani - 8 Ano" \
  -o /tmp/guia.html

# Pós-processar e converter para PDF
# (veja o pipeline completo em AGENTS.md)
chromium --headless --disable-gpu \
  --print-to-pdf=guia-estudos-matematica-${DATA}.pdf \
  --no-pdf-header-footer \
  /tmp/guia.html

# Verificar
pdfinfo guia-estudos-matematica-${DATA}.pdf | grep -E "Pages|Title"
```

O pipeline completo, incluindo o CSS embutido e o pós-processamento Python,
está documentado em [`AGENTS.md`](AGENTS.md).

## O que tem aqui

```
guia-estudos-matematica-YYYY-MM-DD.md   ← fonte Markdown (com data)
guia-estudos-matematica-YYYY-MM-DD.pdf  ← PDF gerado (com data)
AGENTS.md                               ← pipeline completo e regras de conteúdo
README.md                               ← este arquivo
```

## Contribuindo

As regras de conteúdo (Markdown, estilo, estrutura de blocos) estão em
[`AGENTS.md`](AGENTS.md). O pipeline de geração também está documentado lá.
Mude o CSS ou o fluxo de geração somente com aprovação explícita.
