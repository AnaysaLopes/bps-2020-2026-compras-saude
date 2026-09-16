# Mapeamento de discrepâncias entre os arquivos anuais

**Sprint 1 — Entendimento do problema e dos dados**
Aluna: Anaysa Pereira Lopes

Evidência gerada por: [`notebooks/BPS_20_26_projeto_completo.ipynb`](../notebooks/BPS_20_26_projeto_completo.ipynb), seção **DIAGNÓSTICO**
Saída completa em: [`relatorios/relatorio_discrepancias.txt`](../relatorios/relatorio_discrepancias.txt)

---

## 1. Inventário dos sete arquivos

Os sete anos são publicados no portal como **arquivos .ZIP**, cada um contendo
um único CSV nomeado apenas pelo ano.

| Ano | Arquivo no portal | CSV interno | Tamanho | Registros | Colunas | Encoding |
|-----|-------------------|-------------|---------|-----------|---------|----------|
| 2020 | `2020_csv.zip` | `2020.csv` | 50,07 MB | 84.919 | 36 | UTF-8 |
| 2021 | `2021_csv.zip` | `2021.csv` | 50,42 MB | 85.012 | 36 | UTF-8 |
| 2022 | `2022_csv.zip` | `2022.csv` | 53,22 MB | 89.546 | 36 | UTF-8 |
| 2023 | `2023_csv.zip` | `2023.csv` | 20,79 MB | 33.809 | 36 | UTF-8 |
| 2024 | `2024_csv.zip` | `2024.csv` | 17,83 MB | 28.815 | 36 | UTF-8 |
| 2025 | `2025_csv.zip` | `2025.csv` | 22,03 MB | 34.174 | 36 | UTF-8 |
| 2026 | `2026_csv.zip` | `2026.csv` |  7,47 MB | 11.090 | 36 | UTF-8 |
| **Total** | | | **221,8 MB** | **367.365** | | |

---

## 2. Discrepância 1 — Formato de distribuição (.zip e não .csv)

O enunciado pede os arquivos `.csv`, mas o portal entrega `.zip`. Reparei que
o botão "Baixar" de cada recurso aponta pro bucket S3 do CKAN do Ministério da
Saúde, sempre com o mesmo padrão de URL:

```
https://s3.sa-east-1.amazonaws.com/ckan.saude.gov.br/BPS/csv/<ANO>_csv.zip
```

Como a URL é previsível, resolvi automatizar o download e a descompactação
direto no notebook, na seção **BASES**, em vez de baixar os sete arquivos na
mão. Assim o projeto fica reprodutível: qualquer pessoa roda o notebook e
chega na mesma camada raw que eu.

Uma coisa que percebi no caminho: a página do dataset mostra 23 recursos, mas
são repetições dos mesmos sete anos. Conferi o campo `name` de cada recurso
antes de baixar, pra não duplicar nada.

---

## 3. Discrepância 2 — Nomes das colunas: dicionário oficial ≠ arquivo

Esta foi a divergência mais relevante do projeto.

O **dicionário oficial** (`Metadados_BPS_07_04_2026.pdf`) descreve as colunas com
nomes de negócio em português — "Ano Compra", "Nome Instituição", "Código BR",
"Qtd Itens Comprados". Os **arquivos CSV**, porém, trazem nomes técnicos
abreviados. Não existe uma tabela de-para publicada.

| Nome no dicionário oficial | Nome real no CSV |
|----------------------------|------------------|
| Ano Compra | `ano_compra` |
| Nome Instituição | `no_instituicao` |
| CNPJ Instituição | `cnpj_instituicao` |
| Município Instituição | `no_municipio` |
| UF | `sg_uf` |
| Compra | `dt_compra` |
| Inserção | `dt_insercao` |
| Código BR (CATMAT) | `co_catmat` |
| Descrição CATMAT | `ds_item` |
| Unidade de Fornecimento | `un_fornecimento` |
| Genérico | `fg_generico` |
| Anvisa | `registro_anvisa` |
| Modalidade da Compra | `modalidade` |
| Tipo de Compra | `tp_compra` |
| Capacidade | `vl_capacidade` |
| CNPJ / Nome Fornecedor | `cnpj_fornecedor` / `no_fornecedor` |
| CNPJ / Nome Fabricante | `cnpj_fabricante` / `no_fabricante` |
| Qtd Itens Comprados | `qt_medicamento` |
| Preço Unitário | `vl_preco_unitario` |
| Preço Total | `vl_preco_total` |

