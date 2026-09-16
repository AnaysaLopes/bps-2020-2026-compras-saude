# Dicionário de dados da base consolidada

Arquivo: `dados/curated/BPS_20_26_AnaysaPereiraLopes.csv`
367.365 registros · **41 colunas** · UTF-8 · separador `,` · decimal `.` · datas em `AAAA-MM-DD`

> **Princípio adotado:** decidi não descartar nenhuma coluna publicada pelo
> Ministério da Saúde. As 36 colunas originais do BPS seguem íntegras na base
> consolidada, mesmo as de pouco uso analítico, e a base só ganha 5 colunas
> calculadas em cima disso.

Coluna **Origem**:
- **BPS** — coluna original do arquivo, com o nome mantido;
- **BPS (renomeada)** — coluna original, renomeada por clareza;
- **BPS (inferida)** — coluna que existe no arquivo mas **não** consta do dicionário oficial; a descrição foi inferida a partir dos dados;
- **Criada** — coluna calculada neste projeto.

---

## Identificação e tempo

| Coluna | Tipo | Origem | Descrição |
|--------|------|--------|-----------|
| `cd_seq_bps` | Texto | BPS (renomeada, era `co_seq_bps`) | Identificador único do registro no BPS. Usado para conferir duplicidade. Tratado como texto porque é identificador, não número de cálculo. **Inferida** — não consta do dicionário oficial. |
| `nr_ano_compra` | Inteiro | BPS (renomeada, era `ano_compra`) | Ano da compra informado pela instituição. Mantido como coluna própria conforme exigido no enunciado. |
| `dt_compra` | Data | BPS | Data da compra informada pela instituição. Convertida de `DD/MM/AAAA` para `AAAA-MM-DD`. |
| `dt_insercao` | Data | BPS | Data em que a instituição inseriu as informações no BPS. Não representa o evento de compra — serve para avaliar o atraso do registro. Vazia em até 4,5% dos registros (2024). |
| `nr_validade_compra` | Inteiro | BPS (renomeada, era `validade_compra`) | Prazo de validade do registro. Valor predominante `12`; **provavelmente em meses**, mas o dicionário oficial não descreve o campo — usar com cautela. |

## Geografia e instituição compradora

| Coluna | Tipo | Origem | Descrição |
|--------|------|--------|-----------|
| `sg_uf` | Texto | BPS | UF da instituição compradora. 24 UFs presentes (sem AM, AP e DF). |
| `no_municipio` | Texto | BPS | Município da instituição compradora. 748 municípios distintos. |
| `ds_esfera` | Texto | BPS (inferida) | Esfera administrativa: `MUNICIPAL`, `ESTADUAL`, `FEDERAL` ou `PRIVADA`. |
| `cnpj_instituicao` | Texto | BPS | CNPJ da instituição compradora. **Mantido como texto** para não perder zeros à esquerda. É a chave da contagem distinta do KPI 4. |
| `no_instituicao` | Texto | BPS | Nome da instituição compradora. Vazios → `"Não informado"`. |

## Produto

