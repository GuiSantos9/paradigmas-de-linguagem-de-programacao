# Revisão para a prova de paradgmas no segundo bimenstre:

### 1. Tipos de Dados

#### 1.1 Inteiros
Quando tratamos de valores inteiros e de seus limites de memória(overflow) as linguagens de programação se dividem em três grandes filosofias. O ponto central é o que fazer quando uma operação matemática resulta em um número maior do que a memória alocada pode armazenar ? 

1. ***Silêncio(C, C++,Java)***
Nesta filosofia, as linguagens priorízam uma performance muito grande. Quando um limite é ultrapassado o valor "dá a volta", reiniciando do menor número possível, geralmente negativo sem emitir nenhum tipo de aviso, erro ou exceção.A responsabilidade de garantir que o estouro não aconteça é 100% do programador.
    - ***Perigo***: Esse estouro silêncioso é uma das maiores fontes de bugs da historia da computação. Como exemplos temos o foguete Ariane 5 que explodiu porque a conversão de um número de ponto flutuante de 64 bits para um inteiro de 16 bits estourou o limite silenciosamente, invertendo a direção dos motores.
    - ***Como o Java contorna:*** Se precisamos de segurança em Java, podemos abrir mão dos operadores matemáticos convencionais e usar metodos da classe ``Math``. O método ``Math.addExact(a,b)`` faz a adição e lança uma exceção caso ocorra um estouro (``ArithmeticException``).

````java
public class Estouro {
    public static void main(String[] args){
        int max = Integer.MAX_VALUE;

    // Estouro silencioso
    int resultadoSilencioso = max + 1;
    System.out.println(resultadoSilencioso); // Imprime: -2147483648 (Deu a volta sem avisar nada)

    // Verificação explícita
    int resultadoSeguro = Math.addExact(max, 1);
    // O programa quebra aqui, evitando um erro lógico silencioso
    }
}
````

2. ***Escolha Explícita(Rust)***
A linguagem rust entende que tanto segurança como performance são importantes, mas que a intenção do programador não pode ser ambígua, ou seja, a linguagem não vai tomar decisões automaticamente, sendo definidas pelo programador. A linguagem exige declaração de o que quer que aconteça.
    - ***Comportamento padrão:*** No modo de desenvolvimento (debug), o Rust entra em pânico (``panic``) e derruba o programa se houver um estouro, forçando você a consertar o bug. No modo de produção (``release``), ele faz o estouro silencioso por questões de performance.`
    - ***Escolha do programador:*** O diferencial do Rust é em ofereçer métodos explícitos para cada tipo de cenário. Isso é excelente para quando sabemos quais limites queremos atingir. Temos métodos como:
        - ``checked_add``: Retorna nulo se estourar.
        - ``wrapping_add``: Faz o estouro siliencioso
        - ``saturating_add``: Trava no valor máximo possível e não passa.
````Rust
fn main(){
    let max: u8 = 255; // Limite máximo de um inteiro de 8 bits

    // Escolha 1: Quero que dê a volta silenciosamente
    let volta = max.wrapping_add(1);
    assert_eq!(volta,0);

    // Escolha 2: Quero que trave no valor máximo possível
    let trava = max.saturating_add(1);
    assert_eq!(trava,255);

    // Escolha 3: Quero ser avisado se estourar para tratar o erro
    let aviso = max.checked_add(1);
    assert_eq!(aviso, None); // Retorna 'None' na falha
}
````  

3. ***Não tem Estouro***
Aqui, a linguagem abstrai completamente as limitações do hardware. Para o programador, o hardware é um detalhe invisível e a matemática deve funcionar como no mundo real. A prioridade aqui é a conveniência e a ausência de surpresas, mesmo que custe processamento e memória.
    - ***Python:*** O tipo ``int`` não tem limite de tamanho. Quando um número ultrapassa o limite natural do processador, o Python automáticamente converte a representação do inteiro em um array de dígitos na memória.O número cresce infinitamente até esgotar a memória RAM da máquina. Você nunca terá um estouro de limite numérico tradicional.
    - ***JavaScript:*** O JS lida com essa abstração de outra forma. Historicamente, ele nem sequer tem um tipo inteiro nativo clássico; o tipo Number é um double (ponto flutuante de precisão dupla do padrão IEEE 754). Isso significa que ele perde a precisão para números inteiros gigantes (acima de $2^{53} - 1$). Para resolver isso e permitir inteiros massivos (como no Python), o JavaScript introduziu o BigInt, um tipo específico para representar inteiros grandes de forma segura e sem estouros.

```` python
    numero_gigante = 10 ** 50
    print(numero_gigante + 1)
    # Imprime perfeitamente: 100000000000000000000000000000000000000000000000001
````

```` javascript
    const limiteSeguro = Number.MAX_SAFE_INTEGER;

    const calculoSeguro = 9007199254740991n + 50n;

    console.log(calculoSeguro);
    // Imprime: 9007199254741041n (Matemática exata, sem perda de precisão)
````

#### 1.2 Tipos flutuantes e decimais
Quando saímos do mundo dos números inteiros e entramos nas frações matemáticas, a computação moderna adota um padrão quase universal: o IEEE 754 (que é o padrão usado por baixo dos panos no tipo Number do JavaScript, mencionado anteriormente).No entanto, o uso do ponto flutuante esconde ilusões matemáticas que todo programador precisa conhecer.

1. ***Por que 0.1 + 0.2 não é 0.3?***
Em quase todas as linguagens, somar 0.1 + 0.2 resulta em ``0.30000000000000004``.
A cause desse comportamento é a representação binária finita de uma dízima. Assim como na base 10 não conseguimos escrever a fração 1/3 de forma exata, o computador, que opera na base 2 (binário), não consegue representar de forma exata números que parecem simples para nós, como o 0.1.Eles se transformam em dízimas binárias infinitas. Como a memória do computador tem um fim, ele é obrigado a cortar (truncar) essas últimas casas, perdendo a precisão exata do número.

2. ***Memória e Limites(Float x Double)***
O ponto de corte desse arredondamento depende do espaço de memória que você aloca para a variável:
    - ***float (32 bits)***: Ocupa menos memória, mas só consegue garantir uma precisão de aproximadamente 7 dígitos decimais.
    - ***double (64 bits)***: Usando o dobro de espaço, garante uma precisão muito maior, de aproximadamente 15 a 16 dígitos decimais

3. ***Nunca use ``==``***
Por causa dessa perda invisível de precisão no arredondamento, nunca compare números reais com ==. O computador pode achar que $0.30000000000000004$ é diferente de $0.3$.
A abordagem correta é usar uma tolerância (geralmente chamada na matemática de epsilon). Você subtrai um número do outro e verifica se a diferença entre eles é tão pequena que pode ser considerada zero.

4. ***Solução para o Dinheiro:***
Se o ponto flutuante arredonda sozinho e erra em sistemas críticos, como lidamos com transações bancárias onde nenhum centavo pode sumir?
***o valor não é exato em binário, por isso a soma acumula erro; usar decimal ou centavos.***
- Para resolver isso de forma definitiva, temos duas saídas no código:
    1. ***Tipo decimal:*** Algumas linguagens orientadas a negócios, como COBOL, C# e F#, possuem nativamente o tipo decimal (ou money), que faz as contas usando base 10 e não perde a precisão. Em linguagens sem esse recurso nativo, você deve usar bibliotecas oficiais, como BigDecimal (em Java) ou Decimal (em Python).
    2.***Centavos em inteiro***: Esta é a solução mais simples, à prova de falhas e mais rápida. Você ignora os pontos flutuantes por completo e armazena os valores financeiros apenas com o tipo inteiro multiplicando tudo por 100. Um produto de R$ 15,90 é salvo no banco de dados como 1590 centavos. Toda a matemática financeira flui perfeitamente com inteiros (pois não há dízimas de centavos) e você só divide por 100 novamente quando precisar mostrar o valor na tela do usuário.

##### ***Solução 1 em C# e Java***:
```` C#
double a = 0.1;
double b = 0.2;
Console.WriteLine(a + b); // Imprime: 0.30000000000000004 (Acumulou erro)

// Solução
decimal valorA = 0.1m;
decimal valorB = 0.2m;
Console.WriteLine(valorA + valorB); 
// Imprime: 0.3 (Matemática exata)
````

````Java
import java.math.BigDecimal;

public class Dinheiro {
    public static void main(String[] args){
        //Correto
        BigDecimal preco1 = new BigDecimal("0.1");
        BigDecimal preco2 = new BigDecimal("0.2");

        BigDecimal total = preco1.add(preco2);

        System.out.println(total);
    }
}
````
##### ***Solução 2 em JavaScript***:
````javascript
const precoProdutoCentavos = 1590; 
const precoFreteCentavos = 415;

const totalCarrinhoCentavos = precoProdutoCentavos + precoFreteCentavos;
console.log(totalCarrinhoCentavos);

const totalFormatado = (totalCarrinhoCentavos / 100).toFixed(2);

console.log(`Total a pagar: R$ ${totalFormatado.replace('.', ',')}`);
````

#### 1.3 Cadeias e Matrizes 
Quando lidamos com cadeias de texto (strings) e matrizes (arrays), entramos em um dos terrenos mais perigosos e debatidos da ciência da computação: o gerenciamento de memória e a verificação de limites.
1. ***Strings: O que realmente são ?***
A forma como uma linguagem entende uma string muda tudo sobre como programamos. Ela é um tipo primitivo ou uma matriz de caracteres? Tem tamanho estático ou dinâmico?
    - ***Mutabilidade:*** Em linguagens modernas como Java, Python e JavaScript, as cadeias de texto são imutáveis. Uma vez criadas, você não pode alterar um caractere no meio dela; qualquer alteração cria uma nova string na memória. Se precisar de performance para edições constantes, você deve usar estruturas mutáveis específicas (como o StringBuilder no Java). Já em C, uma string é apenas uma matriz de caracteres mutável.
    - ***C:*** Na linguagem C, a famosa função strcpy (usada para copiar strings) não faz a menor ideia do tamanho do destino. Se você copiar um texto de 100 letras para um espaço de 50, o C simplesmente vai sobrescrever o que estiver na frente, causando o famigerado estouro de buffer (buffer overflow).
    - ***Armadilha do Java:*** Como as strings em Java são objetos (e não tipos primitivos), usar == compara o endereço de memória (a referência), não o conteúdo do texto. Para comparar o valor real, você é obrigado a usar o método .equals().

````C
#include <stdio.h>
#include <string.h>

int main() {
    char destino[5] = "Olá"; 
    
    // O array 'destino' só suporta 5 bytes. 
    // O C não reclama e sobrescreve a memória à frente, podendo corromper o sistema.
    strcpy(destino, "Texto muito grande que não cabe aqui"); 
    
    return 0;
}
````

```` Java
public class StringsJava {
    public static void main(String[] args) {
        String a = new String("texto");
        String b = new String("texto");

        // ERRO: Compara se 'a' e 'b' apontam para o mesmo local na memória
        System.out.println(a == b); // Imprime: false

        // CORRETO: O método equals compara o conteúdo lógico do texto
        System.out.println(a.equals(b)); // Imprime: true
    }
}
````


2. ***O que acontece com ```v[3]``***
Imagine que você tem um array v com apenas 3 elementos (índices 0, 1 e 2). O que acontece se você tentar acessar o quarto elemento, fazendo a chamada v[3]?
    - ***C:*** Comportamento indefinido: Pode ler "lixo" de memória de outro programa, corromper dados ou causar o travamento total.
    - ***JavaScript:*** Retorna undefined, passando um erro silencioso para a frente.
    - ***Java:*** Lança a exceção ArrayIndexOutOfBoundsException e interrompe a execução.
    - ***Go/Rust:*** Causa um panic (Em Rust, a alternativa segura v.get(3) devolve None para ser tratado).
    - ***Python:*** Lança um IndexError. (Um charme extra do Python: índices negativos como v[-1] buscam o último elemento de trás para frente!).

