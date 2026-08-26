# ***Realizando a Derivação:***

Referência: https://doc.rust-lang.org/reference/grammar.html#grammar-summary-SimplePath

## ***Código base:***

```rust
for x in y {}
```


## ***Derivação passo a passo:***
1. Simbolo inicial: ``Expression``.
2. O loop for é uma expressão de bloco: ``ExpressionWithBlock``. 
3. Especificando que é um loop: ``LoopExpression``. 
4. Especificando que é um loop de iteração(for): ``IteratorLoopExpression``.
5. Aplicando a produção principa do IteratorLoopExpression: ``for`` ``Pattern`` ``ìn`` ``Expression`` ``BlockExpression``.
6. Resolvendo o ``Patern`` (padrão da variável do loop):
    - ``for`` ``PatternNoTopAlt`` ``ìn`` ``Expression`` ``BlockExpression``.
    - ``for`` ``PatternWithoutRange`` ``ìn`` ``Expression`` ``BlockExpression``.
    - ``for`` ``IdentifierPattern`` ``ìn`` ``Expression`` ``BlockExpression``.
    - ``for`` ``IDENTIFIER`` ``ìn`` ``Expression`` ``BlockExpression``
O Lexer identifica o primeiro identificador como "x":
    - ``for`` ``x`` ``in`` ``Expression`` ``BlockExpression ``.

7. Resolvendo a Expression (o iterador, após o "in"):
    - ``for`` ``x`` ``in`` ``ExpressionWithoutBlock`` ``BlockExpression``

    - ``for`` ``x`` ``in`` ``PathExpression`` ``BlockExpression``

    - ``for`` ``x`` ``in`` ``PathInExpression`` ``BlockExpression``

    - ``for`` ``x`` ``in`` ``PathIdentSegment`` ``BlockExpression``

    - ``for`` ``x`` ``in`` ``IDENTIFIER`` ``BlockExpression``

O Lexer identifica o segundo identificador como "y":
    - ``for`` x ``in`` ``y`` ``BlockExpression``

8. Resolvendo o BlockExpression (o corpo do loop):
    - ``for`` ``x`` ``in`` ``y`` { Statements? }

Como nosso bloco está vazio, omitimos os Statements opcionais:
    - ``for`` ``x`` ``in`` ``y`` ``{`` `` }``