Montei o de-para acima na mão, comparando o dicionário lado a lado com uma
amostra de cada arquivo, e registrei o resultado em
[`dicionario_de_dados.md`](dicionario_de_dados.md).

Um detalhe que quase passou despercebido: a coluna se chama `qt_medicamento`,
mas o dicionário a define como "Qtd Itens Comprados", e ela também é usada
pra dispositivos e materiais médicos, não só medicamento. Renomeei pra
`qt_itens_comprados` pra ninguém olhar o dashboard e achar que os
dispositivos ficaram de fora.

---

## 4. Discrepância 3 — Colunas presentes no CSV e ausentes do dicionário

O dicionário oficial descreve **24 campos**, mas os arquivos trazem **36 colunas**.
As 12 colunas abaixo existem nos dados e **não estão documentadas**:

`ds_esfera`, `validade_compra`, `co_pdm`, `co_grupo`, `no_grupo`, `co_classe`,
`no_classe`, `sg_unidade_medida`, `ds_observacao`, `no_pdm`,
`nu_processo_compra`, `nu_ata`, `co_seq_bps`

Pra essas, inferi o conteúdo olhando os valores distintos e a relação com as
demais colunas, e deixei claro que é inferência — está sinalizado na coluna
"Origem" do dicionário do projeto. Nenhuma delas foi descartada: mesmo as de
pouco uso analítico seguem na base consolidada, porque continuam sendo dado
publicado pelo Ministério.

---

## 5. Discrepância 4 — Caracteres corrompidos (mojibake) só em alguns anos

Os sete arquivos são UTF-8 válido, nenhum deles dá erro de decodificação. Mas
2020 a 2023 têm texto **duplamente codificado**: um texto que já estava em
UTF-8 foi lido como Latin-1 e gravado de novo em UTF-8 na origem. O resultado
são sequências como:

| Aparece como | Deveria ser |
|--------------|-------------|
| `PregÃ£o 217/2022` | `Pregão 217/2022` |
| `ARP NÂº 177/FMS/2022` | `ARP Nº 177/FMS/2022` |
| `nÂ° 25/2022` | `n° 25/2022` |

**Onde está:** exclusivamente na coluna `nu_processo_compra`, em **7.329
valores**:

| Ano | Valores afetados |
|-----|------------------|
| 2020 | 1.876 |
| 2021 | 1.955 |
| 2022 | 2.742 |
| 2023 | 756 |
| 2024–2026 | 0 |

Como o problema desaparece a partir de 2024, provavelmente houve correção no
sistema de origem.

> **Nota sobre os números:** esta tabela conta **valores (células) afetados**
> na coluna `nu_processo_compra` — é o mesmo critério usado em
> [`relatorios/relatorio_tratamento.txt`](../relatorios/relatorio_tratamento.txt).
> Já [`relatorios/relatorio_discrepancias.txt`](../relatorios/relatorio_discrepancias.txt)
> reporta um número maior (ex.: 2.124 em vez de 1.955 para 2021) porque conta
> **ocorrências do padrão** no texto bruto do arquivo, e algumas células têm o
> padrão duplicado (ex.: `"PregÃ£o NÂº 217"` tem duas ocorrências em um único
> valor). Os dois relatórios estão corretos; medem coisas diferentes.

A correção ficou na função `corrigir_mojibake()`, na seção **TRATAMENTO** do
notebook. Ela reescreve o texto em Latin-1 (recuperando os bytes originais) e
reinterpreta como UTF-8.

```python
def corrigir_mojibake(texto):
    try:
        return texto.encode("latin-1").decode("utf-8")
    except (UnicodeEncodeError, UnicodeDecodeError):
        return texto
```

Ela é segura porque, se aplicada a um texto que já está correto, a operação
falha sozinha e o `except` devolve o original intacto. Pra ter certeza,
testei em 13.512 descrições de item que já estavam corretas, e nenhuma foi
alterada.