```` javascript
const v = [10, 20, 30];
console.log(v[3]); // O JS não trava, apenas retorna: undefined
````
````python
v = [10, 20, 30]

# Python detecta o erro e interrompe o programa
# print(v[3]) -> Lança: IndexError: list index out of range

# Bônus: Índices negativos acessam de trás para frente
print(v[-1]) # Imprime: 30
````

3. ***O que é um caractere ?***
Contar "caracteres" não é uma ciência exata e depende da codificação da linguagem. Quando você pede o tamanho de uma string, linguagens diferentes contam coisas diferentes:
    - Go e Rust (usando len): Contam os bytes brutos.
    - Java e JS: Contam as unidades de código UTF-16 (alguns emojis ocupam duas posições em vez de uma!).
    - Python: Conta os code points (o caractere lógico, exatamente como o ser humano lê).
É por causa dessa diferença mecânica que len('ação') em Go dá 6 bytes (caracteres com acento ocupam mais bytes no UTF-8), mas em Python, dá 4 (ele conta os símbolos lógicos).

```` go
package main
import "fmt"

func main() {
    // Letras acentuadas no padrão UTF-8 ocupam 2 bytes cada ('ç' e 'ã').
    // 'a' (1) + 'ç' (2) + 'ã' (2) + 'o' (1) = 6 bytes
    fmt.Println(len("ação")) // Imprime: 6
}
````

````python
print(len('ação'))
````

4. ***Fatias (Slices) de Memória***
Quando queremos apenas um pedaço de uma matriz ou string (como v[1:3]), as linguagens têm filosofias de gestão de memória diferentes:
    - Em Go, as fatias (slices) compartilham a memória original. Se você altera a fatia, você altera o array original. É absurdamente rápido, mas exige muita atenção do programador.
    - Em Python, fatiar uma lista cria uma cópia independente na memória.

```` go
package main
import "fmt"

func main() {
    original := []int{10, 20, 30, 40}
    
    // Cria uma fatia com os elementos 20 e 30
    fatia := original[1:3] 
    
    // Alterar a fatia altera o array ORIGINAL! É rápido, mas perigoso.
    fatia[0] = 999 
    
    fmt.Println(original) 
    // Imprime: [10, 999, 30, 40]
}
````

```` python
original = [10, 20, 30, 40]

# Fatiar cria um novo objeto na memória com os elementos 20 e 30
fatia = original[1:3] 

# Alterar a fatia NÃO afeta o array original
fatia[0] = 999 

print(original) 
# Imprime intacto: [10, 20, 30, 40]
````
#### 1.4 Ponteiros e Referências
Chegamos a um dos tópicos mais complexos e fascinantes da computação: o gerenciamento de memória através de ponteiros e referências. O cientista da computação Tony Hoare, criador do algoritmo Quicksort, chegou a chamar a introdução dos ponteiros de **"um passo para trás"** (p. 280) na engenharia de software, devido à quantidade absurda de bugs que eles introduziram.

Para dominar esse assunto, você precisa treinar e fixar dois conceitos fundamentais que são frequentemente confundidos.

## O Vocabulário Essencial: Solto × Perdido

### 1. O Ponteiro Solto (*Dangling Pointer*)

**Regra:** Solto = usar depois de liberar.
Acontece quando um ponteiro aponta para um endereço de memória que já foi devolvido ao sistema (liberado) ou que simplesmente deixou de existir (saiu de escopo). Tentar acessar esse ponteiro gera **comportamento indefinido**.

Além disso, ponteiros permitem **apelidos** (duas variáveis apontando para o mesmo lugar). Se você liberar a memória usando um nome, o outro ponteiro instantaneamente se torna um "ponteiro solto".

**Exemplo Clássico em C (O escopo que sumiu):**

```c
int *cria(void) {
    int local = 42;
    return &local;  // ERRO GRAVE: Retorna o endereço de uma variável da pilha
                    // Assim que a função termina, essa memória 'some'.
                    // O ponteiro retornado nasce "solto".
}

```

### 2. A Variável do Monte Perdida (*Memory Leak* / Vazamento)

**Regra:** Perdida = perder a referência sem liberar.
Acontece quando você pede um espaço de memória ao sistema (no *Heap* / Monte), mas em algum momento do código você perde o endereço desse espaço antes de avisar ao sistema que não precisa mais dele. A memória fica "sequestrada" pelo seu programa para sempre.

**Exemplo Clássico em C (A referência apagada):**

```c
char *buf = malloc(100); // Aloca 100 bytes na memória
buf = NULL;              // Apagamos o endereço que estava em 'buf'.
                         // Bloco perdido: VAZAMENTO DE MEMÓRIA! 
                         // Ninguém mais consegue alcançar ou liberar esses 100 bytes.

```

---

## Como as Linguagens Evitam o Caos

O ecossistema das linguagens se divide em três grandes estratégias para resolver a equação: Manual (C) × Coleta de Lixo (Java/Go) × Posse (Rust).

| Linguagem | Mecanismo de Prevenção e Gerenciamento |
| --- | --- |
| **C** | **Manual:** Uso de `malloc` e `free` 100% manuais. Permite aritmética de ponteiros (somar e subtrair endereços). O programador tem poder total, mas é o único responsável por evitar ponteiros soltos e vazamentos. |
| **Java** | **Coleta de Lixo (GC):** Usa referências invisíveis. Não permite aritmética de ponteiros e proíbe o uso de `free`. A **coleta de lixo tira o `free` da mão do programador**, varrendo a memória em segundo plano para limpar o que não está mais sendo usado. |
| **Go** | **Ponteiros Seguros + GC:** Tem ponteiros explícitos, mas não tem aritmética de ponteiros e possui Coletor de Lixo. O compilador faz a chamada *Escape Analysis*: se você tentar fazer o erro do `return &local` (como no C acima), o Go percebe, move a variável da pilha para o monte, e a torna segura. |
| **Rust** | **Posse e Empréstimo:** O Rust muda o paradigma. Ele **move a verificação para o momento da compilação**. Através de regras rígidas de quem é o "dono" da memória, o compilador prova matematicamente que não há ponteiros soltos. Tudo isso sem usar um coletor de lixo. |

