# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sobre o repositório

Este repositório contém uma monografia de TCC (Trabalho de Conclusão de Curso) em Ciência da Computação da UEPB, escrita em LaTeX seguindo as normas ABNT. Título: "A Reestruturação da Estratégia de ETL na SEFAZ-PB: uma proposta para o BDMalhas". O documento é construído a partir de um template ABNT modular originalmente desenvolvido pelo Prof. Dr. José Wilker de Lima Silva (UEPB).

## Comandos de build

O projeto compila com `pdflatex` + `bibtex` (via `abntex2cite`/`abntex2-alf`), com passadas de nomenclatura (`makeindex`) e glossário, com saída redirecionada para `build/`:

```bash
latexmk -pdf -output-directory=build documento.tex
```

Pipeline manual equivalente (espelha `build/documento.fdb_latexmk`):

```bash
pdflatex -output-directory=build documento.tex
bibtex build/documento
makeindex build/documento.nlo -s nomencl.ist -o build/documento.nls
pdflatex -output-directory=build documento.tex
pdflatex -output-directory=build documento.tex
```

O diretório `build/` está no `.gitignore` — todos os artefatos (`.aux`/`.bbl`/`.log`/`.pdf`/etc.) são gerados automaticamente e nunca devem ser editados manualmente. Após um build completo, verifique `build/documento.log` em busca de `Warning`/`Undefined` para detectar referências quebradas, citações ausentes ou passadas de glossário/nomenclatura faltando — avisos de `\hbox` under/overfull são apenas cosméticos e não indicam erro de compilação.

Não há suíte de testes nem linter; a correção é avaliada por um build limpo (sem referências/citações indefinidas) e revisão visual do PDF gerado.

## Estrutura do documento

`documento.tex` é o ponto de entrada — ele apenas encadeia chamadas `\input` em uma ordem fixa e raramente deve precisar de mudanças estruturais:

1. `config/pacotes.tex` — todos os pacotes importados e definições globais de macros/ambientes (tipos de float customizados `grafico`/`quadro`, ambientes de teorema, formatação de listas ABNT via `tocloft`, configuração de glossário/nomenclatura, atalhos de referência cruzada `\tabref`/`\figref`/`\secref`/`\chapref`/`\eqnref`). Mudanças em nível de pacote (novos ambientes, macros, estilização) pertencem aqui, não nos arquivos de capítulo.
2. `pre-textual/0-infobasica.tex` — fonte única de verdade para todos os metadados do documento: tipo de trabalho (`\tipodetrabalhonum`, 1=TCC/2=Dissertação/3=Tese — controla se membros extras da banca são exibidos), instituição, título, autor, orientador e nomes da banca examinadora. Edite este arquivo, não as páginas pré-textuais em si, ao atualizar título/autor/banca.
3. `pre-textual/*.tex` (capa, folha de rosto, ficha catalográfica, folha de aprovação, dedicatória, agradecimentos, epígrafe, resumo, abstract) — cada um é uma página pré-textual independente que utiliza as macros definidas em `0-infobasica.tex`.
4. `config/listas.tex` — detecta automaticamente (via marcadores escritos no arquivo `.aux` por comandos "ganchados") quais das listas (Ilustrações/Tabelas/Siglas/Símbolos/Quadros/Gráficos) realmente aparecem no documento e imprime apenas essas, além do `\tableofcontents`. Este arquivo está explicitamente marcado como "NÃO FAZER EDIÇÕES" em `documento.tex` — não o modifique para corrigir um problema de formatação pontual em algum capítulo.
5. `capitulos/*.tex` — um arquivo por capítulo (introducao, fundamentacao, metodologia, desenvolvimento_da_solucao, resultados, discurssao, conclusoes, apendices), incluídos via `\input` nessa ordem em `documento.tex`. Adicionar um novo capítulo significa criar o arquivo em `capitulos/` e adicionar a linha `\input` correspondente em `documento.tex`.
6. `referencias.bib` — bibliografia, citada via `abntex2cite` (`\cite`, `\citeonline`, `\citeauthor`, etc.) no estilo `alf` (autor-data); o título/formatação da seção de referências está fixado em `documento.tex`, logo antes de `\bibliography{referencias}`, e não no arquivo `.bib`.
7. `imagens/` — todas as figuras referenciadas pelos capítulos via `\includegraphics`; `config/logo-uepb.png` é a logo institucional usada na capa/folha de rosto, mantida separadamente em `config/`.

## Convenções específicas deste template

- Os ambientes flutuantes customizados `grafico` (Gráfico) e `quadro` (Quadro) existem junto aos padrões `figure`/`table` — use-os para conteúdo do tipo gráfico e texto emoldurado, respectivamente, para que apareçam na lista automática correta.
- Siglas/abreviações passam pelo comando `\sigla{chave}{Expansão}` (que encapsula `\newacronym`+`\gls` do pacote `glossaries`) ou pelas já pré-definidas em `config/pacotes.tex` (`abnt`, `usp`, `ia`) — texto de sigla digitado diretamente não populará a Lista de Abreviaturas e Siglas.
- Nomenclatura/símbolos usam o comando padrão `\nomenclature{}{}` do pacote `nomencl`.
- Se uma determinada lista pré-textual (figuras, tabelas, siglas, símbolos, quadros, gráficos) é impressa ou não é totalmente automático, com base no uso no texto — não adicione `\newpage`/títulos manuais para essas listas nos arquivos de capítulo.
- Referências cruzadas devem usar as macros de atalho (`\figref{}`, `\tabref{}`, `\secref{}`, `\chapref{}`, `\eqnref{}`) definidas em `config/pacotes.tex` em vez de `\ref{}`/`\eqref{}` crus, para manter a fraseologia consistente no padrão ABNT ("Figura~X"/"Tabela~X").
