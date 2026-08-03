```Rust
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
        
    for input in 1..11 {
        count += 1;
        println!("{}",count * input);
    }
        
    println!("Você escreveu: {}", input);
}

```