---

**Pergunta para a turma:**
Se o Rust não tem coletor de lixo, como evita o ponteiro solto?

#### 1.5 Null e uniões discriminadas
Para finalizar nossa jornada pela tipagem e segurança de memória, precisamos encarar de frente um dos maiores pesadelos da programação: a ausência de valor e como as linguagens lidam com tipos que podem assumir múltiplas formas.

## O Erro de Um Bilhão de Dólares

Na maioria das linguagens antigas ou orientadas a objetos clássicas, quando uma variável não tem valor, ela recebe `null`. O próprio criador do `null`, Sir Tony Hoare, o chamou publicamente de **“o erro de um bilhão de dólares”**, referindo-se à quantidade incalculável de tempo e dinheiro perdidos na indústria devido aos infames *NullPointerExceptions* (quando o programa tenta ler um valor vazio e quebra).

Para fugir do `null`, a computação moderna abraçou o conceito de **Uniões** (estruturas que podem guardar diferentes tipos de dados), mas a forma de implementar isso faz toda a diferença.

## A Evolução das Uniões

### 1. União Livre (C)

Na linguagem C, existe uma estrutura chamada `union`. Nela, vários campos diferentes **dividem exatamente o mesmo espaço físico na memória**. O problema? Não há absolutamente nada na memória que diga qual campo está em uso no momento.
Se você gravar um número decimal (float), mas tentar ler a variável como se fosse um texto (char), o C não vai impedir; ele vai te devolver "lixo" de memória (bits sem sentido).
**A regra aqui é: união livre é responsabilidade inteiramente do programador.**

### 2. União Discriminada (Rust, TypeScript, Java 21)

As linguagens modernas (usando `enum` no Rust, tipos literais no TypeScript ou classes `sealed` no Java 21) criaram a **União Discriminada**.
Nesse modelo, junto com o dado, a linguagem guarda uma "etiqueta" (tag) invisível que diz exatamente qual é o tipo atual daquela variável. Se você tentar ler o tipo errado, o programa te barra.
**A regra aqui é: na união discriminada, o compilador verifica e garante a segurança.**

## A Solução: Option e Optional

Em vez de devolver `null` quando algo dá errado ou não é encontrado, as linguagens modernas usam uniões discriminadas nativas, geralmente chamadas de `Option` ou `Optional`.

O `Option` é uma união que só tem dois estados possíveis: ou ele tem `Some` (Algum valor), ou tem `None` (Nenhum valor). Como é uma união discriminada, a linguagem te obriga a verificar a "etiqueta". **Se você esquecer de tratar o caso `None`, o código simplesmente não compila.**

### O Código na Prática

**Em Rust (`Option` com match exaustivo):**

```rust
// Rust: Option é uma união discriminada nativa
fn buscar(id: u32) -> Option<String> {
    if id == 1 { Some("Ana".into()) } else { None }
}

// O compilador te obriga a usar o 'match' para checar a etiqueta
match buscar(2) {
    Some(nome) => println!("{}", nome),
    None       => println!("não achei"), 
}   
// Se você esquecer de escrever a linha do 'None', dá erro de compilação!

```

**Em Java (Uso de `Optional`):**
O Java envelopa o valor em um objeto `Optional`, forçando o programador a declarar o que acontece se o valor não existir (usando o `orElse`).

```java
// Busca o id 2. Se achar, pega o tamanho do nome. Se não achar, o padrão é 0.
int tamanho = buscar(2).map(String::length).orElse(0); // Resultado: 0

```

### Tipos Anuláveis Checados

Vale ressaltar que algumas linguagens preferiram resolver o problema do `null` melhorando a própria sintaxe, criando o que chamamos de **tipos anuláveis checados**. Linguagens como **Kotlin** (usando `String?`), **TypeScript** (ativando o modo `strictNullChecks`), **C#** e **Swift** avisam em tempo de compilação se você tentar usar uma variável anulável sem antes verificar se ela está vazia.

---

Para resumir e fechar nossa linha de raciocínio: a combinação de **uma união + uma etiqueta** é o que finalmente dá segurança real de tipos à programação. E hoje, o padrão `Option` é, sem dúvida, o uso mais comum, elegante e seguro dessa teoria para exterminar o `null` de uma vez por todas.

**Pergunta para a turma:**
Que tipo de erro o *match* exaustivo impede?

### 2. Expressões, atribuição e controle 
## 1. Precedência, Associatividade e Efeitos Colaterais

A forma como uma linguagem resolve uma expressão matemática ou lógica depende de três pilares: qual operador é executado primeiro (**precedência**), o que fazer quando há operadores iguais lado a lado (**associatividade**) e o que acontece quando uma função altera o estado do programa (**efeitos colaterais**).

Quando temos operadores de mesma precedência, a associatividade dita a direção da leitura. Operadores como exponenciação (`**`), incremento (`++`), decremento (`--`) e o sinal de menos unário são **associativos à direita** (resolvidos da direita para a esquerda). Se houver qualquer dúvida ao ler o código, o uso de parênteses é sempre a melhor prática.

Os **efeitos colaterais** ocorrem quando uma função altera uma variável externa, grava um arquivo ou muda algo além de simplesmente retornar um valor. Se a linguagem não tiver uma ordem de avaliação estrita, o resultado se torna totalmente imprevisível. Em um paradigma sem efeitos colaterais (funcional), temos a "transparência referencial": `f(x) + f(x)` é matematicamente idêntico a `2 * f(x)`.

```python
# Python
print(-2 ** 2)       # Imprime -4 (o ** tem precedência sobre o menos unário)
print(2 ** 3 ** 2)   # Imprime 512 (associatividade à direita: primeiro 3**2=9, depois 2**9=512)

# Curiosidade: No Excel, a fórmula =-2^2 resulta em 4, pois ele prioriza o menos unário!

```

```java
// Java: A ordem de avaliação é estritamente garantida (da esquerda para a direita)
System.out.println(1 + 2 + "3" + 4 + 5); 
// Resultado: 3345. Ele soma 1+2=3, concatena com "3" virando "33", 
// e a partir daí tudo vira concatenação de texto: "334" -> "3345".

```

```c
/* C: A ordem de avaliação dos operandos NÃO é definida pelo padrão */
int a = 5;
int fun1(void) { a = 17; return 3; }

int main() {
    a = a + fun1(); 
    /* O resultado será 8 ou 20? 
       Depende do compilador! Se ele ler o 'a' primeiro (valendo 5), dá 8. 
       Se ele executar 'fun1()' primeiro, o 'a' vira 17, e o resultado dá 20.
       (No GCC, costuma dar 20).
       Efeitos colaterais + ordem não definida = resultado imprevisível. */
}

```

**Pergunta para a turma:** Qual a saída de `1 + 2 + "3" + 4 + 5` em Java? E por quê?

---

## 2. Conversões e Coerção

Quando tipos diferentes interagem, as linguagens precisam decidir se convertem os valores por você (coerção) ou se exigem que você faça isso manualmente.

O **alargamento** (passar um `int` para `long` ou `double`) costuma ser seguro, pois não há perda de dados. Já o **estreitamento** (passar de `double` para `int`) perde informação (como as casas decimais) e normalmente exige uma conversão explícita (um *cast*).

A **coerção** (conversão implícita) é perigosa porque esconde erros lógicos. Linguagens como C, JavaScript e PHP convertem tipos automaticamente para tentar ajudar, tornando o código mais curto, mas muito menos confiável. Linguagens modernas focadas em segurança, como Go e Rust, repudiam a coerção e exigem conversões explícitas (`float64(i)` ou `i as f64`).

Um **caso real** assustador de coerção ocorreu em sistemas antigos feitos em PHP: a expressão `"0e123" == "0e456"` retorna verdadeiro. O PHP interpreta ambos os textos como notação científica ($0 \times 10^{123}$ é igual a $0 \times 10^{456}$), o que permitiu que invasores passassem por verificações de *hash* de senhas enviando textos que o PHP convertia para zero.

```javascript
// JavaScript e a loucura da coerção implícita
"5" - 2       // 3
"5" + 2       // "52" (se tem string e o operador é +, ele concatena)
true + 1      // 2
null + 1      // 1
undefined + 1 // NaN

// O perigo do == (que faz coerção) vs === (que checa tipo rigorosamente)
0 == ""       // true
"" == "0"     // false
0 == "0"      // true   
// Atenção: O == do JS não é transitivo!
0 === "0"     // false (o === não converte, logo número é diferente de texto)

```

```java
// Java: Alargamento e a armadilha da divisão inteira
int grande = 123456789;
float f = grande;              // alargamento implícito
System.out.println((int) f);   // 123456792 (perdeu precisão devido ao limite do float)

// Armadilha clássica de divisão inteira:
System.out.println(1 / 2);     // 0 (Se ambos são inteiros, a resposta é inteira).
System.out.println(1 / 2.0);   // 0.5 (Forçando um double, a resposta vem em decimal).

// Se você fizer 'acertos / total * 100' com inteiros, o resultado será sempre 0.0!

```