| Coluna | Tipo | Origem | Descrição |
|--------|------|--------|-----------|
| `cd_catmat` | Texto | BPS (renomeada, era `co_catmat`) | Código BR / CATMAT do item. Identificador padronizado do produto. 13.513 códigos distintos. |
| `ds_item` | Texto | BPS | Descrição CATMAT completa, com dosagem e apresentação. Ex.: `DIPIRONA SÓDICA, DOSAGEM:500 MG`. |
| `co_pdm` | Texto | BPS (inferida) | Código do item padronizado (PDM). Chave técnica correspondente a `no_pdm`. |
| `no_pdm` | Texto | BPS (inferida) | Nome padronizado do item (princípio ativo ou produto), sem dosagem. Ex.: `DIPIRONA SÓDICA`. É o campo ideal para os rankings de itens. |
| `co_grupo` | Texto | BPS (inferida) | Código do grupo CATMAT. |
| `no_grupo` | Texto | BPS (inferida) | Nome do grupo CATMAT. **Pouco discriminante:** 98,7% dos registros têm o mesmo valor (`Equipamentos e artigos para uso médico, dentário e veterinario`). |
| `co_classe` | Texto | BPS (inferida) | Código da classe CATMAT. Chave técnica correspondente a `no_classe`. |
| `no_classe` | Texto | BPS (inferida) | Classe CATMAT do item. Ex.: `DROGAS E MEDICAMENTOS`, `INSTRUMENTOS, EQUIPAMENTOS E SUPRIMENTOS DENTÁRIOS`. |
| `fg_generico` | Texto | BPS | Indica se é medicamento genérico. Convertido de `S`/`N` para `Generico` / `Nao generico`; vazio → `Nao informado`. |
| `cd_registro_anvisa` | Texto | BPS (renomeada, era `registro_anvisa`) | Registro do produto na Anvisa. Vazio → `Sem registro informado`. |
| `un_fornecimento` | Texto | BPS | Unidade em que o item é fornecido: `COMPRIMIDO`, `AMPOLA`, `FRASCO`... **Componente obrigatório da comparação de preços.** |
| `un_medida_capacidade` | Texto | BPS (inferida) | Apresentação completa do item. Ex.: `AMPOLA 2,00 ML`, `FRASCO 100,00 ML`. |
| `sg_unidade_medida` | Texto | BPS (inferida) | Sigla da unidade de medida da capacidade: `ML`, `G`, `UN`, `DOSES`... Vazia em ~63% dos registros. |
| `vl_capacidade` | Decimal | BPS | Capacidade numérica da unidade de fornecimento. Vazia em ~63% dos registros (só se aplica a apresentações líquidas). |

## Fornecedor e fabricante

| Coluna | Tipo | Origem | Descrição |
|--------|------|--------|-----------|
| `cnpj_fornecedor` | Texto | BPS | CNPJ de quem vendeu. Chave da contagem distinta do KPI 5. |
| `no_fornecedor` | Texto | BPS | Nome do fornecedor. 3.668 CNPJs distintos. |
| `cnpj_fabricante` | Texto | BPS | CNPJ de quem fabricou. |
| `no_fabricante` | Texto | BPS | Nome do fabricante. 2.373 CNPJs distintos. |

## Dados da compra

| Coluna | Tipo | Origem | Descrição |
|--------|------|--------|-----------|
| `ds_modalidade_compra` | Texto | BPS (renomeada, era `modalidade`) | Modalidade da aquisição: `Pregão`, `Registro de Preços`, `Dispensa de Licitação`... 10 modalidades distintas. |
| `tp_compra` | Texto | BPS | Natureza da compra: `ADMINISTRATIVA` ou `JUDICIAL`. |
| `nu_processo_compra` | Texto | BPS (inferida) | Número do processo de compra. **É a coluna que continha os caracteres corrompidos** corrigidos no tratamento. |
| `nu_ata` | Texto | BPS (inferida) | Número da ata de registro de preços. Vazio → `Sem ata informada`. |
| `ds_observacao` | Texto | BPS (inferida) | Observação livre digitada pela instituição. Sem padronização e vazia em até 72% dos registros (2024) — mantida na base por ser dado de origem, mas de uso analítico limitado. |

## Métricas

| Coluna | Tipo | Origem | Descrição |
|--------|------|--------|-----------|
| `qt_itens_comprados` | Inteiro | BPS (renomeada, era `qt_medicamento`) | Quantidade adquirida. Renomeada porque o campo cobre **medicamentos e dispositivos**, apesar do nome original. Base do KPI 2. |
| `vl_preco_unitario` | Decimal | BPS | Preço pago por unidade. **Nunca deve ser somado** — só média, mediana ou razão. |
| `vl_preco_total` | Decimal | BPS | Preço unitário × quantidade. Base do KPI 1. Conferido: bate em 367.365 de 367.365 registros. |

## Colunas criadas neste projeto

Cada uma responde a uma exigência específica do enunciado:

