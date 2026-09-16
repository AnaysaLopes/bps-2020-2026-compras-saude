# Perguntas de negócio

**Sprint 1 — Entendimento do problema e dos dados**
Mini-Projeto Avaliativo · Módulo 2 · Semana 07
Aluna: Anaysa Pereira Lopes

---

Antes de abrir qualquer ferramenta de BI, defini as perguntas que o dashboard
precisa responder. Cada pergunta está amarrada a uma coluna que realmente
existe na base do BPS — foi assim que decidi quais visuais construir e quais
filtros disponibilizar.

## 1. Evolução no tempo

| # | Pergunta | Campos usados | Como o dashboard responde |
|---|----------|---------------|---------------------------|
| 1.1 | Quanto foi registrado em compras a cada ano entre 2020 e 2026? | `nr_ano_compra`, `vl_preco_total` | Gráfico de série temporal |
| 1.2 | O volume financeiro está crescendo ou caindo? Houve algum ano fora da curva? | `dt_compra`, `vl_preco_total` | Série temporal + variação anual |
| 1.3 | O número de registros acompanha o crescimento do valor, ou o valor cresce por causa de poucas compras grandes? | `vl_preco_total`, contagem de registros | Série temporal com duas métricas |

## 2. Distribuição geográfica e institucional

| # | Pergunta | Campos usados | Como o dashboard responde |
|---|----------|---------------|---------------------------|
| 2.1 | Quais estados concentram o maior volume financeiro de compras? | `sg_uf`, `vl_preco_total` | Mapa do Brasil |
| 2.2 | Quais municípios e instituições mais compram? | `no_municipio`, `no_instituicao` | Tabela com barras |
| 2.3 | A compra é mais municipal ou estadual? | `ds_esfera` | Filtro + gráfico de rosca |
| 2.4 | Quantas instituições diferentes estão registrando compras? Esse número cresce ou diminui? | `cnpj_instituicao` (contagem distinta) | Cartão de KPI + série temporal |

## 3. Produtos adquiridos

| # | Pergunta | Campos usados | Como o dashboard responde |
|---|----------|---------------|---------------------------|
| 3.1 | Quais medicamentos e dispositivos médicos têm maior valor total registrado? | `no_pdm`, `vl_preco_total` | Gráfico de barras horizontais |
| 3.2 | Quais itens são comprados em maior quantidade? (a resposta é diferente da anterior) | `no_pdm`, `qt_itens_comprados` | Gráfico de barras horizontais |
| 3.3 | Quanto do gasto vai para medicamentos e quanto vai para dispositivos e materiais médicos? | `ds_tipo_produto` | Gráfico de rosca + filtro |
| 3.4 | Os itens genéricos têm participação relevante? | `fg_generico` | Filtro |

## 4. Fornecedores e fabricantes

| # | Pergunta | Campos usados | Como o dashboard responde |
|---|----------|---------------|---------------------------|
| 4.1 | Quais fornecedores concentram a maior parte do valor registrado? | `no_fornecedor`, `vl_preco_total` | Tabela com barras |
| 4.2 | Quais fabricantes aparecem com mais frequência? | `no_fabricante` | Tabela com barras |
| 4.3 | O mercado é concentrado ou pulverizado? | `cnpj_fornecedor` (contagem distinta) | Cartão de KPI + participação % |

## 5. Preços e modalidades

| # | Pergunta | Campos usados | Como o dashboard responde |
|---|----------|---------------|---------------------------|
| 5.1 | Qual o preço unitário médio ponderado do conjunto filtrado? | `vl_preco_total` / `qt_itens_comprados` | Cartão de KPI |
| 5.2 | Quais modalidades de compra são mais utilizadas? | `ds_modalidade_compra` | Gráfico de barras + filtro |
| 5.3 | O mesmo item é comprado por preços muito diferentes entre instituições? | `vl_indice_preco_vs_mediana` | Tabela de investigação |
| 5.4 | Onde estão as maiores diferenças de preço unitário para produtos comparáveis? | `ds_faixa_alerta_preco`, `cd_catmat`, `un_fornecimento` | Filtro de faixa + tabela mín/mediana/máx |

---

## Pergunta central do projeto

> **Onde está concentrado o gasto público registrado no BPS entre 2020 e 2026, e
> quais registros de preço unitário se afastam tanto da mediana do próprio item
> que merecem verificação antes de servirem de referência para uma nova compra?**

É essa pergunta que amarra o dashboard inteiro. As quatro primeiras seções
mostram onde está o dinheiro; a quinta mostra onde o dado pode não ser
confiável, que é justamente o que impede um gestor de usar o BPS como
referência de preço sem conferir antes.

## Ressalva metodológica

Diferença de preço não é prova de sobrepreço nem de irregularidade. Um mesmo
código CATMAT pode ter preços legitimamente diferentes por causa de
fabricante, apresentação, unidade de fornecimento, quantidade adquirida,
localidade, modalidade de compra e período da negociação. Foi por isso que
chamei o campo criado no projeto de **faixa de alerta**, e não "faixa de
irregularidade": ele serve pra priorizar a análise, nunca pra concluir nada
sozinho.
