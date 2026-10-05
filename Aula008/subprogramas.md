# Subprogramas:


Os subprogramas (métodos, funções, etc.) permitem a manutenabilidade dos seus programas, e o reúso de partes do código, além de entender o que acontece com os dados quando uma função é chamada.

Para o Engenheiro de software isso significa:
- dicidir se uma função pode ou não alterar os dados que recebe
- evitar efeitos colaterais e apelidos inesperados 
- entender erros como ``StackOverFlowError`` e ``RecursionError``

--- 
### Fundamentos dos subprogramas:

1. Uma única entrada: a execução começa sempre no início do subprograma
2. Quem chama fica suspenso: só o subprograma executa por vez 
3. Controle sempre volta: ao terminar, retorna a quem a chamou 


```python
# cabeçalho
def calcular_salario(renda,aliquota,dependentes=1):
    return renda * (1 - aliquota) + 100 * dependentes

# chamada: parâmetros REAIS
pago = calcular_salario(2000.0, 0.15)
```

--- 
### Parâmetros: posição, palavra-chave, padrão e variádicos

---
### Procedimentos
- Procedimento é uma nova sentença que produz resultados alterando variáveis ou parâmetros(void, None)
- Função modela uma função matemática, o ideal é que o valor devolvido seja o único efeito. 

---
### Ambientes de referenciamento local 

```c
int contador_automatico(void){
    int n = 0; //dinâmica da pilha
    return n++;
}

int contador_estatico(void){
    static int n = 0; //estático: lembra do valor 
    return n++;
}

//automático = 1 1 1
//estático = 1 2 3 
```
---

### C e C++, ponteiros e referências
```c
void troca_valor(int a, int b){
    int t = a;
    a = b;
    b = t;
}
void troca_ponteiro(int *a, int *b) {
    int t = *a; *a = *b; *b = t;
}
troca_valor(x, y);       // x=1 y=2
troca_ponteiro(&x, &y);  // x=2 y=1  (o & aparece)

```

```c++
void troca(int &a, int &b) {     // referência
    int t = a; a = b; b = t;
}
int tamanho(const std::string &s) {
    return s.size();   // sem cópia, sem alterar
}
troca(x, y);   // x=2 y=1: a chamada não
               // mostra que x e y mudam
```

---
### Java tudo é passado por valor 

```java
static void troca(int a, int b) { 
    int t = a; 
    a = b; 
    b = t; 
}
static void altera(Ponto p) { 
    p.x = 99;
}
static void reatribui(Ponto p) {
    p = new Ponto(-1); 
}

troca(x, y);       // x=1 y=2: troca só as cópias
altera(p);         // p.x = 99: o OBJETO é compartilhado
reatribui(p);      // p.x continua 99: a referência
                   // do chamador não muda
```

--- 
### python, c#, go e rust

```python
def altera_lista(lista):
    lista.append(4)      # o chamador vê
def reatribui_lista(lista):
    lista = [0, 0, 0]    # só o nome local muda
def incrementa(n):
    n = n + 1            # int é imutável

nums = [1, 2, 3]; x = 10
altera_lista(nums)     # [1, 2, 3, 4]
reatribui_lista(nums)  # [1, 2, 3, 4]
incrementa(x)  # 10
```

---
### apelidos criados pela passagem por referência
```c
// a e b são o mesmo endereço
void soma_duas_vezes(int *a, int *b) {
    *a += *b;
    *a += *b;
}

int x = 1, y = 1;
soma_duas_vezes(&x, &y);          // x = 3

int total = 1;
soma_duas_vezes(&total, &total);  // total = 4 !
```

```rust
// o compilador proíbe apelidos
fn soma_duas_vezes(a: &mut i32, b: &i32) {
    *a += *b;
    *a += *b;
}

let mut total = 1;
soma_duas_vezes(&mut total, &total);
// ERRO E0502: não pode haver &mut e &
// do mesmo valor ao mesmo tempo
```
---
### Chamada indireta:

```c
int decrescente(const void *a, const void *b) {
    return *(const int *)b - *(const int *)a;
}
int dobro(int x)    { return 2 * x; }
int quadrado(int x) { return x * x; }

qsort(v, 4, sizeof(int), decrescente);  // 9 5 2 1

int (*ops[])(int) = {dobro, quadrado};
ops[1](7);  

```
```java
static int aplicar(IntUnaryOperator f, int x) {
    return f.applyAsInt(x);
}
aplicar(x -> x * 2, 7);      // 14 (lambda)
aplicar(Math::abs, -7);      // 7  (ref. a método)

nomes.sort(Comparator.comparing(
        String::toLowerCase));
// [Ana, bia, caio]
```

--- 
### Fechamentos (Closures)
