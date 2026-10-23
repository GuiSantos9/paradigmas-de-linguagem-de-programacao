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