**Pergunta para a turma:** Qual é o risco de usar `==` em vez de `===` em JavaScript?

---

## 3. Expressões Booleanas e Curto-Circuito

Nem todo operando de uma expressão lógica precisa ser lido. Os operadores `&&` (E) e `||` (OU) fazem **curto-circuito**: eles param de avaliar a expressão assim que o resultado matemático final já é conhecido. Em contraste, operadores como `&` e `|` (bit a bit ou booleanos estritos) avaliam sempre os dois lados.

O uso defensivo do curto-circuito salva sistemas todos os dias, garantindo que o lado direito só seja verificado se o lado esquerdo permitir. Se você escrever `i < n && v[i] != x`, o curto-circuito impede o acesso fora dos limites da matriz caso `i` já seja igual a `n`. O único cuidado é: se houver um efeito colateral no segundo operando, ele pode nunca ser executado.

Outro detalhe é a "verdade" dos valores (*truthiness*): C, Python e JS aceitam o valor `0`, matrizes vazias ou valores nulos como sendo equivalentes a "falso" diretamente no `if`. Java, Go e Rust são mais rigorosos e exigem que a expressão resulte estritamente em um Booleano (`true` ou `false`).

```java
// Java: O curto-circuito evita o erro fatal
String s = null;

// CORRETO: Retorna false imediatamente no primeiro teste.
boolean seguro = (s != null && s.length() > 0); 

// ERRO GRAVE: O '&' tenta avaliar os dois lados. 
// Acessar s.length() quando 's' é null causa um NullPointerException!
boolean erro = (s != null & s.length() > 0); 

```

```javascript
// JavaScript: Armadilhas do || e soluções modernas
const u = { nome: "Ana", endereco: null };

// Operador de Encadeamento Opcional (?.) 
// Não dá erro ao acessar propriedade de null, apenas retorna undefined
console.log(u.endereco?.cidade); 

const qtd = 0;
// Armadilha do || com valor 0 (que é lido como falso)
console.log(qtd || 10); // Imprime 10. O zero autêntico se perdeu!

// Solução: Operador de Coalescência Nula (??)
// Só substitui se for estritamente null ou undefined.
console.log(qtd ?? 10); // Imprime 0.

```

```python
# Python: and/or devolvem um dos operandos, não apenas True/False
print(0 or "padrão") # Imprime "padrão" (pois 0 é falso)

# Em Python, isso é válido e resulta True, 
# pois é avaliado como (3 > 2) and (2 > 1)
print(3 > 2 > 1)     

# Atenção: Se esse mesmo código (3 > 2 > 1) fosse escrito em C, o resultado seria 0 (falso). 
# Em C, (3 > 2) resulta em 1 (verdadeiro). Em seguida, ele avalia 1 > 1, o que é falso (0).

```

**Pergunta para a turma:** Em quais situações o curto-circuito evita um erro de execução?

---

## 4. Seleção: Else Pendente e Switch

O controle de fluxo muitas vezes engana a visão do programador: a estrutura "parece certa" por causa da endentação, mas o compilador entende outra coisa.

O **else pendente** (*dangling else*) é um problema clássico em C, Java, C++ e C#. A regra da linguagem diz que um `else` sempre pertence ao `if` mais próximo que não tenha um `else`, ignorando completamente os espaços e parágrafos do texto. O uso de chaves `{}` obrigatórias (como feito no Go, Rust e Perl) elimina esse problema estruturalmente.

Já o **switch do C** sofre de uma escolha de projeto controversa: o *fall-through* (cair através). Se você não colocar um `break` explicitamente, o controle do programa simplesmente continua descendo e executando os casos seguintes.

```c
/* C: O perigo do fall-through no switch */
int index = 1;
int odd = 0, even = 0;

switch (index) {
  case 1:
  case 3: odd += 1;
  case 2:
  case 4: even += 1;
  default: printf("Error\n");
}  
/* O que acontece se index for 1?
   O fluxo entra no case 1, desce pro 3 e soma odd. 
   COMO NÃO TEM BREAK, ele continua caindo! 
   Entra no case 2/4 e soma even. 
   Continua caindo, entra no default e imprime "Error".
   Ele executa tudo de uma vez. */

```

Linguagens modernas consertaram essa falha arquitetônica com novas abordagens:

```java
// Java 14+: Switch introduzido como expressão (usa '->' e não tem fall-through)
String tipo = switch (dia) {
  case SAB, DOM -> "fim de semana";
  case SEG, TER, QUA, QUI, SEX -> "dia útil";
};

```

```rust
// Rust: Match rigoroso e exaustivo. 
// Esquecer de mapear um caso gera erro de compilação imediato.
match n { 
    i32::MIN..=-1 => "negativo", 
    0 => "zero",
    1..=9 => "um dígito", 
    _ => "vários dígitos" // O '_' captura o restante (default)
}

```

**Pergunta para a turma:** Como você descreveria, em uma frase, o que é *fall-through* e qual a sua consequência?

---

## 5. Laços, Goto e Comandos Protegidos

O fluxo de controle de um programa não é apenas uma questão de estilo; é a base da segurança e da legibilidade. Em 1968, o cientista da computação Edsger Dijkstra publicou o famoso artigo **"O comando Goto é considerado prejudicial"**, provando que o código macarrônico criado por pulos desenfreados na memória tornava impossível verificar se um programa estava logicamente correto. Böhm e Jacopini já haviam provado em 1966 que apenas três pilares (sequência, seleção e laço lógico) são suficientes para escrever qualquer algoritmo do mundo.

Hoje, os "gotos" sobrevivem de forma estritamente controlada e legível através de comandos como `break`, `continue` e `return` (ou o `break` rotulado do Java). Da mesma forma, os **comandos protegidos** idealizados por Dijkstra (onde múltiplas condições verdadeiras levam a uma escolha não determinística) serviram de inspiração direta para blocos avançados como o comando `select` em Go.

**A falta de laços estruturados e os erros de controle causam tragédias corporativas reais:**

1. **Bug Zune (2008):** Os tocadores de MP3 da Microsoft travaram no mundo todo. O motivo? Um laço lógico `while` de cálculo de anos bissextos que, no último dia do ano, não tinha garantia de progresso. A variável nunca era incrementada e o laço rodava infinitamente.
2. **O "Goto Fail" da Apple (2014):** Uma das maiores falhas de segurança do iOS. Uma linha de código duplicada (`goto fail;`) sem estar envolvida por chaves contornou silenciosamente a verificação de assinatura dos certificados SSL.

```javascript
// JavaScript: A pegadinha clássica do for...in vs for...of
const notas = [10, 20, 30];
let total = 0;

// for...in percorre as CHAVES/ÍNDICES (como strings "0", "1", "2")
for (const i in notas) total += i;
console.log(total); // Resultado bizarro: "0012" (ele concatenou strings!)

total = 0;
// for...of percorre os VALORES reais do vetor (10, 20, 30)
for (const v of notas) total += v;
console.log(total); // Resultado correto: 60

```

```java
// Java: O "goto" moderno, seguro e restrito (Break rotulado)
// Permite quebrar múltiplos laços aninhados de forma controlada
externo:
for (int i = 0; i < m.length; i++) {
    for (int j = 0; j < m[i].length; j++) {
        if (m[i][j] < 0) { 
            break externo; // Pula diretamente para fora do laço principal
        }
    }
}

```

**Pergunta para a turma:** Que mecanismos do Go e do Rust teriam evitado o erro 'goto fail'?

### Subprogramas
## 1. Fundamentos e Parâmetros

Para entender como as funções operam, precisamos separar a teoria da prática, ou seja, separar quem define a função de quem a chama.

* **Cabeçalho e Chamada:** O cabeçalho define o nome e os **parâmetros formais** (as variáveis internas da função). A chamada da função é quem injeta os **parâmetros reais** (os valores de fato).
* **Perfil vs Protocolo:** O **perfil** (ou assinatura) é estritamente o número, a ordem e os tipos dos parâmetros. Já o **protocolo** vai além: é o perfil + o tipo do valor de retorno.
* **Tipos de Parâmetros:** Eles podem ser posicionais (ordem importa), por palavra-chave (ordem não importa, mas exige conhecer o nome exato do parâmetro), com valor padrão (se não for passado, assume um valor) e variádicos (aceitam uma quantidade indefinida de itens).

**Atenção (A Armadilha do Python):** Em Python, um valor padrão mutável (como `lista=[]`) é criado **uma única vez** quando a função é definida e acaba sendo compartilhado entre todas as chamadas futuras.

```python
# Python: formais x reais, palavra-chave e padrão
def calcular_salario(renda, aliquota, dependentes=1):
    return renda * (1 - aliquota) + 100 * dependentes

calcular_salario(2000.0, 0.15)                 # posicional + padrão
calcular_salario(aliquota=0.15, renda=2000.0)  # palavra-chave (ordem invertida)

def media(*valores): 
    pass                                       # variádico (recebe tupla)

# A clássica armadilha do padrão mutável
def adicionar(item, lista=[]):   # lista é criada UMA vez na memória
    lista.append(item)
    return lista

```

