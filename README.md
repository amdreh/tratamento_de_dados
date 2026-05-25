# Limpeza e Padronização de Base de Contatos no Google Sheets

## Descrição

Este projeto consistiu na limpeza, organização, padronização e consolidação de uma base de contatos distribuída em duas abas de uma planilha no Google Sheets:

- `BASE_BRUTA`
- `CONTATOS NOVOS`

O objetivo foi transformar dados desorganizados, inconsistentes e parcialmente preenchidos em uma base estruturada, limpa e pronta para consulta, filtros e integração com ferramentas analíticas.

---

## Diagnóstico Inicial

Foi realizada uma análise inicial da estrutura da planilha.

### Resultado da análise

Ambas as abas continham informações relevantes e foram mantidas no processo de consolidação.

Na aba `CONTATOS NOVOS`, a coluna `F` (sem cabeçalho) continha dois registros compostos apenas por anotações operacionais sem valor estrutural, que foram removidos.

---

# Tecnologias Utilizadas

- **Google Sheets**
- **Google Apps Script (JavaScript)**
- **Regex (Expressões Regulares)**
- **Funções nativas do Google Sheets**

---

# Tratamento da Aba `CONTATOS NOVOS`

## 1. Correção de e-mails com acentuação

A coluna `telefone/email` possuía:

- telefones e e-mails misturados
- erros de preenchimento com acentuação em e-mails

Para normalização, foi criada no Apps Script a função:

```javascript
function REMOVER_ACENTOS(texto) {
  return texto.normalize("NFD").replace(/[\u0300-\u036f]/g, "");
}
```

### Funcionamento

- `.normalize("NFD")` decompõe caracteres acentuados
- `.replace(...)` remove os diacríticos
- `return` devolve o resultado para a planilha

---

## 2. Extração de telefones

```excel
=REGEXEXTRACT(C5;"(\(?\d{2}\)?\s?9?\d{4,5}-?\d{4})")
```

A expressão identifica telefones em diferentes formatos, incluindo:

- `(21)99999-9999`
- `21999999999`
- `21 99999-9999`

---

## 3. Extração de e-mails

```excel
=REGEXEXTRACT(C2;"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}")
```

---

## 4. Tratamento de erros

Campos sem correspondência geravam `#N/A`.

Para evitar poluição visual:

```excel
=SEERRO(REGEXEXTRACT(C2;"[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}");"")
```

```excel
=SEERRO(REGEXEXTRACT(C5;"(\(?\d{2}\)?\s?9?\d{4,5}-?\d{4})");"")
```

---

## Resultado

A coluna original `telefone/email` foi desmembrada em:

- `Telefone`
- `Email`

Após validação, a coluna original foi removida.

---

# Tratamento da Aba `BASE_BRUTA`

## Problema identificado

A coluna `dados` concentrava múltiplos tipos de informação:

- nome da empresa
- nome de contato
- telefone
- data
- e-mail
- observações

Foi necessário realizar extração seletiva.

---

## 1. Remoção de telefones e datas

```excel
=IFERROR(REGEXREPLACE(REGEXREPLACE(A4; "\(?\d{2}\)?\s?9?\d{4}-?\d{4}|\d{10,11}"; ""); "\d{2,4}[/-]\d{2}[/-]\d{2,4}"; ""); A4)
```

---

## 2. Extração de telefones

```excel
=IFERROR(REGEXEXTRACT(A4; "(\(?\d{2}\)?\s?9?\d{4}-?\d{4}|\d{10,11})"); "")
```

---

## 3. Extração de datas

```excel
=IFERROR(REGEXEXTRACT(A4; "(\d{2,4}[/-]\d{2}[/-]\d{2,4})"); "")
```

---

## 4. Extração de e-mails

```excel
=IFERROR(REGEXEXTRACT(A4; "[^[:space:]]+@[^[:space:]/]+"); "")
```

---

# Validação dos Dados

## Validação de datas

```excel
=IF(D4=""; "Vazio"; IF(AND(REGEXMATCH(D4; "[/-]"); LEN(D4)>=8; LEN(D4)<=10); "OK"; "Revisar Data"))
```

---

## Validação de telefones

```excel
=IF(C4=""; "Vazio"; IF(AND(LEN(REGEXREPLACE(C4; "\D"; ""))>=10; LEN(REGEXREPLACE(C4; "\D"; ""))<=11); "OK"; "Revisar Telefone"))
```

Após conferência, as fórmulas foram substituídas por valores fixos.

---

# Limpeza Textual

## Remoção de espaços, e-mails e separadores residuais

```excel
=TRIM(REGEXREPLACE(REGEXREPLACE(B4; "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b|\s*/\s*TI"; ""); "\s*[-/]\s*$"; ""))
```

---

## Padronização de separadores

```excel
=TRIM(REGEXREPLACE(C4; "\s*/\s*/\s*|\s*/\s*"; " - "))
```

---

## Extração complementar

```excel
=IFERROR(TRIM(REGEXEXTRACT(D4; "-\s*(.+)$")); "")
```

---

# Tratamento Manual

Após automação, foi realizada revisão manual para:

- mover observações classificadas incorretamente
- corrigir posicionamento de registros
- realocar informações extraídas para colunas corretas

---

# Tratamento da Coluna `Contato`

Foram removidos:

- expressões operacionais
- prefixos como `Falar com`

Nomes encontrados na coluna `extras` foram realocados manualmente.

---

# Correções na Coluna `Extras`

Foram extraídos e realocados:

- nomes → coluna `Contato`
- e-mails → coluna `Email`
- telefones → coluna `Telefone`
- marcadores como `sem email`

Também foi aplicada a função `REMOVER_ACENTOS`.

---

# Padronização de Telefones

## Remoção de caracteres não numéricos

```excel
=REGEXREPLACE(TO_TEXT(D2); "\D"; "")
```

---

## Formatação final

```excel
=IF(LEN(D3)=10; 
  REGEXREPLACE(TO_TEXT(D3); "(\d{2})(\d{4})(\d{4})"; "($1) $2-$3"); 
  REGEXREPLACE(TO_TEXT(D3); "(\d{2})(\d{5})(\d{4})"; "($1) $2-$3")
)
```

Formato padronizado:

- `(21) 9999-9999`
- `(21) 99999-9999`

---

# Remoção de Duplicatas

Processo:

**Dados → Limpeza de dados → Remover cópias**

## Resultado

### BASE_BRUTA
- 23 linhas removidas
- 182 linhas mantidas

### CONTATOS NOVOS
Nenhuma duplicata encontrada

---

# Padronização de Datas

Foram realizados:

- remoção de textos indevidos
- substituição de traços por barras
- aplicação de formatação nativa de data

---

# Consolidação Final

As abas foram alinhadas estruturalmente:

- mesmos nomes de colunas
- mesma ordem
- mesmo padrão de formatação

Após alinhamento:

- os registros de `CONTATOS NOVOS` foram incorporados à `BASE_BRUTA`
- nova verificação de duplicidade foi executada
- nenhuma duplicata adicional foi encontrada

---

# Resultado Final

A base consolidada ficou:

- estruturada
- padronizada
- deduplicada
- validada
- pronta para filtros e análises

---

## Competências Aplicadas

- Limpeza de dados
- Regex
- Tratamento de exceções
- Google Apps Script
- Normalização textual
- Padronização de dados
- Deduplicação
- Validação automatizada
- Revisão manual orientada por regras
