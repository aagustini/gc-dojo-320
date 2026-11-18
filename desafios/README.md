# Desafios — extensões do gerador de código

Cada pasta testa **uma** funcionalidade a ser acrescentada ao código inicial
(`exemploGC.y` e `exemploGC.flex`). Os programas usam só variáveis globais e a
`main`, ou seja, não dependem da geração de código de funções.

```
make teste-desafios                 # todos os desafios
make desafio D=03                   # só a pasta 03_incremento
./run.x desafios/09_aif/d09a_sinal.cmm
```

Cada `prog.cmm` tem `prog.esperado` (a saída correta do programa) e, se lê
dados, `prog.entrada`. Os comentários de cada programa mostram o valor
esperado de cada `write`.

## Funcionalidades

| Pasta | Funcionalidade | Sintaxe / comportamento |
|---|---|---|
| `01_atribuicao_expressao` | atribuição passa a ser **expressão** | `x = e` vale `e`; associa à direita (`a = b = 0`); `exp ;` passa a ser comando |
| `02_mais_igual` | operador `+=` | `x += e` ≡ `x = x + (e)`; é expressão, com o novo valor de `x` |
| `03_incremento` | `++` e `--`, pré e pós | `x++` vale o valor antigo, `++x` vale o novo; só para variáveis `int` |
| `04_do_while` | `do-while` | `do cmd while (exp);`, o corpo executa ao menos uma vez |
| `05_break_continue` | `break` e `continue` | atuam no laço mais interno; `continue` vai ao teste (`while`, `do-while`) |
| `06_for` | `for` estilo C | `for (exp? ; exp? ; exp?) cmd`; condição omitida = verdadeira; `continue` vai ao passo |
| `07_swap` | `swap` | `swap a, b;` troca os valores de duas variáveis do mesmo tipo |
| `08_times` | `times` | `times exp do cmd` executa `cmd` `exp` vezes; `exp` é avaliada **uma vez**; `exp <= 0` não executa |
| `09_aif` | *arithmetic if* | ver abaixo |

### Sintaxe proposta para o *arithmetic if*

Inspirada no IF aritmético do FORTRAN (`IF (e) l1, l2, l3`), mas com comandos
no lugar de rótulos:

```c
aif (exp)
    neg:  cmd        // exp <  0
    zero: cmd        // exp == 0
    pos:  cmd        // exp >  0
```

- `aif`, `neg`, `zero` e `pos` são palavras reservadas; os três ramos são
  obrigatórios e aparecem nessa ordem.
- `exp` é avaliada uma vez e exatamente um dos três comandos é executado.
- Cada ramo é um `cmd` qualquer: um comando simples, um bloco `{ }` ou outro `aif`.
- Uso típico: `aif (a - b)` compara dois valores com um único teste.

## Ordem sugerida e dependências

1. **01** primeiro: os testes de 02, 03 e 06 usam atribuições e incrementos
   dentro de expressões e como comandos (`i++;`).
2. **04** e **05** juntos: `break` e `continue` precisam saber os rótulos do
   laço mais interno, que pode ser `while`, `do-while`, `for` ou `times`.
3. **06** usa 01, e o teste `d06b` usa `break` (05).
4. **07**, **08** e **09** são independentes.

## Dicas de implementação

- **Atribuição como expressão**: o `cmd: ID '=' exp ';'` sai e entra
  `exp: ID '=' exp`, que armazena o valor **e o deixa na pilha**. O comando
  `cmd: exp ';'` descarta o valor (`ADDL $4, %ESP`). Atenção: a chamada
  `ID '(' ... ')'` aparece como comando e como expressão, e as ações no meio
  da regra (`{ iniciaChamada(...) }`) geram um conflito redução/redução. Uma
  das duas formas precisa sair (ou ser fatorada em um não terminal comum).
- **Precedência**: `'='` e `+=` têm a menor precedência e associam à direita
  (`%right`, antes de `'?'`). `++` e `--` ficam com a dos operadores unários.
- **Analisador léxico**: `"++"`, `"--"` e `"+="` antes das regras de um caractere
  (`a--b` passa a ser `a -- b`).
- **Pré/pós incremento**: `INCL`/`DECL` direto na memória da variável, antes
  ou depois do `PUSHL`.
- **`break` / `continue`**: uma pilha com os rótulos de *continuar* e de *sair*
  de cada laço aberto; `break` ou `continue` fora de laço é erro semântico.
- **`for`**: em uma passagem, o passo aparece **antes** do corpo no fonte, mas
  executa **depois**. Uma solução usa quatro rótulos:
  `teste: cond; JE fim; JMP corpo; passo: ...; JMP teste; corpo: cmd; JMP passo; fim:`
- **`times`**: o contador pode ficar **no topo da pilha** durante o laço
  (`CMPL $0, (%ESP)` · `JLE fim` · `DECL (%ESP)`), o que dispensa variáveis
  temporárias e funciona com `times` aninhados. Lembre de remover o contador
  no fim (inclusive quando se sai por `break`).
- **`swap`**: na máquina de pilha bastam quatro instruções
  (`PUSHL a` · `PUSHL b` · `POPL a` · `POPL b`).
- **`aif`**: após calcular `exp`: `POPL %EAX` · `CMPL $0, %EAX` · `JL neg` · `JE zero` · `JMP pos`.
  Cada ramo termina com `JMP fim`.

## Verificações semânticas esperadas

Além da geração de código, o compilador deve rejeitar:

- `break` ou `continue` fora de um laço;
- `++`, `--` ou `+=` aplicados a variável `bool`, ou a algo que não é variável (`3++`);
- `swap` entre variáveis de tipos diferentes, ou com algo que não é variável.

## Observação sobre `exemplos/erros`

Os testes `e01` a `e09` de `exemplos/erros` verificam que o código inicial
**rejeita** as construções acima. Depois de implementada cada funcionalidade,
o teste correspondente deixa de produzir erro: isso é o esperado, e esses
testes podem ser removidos ou atualizados.