Faça uma tabela mental sobre os padrões linguísticos: Python suporta palavra-chave, valor padrão e variádico; Java não tem valor padrão (usa sobrecarga de métodos); C++ exige que valores padrão fiquem apenas no final da assinatura. Não decore, apenas entenda o porquê de cada linguagem adotar essa flexibilidade (ou a falta dela).

**Pergunta para a turma:** O que imprime `adicionar(1)` seguido de `adicionar(2)` no código Python acima?

---

## 2. Modos e Métodos de Passagem

Uma coisa é o **modelo semântico** (para onde vão os dados: entram, saem, ou ambos). Outra é a **implementação** (como a linguagem faz isso acontecer: copiando o dado ou passando o caminho de acesso até ele).

| Método | Modo | Característica |
| --- | --- | --- |
| **Valor** | entrada | Copia o valor. É seguro para o chamador, mas caro/lento para dados muito grandes. |
| **Resultado** | saída | Copia o valor de volta para o chamador no fim da função. |
| **Valor-resultado** | entrada e saída | Duas cópias (na ida e na volta). Não gera apelidos de memória. |
| **Referência** | entrada e saída | Passa o endereço original. É altamente eficiente e gera apelidos (quem altera a cópia, altera o original). |
| **Nome** | entrada e saída | Reavalia o texto a cada uso (histórico, usado no ALGOL 60). |

**Como as linguagens implementam a alteração da variável de quem chama?**

* **C:** Exige ponteiros: `f(&x)`. Detalhe importante: o ponteiro em si é sempre passado por valor (é uma cópia do endereço).
* **C++:** Possui passagem real por referência: `void f(int &x)`.
* **Java:** Não dá para alterar tipos primitivos; você é obrigado a devolver o novo valor com `return`.
* **C#:** Usa palavras-chave explícitas `ref` / `out`.
* **Python:** Passagem por atribuição. Você não altera imutáveis, deve devolver o novo valor.
* **Go / Rust:** Usa ponteiros expressos `&x` / `&mut x`.

**Pergunta para a turma:** Por que a função de inversão `troca(a, b)` em C só funciona se usarmos ponteiros?

---

## 3. A Grande Regra: Java Passa TUDO Por Valor

Este é o ponto mais cobrado em testes e entrevistas. Para dominar o conceito, treine o raciocínio: *"o que está sendo copiado para a função? O valor literal ou a referência (endereço)? E o que a função está fazendo com essa cópia?"*

* **Primitivos (`int`, `boolean`, etc):** Copia-se o valor literal. O chamador **nunca** vê a alteração, pois a função está mexendo apenas no clone local.
* **Objetos:** Copia-se o valor da referência (o endereço). Como as duas referências (a do chamador e a local da função) apontam para a mesma casa na memória, se a função alterar o **conteúdo** do objeto (ex: `p.x = 99`), o chamador enxerga a mudança.
* **Reatribuição:** Se a função tentar reatribuir a variável (fazer `p = new Ponto()`), ela só alterou a própria cópia da referência. O objeto original do chamador fica intacto.

```java
// Java
static void troca(int a, int b) { int t = a; a = b; b = t; }
static void altera(Ponto p)     { p.x = 99; }
static void reatribui(Ponto p)  { p = new Ponto(-1); }

troca(x, y);        // x=1, y=2: a função trocou apenas as cópias locais. x e y seguem 1 e 2.
altera(p);          // p.x = 99: o CONTEÚDO do objeto compartilhado foi modificado.
reatribui(p);       // O 'p' do chamador continua com x = 99. Apenas a referência local da função foi reatribuída.

```

O **Python** é parecido sob o nome de *passagem por atribuição*. Listas são mutáveis (o chamador vê `l.append(4)`), inteiros são imutáveis (`n = n + 1` altera apenas o nome local), e reatribuir um nome inteiro (`l = [0,0,0]`) não afeta o chamador.

**Pergunta para a turma:** Se Java não passa tipos primitivos por referência, como um método Java poderia 'devolver' dois ou mais valores ao chamador simultaneamente?

---

## 4. Fechamentos (Closures)

Pense no fechamento como "uma função que vai viajar e leva nas costas a mochila das variáveis que costumava usar".
O **fechamento** é a combinação de um subprograma com o ambiente exato em que ele foi definido.

O grande pulo do gato é que o fechamento **captura a variável** (a célula de memória em si), não apenas uma cópia do valor dela. Como a função externa que a criou vai terminar e sair da pilha de execução, as variáveis capturadas ganham "extensão ilimitada" e são movidas para o **monte (heap)**, sobrevivendo ao fim da execução da função original.

Eles são a base de estados privados, callbacks em web e fábricas de funções.

```javascript
// JavaScript: Fábrica de Funções (Livro Sebesta, p. 405)
function makeAdder(x) {
  return function (y) { return x + y; };
}
const add10 = makeAdder(10);
const add5  = makeAdder(5);

console.log(add10(20)); // 30
console.log(add5(20));  // 25

// Estado Privado: A variável 'n' sobrevive após o return graças ao closure!
function criarContador() {
  let n = 0;
  return { inc: () => ++n, get: () => n };
}
const c = criarContador();
c.inc(); c.inc();
console.log(c.get());   // 2

```

*(Atenção aos loops no JS antigo: `var` criava uma única variável para a função toda, fazendo o closure capturar sempre o último valor. `let` cria uma nova variável por volta do laço, resolvendo o problema).*

**Pergunta para a turma:** Se o fechamento copiasse estaticamente apenas o valor e não a célula de memória original, o que aconteceria ao tentarmos usar o método `inc()` no contador?

---

## 5. Pilha de Execução e Recursão

Cada vez que um subprograma é chamado, o sistema não pode simplesmente pular para lá. Ele precisa salvar o estado atual, passar os parâmetros e guardar o endereço de retorno para saber para onde voltar. Tudo isso é empacotado no que chamamos de **Registro de Ativação** (ou IRA). O IRA armazena as variáveis locais, parâmetros, a ligação dinâmica e o tal endereço de retorno.

Na **recursão**, várias chamadas da mesma função existem e rodam simultaneamente. Como a pilha segue a regra do "último a entrar, primeiro a sair", a ativação mais recente fica sempre no topo; o último chamado é o primeiro a terminar. Se você esquecer de colocar o caso base (a condição de parada), as ativações se empilham até a memória acabar, resultando num erro fatal: `RecursionError` (Python), `StackOverflowError` (Java) ou `Segmentation Fault` (C).

```c
int fatorial(int n) {
    if (n <= 1)
        return 1;
    return n * fatorial(n - 1);
}
// fatorial(3) empilha as chamadas na memória nesta ordem:
// Topo: fatorial(1)
// Meio: fatorial(2)
// Base: fatorial(3)

int contador(void) {
    static int n = 0;   // A memória estática lembra do valor entre chamadas! 
                        // Ela mora na área de dados, não na pilha.
    return ++n;
}
// 3 chamadas sucessivas retornam: 1, 2, 3
// (Se fosse int n = 0 normal, retornaria: 1, 1, 1)

```

*(Dica: A recursão em cauda — onde a chamada recursiva é estritamente a última coisa que acontece antes de retornar — pode ser otimizada por alguns compiladores para virar uma iteração comum, impedindo a pilha de crescer infinitamente. Linguagens como Scheme garantem isso por padrão).*

**Pergunta para a turma:** Quando a função de fatorial acima for chamada com `n = 1` a partir de um `fatorial(3)` inicial, quantas ativações de `fatorial` existem ativas na pilha naquele exato milissegundo?

### TAD e orientação a objetos
Chegamos agora ao coração do design de software moderno: a estruturação dos dados e o paradigma da Orientação a Objetos (OO). Vamos desconstruir como as linguagens encapsulam, herdam e despacham comportamentos.

## 1. Tipo Abstrato de Dados (TAD) e Ocultação

Na literatura de linguagens de programação, para que uma estrutura seja considerada um verdadeiro **Tipo Abstrato de Dados (TAD)**, ela precisa obrigatoriamente satisfazer duas condições:

1. **Representação oculta:** Os dados internos não podem ser vistos nem manipulados diretamente pelo cliente (o código que usa o TAD).
2. **Unidade sintática única:** A declaração do tipo e os protocolos (assinaturas) das operações permitidas devem ficar agrupados em um único lugar.

As vantagens são imensas: aumenta a confiabilidade (protege a invariante do tipo), exige que o cliente leia menos código para entender como usar, reduz o conflito de nomes e, principalmente, **permite que o criador da classe mude a representação interna sem quebrar os clientes**.
Por exemplo, os métodos de acesso (como um "setter") garantem que o cliente só insira dados válidos.

**A Ocultação nas Diferentes Linguagens:**

* **C++ / Java / C#:** Possuem a palavra-chave explícita `private`.
* **Go:** Letra inicial minúscula define que é privado ao pacote.
* **Rust:** Tudo é privado por padrão; usa-se `pub` para exportar.
* **JavaScript:** Usa o prefixo `#` para campos estritamente privados (recurso moderno).
* **Python:** A ocultação é **apenas uma convenção**. Usa-se o prefixo `_` para avisar "por favor, não mexa aqui", mas a linguagem não impede o acesso. Sem uma ocultação real forçada, mudar a representação pode quebrar clientes desavisados.

