# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão geral

NutriPlano — calculadora/planejador nutricional em **um único arquivo** (`index.html`) com HTML, CSS e JavaScript vanilla embutidos. Não há build, dependências, testes nem framework. Para desenvolver, basta abrir `index.html` no navegador. Idioma da UI e do código (nomes de variáveis/funções/comentários): **português**.

Deploy: GitHub Pages a partir da branch `main` — https://vitormattosdev.github.io/calculadora-nutricional/. Qualquer push em `main` publica direto.

## Arquitetura (tudo dentro de index.html)

O arquivo tem três blocos: CSS no `<head>`, marcação das telas no `<body>`, e um único `<script>` no final.

### Telas (seções alternadas via `style.display`)
- `#etapa1` — formulário de perfil (peso, altura, idade, sexo, atividade), método de cálculo (`cientifico` = Mifflin-St Jeor × atividade, ou `rapido` = peso × fator), preferência alimentar (`todos`/`veg`) e objetivo.
- `#etapa2` — montagem da dieta: cards de refeições + painel lateral fixo (`.painel`) com anel de calorias, barras de macros e sugestões "para fechar o dia".
- `#consulta` — consulta rápida de valores nutricionais sem metas (estado separado em `consultaItens`).
- `#modal-fundo` — modal de busca/adição de alimentos, reutilizado para 3 fluxos: adicionar a uma refeição (`refDestino` = índice), adicionar à consulta (`refDestino === "consulta"`) e substituição por equivalente (`modoTroca = {ri, ii}`).
- `#folha` — planilha de impressão, montada por `montarFolha()` e exibida só em `@media print`.

### Dados e estado
- `ALIMENTOS` — banco de alimentos hardcoded, valores **por 100 g** (base Tabela TACO / rótulos). Campos: `id, nome, cat, kcal, p, c, g, fib, medida, porcoes` (porções rápidas em gramas), opcionais `veg:false` (oculto no modo vegetariano) e `un:{g, nome}` (alimento contável: gramas por unidade e nome da unidade — exibe/edita quantidades em unidades em vez de gramas).
- Estado global em variáveis soltas: `perfil`, `metas`, `refeicoes` (`[{nome, itens:[{foodId, gramas}]}]`), `consultaItens`, `objetivoSel`, `modoTroca`, etc. **Nada é persistido** (sem localStorage) — recarregar perde tudo. As quantidades são sempre armazenadas em **gramas**, mesmo para alimentos com `un` (conversão só na exibição/entrada).
- `calcularMetas()` — calorias por objetivo (com piso de segurança), proteína por g/kg (usa peso desejado nos objetivos de emagrecimento), gordura 0,7 g/kg, fibra 14 g/1000 kcal, carbo pelo restante, água 35 ml/kg.

### Padrão de renderização
Sem framework: funções `render*()` reconstroem `innerHTML` e religam listeners a cada mudança (`renderTudo()` = resumo + refeições + painel). Exceção: digitação nos inputs de quantidade usa `atualizaNumeros()` (atualiza só os números no DOM existente, para não perder o foco), e o `change` dispara o re-render completo.

## Convenções

- Commits em português, mensagens curtas (ver `git log`).
- Estilo: cores/fontes via variáveis CSS em `:root`, classes em português (`.refeicao`, `.painel`, `.folha`).
- Ao adicionar alimento: manter coerência entre `medida`, `porcoes` e `un` (porções devem dar números redondos de unidades) e marcar `veg:false` se tiver carne/ave/peixe. Categorias válidas estão em `CATEGORIAS` (o filtro "Ricos em fibra" é virtual, por `fib >= LIMIAR_FIBRA`).
