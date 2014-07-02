---
layout: post
lang: "pt"
trans: /development-workspace-craftsmanship
title: Artesão de máquinas de desenvolvimento
---

Artesãos são geralmente conhecidos comos pessoas que produzem algo manualmente, sem um processo industrial estabelecido. Muitas vezes, o valor do produto é superior a equivalentes manufaturados justamente pela originalidade e exclusividade da peça adquirida. Em teoria, nenhuma peça é exatamente igual à outra.

Artesãos de máquinas de desenvolvimento trabalham de forma semelhante: não utilizam (ou utilizam poucos) processos automatizados e, **raramente, duas máquinas de desenvolvimento se comportam da mesma maneira**. Os efeitos disso são muitas dores de cabeça e as mesmas perguntas sobrevoando a sala semanalmente: "*Tinha que instalar isso?*", "*Podes abrir tuas configurações para eu ver uma coisa?*", "*Como assim o plugin não funciona? Na minha máquina tá ok*". É um processo doloroso que muitas vezes é ignorado.

Mas verdade seja dita: 90% do que precisa ser configurado é uma receita de bolo. Baixa aqui, instala lá, muda umas variáveis de ambiente, roda um script, reinicia (no caso de Windows) e pronto: agora deve compilar. É senso comum automatizar o build, mas por que não automatizamos o processo de criação de máquinas novas? Ferramentas não faltam.

Entre as diversas opções disponíveis ([Vagrant][vagrant], [Chef][chef] e etc) uma que me chamou atenção, principalmente pela simplicidade, foi [Babushka][babushka]. Imagine um mundo em que tudo que é necessário para ter um ambiente de desenvolvimento está em alguns poucos arquivos ``.rb`` em um repositório qualquer e tudo que você precisa fazer é executar ``babushka maquina-desenvolvimento``. A ferramente possui uma DSL simples e a curva de aprendizado é muito pequena. Ela integra maravilhosamente bem com ``homebrew`` e ``apt`` e, basicamente, o céu é o limite. Mais detalhes sobre Babushka podem ser lidos na [oitava edição da P2 Magazine da **Thought**Works][p2-babushka] (em inglês).

Outra opção é utilizar o bom e velho bash script. É melhor do que fazer tudo manualmente, sem dúvidas, mas a primeira desvantagem óbvia é que o código é bizarro. A segunda é que isso pode se tornar difícil de manter a longo prazo (parte por culpa da primeira). Ainda assim, vale o experimento e certamente poupará tempo futuro.

No fim do dia, pouco importa a ferramenta que será utilizada. O importante é possuir uma forma automatizada de criar ambientes de desenvolvimento que evitem dores de cabeças e que funcionem para todos. Se um membro novo no seu time precisa de oito horas para configurar tudo, talvez você esteja obrigando-o a ser um artesão de máquinas de desenvolvimento. 

[vagrant]:http://www.vagrantup.com/
[chef]:http://www.getchef.com/
[babushka]:https://babushka.me/
[p2-babushka]:http://thoughtworks.github.io/p2/issue08/babushka/
