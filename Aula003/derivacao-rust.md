# Gramática do loop `for` em Rust

## 1. Fonte da Gramática
Optaremos pela linguagem Rust, uma linguagem de programação multiparadigma, compilada e desenvolvida originalmente pela Mozilla.

Rust é projetada com foco em segurança, concorrência e desempenho (https://pt.wikipedia.org/wiki/Rust_(linguagem_de_programação)).

Para uma descrição oficial da linguagem, é recomendado utilizar a documentação oficial do Rust (https://doc.rust-lang.org/stable/).

A gramática oficial do Rust utiliza uma notação gramatical própria, baseada em conceitos de EBNF, mas com extensões relacionadas ao processo de parsing e inspiradas em PEG (Parsing Expression Grammar).

Por exemplo, a notação utiliza construções semelhantes às encontradas em EBNF:

| — representa uma alternativa;
? — indica zero ou uma ocorrência;
* — indica zero ou mais ocorrências;
+ — indica uma ou mais ocorrências;
( ... ) — indica agrupamento.
Além dessas construções, a notação do Rust possui operadores específicos relacionados ao parsing. Um exemplo é:

^ — chamado de hard cut, indica um ponto após o qual o parser não deve voltar atrás (backtrack) para tentar uma alternativa diferente.
Assim, a notação da gramática de Rust não pode ser caracterizada simplesmente como BNF, EBNF ou PEG puro.

## 2. Loop `for` em Rust

Em Rust, o loop `for` funciona como um iterador de coleções ou intervalos, e sua gramática básica segue o formato:

```bnf
<for-statement> ::= "for" <identifier> "in" <iterable-expression> "{" <body> "}"

<iterable-expression> ::= <identifier> | <expression>

<body> ::= <statement> | <statement> <body>

<statement> ::= <assignment> | <expression>

<assignment> ::= <identifier> "=" <expression>

<expression> ::= <identifier> | <literal> | <expression> <operator> <expression>

<operator> ::= "+" | "-" | "*" | "/"
```
 | Regra | Significado |
|---|---|
| `<for-statement>` | Representa a estrutura completa do laço for |
| `<identifier>` | Representa um nome de variável |
| `<iterable-expression>` | Representa o objeto ou expressão que será percorrida pelo laço |
| `<body>` | Representa o conjunto de instruções executadas a cada iteração |
| `<statement>` | Representa uma instrução individual, como atribuição ou expressão |
| `<assignment>` | Representa uma atribuição de valor a uma variável |
| `<expression>` | Representa uma expressão no geral |
| `<operator>` | Representa operadores aritméticos |

## 3. Defina o código a ser gerado

Escolha um pequeno trecho de código válido na linguagem.  O código deve ser suficientemente simples para que seja possível demonstrar sua derivação. Recomenda-se utilizar uma construção como uma atribuição, uma expressão aritmética, uma estrutura condicional ou uma chamada de função.  

```rust
let x = 10;
let y = x + 5;
```
A primeira declaração:

```rust
let x = 10;
```

possui:

`let` → palavra-chave;  
`x` → identificador;  
`=` → operador de atribuição;  
`10` → número inteiro;  
`;` → terminador da declaração. 

A segunda declaração:

```rust
let y = x + 5;
```

possui:

`let` → palavra-chave;  
`y` → identificador;  
`=` → operador de atribuição;  
`x` → identificador;  
`+` → operador aritmético;  
`5` → número inteiro;  
`;` → terminador.  

## 4. Realize a derivação

Comece pelo símbolo inicial da gramática.
Aplique sucessivamente as produções escolhidas.
Mostre cada etapa da derivação até chegar ao código ou à sequência de tokens correspondente ao código escolhido.  

### 4.1 Primeira etapa

Aplicamos:
```
<Programa> ::= <ListaDeclaracoes>
```

Resultado:
```
<Programa>
⇒ <ListaDeclaracoes>
```

Como temos duas declarações, utilizamos:
```
<ListaDeclaracoes> ::= <Declaracao> <ListaDeclaracoes>
```

Então:
```
<Programa>
⇒ <ListaDeclaracoes>
⇒ <Declaracao> <ListaDeclaracoes>
```

Agora precisamos gerar a primeira declaração.

Aplicamos:
```
<Declaracao> ::= "let" <Identificador> "=" <Expressao> ";"
```

Resultado:
```
<Programa>
⇒ <ListaDeclaracoes>
⇒ <Declaracao> <ListaDeclaracoes>
⇒ "let" <Identificador> "=" <Expressao> ";" <ListaDeclaracoes>
```

Substituímos <Identificador> por x:
```
⇒ "let" "x" "=" <Expressao> ";" <ListaDeclaracoes>
```

Agora substituímos <Expressao> por <Numero>:
```
⇒ "let" "x" "=" <Numero> ";" <ListaDeclaracoes>
```

E substituímos <Numero> por 10:
```
⇒ "let" "x" "=" "10" ";" <ListaDeclaracoes>
```

Temos:

```
let x = 10; <ListaDeclaracoes>
```

Versão BNF final:
```
<Programa>
⇒ <ListaDeclaracoes>
⇒ <Declaracao> <ListaDeclaracoes>
⇒ "let" <Identificador> "=" <Expressao> ";" <ListaDeclaracoes>
⇒ "let" "x" "=" <Expressao> ";" <ListaDeclaracoes>
⇒ "let" "x" "=" <Numero> ";" <ListaDeclaracoes>
⇒ "let" "x" "=" "10" ";" <ListaDeclaracoes>
⇒ "let" "x" "=" "10" ";" <Declaracao>
⇒ "let" "x" "=" "10" ";" "let" <Identificador> "=" <Expressao> ";"
⇒ "let" "x" "=" "10" ";" "let" "y" "=" <Expressao> ";"
⇒ "let" "x" "=" "10" ";" "let" "y" "=" <Expressao> "+" <Expressao> ";"
⇒ "let" "x" "=" "10" ";" "let" "y" "=" <Identificador> "+" <Expressao> ";"
⇒ "let" "x" "=" "10" ";" "let" "y" "=" "x" "+" <Expressao> ";"
⇒ "let" "x" "=" "10" ";" "let" "y" "=" "x" "+" <Numero> ";"
⇒ "let" "x" "=" "10" ";" "let" "y" "=" "x" "+" "5" ";"
```

## 5. Apresente o resultado

Mostre o código final gerado. Explique, com suas palavras, como as regras da gramática foram utilizadas para chegar ao código.  

O código gerado pela derivação foi:

```rust
let x = 10;
let y = x + 5;
```

Os não terminais são os símbolos que podem ser substituídos por outras produções:

```
<Programa> <ListaDeclaracoes> <Declaracao>
<Expressao> <Identificador> <Numero>
```

Os terminais são os símbolos que aparecem no código final:

```rust
let  x  y  =  +  10  5  ;
```

Assim, a aplicação das produções BNF transforma o símbolo inicial <Programa> no código Rust válido apresentado acima.