```java
// Java: Representação estritamente privada (Garante o TAD)
class Pilha {
    private int[] dados;
    private int indiceTopo = -1;
    
    Pilha(int capacidade) { dados = new int[capacidade]; }
    public void empilhar(int x) { dados[++indiceTopo] = x; }
    public boolean vazia() { return indiceTopo == -1; }
}
// Se o cliente tentar: p.indiceTopo = 5; 
// O compilador bloqueia com erro! A invariante está protegida.

```

```python
# Python: Privado por convenção (A ilusão do TAD)
class Pilha:
    def __init__(self): 
        self._dados = []  # O '_' é só um aviso visual

p = Pilha()
p._dados.insert(0, 999)   # Funciona perfeitamente! Nada é realmente privado.

```

**Pergunta para a turma:** O que o cliente pode (e não pode) fazer com uma pilha bem projetada?

---

## 2. Herança, Composição e Subtipos

Na Orientação a Objetos, quando queremos criar uma classe a partir de outra, temos duas rotas principais: Herança ("é um") e Composição ("tem um").

* **Herança:** A subclasse herda o comportamento e os campos do pai. Campos `protected` são visíveis à subclasse, mas `private` não. O grande risco da herança é expor toda a interface do pai, o que pode quebrar a ocultação do filho. (Um caso real terrível da biblioteca padrão do Java é a classe `Stack` herdando de `Vector`: como herdou métodos genéricos do vetor, o cliente consegue inserir um elemento no MEIO da pilha, ferindo completamente a definição de TAD!).
* **Composição:** A classe guarda o objeto dentro dela em um campo privado e expõe apenas os métodos estritamente necessários. **Prefira composição quando a subclasse não honra o contrato do pai.**
* **Subtipo (Princípio da Substituição de Liskov):** Para que algo seja um subtipo seguro, a subclasse deve poder substituir a classe pai em qualquer lugar do código sem alterar o comportamento esperado pelo cliente.

Herdar apenas para "reutilizar código" fere esse princípio frontalmente. Veja o clássico problema do Retângulo e Quadrado:

```python
# Python: A quebra do Princípio da Substituição
class Retangulo:
    def __init__(self, l, a): 
        self.l, self.a = l, a
        
    def definir_largura(self, l): 
        self.l = l
        
    def area(self): 
        return self.l * self.a

class Quadrado(Retangulo):
    def __init__(self, l): 
        super().__init__(l, l)
        
    def definir_largura(self, l): 
        # Em um quadrado, alterar a largura OBRIGA a alterar a altura
        self.l = self.a = l

# O cliente espera lidar com a matemática de um Retângulo
def redimensiona(r):          
    r.definir_largura(4)
    return r.area()

print(redimensiona(Retangulo(2, 3)))   # Imprime 12 (Largura 4 x Altura 3). Correto.
print(redimensiona(Quadrado(2)))       # Imprime 16 (Largura 4 x Altura 4). 
# O comportamento MUDOU! O cliente não esperava que a altura fosse alterada sozinha.
# Lição: Quadrado NÃO É um subtipo seguro de Retângulo na programação.

```

**Pergunta para a turma:** Em que situação a herança é mais adequada que a composição?

---

## 3. Polimorfismo e Vinculação Dinâmica

A magia do polimorfismo acontece através da **referência polimórfica**: quando uma variável declarada como o tipo `Forma` aponta na memória para um objeto filho do tipo `Circulo`.

A pergunta-chave de qualquer questão de polimorfismo em provas e entrevistas é: **a decisão de qual código executar é tomada na compilação ou na execução?**
Isso é o que chamamos de **Vinculação Dinâmica** (ou despacho dinâmico): o método executado é escolhido em tempo de execução, baseando-se no tipo real do *objeto* na memória, e não no tipo da variável que o aponta.

* **Java:** Por padrão, a vinculação de métodos é dinâmica (em tempo de execução). As únicas exceções são métodos `static`, `private` e `final`, que são ligados na compilação. **Cuidado:** Campos (variáveis de classe) **nunca** são polimórficos! `A x = new B(); x.nome` usa sempre o campo da classe da variável (`A`).
* **C++:** É necessário usar a palavra-chave `virtual` explicitamente. Sem o `virtual`, a ligação é estática (decidida pelo tipo do ponteiro na compilação).

```java
// Java: Vinculação Dinâmica (Pelo tipo do Objeto)
abstract class Forma {
    abstract double area(); // Classe abstrata: tem protocolo, mas não implementa.
    
    String descrever() {
        return getClass().getSimpleName() + ": " + area();
    }
}
class Circulo extends Forma { double area() { return 3.14; } }
class Retangulo extends Forma { double area() { return 10.0; } }

// O laço não sabe quais são as formas específicas na compilação,
// mas em tempo de execução o método area() correto de cada filho é chamado!
for (Forma f : formas) {
    System.out.println(f.descrever());   
}

```

```cpp
// C++: A armadilha do Virtual
B b; 
A *p = &b; // Ponteiro do tipo A apontando para objeto do tipo B

p->estatico();   // Chama A::estatico (Não era virtual, decidiu pelo tipo do ponteiro)
p->dinamico();   // Chama B::dinamico (Era virtual, decidiu pelo tipo real do objeto)

```

**Pergunta para a turma:** Na Aula 09, o que imprime `x.nome + x.getNome()` quando executamos `A x = new B()` e ambas as classes (`A` e `B`) têm um campo chamado `nome` e sobrescrevem o método `getNome()`?

---

## 4. Por Dentro: Vtable, Genéricos e OO sem Classes

Como as linguagens implementam a vinculação dinâmica por baixo dos panos? Elas usam a **vtable** (Virtual Method Table).

Para cada classe, o compilador cria uma tabela de ponteiros para os métodos virtuais. Quando você cria um objeto, o sistema insere secretamente dentro dele um ponteiro para a vtable da classe dele.
Se a subclasse não sobrescreveu um método, a entrada na vtable dela simplesmente aponta para o código da classe pai. Quando você chama `f.draw()`, o processador segue o ponteiro secreto do objeto até a vtable, pega o endereço de memória do método correto e pula para lá. (Curiosidade: O núcleo do Linux faz exatamente isso manualmente usando `structs` em C e ponteiros explícitos para funções).

### Genéricos (TADs Parametrizados)

Quando lidamos com tipos genéricos (ex: `List<T>`), as linguagens têm filosofias de implementação totalmente diferentes:

* **C++ e Rust:** Fazem a expansão macro (Templates/Monomorfização). O compilador gera uma cópia do código completa para cada tipo usado (gera um código para `List<Int>`, outro para `List<String>`). Fica muito rápido, mas aumenta o tamanho do binário.
* **Java:** Usa o *Apagamento de Tipo* (Type Erasure). Para a máquina virtual, não existe `<T>`, tudo vira `Object`. É ótimo para retrocompatibilidade, mas você não consegue saber o tipo real em tempo de execução.
* **C#:** Usa tipos *Reificados*. O tipo genérico é preservado até a execução, evitando os problemas do Java.
* **Go (1.18+):** Implementa genéricos suportados por interfaces como restrições.

### Orientação a Objetos SEM Herança de Classes

Para fugir dos problemas clássicos da herança, muitas linguagens modernas não possuem classes tradicionais:

| Linguagem | Alternativa à Herança de Classe Tradicional |
| --- | --- |
| **Go** | Não tem classe nem `extends`. Usa composição explícita (*embedding*) e interfaces implícitas (se você tem os métodos, você satisfaz a interface automaticamente). |
| **Rust** | Usa `trait` para definir comportamentos compartilhados e `dyn Trait` quando precisa de despacho dinâmico. |
| **JavaScript** | Orientação a objetos baseada em Protótipos: não existem classes (a palavra `class` é só açúcar sintático), objetos herdam diretamente de outros objetos em tempo de execução. |
| **Ruby / Python** | Tudo no sistema é um objeto e o despacho é **sempre** dinâmico (Duck Typing). |

**Resumo de O.O.:** Esconder: private/pub/convenção. Reutilizar: herança, interfaces, composição, traits. Despachar: sempre (Java), por padrão (Python) ou explicitamente com `virtual` (C++).

**Pergunta para a turma:** O que a vtable de `B` guarda na posição de um método que `B` herdou de `A`, mas não sobrescreveu?

### Concorrencia

Chegamos ao último e mais desafiador nível do design de software: a **concorrência**. Fazer o código rodar linha por linha é fácil; o problema começa quando múltiplas tarefas tentam manipular a mesma memória ao mesmo tempo.

## 1. Tarefas, Sincronização e Condição de Corrida

Uma **tarefa** (ou *thread*) é uma unidade de execução. Quando uma tarefa inicia outra de forma concorrente, ela não fica parada esperando a outra terminar; ambas prosseguem de forma independente.
Essa concorrência pode ser **física** (vários núcleos do processador executando código simultaneamente, o verdadeiro paralelismo) ou **lógica** (um único núcleo intercalando milissegundos de atenção entre várias tarefas).

