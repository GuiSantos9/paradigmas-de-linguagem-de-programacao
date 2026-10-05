1. python

```python
# Questão 1: qual é a saída? Por quê?
# Online: https://onecompiler.com/python  (cole o código inteiro)
def adicionar(item, lista=[]):
    lista.append(item)
    return lista

print(adicionar(1))
print(adicionar(2))
```
resposta: [1] [1,2], isso acontece porque não temos outra chamada por atribuição, o chamador vê e guarda os valores da primeira chamada da função adicionar. 

```java
// Questão 2: qual é a saída? Por quê?
// Online: https://onecompiler.com/java  (cole o código inteiro; a classe se chama Main)
public class Main {
    static void zera(int[] v, int n) {
        v[0] = 0;
        n = 0;
    }

    public static void main(String[] args) {
        int[] v = {5, 5};
        int n = 5;
        zera(v, n);
        System.out.println(v[0] + " " + n);
    }
}
```
Respostas: 0 e 5 isso acontece por que no Java variáveis passadas com tipos primitivos ``int``, ``float``, etc. São passadas por valor dentro das funções, fazendo com que o metodo zera receba somente uma cópia da variável n que contém o número 5, quando fazemos n = 0 no método estamos alterando somente a variável local, o n que temos dentro do main ainda é 5. Já quando enviamos um array o java manda para o método uma cópia de referência (com o endereço de memória), ou seja, a variavel v da função zera e da função main apontam para o mesmo endereço de memória. 



```python
# Questão 3: qual é a saída? Por quê?
# Online: https://onecompiler.com/python  (cole o código inteiro)
fs = [lambda: i for i in range(3)]
print([f() for f in fs])
```
Resposta: [2,2,2], isso acontence porque quando fazemos ``lambda:i``, a função lambda não guarda o valor que i assume no exato momento do loop, enves disso ele guarda um referência para a variável i. O código ``for i in range(3)`` faz a variável i assumir os valores 0, 1 e finalmente 2 quando o loop para e a lista é criada o valor final da variável i é 2. Quando fazemos ``f() for f in fs`` estamos executando as 3 funções que estavam guardadas. Ou seja, cada uma das funções vai buscar o respectivo valor de i. Como todas as funções de i apontam para a mesma variável e o loop já terminou deixando o loop em i=2, todas as funções retornam 2.

```c
/* Questão 4: qual é a saída? Por quê?
 * Online: https://onecompiler.com/c  (cole o código inteiro)
 */
#include <stdio.h>

int contador(void) {
    static int n = 0;
    return ++n;
}

int main(void) {
    contador();
    contador();
    printf("%d\n", contador());
    return 0;
}
```
Resposta: 3, isso acontece por conta da dinâmica da pilha que permite a recursão da função, onde a função em sí não lembra dos valores mas lembra da quantidade de vezes que foi chamada

```rust
// Questão 5: qual é a saída? Por quê?
// Online: https://onecompiler.com/rust  (cole o código inteiro)
fn dobra(v: Vec<i32>) -> Vec<i32> {
    v.iter().map(|x| x * 2).collect()
}

fn main() {
    let v = vec![1, 2, 3];
    let d = dobra(v);
    println!("{:?} {:?}", v, d);
}
```

Resposta: error[E0382]: borrow of moved value: v
Isso acontece por causa da principal e mais famosa característica do Rust: o seu sistema de Ownership (Posse).

A regra do "Move" (Movimentação): Em Rust, tipos de dados que alocam memória na heap, como o Vec<i32>, não implementam a trait Copy por padrão. Isso significa que, quando você os passa como argumento para uma função, o Rust não faz uma cópia silenciosa deles.

Transferência de Posse: Quando você chama let d = dobra(v);, a variável v é passada por valor. Nesse momento, a "posse" (ownership) daquele vetor é transferida da função main para dentro da função dobra. Dizemos que o valor foi movido (moved).

O uso após a movimentação: Na linha seguinte, no println!("{:?} {:?}", v, d);, você tenta imprimir o v e o d. Porém, como o v já foi doado para a função dobra, ele não é mais válido dentro do main. O compilador percebe que você está tentando usar algo que não pertence mais a essa parte do código e trava a compilação por segurança.

```python
# Questão 6: qual é a saída? Por quê?
# Online: https://onecompiler.com/python  (cole o código inteiro)
total = 0

def adiciona(x):
    total = total + x
    return total

print(adiciona(5))
```
Isso dá um erro porconta da variavel local ``total`` que não está associada a um valor concreto 