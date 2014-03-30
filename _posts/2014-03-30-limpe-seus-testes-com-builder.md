---
layout: post
title: Limpe seus Unit Tests com Builder
---

Utilizar *unit tests*, como sugerido pelo ciclo TDD, traz grandes benefícios para o ciclo de desenvolvimento. Os ganhos mais comuns aparecem como uma solução mais simples e flexível, bem como uma maior confiança de que o que foi produzido atende as necessidades do usuário. Por ser tão importante, código de teste deve ser tratado com o mesmo cuidado que código de produção e uma das maneiras de fazer isso é utilizando [Builder Pattern][builder].

O padrão Builder é um dos padrões de criação mais fáceis de serem implementados e, no caso de unit tests, traz um ganho muito grande em legibilidade e simplicidade. O exemplo utilizado aqui envolve um objeto imutável e é o caso em que mais aplico o padrão Builder.

Para termos um exemplo simples, vamos considerar uma classe `Produto` e outra chamada `CaluladorPreco`. Essas classes são descritas da seguinte forma:

```java
package me.pedrovereza.examples;
public class Produto {
    private String nome;
    private double precoNormal;
    private int quantidadeEmEstoque;
    private String modelo;
    private String cor;
    
    public Produto(String nome, double precoNormal, int quantidadeEmEstoque, 
    String modelo, String cor) {
        this.nome = nome;
        this.precoNormal = precoNormal;
        this.quantidadeEmEstoque = quantidadeEmEstoque;
        this.modelo = modelo;
        this.cor = cor;
    }
    public String getNome() { return nome; }
    public double getPrecoNormal() { return precoNormal; }
    public int getQuantidadeEmEstoque() { return quantidadeEmEstoque; }
    public String getModelo() { return modelo; }
    public String getCor() { return cor; }
}
```

```java
package me.pedrovereza.examples;
public class CalculadorPreco {
    public double calculaPrecoPara(Produto produto) {
        return 0;
    }
}

```

A classe `Produto` representa um objeto imutável, isto é, depois de inicializado, não deve ser possível fazer alteração nos valores. Neste caso, a classe não possui nenhum método `set` e todos os campos são preenchidos através do construtor.

Para melhorar, vamos definir que o preço do produto varia conforme a quantidade em estoque: quando a quantidade em estoque é maior que 50 vale o preço normal, quando a quantidade está entre 10 e 49 o preço é 70% do valor normal e quando o estoque está abaixo de 10 unidades o preço é 50% do valor normal. Seguindo o ciclo de TDD, escrevemos testes para a classe `CalculadorPreco` para descrever o comportamento do método `calculaPrecoPara()`

```java
package me.pedrovereza.examples;
import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class CalculadorPrecoTest {

    @Test
    public void
    calula_preco_normal_quando_estoque_maior_que_50() {
        Produto produto = new Produto(null, 10.0, 51, null, null);
        CalculadorPreco calculadorPreco = new CalculadorPreco();
        assertEquals(10.0, calculadorPreco.calculaPrecoPara(produto), 0.0);
    }

    @Test
    public void
    calula_preco_como_setenta_porcento_quando_estoque_menor_que_50() {
        Produto produto = new Produto(null, 10.0, 49, null, null);
        CalculadorPreco calculadorPreco = new CalculadorPreco();
        assertEquals(7.0, calculadorPreco.calculaPrecoPara(produto), 0.0);
    }

    @Test
    public void
    calula_preco_como_cinquenta_porcento_quando_estoque_menor_que_10() {
        Produto produto = new Produto(null, 10.0, 8, null, null);
        CalculadorPreco calculadorPreco = new CalculadorPreco();
        assertEquals(5.0, calculadorPreco.calculaPrecoPara(produto), 0.0);
    }
}

```


Feito isso, e garantindo que os testes estão falhando, seguimos para a implementação do método.

```java
package me.pedrovereza.examples;

public class CalculadorPreco {
    public double calculaPrecoPara(Produto produto) {
        if (produto.getQuantidadeEmEstoque() >= 50) {
            return produto.getPrecoNormal();
        }
        if (produto.getQuantidadeEmEstoque() >= 10 &&
                produto.getQuantidadeEmEstoque() < 50) {
            return produto.getPrecoNormal() * 0.7;
        }
        return produto.getPrecoNormal() * 0.5;
    }
}

```

E *voilà* , temos todos os testes verdes. Algumas refatorações depois e o método `calculoPrecoPara()` se parece com isso:

```java
package me.pedrovereza.examples;

public class CalculadorPreco {
    private static final double CINQUENTA_PORCENTO = 0.5;
    private static final double SETENTA_PORCENTO = 0.7;

    public double calculaPrecoPara(Produto produto) {
        if (estoqueNaFaixaDePrecoNormal(produto))
            return produto.getPrecoNormal();
            
        if (estoqueNaFaixaDeSetentaPorcento(produto))
            return produto.getPrecoNormal() * SETENTA_PORCENTO;
            
        return produto.getPrecoNormal() * CINQUENTA_PORCENTO;
    }
    private boolean estoqueNaFaixaDePrecoNormal(Produto produto) {
        return produto.getQuantidadeEmEstoque() >= 50;
    }
    private boolean estoqueNaFaixaDeSetentaPorcento(Produto produto) {
        return produto.getQuantidadeEmEstoque() >= 10 &&
                produto.getQuantidadeEmEstoque() < 50;
    }
}
```

Até aí tudo bem, mas temos um pequeno problema nos métodos de testes. A linha ``Produto produto = new Produto(null, 10.0, 51, null, null);`` é pouco expressiva. Alguém que nunca leu este código seria obrigado a procurar o que significa cada parâmetro até entender o que é relevante para o teste e o que não é. Para facilitar a vida de todos, utilizamos o padrão Builder.

O que o Builder fará para nós é esconder a criação do objeto `Produto` atrás de uma interface (não deve ser confundida com Interface Java) de criação mais expressiva. 
Vale lembrar que, por ser uma classe que nos ajudará apenas nos testes, ela será criada e utilizada no pacote de testes. O uso que queremos é o seguinte:

```java
    @Test
    public void
    calula_preco_normal_quando_estoque_maior_que_50() {
        Produto produto = ProdutoBuilder.criaProduto()
                			.comPrecoNormal(10.0)
                			.comQuantidadeEmEstoque(51).build();
        CalculadorPreco calculadorPreco = new CalculadorPreco();
        assertEquals(10.0, calculadorPreco.calculaPrecoPara(produto), 0.0);
    }

```

A classe `ProdutoBuilder` basicamente terá os mesmo campos da classe `Produto` mas terá `setters` para podermos preencher apenas valores que são relevantes.  Maravilhoso, mas me mostre código:

```java
package me.pedrovereza.examples;

public class ProdutoBuilder {
    private double precoNormal;
    private int quantidadeEmEstoque;
    private String nome;
    private String modelo;
    private String cor;

    private ProdutoBuilder(){ }

    public static ProdutoBuilder criaProduto() {
        return new ProdutoBuilder();
    }
    public ProdutoBuilder comNome(String nome) {
        this.nome = nome;
        return this;
    }
    public ProdutoBuilder comModelo(String modelo) {
        this.modelo = modelo;
        return this;
    }
    public ProdutoBuilder comCor(String cor) {
        this.cor = cor;
        return this;
    }
    public ProdutoBuilder comPrecoNormal(double precoNormal) {
        this.precoNormal = precoNormal;
        return this;
    }
    public ProdutoBuilder comQuantidadeEmEstoque(int quantidadeEmEstoque) {
        this.quantidadeEmEstoque = quantidadeEmEstoque;
        return this;
    }
    public Produto build() {
        return new Produto(nome, precoNormal,
                quantidadeEmEstoque, modelo, cor);
    }
}
```

O padrão basicamente consiste em reter as informações recebida através dos métodos `set` e usar essas informações no método `build()` para criar um objeto `Produto`. Note que todo `set` retorna o próprio `ProdutoBuilder`, isso possibilita que se tenha um jeito mais claro e simples de informar o que queremos no objeto.

```java
package me.pedrovereza.examples;
import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class CalculadorPrecoTest {
    private CalculadorPreco calculadorPreco = new CalculadorPreco();
    
    @Test
    public void
    calula_preco_normal_quando_estoque_maior_que_50() {
        Produto produto = ProdutoBuilder.criaProduto()
                			.comPrecoNormal(10.0)
                			.comQuantidadeEmEstoque(51).build();
        assertEquals(10.0, calculadorPreco.calculaPrecoPara(produto), 0.0);
    }

    @Test
    public void
    calula_preco_como_setenta_porcento_quando_estoque_menor_que_50() {
        Produto produto = ProdutoBuilder.criaProduto()
                			.comPrecoNormal(10.0)
                			.comQuantidadeEmEstoque(49).build();
        assertEquals(7.0, calculadorPreco.calculaPrecoPara(produto), 0.0);
    }

    @Test
    public void
    calula_preco_como_cinquenta_porcento_quando_estoque_menor_que_10() {
        Produto produto = ProdutoBuilder.criaProduto()
                			.comPrecoNormal(10.0)
                			.comQuantidadeEmEstoque(8).build();
        assertEquals(5.0, calculadorPreco.calculaPrecoPara(produto), 0.0);
    }
}
```

Executando os testes novamente, tudo continua funcionando. A diferença é que agora se tem uma interface simples e expressiva para criar novos objetos `Produto`. Cada caso de teste cria um ``Produto`` contendo apenas as informações que são necessárias para o contexto. 

Nesse exemplo pode parecer overkill, mas imagine outras classes que precisam receber um objeto `Produto` nos testes, todas poderão utilizar o mesmo Builder.

[builder]: http://en.wikipedia.org/wiki/Builder_pattern