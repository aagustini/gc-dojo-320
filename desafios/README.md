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
| `05_swap` | `swap` | `swap a, b;` troca os valores de duas variáveis do mesmo tipo |
| `06_break_continue` | `break` e `continue` | atuam no laço mais interno; `continue` vai ao teste (`while`, `do-while`) |

## Ordem sugerida e dependências

1. **01** primeiro: os testes de 02, 03 e 06 usam atribuições e incrementos
   dentro de expressões e como comandos (`i++;`).
2. **04** e **05** juntos: `break` e `continue` precisam saber os rótulos do
   laço mais interno, que pode ser `while`, `do-while`, `for` ou `times`.
3. **06** usa 01, e o teste `d06b` usa `break` (05).


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
- **`swap`**: na máquina de pilha bastam quatro instruções
  (`PUSHL a` · `PUSHL b` · `POPL a` · `POPL b`).

## Verificações semânticas esperadas

Além da geração de código, o compilador deve rejeitar:

- `break` ou `continue` fora de um laço;
- `++`, `--` ou `+=` aplicados a variável `bool`, ou a algo que não é variável (`3++`);
- `swap` entre variáveis de tipos diferentes, ou com algo que não é variável.

