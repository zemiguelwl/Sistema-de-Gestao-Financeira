# Problemas encontrados e como os resolvi

Estes são os problemas reais que apareceram enquanto montava o sistema. Cada um tem o sintoma (o que se via), a causa (o que estava realmente a acontecer) e a solução. Quase todos têm a mesma origem: duas ferramentas a interpretar os mesmos dados de maneiras diferentes.

## 1. Valores como 2,20 apareciam como números enormes

**Sintoma.** Uma mensagem como `-2,20 cafe` aparecia na coluna Valor como um número de cinco dígitos, do tipo 46 330, em vez de -2,2.

**Causa.** A primeira versão da fórmula trocava a vírgula por ponto (`2,20` passava a `2.20`) e depois usava `VALOR` para converter em número. Numa folha em localidade Portugal o separador decimal é a vírgula, por isso o Sheets não lia `2.20` como um número decimal: lia-o como uma **data**. Internamente, uma data é guardada como um número de série (o número de dias desde 30/12/1899), e era esse número que aparecia.

**Solução.** A fórmula C2 deixou de pedir ao Sheets que interprete um número com separador decimal. Extrai com expressões regulares a parte inteira (`2`) e a parte decimal (`20`) como dígitos soltos, converte cada uma e junta-as: `2 + 20/100`. Dígitos soltos leem-se da mesma maneira em qualquer localidade. Explicação completa em [formulas.md](formulas.md#c2-valor-).

**Lição.** Nunca depender da localidade para converter texto em número. Se o formato de entrada é conhecido, é mais seguro fazer a conversão à mão.

## 2. As fórmulas desapareciam ao importar o modelo .xlsx

**Sintoma.** Preparei o modelo da folha num ficheiro .xlsx e importei-o para o Google Sheets. A estrutura e os dados apareciam, mas as colunas da tab Data ficavam vazias: as fórmulas tinham desaparecido.

**Causa.** Na conversão de .xlsx para Google Sheets, as funções que o Excel não tem (`ARRAYFORMULA`, `REGEXEXTRACT`, `REGEXREPLACE`, `REGEXMATCH`) são descartadas **sem aviso**. Um ficheiro .xlsx não consegue guardar uma função que o Excel não conhece.

**Solução.** Escrever (ou colar) essas fórmulas diretamente no Google Sheets, com `;` como separador por causa da localidade Portugal. É por isso que o [modelo](../sheets/modelo.xlsx) deste repositório traz a estrutura, o Mapeamento e as fórmulas do Dashboard (que só usam funções comuns aos dois programas), mas **não** as fórmulas da tab Data: essas colam-se à mão a partir de [formulas.md](formulas.md).

**Lição.** Um formato de ficheiro comum não garante que tudo passa de um programa para o outro. Depois de qualquer importação, confirmar que o que importa lá está.

## 3. O Make dava "Unable to parse range"

**Sintoma.** O módulo Google Sheets: Add a Row falhava com o erro `Unable to parse range`.

**Causa.** No campo **Sheet Name** estava o nome do **ficheiro** (a folha de cálculo inteira). Esse campo pede o nome do **separador** onde se escreve. O Make construía um intervalo com um separador que não existia, e a API do Google não o conseguia interpretar.

**Solução.** Escolher o ficheiro no campo Spreadsheet e pôr `Histórico` no campo Sheet Name.

**Lição.** "Sheet" é ambíguo: em inglês pode ser o ficheiro ou o separador. Quando um erro fala de "range", vale a pena confirmar primeiro os nomes.

## 4. Entradas com "+" apareciam como #ERROR!

**Sintoma.** Os gastos (`-40 gasoleo`) entravam bem, mas os ganhos (`+1800 salário`) apareciam no Histórico como `#ERROR!`.

**Causa.** O módulo do Google Sheets tinha a opção **Value input option = User entered**. Com essa opção, o Sheets trata o texto como se alguém o estivesse a escrever à mão numa célula. Um texto que começa por `+` é interpretado como o início de uma fórmula, e `+1800 salário` não é uma fórmula válida.

**Solução.** Mudar a **Value input option** para **Raw**. O texto é guardado exatamente como chega, sem interpretação. A conversão em número passa a ser feita só pelas fórmulas da tab Data, onde está controlada.

**Lição.** A camada de dados em bruto deve guardar os dados **sem transformações**. Interpretar é trabalho da camada de processamento.

## 5. Mensagens com várias linhas ficavam num só registo

**Sintoma.** Uma mensagem com três linhas (três transações) ficava numa única linha do Histórico, com o texto todo junto.

**Causa.** No módulo Set variable, a fórmula era `split(1. Message: Text; newline)`, mas `newline` tinha sido **escrito à mão** como texto. Para o Make isso é a palavra "newline", não o carácter de mudança de linha. O `split` procurava a palavra "newline" na mensagem, não a encontrava e devolvia a mensagem inteira como um só elemento.

**Solução.** Apagar o texto e inserir a palavra-chave `newline` a partir do painel de funções do Make, no separador das funções de texto. Quando está certa, aparece como um bloco colorido e não como texto simples.

**Lição.** Em ferramentas no-code, um valor que parece igual pode ser uma constante ou um simples texto. Olhar para a forma como o editor o mostra.

## 6. `-40 presente` era classificado como Presentes (Ganho)

**Sintoma.** Um presente que eu comprei (`-40 presente`) aparecia na categoria `🎁 Presentes (Ganho)`, junto com os presentes recebidos.

**Causa.** A categoria era procurada só pela palavra. No Mapeamento, `presente` está associado a `🎁 Presentes (Ganho)`, mas a mesma palavra serve para um presente recebido e para um presente oferecido. A palavra sozinha não chega para decidir.

**Solução.** A fórmula E2 passou a olhar também para o **sinal do valor**. Encontra a categoria pela palavra e, se o valor for negativo, troca `(Ganho)` por `(Gasto)` no nome da categoria (e o inverso para valores positivos). Assim `-40 presente` dá `🎁 Presentes (Gasto)` e `+50 presente` dá `🎁 Presentes (Ganho)`, com uma só linha no Mapeamento. Explicação completa em [formulas.md](formulas.md#e2-categoria).

**Lição.** Quando uma classificação está errada, a pergunta é que informação falta para a decidir. Aqui o sinal do valor já existia, só não estava a ser usado.
