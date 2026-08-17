Exercícios 9 - 11 baseados no livro Concepts of Programming Languages 11th Edition by Robert Sebesta

#### Exercício 9

Pergunta: ***APL, SNOBOL e SIMULA 67 seguiram direções distintas. Associe cada linguagem ao seu foco e identifique uma contribuição duradoura de cada uma.***

-> Com base na evolução das principais linguagens de programação descritas pelo autor, APL, SNOBOL e SIMULA67 de fato tomaram rumos tecnológicos completamente distintos. A associação de cada uma ao seu foco e sua respectíva contribuição duradoura é a seguinte: 

##### APL (A programming language):
- Seu foco original: Ela foi desenvolvida por Kenneth E. Iverson na IBM por volta dos anos 60, ela não foi planejada inicialmente para ser uma linguagem de programação implementada, mas sim como um veículo para descrever arquiteturas de computadores. Caracteriza-se por ser uma linguagem dinãmico com tipagem e alocação dinâmica de armazenamento. 

- Contribuição: Sua grande contribuição é a inclusão de operadores extremamente poderosos voltados para operações unitárias em vetores e matrizes, isso da para a APL uma altíssima expressividade, pois permite realizar computações matemáticas complexas com programas pequenos. 

##### SNOBOL
- Foco principal: A linguagem foi projetada nos anos 60  por três pesquisadores nos Labaratórios Bell com o propósito claro e específico de fazer processamento de textos. Assim com a APL, ela compartilha uma característica de tipagem dinâmica de armazenamento. 

- Contribuição: SNOBOL foi a primeira linguagem aplamente conhecido a introduzir e popularir o pattern matching (casamento de padrões) de cadeias de caracteres de maneira integrada à linguagem. Embora tenha caído em desuso para editores de texto por ser lenta, suas operações de manipulação de strings influenciaram fortemente as linguagens de scripting modernas. 

##### SIMULA 67
- Foco principal:Desenvolvida pelos noruegueses Kristen Nygaard e Ole-Johan Dahl, o foco exclusivo de sua primeira versão (SIMULA I) era a simulação de sistemas complexos e a pesquisa operacional.

- Contribuição duradoura: Para permitir que subprogramas de simulação pudessem ser pausados e reiniciados de onde pararam (mecanismo conhecido como corrotinas), a linguagem introduziu a construção de classes. Essa inovação deu início ao conceito de abstração de dados e estabeleceu as fundações fundamentais para o surgimento da programação orientada a objetos


#### Exercício 11: 

Pergunta: ***11. Construa uma cadeia de influência que passe por ALGOL, Pascal e C. Depois contraste essa linhagem imperativa com a proposta declarativa de Prolog.***

A evolução das linguagens de programação imperativas possui uma árvore genealógica em que o ALGOL 60 atua como o ancestral comum mais significativo de sofisticação técnica. A partir dele, estabeleceu-se uma linhagem que moldou o desenvolvimento de Pascal e C:

- ALGOL 60: Introduziu conceitos revolucionários para o paradgma imperativo, como a estrutura em blocos (criação de escopos globais), subprogramas recursivos, passagem de parâmetros(por valor e nome) e vetores dinâmicos na pilha. Praticamente todas as linguagens imperativas subsequêntes herdam direta ou indiretamente suas características.

- ALGOL 60 → Pascal: Niklaus Wirth, que participou do comitê de evolução do ALGOL, propôs modificações à linguagem (como o ALGOL-W, criado em conjunto com Tony Hoare) que introduziram a estrutura de seleção múltipla case. Posteriormente, utilizando o ALGOL 60 como base de projeto direto, Wirth desenvolveu o Pascal. Pascal incorporou a estrutura case do ALGOL-W, registros similares aos de COBOL/PL/I e adotou a definição de tipos de dados definidos pelo usuário que havia sido pioneira no ALGOL 68.

- ALGOL 60 → C: Embora a linhagem sintática do C passe por seus ancestrais diretos CPL, BCPL e B, a linguagem C foi profundamente influenciada de forma direta pelo ALGOL 68. Essa herança do ALGOL 68 é claramente visível no C através do projeto de suas sentenças de controle for e switch, em seus operadores de atribuição acumulada e em seu mecanismo de tratamento de ponteiros.

#### ***Contraste: Linhagem Imperativa vs. Proposta Declarativa do Prolog:***
O contraste entre a linhagem imperativa (representada por ALGOL, Pascal e C) e a proposta declarativa (representada por Prolog) baseia-se em filosofias computacionais radicalmente opostas:

- ***Modelo Físico / Teórico:***
    - Linhagem Imperativa (ALGOL, Pascal, C):
        Baseada diretamente na arquitetura de von Neumann. As variáveis modelam as células físicas de memória, e as computações ocorrem por meio de alterações sequenciais desses estados

    - Proposta Declarativa (Prolog):
        Baseada em lógica simbólica (especificamente no cálculo de predicados de primeira ordem e nas cláusulas de Horn). Não depende de um modelo de máquina com estados físicos.

- ***Abordagem de Programação:***
    - Linhagem Imperativa (ALGOL, Pascal, C):
        Procedural (orientada a procedimentos). O programador precisa especificar detalhadamente como o computador deve processar os dados e em qual ordem exata as instruções e sentenças devem ser executadas

    - Proposta Declarativa (Prolog):
        Não procedural (declarativa). O programador não exprime o passo a passo da computação, mas define as características e a forma necessária que o resultado final deve possuir.

- ***Estrutura de Código:***
    - Linhagem Imperativa (ALGOL, Pascal, C):
        O programa é uma sequência lógica de instruções algorítmicas, atribuições e estruturas de controle (como laços while ou for).

    - Proposta Declarativa (Prolog):
        O programa é uma coleção estática de fatos (asserções que se assume verdadeiras) e regras (implicações lógicas entre proposições).

- ***Mecanismo de Execução:***
    - Linhagem Imperativa (ALGOL, Pascal, C):
        O compilador ou interpretador apenas traduz e executa as instruções e laços fornecidos pelo programador na ordem descrita.

    - Proposta Declarativa (Prolog):
        O computador utiliza um sistema de inferência lógico interno (baseado no princípio de resolução e unificação) para responder a consultas/objetivos de forma automática, buscando correspondências na base de dados.

Enquanto na linhagem imperativa o programador dita as ações passo a passo para alterar estados de memória, no Prolog o programador fornece o conhecimento lógico sobre o problema e deixa que o motor de inferência da linguagem encontre o caminho para a solução.