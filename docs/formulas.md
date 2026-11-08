# Fórmulas explicadas

Este documento explica, parte a parte, todas as fórmulas da folha. Cada fórmula aparece em duas versões:

- **PT**: a que está na folha real, em localidade Portugal (separador `;`, nomes de funções em português).
- **Internacional**: a mesma fórmula para localidades em inglês (separador `,`, nomes em inglês).

Usa a versão que corresponde à localidade da tua folha (Ficheiro > Definições > Localização). Se colares a versão errada, o Sheets mostra um erro de análise.

## Tradução dos nomes das funções

| PT | Internacional | PT | Internacional |
|---|---|---|---|
| `SE` | `IF` | `TEXTO` | `TEXT` |
| `SE.ERRO` | `IFERROR` | `DATA.VALOR` | `DATEVALUE` |
| `COMPACTAR` | `TRIM` | `ESQUERDA` | `LEFT` |
| `VALOR` | `VALUE` | `HOJE` | `TODAY` |
| `NÚM.CARACT` | `LEN` | `SOMA.SE.S` | `SUMIFS` |
| `MINÚSCULAS` | `LOWER` | `MÁXIMO` | `MAX` |
| `SUBST` | `SUBSTITUTE` | `REPETIR` | `REPT` |
| `PROCV` | `VLOOKUP` | `ARRED` | `ROUND` |
| `FALSO` | `FALSE` | `DATAM` | `EDATE` |

`ARRAYFORMULA`, `REGEXMATCH`, `REGEXEXTRACT`, `REGEXREPLACE` e `ABS` têm o mesmo nome nas duas localidades.

## Ideia geral: uma fórmula por coluna

A tab **Data** não tem dados escritos à mão. Cada coluna tem **uma única fórmula na linha 2**, dentro de `ARRAYFORMULA`, que se aplica a todas as linhas abaixo. Quando o Make acrescenta uma linha ao Histórico, a Data atualiza-se sozinha.

O padrão `SE(X2:X="";"";...)` aparece em todas: se a linha está vazia, a célula fica vazia em vez de mostrar 0 ou um erro.

> **Importante:** abaixo da linha 2 as colunas têm de estar vazias. Se houver qualquer valor escrito numa célula que a `ARRAYFORMULA` quer preencher, ela dá `#REF!`.

## Tab Data

### A2, Data e B2, Texto Original

Copiam o Histórico tal como está. O Histórico é a camada de dados em bruto (só o Make escreve lá) e a Data é a camada de processamento. Assim podes mudar as fórmulas à vontade sem tocar nos dados originais.

PT:
```
=ARRAYFORMULA(SE('Histórico'!A2:A="";"";'Histórico'!A2:A))
=ARRAYFORMULA(SE('Histórico'!B2:B="";"";'Histórico'!B2:B))
```
Internacional:
```
=ARRAYFORMULA(IF('Histórico'!A2:A="","",'Histórico'!A2:A))
=ARRAYFORMULA(IF('Histórico'!B2:B="","",'Histórico'!B2:B))
```

O nome `'Histórico'` vai entre plicas por ter um acento.

### C2, Valor (€)

Transforma o texto `-2,50 cafe` no número `-2,5`.

PT:
```
=ARRAYFORMULA(SE(B2:B="";"";SE.ERRO(SE(REGEXMATCH(COMPACTAR(B2:B);"^\+");1;-1)*(VALOR(REGEXEXTRACT(B2:B;"(\d+)"))+SE.ERRO(VALOR(REGEXEXTRACT(B2:B;"^\D*\d+[.,](\d+)"))/10^NÚM.CARACT(REGEXEXTRACT(B2:B;"^\D*\d+[.,](\d+)"));0));0)))
```
Internacional:
```
=ARRAYFORMULA(IF(B2:B="","",IFERROR(IF(REGEXMATCH(TRIM(B2:B),"^\+"),1,-1)*(VALUE(REGEXEXTRACT(B2:B,"(\d+)"))+IFERROR(VALUE(REGEXEXTRACT(B2:B,"^\D*\d+[.,](\d+)"))/10^LEN(REGEXEXTRACT(B2:B,"^\D*\d+[.,](\d+)")),0)),0)))
```