> Vale registrar uma armadilha que caí no início: minha primeira tentativa de
> detecção procurava qualquer `Ã` ou `Â` no texto e acusou 243.876 registros.
> Eram quase todos falsos positivos — `Ã` e `Â` maiúsculos aparecem
> legitimamente em palavras como `APRESENTAÇÃO`, `SOLUÇÃO` e `ÂMBAR`. A
> detecção correta precisa exigir `Ã`/`Â` seguido de um caractere da faixa
> alta do Latin-1, que é o que realmente caracteriza a dupla codificação.

---

## 6. Discrepância 5 — Formato brasileiro de data e formato inglês de número

No mesmo arquivo convivem duas convenções:

- **datas** no padrão brasileiro: `21/10/2020` (texto `DD/MM/AAAA`);
- **números** no padrão inglês: `0.15`, `7500.0` (ponto como separador decimal).

Resolvi isso convertendo as datas explicitamente com
`pd.to_datetime(..., format="%d/%m/%Y")` e gravando a base final em ISO
(`AAAA-MM-DD`), formato que o BigQuery e o Looker Studio leem sem ajuste. Os
números já vinham com ponto decimal, então só precisaram de tipagem, sem
conversão de separador.

---

## 7. Discrepância 6 — Preenchimento das colunas varia muito entre os anos

A estrutura é a mesma, mas o **preenchimento** não é. Percentual de vazios:

| Coluna | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 | 2026 |
|--------|------|------|------|------|------|------|------|
| `nu_ata` | 99,3% | 97,4% | 96,4% | 64,3% | 3,9% | 4,4% | 1,9% |
| `ds_observacao` | 0,0% | 1,5% | 2,6% | 19,9% | 72,2% | 62,0% | 56,0% |
| `fg_generico` | 56,3% | 53,4% | 55,0% | 45,8% | 34,1% | 26,7% | 24,8% |
| `registro_anvisa` | 56,3% | 53,4% | 55,0% | 45,8% | 34,1% | 26,7% | 24,8% |
| `sg_unidade_medida` | 63,0% | 63,9% | 64,8% | 62,2% | 63,6% | 63,0% | 62,7% |
| `dt_insercao` | 0,0% | 0,0% | 0,1% | 2,2% | 4,5% | 0,0% | 0,0% |

`nu_ata` e `ds_observacao` trocam de comportamento entre 2023 e 2024, o que
parece sinal de mudança no sistema de coleta. Já `fg_generico` e
`registro_anvisa` têm exatamente o mesmo percentual em todos os anos, o que
faz sentido: são preenchidos juntos, só quando o medicamento tem registro
Anvisa informado.

Decidi não inventar valor nenhum e não descartar nenhuma coluna por causa
disso. Campos categóricos vazios receberam a categoria explícita
`"Não informado"`, pra o registro continuar aparecendo nos gráficos em vez de
sumir sem avisar. As colunas de baixo aproveitamento continuam na base, com a
limitação já registrada no dicionário de dados.

---

## 8. O que **não** era discrepância

Também é resultado do mapeamento descartar hipóteses. Verifiquei e **não**
encontrei divergência em:

| Verificação | Resultado |
|-------------|-----------|
| Nomes das colunas entre os sete anos | Idênticos |
| Quantidade de colunas | 36 em todos os anos |
| Ordem das colunas | Idêntica |
| Separador do CSV | `;` em todos os anos |
| Linhas duplicadas idênticas | 0 em todos os anos |
| `co_seq_bps` repetido (dentro do ano e entre anos) | 0 |
| `ano_compra` divergente do ano de `dt_compra` | 0 |
| `vl_preco_total` ≠ `vl_preco_unitario` × `qt_medicamento` | 0 de 367.365 |
| Quantidade ou preço ≤ 0 | 0 |

No fim, o layout dos sete arquivos é estável. A divergência real do BPS está
no conteúdo — encoding, preenchimento das colunas e, principalmente, a
plausibilidade dos preços unitários, que é o que a Sprint 5 investiga.

---

## 9. Limitação de cobertura identificada

A base tem registros de 24 unidades federativas. Não há nenhum registro de
AM, AP e DF em nenhum dos sete anos. Como o BPS depende de alimentação
voluntária pelas instituições, essa ausência significa "não informou ao BPS",
não necessariamente "não comprou". Registrei isso como limitação da análise.
