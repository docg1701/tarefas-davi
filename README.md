# tarefas-davi

Material didático de matemática para o Davi Galvani (13 anos, 8º ano).
Markdown → pandoc → HTML pós-processado → Chromium headless → PDF.

Pipeline determinístico: um arquivo Markdown fonte vira PDF com sumário
clicável, vídeos do YouTube referenciados e tipografia limpa. As regras de
conteúdo, o CSS aprovado e os comandos completos estão em
[`AGENTS.md`](AGENTS.md).

## Pré-requisitos

- [pandoc](https://pandoc.org/) ≥ 3.0
- [Chromium](https://www.chromium.org/) (headless)
- Python 3

```bash
which pandoc && pandoc --version | head -1
which chromium
```
