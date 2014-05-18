---
layout: post
lang: "pt"
title:  "Os problemas de contar velocidade em Dev Done"
---


Tive algumas experiências em projetos em ambientes curiosos e acho que seria interessante compartilhar algumas coisas que aprendi.
Uma dessas coisas é o lado negativo de contar velocidade baseada em Dev done.

>Dev done representa o exato momento depois de um commit. A tarefa está feita pelo desenvolvedor e é movida para validação.


Na minha experiência, tarefas consideradas terminadas por desenvolvedores não chegariam na etapa de testes até a próxima iteração. Nos piores casos, estes testes seriam feitos pelo QA apenas três semanas mais tarde.

Essa abordagem implicada em alguns problemas:

**Mata o loop de feedback:** Isso é um dos principios básicos do desenvolvimento Ágil (e também do ciclo *build, measure, learn* do Lean). Feedback rápido significa que você pode encontrar seus erros de forma rápida e consertá-los rapidamente. Demorar três semanas para descobrir que a funcionalidade criada não saiu como desejada é tempo demais.

**É ruim para todos:** O desenvolvedor deve parar o que está fazendo, voltar para o contexto de semanas atrás para se lembrar tudo que estava envolvido naquela tarefa; o BA contava que a tarefa já estava terminada e fez novas promessas para o cliente; o time agora tem uma nova tarefa para a iteração que causará impacto no planejamento e que, provavelmente, não será considerado. 

**O retrabalho não é medido:** A funcionalidade que você considerou pronta voltou para te assombrar e trouxe notícias: todos aqueles pontos que contaram para a iteração na verdade não entregaram valor algum. Corrigir essa antiga estória é uma nova tarefa? Esse trabalho deve contar na pontuação do time? Algo que foi prometido deve ser deixado de lado para que a correção seja feita?

**Esconde os reais problemas:** Se sua QA não consegue dar conta da quantidade de código novo sendo jogado pelos desenvolvedores e os testes passam a ser o gargalo do seu fluxo, o que você faz? Você pode reconhecer que há um problema e que, talvez, você precise de mais pessoas testando (outro QA ou uma desenvolvedora) ou você pode simplesmente remover a coluna de validação e considerar cada commit como uma tarefa feita. Viver perigosamente, certo?

Até hoje, não vi absolutamente nenhum benefício de contar pontos em Dev done. Essa abordagem é geralmente tomada para satisfazer [Velocityraptors][velocityraptor], mantendo a velocidade média alta enquanto esconde problemas no time/projeto.

Uma das melhores coisas do desenvolvimento Ágil é trazer os problemas para a superfície para que possam ser analisados e resolvidos. Não é sobre quantos pontos você terminou em um período de tempo, muito menos uma ferramenta para *micromanagement*. Não esconda seus problemas na coluna de Done.

[velocityraptor]:http://thoughtworks.github.io/p2/issue03/velocityraptors/