Quando tarefas coexistem, elas podem atuar de duas formas:

1. **Cooperação:** Uma tarefa depende da outra. Exemplo clássico: o padrão Produtor-Consumidor (a tarefa que processa dados precisa esperar a tarefa que baixa os dados).
2. **Competição:** Duas tarefas independentes querem alterar o mesmo recurso ao mesmo tempo. Aqui nasce a necessidade de **exclusão mútua** (trancar a porta para só um usar por vez).

### A Condição de Corrida (*Race Condition*)

Se não houver sincronização, temos a **condição de corrida**: o resultado final do programa passa a depender puramente da sorte (ou seja, da ordem de intercalação que o processador escolher). O maior causador de bugs de concorrência é a ilusão de que operações simples são atômicas (feitas em um passo só).

O comando `contador++` no código **não é um passo**, mas três: (1) ler o valor da memória, (2) somar 1 na CPU, (3) gravar o novo valor na memória.

**O Exemplo do Livro (Figura 13.1):**
Imagine um programa onde um total começa em 3. Duas tarefas (A e B) tentam somar valores diferentes a esse total ao mesmo tempo. O resultado varia dependendo de quem o processador pausa e quando:

| Ordem dos passos na CPU | Resultado (TOTAL) |
| --- | --- |
| **A roda inteira, depois B** | 8 (O resultado matematicamente correto) |
| **B roda inteira, depois A** | 7 (A matemática já falhou dependendo das variáveis) |
| **As duas leem 3 juntas; A grava por último** | 6 (A sobrescreve o trabalho de B - *atualização perdida*) |
| **As duas leem 3 juntas; B grava por último** | 4 (B sobrescreve o trabalho de A - *atualização perdida*) |

*(Se você rodar esse programa mil vezes, verá quatro saídas diferentes, e ironicamente, o resultado certo costuma ser o menos provável em alto volume).*

```java
// Java: O perigo da falta de exclusão mútua
static int contador = 0;
Runnable r = () -> {
    for (int i = 0; i < 1_000_000; i++)
        contador++; // ler, somar, gravar
};
// Se duas threads executarem 'r' ao mesmo tempo...
// O esperado: 2.000.000. 
// O obtido na vida real: 1.011.017 (e o número muda a cada execução!)

```

**Pergunta para a turma:** Por que `contador++` não é uma operação atômica?

---

## 2. Impasse (Deadlock)

A exclusão mútua (usar travas) resolve a condição de corrida, mas se for mal usada, cria um problema ainda pior: o **Impasse** (ou *Deadlock*).

* **Corrida × Impasse:** A corrida corrompe os dados em silêncio (o programa termina, mas os valores estão errados). O impasse congela o programa para sempre, sem disparar nenhum erro na tela. A "vivacidade" (*liveness* - a garantia de que o programa continua progredindo) é destruída.

O impasse ocorre quando duas ou mais tarefas ficam bloqueadas, cada uma segurando uma trava e esperando o recurso que a outra está segurando. É a clássica **espera circular**:

> **Diagrama do Impasse:**
> Tarefa A segura o **Recurso X** e precisa do **Recurso Y**.
> Tarefa B segura o **Recurso Y** e precisa do **Recurso X**.
> Resultado: Ninguém avança.

```java
// Java: Adquirir travas em ordens opostas = IMPASSE GARANTIDO
// Tarefa A
synchronized (X) { 
    pausa();
    synchronized (Y) { /* ... */ } 
}     

// Tarefa B
synchronized (Y) { 
    pausa();
    synchronized (X) { /* ... */ } 
}     

```

**As soluções clássicas:**

1. **Ordem global:** Quebre a espera circular obrigando o sistema inteiro a adquirir as travas sempre na mesma ordem (ex: todo mundo pega o X primeiro, depois o Y).
2. **Timeout:** Usar mecanismos como `tryLock(tempo)`, onde a tarefa tenta pegar a trava por 5 segundos. Se não conseguir, ela desiste, solta o que tem, e tenta de novo depois.
3. **Passagem de mensagens:** Evitar compartilhar memória (e portanto, evitar travas) trocando apenas mensagens.

**Pergunta para a turma:** Por que o impasse só aparece "às vezes" nos servidores em produção e quase nunca nos testes do desenvolvedor?

---

## 3. Semáforos, Monitores e Mensagens

Para lidar com travas e cooperação, a computação criou três níveis de abstração evolutivos (quanto mais alto, menos erros você comete e menos controle manual você tem):