| Coluna | Tipo | Descrição | Exigência que atende |
|--------|------|-----------|----------------------|
| `ds_tipo_produto` | Texto | `Medicamento` quando `no_classe` = `DROGAS E MEDICAMENTOS`; `Dispositivo medico` nos demais casos. | Desafio: *"Medicamentos e dispositivos médicos mais adquiridos"* |
| `qt_registros_item_comparavel` | Inteiro | Quantos registros existem no grupo comparável (mesmo `cd_catmat` + mesma `un_fornecimento`). Serve para julgar a confiabilidade da comparação. | Sprint 3: *"Definir os critérios utilizados para comparação de preços entre produtos"* |
| `vl_preco_unitario_mediano_item` | Decimal | Mediana do preço unitário do grupo comparável. Vazia quando o grupo tem menos de 5 registros. **Mediana e não média**, porque a média seria distorcida justamente pelos extremos que queremos detectar. | Sprint 3: *"Criar os campos calculados necessários"* |
| `vl_indice_preco_vs_mediana` | Decimal | `vl_preco_unitario` ÷ `vl_preco_unitario_mediano_item`. Igual a 1 significa preço na mediana do próprio item; 10 significa dez vezes a mediana. | Desafio: *"Variação dos preços unitários entre produtos, instituições, fornecedores e períodos"* |
| `ds_faixa_alerta_preco` | Texto | Faixa do índice acima: `1 - Muito acima (>=5x)`, `2 - Acima (2x a 5x)`, `3 - Faixa esperada`, `4 - Abaixo (<=0,5x)`, `5 - Sem comparacao`. **Indicador de priorização de análise, não de irregularidade.** | Desafio: *"Identificação de oportunidades de investigação sobre diferenças relevantes de preços"* |

---

## Critério de comparabilidade entre produtos

Dois registros só são comparáveis se tratarem do **mesmo código CATMAT** e da
**mesma unidade de fornecimento**.

Comparar apenas pelo nome do medicamento estaria errado: `DIPIRONA SÓDICA`
aparece como comprimido, ampola e frasco, com preços de ordens de grandeza
diferentes por razões perfeitamente legítimas. O código CATMAT já carrega
dosagem e forma farmacêutica; somado à unidade de fornecimento, define um
conjunto de registros que faz sentido comparar entre si.

---

## De/Para dos nomes de colunas

O dicionário oficial usa nomes de negócio e o arquivo usa nomes técnicos, sem
tabela de-para publicada. Sete colunas foram renomeadas para eliminar
ambiguidade, seguindo os prefixos usados no módulo:

| Nome original no CSV | Nome na base final | Motivo |
|----------------------|--------------------|--------|
| `qt_medicamento` | `qt_itens_comprados` | O campo cobre medicamentos **e** dispositivos; o dicionário oficial o chama de "Qtd Itens Comprados" |
| `ano_compra` | `nr_ano_compra` | Prefixo de número |
| `modalidade` | `ds_modalidade_compra` | Prefixo de descrição |
| `registro_anvisa` | `cd_registro_anvisa` | Prefixo de código |
| `co_catmat` | `cd_catmat` | Padroniza o prefixo de código |
| `co_seq_bps` | `cd_seq_bps` | Padroniza o prefixo de código |
| `validade_compra` | `nr_validade_compra` | Prefixo de número |

As outras 29 colunas mantiveram o nome original.

---

## Colunas que existem só na camada staging

Estas são colunas **criadas** no tratamento (nenhuma é dado do BPS) e ficam
apenas em `dados/staging/`, onde são usadas na análise em Python. Não seguem para
a base final porque o Looker Studio as deriva sozinho, ou porque são auditoria:

| Coluna | Por que não vai para a base final |
|--------|-----------------------------------|
| `nr_mes_compra`, `nm_mes_compra`, `nr_trimestre_compra`, `ds_ano_mes` | O Looker Studio deriva mês, trimestre e ano-mês a partir de `dt_compra` |
| `vl_preco_total_calculado`, `vl_diferenca_preco_total` | Colunas de auditoria; a conferência está em `relatorios/relatorio_kpis.txt` |
| `cd_item_comparavel` | Reconstruído no Looker Studio com `CONCAT(cd_catmat, " | ", un_fornecimento)` |

---

## Fonte oficial

Dicionário de dados do Ministério da Saúde:
[`Metadados_BPS_07_04_2026.pdf`](Metadados_BPS_07_04_2026.pdf) (cópia local)
Original: <https://dadosabertos.saude.gov.br/dataset/bps/resource/0e76f527-5e7e-417d-9d0b-f46d00afb717>
