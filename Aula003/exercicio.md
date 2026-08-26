# ***Realizando a Derivação:***

Referência: https://doc.rust-lang.org/reference/grammar.html#grammar-summary-SimplePath

## ***Código base:***

```rust
for x in y {
}
```

## ***Regras gramaticais usadas:***
Para concentrar a derivação no comando de repetição, usa-se uma gramática reduzida. Os símbolos entre < > são não terminais; os demais representam terminais (palavras-chave e pontuação) da linguagem.

```plainText
<expression> ::= <loop_expression> | <path_expression>

<loop_expression> ::= <iterator_loop_expression>

<iterator_loop_expression> ::= for <pattern> in <expression> 
<block_expression>

<pattern> ::= <identifier_pattern>

<identifier_pattern> ::= <identifier>

<path_expression> ::= <identifier>

<block_expression> ::= { <statements> } | { }

<identifier> ::= x | y
```

## **Derivação:**
A derivação abaixo começa no símbolo não terminal inicial <expression> (já que um loop em Rust é uma expressão) e substitui, passo a passo, cada não terminal até obter a sequência de terminais correspondente ao trecho desejado.

```PlanText
<expression>

<loop_expression>

<iterator_loop_expression>

for <pattern> in <expression> <block_expression>

for <identifier_pattern> in <expression> <block_expression>

for <identifier> in <expression> <block_expression>

for x in <expression> <block_expression>

for x in <path_expression> <block_expression>

for x in <identifier> <block_expression>

for x in y <block_expression>

for x in y { }
```