| Mecanismo | Origem | Hoje em dia usado em |
| --- | --- | --- |
| **Semáforos** | Dijkstra, 1965 | `Semaphore` (Java), Bibliotecas POSIX |
| **Monitores** | Brinch Hansen (1973), Hoare (1974) | `synchronized` (Java), `lock` (C#), Ada |
| **Mensagens** | Hoare (CSP, 1978) | Canais (Go, Rust), Ada, Erlang |

1. **Semáforo:** É literalmente um contador numérico acoplado a uma fila. O método `wait` diminui o contador (ou bloqueia a tarefa se for 0); o `release` aumenta e acorda quem está esperando. **Fragilidade:** Ele é muito solto. Esquecer de escrever um `release` em alguma linha de erro causa um impasse eterno.
2. **Monitor:** Pense nele como um Tipo Abstrato de Dados (TAD). Ele junta os dados e as operações num lugar só. A exclusão mútua (competição) é automática (só um entra por vez), mas a cooperação ainda é manual (usando `wait()` e `notifyAll()`).
3. **Mensagens:** Abandona-se completamente a memória compartilhada. Em vez de travar variáveis, as tarefas enviam mensagens prontas através de canais estruturados (*rendezvous*).

```java
// 1. Java (Monitor): Exclusão mútua automática no bloco
synchronized void depositar(int v) throws InterruptedException {
    while (ocupados == 3) wait(); // Cooperação manual: espera esvaziar
    // ... deposita ...
    notifyAll();                  // Acorda quem estava esperando
}

// Em Java moderno, usamos AtomicInteger para evitar travas pesadas. 
// Ele usa instruções atômicas direto no hardware.

```

```go
// 2. Go (Mensagens via canais / padrão CSP)
ch := make(chan int)          // Canal sem buffer (comunicação síncrona)
go produtor(5, ch)            // Lança uma nova goroutine

// O for aguarda automaticamente as mensagens chegarem no canal
for v := range ch { soma += v }

```

```rust
// 3. Rust (Segurança forçada pelo compilador)
// O próprio DADO mora DENTRO da trava (Mutex). 
// É impossível acessar o '0' sem abrir o cadeado antes.
let contador = Arc::new(Mutex::new(0));

// No Rust, uma condição de corrida de dados nem sequer compila! 
// O compilador recusa o código inseguro (embora impasses ainda sejam possíveis).

```

**Pergunta para a turma:** Qual mecanismo você escolheria para construir um sistema distribuído rodando em 10 máquinas diferentes conectadas pela rede?

---

## 4. Laço de Eventos e Async (Concorrência sem Paralelismo)

O **JavaScript** popularizou um modelo arquitetural totalmente diferente: ele tem **apenas uma thread** (*single-thread*) aliada a um **laço de eventos** (*Event Loop*).

Como ele faz várias coisas ao mesmo tempo sendo single-thread?
**Concorrência ≠ Paralelismo.** O JS usa o sistema operacional para realizar as operações lentas (I/O, disco, rede) "por fora" do código principal. Quando essas operações externas terminam, seus *callbacks* entram numa fila para serem executados pelo laço.

A fila obedece a uma ordem rigorosa: o código síncrono roda primeiro. Depois, todas as **Microtarefas** (Promises) rodam. E só depois as **Tarefas** regulares (como um `setTimeout`). Mesmo que o `setTimeout` esteja configurado com "zero segundos", ele perde a vez para as microtarefas.

O recurso `async/await` apenas intercala as esperas, devolvendo o controle para o laço enquanto aguarda, mas continua rodando no mesmo núcleo.

```javascript
console.log("1: início");

// Entra na fila de Tarefas (roda por último, mesmo com 0ms de atraso)
setTimeout(() => console.log("4: setTimeout"), 0);

// Entra na fila de Microtarefas (roda assim que o fluxo síncrono acabar)
Promise.resolve().then(() => console.log("3: promise"));

console.log("2: fim do código síncrono");

// A ordem impressa será garantidamente: 1, 2, 3, 4

```

**Como agilizar esperas de rede:**

```javascript
// Ruim: Espera 300ms pelo primeiro, DEPOIS espera 300ms pelo segundo (~600ms total)
await espera(300); 
await espera(300);        

// Excelente: Dispara os dois simultaneamente. (~300ms total)
await Promise.all([espera(300), espera(300)]); 

```

**O perigo deste modelo:** Nunca bloqueie o laço! Se você escrever um `while(true)` pesado matematicamente, a interface inteira trava e a rede para de responder, pois não há outra *thread* para assumir o trabalho. O risco residual do JS não é a condição de corrida de variáveis (pois só há uma thread), mas sim a **lentidão global por esgotamento do laço**.

Cada modelo de concorrência evita o caos de um jeito, mas traz um risco residual diferente.

**Pergunta para a turma:** Qual é o risco residual de cada modelo de concorrência estudado (Travas, Canais, e Laço de Eventos)?

### Paradigma Funcional 

Chegamos ao nosso último tema, que coroa a evolução das linguagens de programação: o **Paradigma Funcional**.

Para entender o funcional, precisamos desapegar da arquitetura de von Neumann (onde a memória é um quadro-negro que apagamos e reescrevemos o tempo todo). Aqui, o código se aproxima da matemática pura.

## 1. Imperativo × Funcional

* **Imperativo:** É centrado no estado (variáveis que mudam com atribuições) e na repetição por laços (`for`, `while`). O resultado final depende fortemente da ordem em que os comandos são executados.
* **Funcional:** É centrado em funções matemáticas e valores imutáveis. Em vez de laços, usa-se recursão e formas funcionais de alta ordem (funções que recebem outras funções).

O pilar desse paradigma é a **função pura**: ela garante que para o mesmo argumento de entrada, o resultado será *sempre* o mesmo, sem nenhum efeito colateral (sem alterar o banco de dados, sem mudar variáveis globais, sem imprimir na tela). Isso gera o que chamamos de **transparência referencial**: a expressão pode ser substituída diretamente pelo seu valor, o que permite fazer cache automático de resultados (*memoização*).

Se você empurrar os "efeitos colaterais" sujos (como E/S e banco de dados) para as bordas do seu sistema e mantiver o núcleo puro, você ganha **testes extremamente simples** e **paralelização 100% segura** (pois sem estado compartilhado e mutável, a condição de corrida deixa de existir — uma ligação direta com nossa aula de Concorrência!).

```python
# Impura: O retorno depende de um estado externo mutável
contador = 0

def proximo_id(prefixo):
    global contador
    contador += 1                  # EFEITO COLATERAL (mudou algo fora do retorno)
    return f"{prefixo}-{contador}"

# Pura: Transparência referencial total
def cubo(x):
    return x * x * x

# Pela matemática, isso é sempre verdade (sem ordem de execução ou efeitos ocultos)
# cubo(2) + cubo(2) == 2 * cubo(2) == 16

```

**Pergunta para a turma:** Pensando em ambientes isolados, o que torna a função `proximo_id` tão difícil de testar e paralelizar?

---

## 2. Lambda e Formas Funcionais

Na década de 1930, Alonzo Church criou o Cálculo Lambda. Hoje, toda linguagem moderna tem uma **expressão lambda** (uma função anônima, sem nome).

As formas funcionais baseiam-se em aplicar essas lambdas a coleções de dados, formando *pipelines* de operações:

* **Composição:** `h = f ∘ g` (ou seja, `h(x) = f(g(x))`).
* **Map (Aplicar-a-todos):** Transforma cada elemento de uma lista.
* **Filter:** Seleciona apenas os elementos que satisfazem um predicado (teste lógico).
* **Reduce / Fold / Sum (Redução):** Combina todos os elementos em um único valor.

> **Receita para reescrever qualquer laço imperativo:**
> 1. O laço usa um `if` para selecionar elementos? → Use **`filter`**.
> 2. O laço transforma o dado (ex: calcula o quadrado)? → Use **`map`**.
> 3. O laço usa um acumulador para juntar o resultado? → Use **`reduce`**.
> 
> 

```javascript
// JavaScript: O laço clássico e imperativo
let soma = 0;
for (let i = 1; i <= 6; i++) {
  if (i % 2 === 0) {  // 1) Seleciona (pares)
      soma += i * i;  // 2) Transforma (quadrado) e 3) Acumula
  }
}

// JavaScript: O mesmo código em funcional (O laço vira um pipeline)
// Sem laço, sem estado mutável, sem acumulador visível. O resultado também é 56.
[1, 2, 3, 4, 5, 6]
  .filter((n) => n % 2 === 0)
  .map((n) => n * n)
  .reduce((acc, n) => acc + n, 0); 

```

```python
# Python: A matemática da composição de funções
def compor(f, g): 
    return lambda x: f(g(x))

# h(x) vai primeiro multiplicar x por 3, e depois somar 2.
h = compor(lambda x: x + 2, lambda x: 3 * x)
print(h(4)) # Imprime 14  ((4 * 3) + 2)

```

**Pergunta para a turma:** Como você reescreveria em formato funcional um laço que percorre uma lista de nomes, filtra apenas os que começam com 'A' e depois conta quantos são?

---

## 3. Scheme e Haskell em Resumo

Você não precisa decorar a sintaxe dessas linguagens acadêmicas, mas precisa entender as ideias brilhantes que elas introduziram.

### Scheme (Família LISP)

Usa notação prefixa cheia de parênteses: `(+ 3 7 10 2)`. Tudo é uma expressão, até o `if` devolve valor. A essência do Scheme é o processamento de listas através do padrão de recursão: `car` (pega o 1º elemento), `cdr` (pega o resto da lista), e `cons` (constrói uma nova lista). E para a memória não explodir, ele exige otimização de **recursão em cauda**, mantendo a pilha de execução constante (não cresce).

```scheme
(car '(A B C))      ; Devolve: A
(cdr '(A B C))      ; Devolve: (B C)
(cons 'A '(B C))    ; Devolve: (A B C)

; O Padrão Universal de Recursão em Listas:
; 1) Lista vazia? (null?)
; 2) Achou no 'car'? (eq?)
; 3) Se não, chama a recursão passando o resto (cdr)
(define (membro atm a_list)
  (cond ((null? a_list) #f)
        ((eq? atm (car a_list)) #t)
        (else (membro atm (cdr a_list)))))

```

### Haskell

Fortemente tipado, 100% puro. Em vez de muitos `if/else`, usa **casamento de padrões** (Pattern Matching).
Suas maiores inovações:

1. **Currying:** Toda função com vários parâmetros é, por baixo dos panos, uma cadeia de funções que recebe apenas um. (Em JS, currying manual seria: `const add = (a) => (b) => a + b;`).
2. **Avaliação Preguiçosa (*Lazy Evaluation*):** O Haskell só calcula um valor no milissegundo exato em que ele for ser usado na tela. Isso permite criar **listas infinitas** na memória sem travar o PC! (Hoje presente no Java Streams, Iteradores do Rust e Geradores do Python).

```haskell
-- Haskell: Pattern matching e recursão sem 'if'
fact 0 = 1                        
fact n = n * fact (n - 1)

-- Haskell: Listas Preguiçosas
[1, 3 .. 11]                      -- Pega os ímpares até 11: [1,3,5,7,9,11]
take 5 [2, 4 ..]                  -- Pede os 5 primeiros de uma lista infinita: [2,4,6,8,10]

```

**Pergunta para a turma:** O que devolve `(cdr '(A B C))` no Scheme? E o que devolve `(car (cdr '(A B C)))`?

---

## 4. Funcional × Imperativo (Você já usa Funcional!)

Quando colocamos os dois paradigmas lado a lado, vemos os custos e os ganhos:

* **Tamanho e Legibilidade:** O funcional esconde variáveis e laços, resultando em códigos entre 10% e 25% do tamanho do imperativo original.
* **Velocidade e Memória:** Historicamente, o funcional puro é mais lento (Wadler cravou ~2x mais lento em 1998) porque cria cópias imutáveis na memória em vez de apenas alterar os bytes. E a avaliação preguiçosa consome mais metadados na memória RAM.
* **Concorrência:** O funcional vence de lavada, sendo paralelizado automaticamente sem medo de *deadlocks*.

O fato é que **a indústria de software decidiu juntar o melhor dos dois mundos**. Você já usa funcional todos os dias, apenas o mistura com a casca imperativa:

* **Java:** Injetou as lambdas e as *Streams* preguiçosas (`map`, `filter`, `reduce`).
* **JavaScript:** Gira em torno dos métodos de *array* e de recursos como `Object.freeze()` e do operador *spread* (`...`) para criar valores novos em vez de mutar os velhos (base do React!).
* **Python e Rust:** Absorveram compreensões de listas, iteradores imutáveis nativos e funções como `map`/`fold`.

```c
// C: Imperativo focado no "COMO" calcular
int sum_cubes(int n) {   
  int sum = 0;
  for (int i = 1; i <= n; i++)
    sum += i * i * i;
  return sum;
}

```

```haskell
-- Haskell: Funcional focado no "O QUE" calcular
sumCubes n = sum (map (^3) [1 .. n])

```

Código de livro didático é lindo, mas sistemas em produção precisam considerar performance e memória. O aprendizado da curva funcional recompensa com sistemas incrivelmente robustos e testáveis, mas as linguagens evoluem adotando o estilo híbrido.

**Pergunta para a turma:** Conhecendo os custos de velocidade e memória das cópias imutáveis, em quais casos (tipos de software ou arquiteturas) você ainda preferiria usar o estilo estritamente imperativo em C/C++?