```Rust
use std::io;

fn main () {
    println!("Escolha um número");
    
    tabuada();
}

fn tabuada(){
    let mut input = String::new();
    let mut count = 0;
    
    io::stdin()
        .read_line(&mut input)
        .expect("Erro ao ler");
        
    let numero: i32 = input.trim().parse().expect("Por favor, digite um número válido!");
    
    println!("Você escreveu: {}", numero);
    println!("--- Tabuada do {} ---", numero);
    
    for i in 1..11 {
        println!("{} x {} = {}", numero, i, numero * i);
}

```
