---
layout: post
lang: "pt"
title: Cobertura de testes como garantia de código testado
---

Cobertura de testes é um valor que representa a porcentagem de linhas de código de produção que é executada durante os testes. Por exemplo, se uma classe possui 50 linhas de código e os testes executam 25 destas, a cobertura de testes dessa classe é 50%. Mas, como muita informação representada em números, este valor é comumente interpretado de forma incorreta.

Muitas pessoas consideram a porcentagem de cobertura como um indicador de que o código está bem testado. O raciocínio é justo: se executa todas as linhas de código e passa todos os testes, é correto afirmar que o código em questão está 100% testado.

O problema dessa abordagem é seu simplismo. É ingênuo chegar a uma conclusão baseando-se somente em "*quantas linhas de código são executadas* vs *quantas linhas existe no total*". Vamos a um exemplo:

Digamos que se tenha uma classe ``Somador`` que possui um método chamado ``somaUm``. Este método recebe um valor inteiro e retorna o sucessor deste valor. Seguindo as práticas de TDD, escrevemos primeiro o teste para essa classe:

{% highlight java %}
public class SomadorTest {
	@Test
	public void dado_um_numero_inteiro_deve_retornar_seu_sucessor(){
		Somador somador = new Somador();
		assertEquals(3, somador.somaUm(2));
	}
}
{% endhighlight %}

Tendo um teste que representa o comportamento desejado, temos certeza de construir apenas o necessário para que esse comportamento seja atingido:

{% highlight java %}
public class Somador {
	public int somaUm(int valor){
		return valor + 1;
	}
}
{% endhighlight %}


Se executarmos algum medidor de cobertura de testes, obteremos 100% como resultado. Seguindo o raciocínio anterior, isto indica que o código está testado e livre de problemas. Afinal, ``2 + 1 = 3`` certo?

Aos olhos de um desenvolvedor mais experiente, um problema fica claro: não por ser óbvio, mas por ser um problema conhecido. Este problema aparece sob a forma do número ``2147483647``. Este número representa o maior valor possível de se representar utilizando uma variável numérica de 32 bits que também comporta números negativos. Adicionando este caso de teste:

{% highlight java %}
public class SomadorTest {
	@Test
	public void dado_um_numero_inteiro_deve_retornar_seu_sucessor(){
		Somador somador = new Somador();
		assertEquals(3, somador.somaUm(2));
		assertEquals(2147483648, somador.somaUm(2147483647));
	}
}
{% endhighlight %}

``AssertionError: 
Expected :2147483648
Actual   :-2147483648
``


Ao adicionar ``1`` à ``2147483647`` esperava-se como resultado o número ``2147483648``, mas obteve-se ``-2147483648``. A explicação para isso foge do escopo deste artigo e pode ser encontrada [aqui][wikipedia]. O ponto principal é que **a cobertura de testes estando em 100% não garante que código está testado de forma correta**. Da mesma forma, uma cobertura baixa não indica, necessariamente, que o código esteja mal testado.

Cobertura de testes é melhor utilizada junto de outras métricas. Se uma aplicação apresenta um número de bugs muito alto **e a cobertura de testes está baixa**, é válido imaginar que coisas importantes não estão sendo testadas. A cobertura por si só raramente é um indicador confiável de qualidade da codebase.

[wikipedia]:http://pt.wikipedia.org/wiki/Complemento_para_dois