Parte a parte, com o exemplo `-2,50 cafe`:

| Parte | O que faz | Resultado |
|---|---|---|
| `REGEXMATCH(COMPACTAR(B2:B);"^\+")` | O texto começa por `+`? | `FALSO` |
| `SE(...;1;-1)` | Sinal: `+` dá 1, qualquer outra coisa dá -1 (sem sinal conta como gasto) | `-1` |
| `REGEXEXTRACT(B2:B;"(\d+)")` | Primeiro grupo de dígitos: a parte inteira | `"2"` |
| `REGEXEXTRACT(B2:B;"^\D*\d+[.,](\d+)")` | Dígitos depois do primeiro `,` ou `.`: a parte decimal | `"50"` |
| `/10^NÚM.CARACT(...)` | Divide pela potência de 10 com tantos zeros quantos os dígitos decimais | `50 / 100 = 0,5` |
| `SE.ERRO(...;0)` interior | Sem parte decimal (ex.: `-45 gasoleo`), a decimal vale 0 | |
| `-1 * (2 + 0,5)` | Junta tudo | `-2,5` |
| `SE.ERRO(...;0)` exterior | Linha sem número nenhum fica com valor 0 | |

**Porquê tão complicado?** Porque nunca se pede ao Sheets que interprete `"2.50"` ou `"2,50"` como número. Isso depende da localidade, e em Portugal `"2.20"` pode ser lido como uma data. Só se converte dígitos soltos (`"2"`, `"50"`), que são iguais em qualquer localidade. Ver o [problema 1](problemas-e-solucoes.md#1-valores-como-220-apareciam-como-números-enormes).

### D2, Palavra

Tira o valor do início e normaliza o resto: `-2,50 Café` passa a `cafe`.

PT:
```
=ARRAYFORMULA(SE(B2:B="";"";MINÚSCULAS(COMPACTAR(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(COMPACTAR(REGEXREPLACE(SUBST(B2:B;"−";"-");"^\s*[+\-]?\s*\d+[.,]?\d*\s*";""));"[áàâãä]";"a");"[éèêë]";"e");"[íìîï]";"i");"[óòôõö]";"o");"[úùûü]";"u");"[ç]";"c")))))
```
Internacional:
```
=ARRAYFORMULA(IF(B2:B="","",LOWER(TRIM(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(TRIM(REGEXREPLACE(SUBSTITUTE(B2:B,"−","-"),"^\s*[+\-]?\s*\d+[.,]?\d*\s*","")),"[áàâãä]","a"),"[éèêë]","e"),"[íìîï]","i"),"[óòôõö]","o"),"[úùûü]","u"),"[ç]","c")))))
```

Lê-se de dentro para fora:

1. `SUBST(B2:B;"−";"-")`: troca o sinal de menos tipográfico (`−`, que alguns teclados de telemóvel inserem) pelo hífen normal (`-`).
2. `REGEXREPLACE(...;"^\s*[+\-]?\s*\d+[.,]?\d*\s*";"")`: apaga, no início do texto, os espaços, o sinal opcional, o número (com ou sem decimais) e os espaços seguintes. Sobra `Café`.
3. `COMPACTAR(...)`: tira espaços a mais (no início, no fim e repetidos no meio).
4. Seis `REGEXREPLACE` encadeados: trocam vogais acentuadas e o `ç` pela letra simples. `Café` passa a `Cafe`.
5. `COMPACTAR` e `MINÚSCULAS`: limpeza final e tudo em minúsculas. Resultado: `cafe`.

Assim `café`, `Cafe` e `cafe` dão todos `cafe`, e o Mapeamento só precisa de uma linha por palavra. A palavra pode ter espaços (`pingo doce`).

**Limitação conhecida:** como as minúsculas são aplicadas **no fim**, uma maiúscula acentuada (`CAFÉ`) não é apanhada pelos padrões de acentos (que são minúsculos) e o resultado é `café`, que não existe no Mapeamento. A correção é passar `MINÚSCULAS` para antes da remoção dos acentos. Esta versão **ainda não foi testada na folha real**:

PT:
```
=ARRAYFORMULA(SE(B2:B="";"";COMPACTAR(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(MINÚSCULAS(COMPACTAR(REGEXREPLACE(SUBST(B2:B;"−";"-");"^\s*[+\-]?\s*\d+[.,]?\d*\s*";"")));"[áàâãä]";"a");"[éèêë]";"e");"[íìîï]";"i");"[óòôõö]";"o");"[úùûü]";"u");"[ç]";"c"))))
```
Internacional:
```
=ARRAYFORMULA(IF(B2:B="","",TRIM(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(REGEXREPLACE(LOWER(TRIM(REGEXREPLACE(SUBSTITUTE(B2:B,"−","-"),"^\s*[+\-]?\s*\d+[.,]?\d*\s*",""))),"[áàâãä]","a"),"[éèêë]","e"),"[íìîï]","i"),"[óòôõö]","o"),"[úùûü]","u"),"[ç]","c"))))
```

### E2, Categoria

Procura a palavra no Mapeamento e escolhe a variante Ganho ou Gasto pelo sinal do valor.

PT:
```
=ARRAYFORMULA(SE(D2:D="";"";SE(C2:C<0;SUBST(SE.ERRO(PROCV(D2:D;Mapeamento!$A$2:$B;2;FALSO);"❓ Outros");"(Ganho)";"(Gasto)");SUBST(SE.ERRO(PROCV(D2:D;Mapeamento!$A$2:$B;2;FALSO);"❓ Outros");"(Gasto)";"(Ganho)"))))
```
Internacional:
```
=ARRAYFORMULA(IF(D2:D="","",IF(C2:C<0,SUBSTITUTE(IFERROR(VLOOKUP(D2:D,Mapeamento!$A$2:$B,2,FALSE),"❓ Outros"),"(Ganho)","(Gasto)"),SUBSTITUTE(IFERROR(VLOOKUP(D2:D,Mapeamento!$A$2:$B,2,FALSE),"❓ Outros"),"(Gasto)","(Ganho)"))))
```

| Parte | O que faz |
|---|---|
| `PROCV(D2:D;Mapeamento!$A$2:$B;2;FALSO)` | Procura a palavra na coluna A do Mapeamento e devolve a coluna 2 (Categoria). `FALSO` exige correspondência exata. |
| `SE.ERRO(...;"❓ Outros")` | Palavra que não está no Mapeamento fica em `❓ Outros`. |
| `SE(C2:C<0;...;...)` | Valor negativo: usa o primeiro ramo. Positivo ou zero: o segundo. |
| `SUBST(...;"(Ganho)";"(Gasto)")` | Num gasto, se a categoria encontrada for `🎁 Presentes (Ganho)`, troca para `🎁 Presentes (Gasto)`. Categorias sem `(Ganho)` no nome ficam iguais. |
| `SUBST(...;"(Gasto)";"(Ganho)")` | O inverso, para valores positivos. |

É isto que resolve o [problema 6](problemas-e-solucoes.md#6--40-presente-era-classificado-como-presentes-ganho): `-40 presente` dá `🎁 Presentes (Gasto)` e `+50 presente` dá `🎁 Presentes (Ganho)`.

O intervalo `$A$2:$B` não tem linha final, por isso as palavras novas acrescentadas ao Mapeamento entram logo na pesquisa e todo o histórico é reclassificado.

### F2, Tipo

PT:
```
=ARRAYFORMULA(SE(C2:C="";"";SE(C2:C>0;"Ganho";SE(C2:C<0;"Gasto";""))))
```
Internacional:
```
=ARRAYFORMULA(IF(C2:C="","",IF(C2:C>0,"Ganho",IF(C2:C<0,"Gasto",""))))
```

O tipo vem do **sinal do valor**, não do Mapeamento. Um valor 0 (linha sem número) fica sem tipo e por isso não entra nos totais do Dashboard.

### G2, Mês

PT:
```
=ARRAYFORMULA(SE(A2:A="";"";TEXTO(SE.ERRO(DATA.VALOR(ESQUERDA(A2:A&"";10));A2:A);"YYYY-MM")))
```
Internacional:
```
=ARRAYFORMULA(IF(A2:A="","",TEXT(IFERROR(DATEVALUE(LEFT(A2:A&"",10)),A2:A),"YYYY-MM")))
```

| Parte | O que faz |
|---|---|
| `A2:A&""` | Garante que o valor é texto, mesmo que o Sheets o tenha guardado como data. |
| `ESQUERDA(...;10)` | Fica com os 10 primeiros caracteres: `2026-11-08T10:15:00.000Z` passa a `2026-11-08`. |
| `DATA.VALOR(...)` | Converte esse texto numa data. O formato ISO (`AAAA-MM-DD`) é lido igual em qualquer localidade. |
| `SE.ERRO(...;A2:A)` | Se a conversão falhar (por exemplo, o valor já é uma data), usa o valor original. |
| `TEXTO(...;"YYYY-MM")` | Escreve o mês como `2026-11`. É esta chave que o Dashboard usa para filtrar. |

## Dashboard

As fórmulas do Dashboard só usam funções que também existem no Excel, por isso **vêm já no [modelo](../sheets/modelo.xlsx)** e sobrevivem à importação.

### B6, Mês em análise

PT: `=TEXTO(HOJE();"YYYY-MM")`
Internacional: `=TEXT(TODAY(),"YYYY-MM")`

Por defeito mostra o mês atual. Para ver outro mês, escreve por cima, por exemplo `'2026-10`. A plica no início obriga o Sheets a guardar o valor como texto (`AAAA-MM`), que é o que a coluna G da Data tem.

### D6, G6, J6: Entradas, Gastos e Saldo do mês

PT:
```
=SE.ERRO(SOMA.SE.S(Data!C:C;Data!F:F;"Ganho";Data!G:G;$B$6);0)
=SE.ERRO(ABS(SOMA.SE.S(Data!C:C;Data!F:F;"Gasto";Data!G:G;$B$6));0)
=SE.ERRO(SOMA.SE.S(Data!C:C;Data!G:G;$B$6);0)
```
Internacional:
```
=IFERROR(SUMIFS(Data!C:C,Data!F:F,"Ganho",Data!G:G,$B$6),0)
=IFERROR(ABS(SUMIFS(Data!C:C,Data!F:F,"Gasto",Data!G:G,$B$6)),0)
=IFERROR(SUMIFS(Data!C:C,Data!G:G,$B$6),0)
```

`SOMA.SE.S` soma a coluna C (Valor) só nas linhas que cumprem todas as condições: tipo igual a `Ganho` (ou `Gasto`) e mês igual a B6. Os gastos são negativos, por isso `ABS` mostra-os como número positivo. O saldo soma tudo do mês, com sinal, e por isso não precisa de filtrar o tipo.

### C12, D12, E12: Gastos por categoria (linhas 12 a 20)

As categorias de gasto estão escritas em B12:B20. Em cada linha:

PT:
```
C12  =SE.ERRO(ABS(SOMA.SE.S(Data!C:C;Data!E:E;B12;Data!F:F;"Gasto";Data!G:G;$B$6));0)
D12  =SE.ERRO(C12/ABS(SOMA.SE.S(Data!C:C;Data!F:F;"Gasto";Data!G:G;$B$6));0)
E12  =SE.ERRO(SE(MÁXIMO($C$12:$C$20)=0;"";REPETIR("█";ARRED(C12/MÁXIMO($C$12:$C$20)*20;0)));"")
```
Internacional:
```
C12  =IFERROR(ABS(SUMIFS(Data!C:C,Data!E:E,B12,Data!F:F,"Gasto",Data!G:G,$B$6)),0)
D12  =IFERROR(C12/ABS(SUMIFS(Data!C:C,Data!F:F,"Gasto",Data!G:G,$B$6)),0)
E12  =IFERROR(IF(MAX($C$12:$C$20)=0,"",REPT("█",ROUND(C12/MAX($C$12:$C$20)*20,0))),"")
```

- **C12, Total**: como os gastos do mês, mas com mais uma condição: categoria igual a B12.
- **D12, %**: total da categoria a dividir pelo total de gastos do mês (formatado como percentagem). `SE.ERRO` evita a divisão por zero num mês sem gastos.
- **E12, "heatmap"**: uma barra de texto. A categoria com mais gastos leva 20 blocos `█`; as outras levam um número proporcional (`ARRED(C12/máximo*20;0)`). É um gráfico de barras sem gráfico.

As referências relativas (`B12`, `C12`) avançam quando a fórmula é copiada para baixo; as absolutas (`$B$6`, `$C$12:$C$20`) ficam fixas.

### H12, I12, J12: Entradas por categoria (linhas 12 a 14)

Iguais às anteriores, com as categorias de ganho em G12:G14 e o tipo `Ganho` (sem `ABS`, porque os ganhos já são positivos).

PT:
```
H12  =SE.ERRO(SOMA.SE.S(Data!C:C;Data!E:E;G12;Data!F:F;"Ganho";Data!G:G;$B$6);0)
I12  =SE.ERRO(H12/SOMA.SE.S(Data!C:C;Data!F:F;"Ganho";Data!G:G;$B$6);0)
J12  =SE.ERRO(SE(MÁXIMO($H$12:$H$14)=0;"";REPETIR("█";ARRED(H12/MÁXIMO($H$12:$H$14)*20;0)));"")
```
Internacional:
```
H12  =IFERROR(SUMIFS(Data!C:C,Data!E:E,G12,Data!F:F,"Ganho",Data!G:G,$B$6),0)
I12  =IFERROR(H12/SUMIFS(Data!C:C,Data!F:F,"Ganho",Data!G:G,$B$6),0)
J12  =IFERROR(IF(MAX($H$12:$H$14)=0,"",REPT("█",ROUND(H12/MAX($H$12:$H$14)*20,0))),"")
```

### B25 a E36: Evolução dos últimos 12 meses

PT:
```
B25  =TEXTO(DATAM(DATA.VALOR($B$6&"-01");-11);"YYYY-MM")
C25  =SE.ERRO(SOMA.SE.S(Data!C:C;Data!F:F;"Ganho";Data!G:G;B25);0)
D25  =SE.ERRO(ABS(SOMA.SE.S(Data!C:C;Data!F:F;"Gasto";Data!G:G;B25));0)
E25  =C25-D25
```
Internacional:
```
B25  =TEXT(EDATE(DATEVALUE($B$6&"-01"),-11),"YYYY-MM")
C25  =IFERROR(SUMIFS(Data!C:C,Data!F:F,"Ganho",Data!G:G,B25),0)
D25  =IFERROR(ABS(SUMIFS(Data!C:C,Data!F:F,"Gasto",Data!G:G,B25)),0)
E25  =C25-D25
```

- **B25**: `$B$6&"-01"` passa `2026-11` a `2026-11-01`; `DATA.VALOR` converte em data; `DATAM(...;-11)` recua 11 meses; `TEXTO` volta ao formato `AAAA-MM`. Nas linhas 26 a 36 o deslocamento vai de `-10` até `0`, por isso a última linha é o mês em análise.
- **C25 e D25**: entradas e gastos desse mês, como nos cartões, mas com o mês da própria linha (B25) em vez de B6.
- **E25**: saldo do mês.

## Looker Studio: campos calculados

O relatório lê a tab Data. Para separar entradas e gastos na mesma métrica, criam-se dois campos (a sintaxe é a mesma em qualquer localidade):

```
Gastos    = CASE WHEN Valor (€) < 0 THEN -Valor (€) ELSE 0 END
Entradas  = CASE WHEN Valor (€) > 0 THEN Valor (€) ELSE 0 END
```

Para cada transação, `Gastos` é o valor positivo se for um gasto e 0 se não for. Somados, dão o total de gastos, que pode ir num gráfico de barras ao lado de `Entradas` sem valores negativos.
