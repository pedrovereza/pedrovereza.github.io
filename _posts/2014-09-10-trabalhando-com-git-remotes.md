---
layout: post
lang: "pt"
title: Trabalhando com git - Remotes
---

O [post anterior][git-part-one] deu uma ideia das principais diferenças entre git e outros VCS. Sempre que foi feita referência ao repositório "central" do git, usou-se aspas. Este post explica o motivo.

##Git não possui repositório central
Motivo explicado. Sendo um VCS disitruído, é esperado que não haja a figura de um servidor que contém toda a verdade e clientes que acessam e salvam suas informações nesse servidor. Toda cópia de um repositório git é um repositório git. Isto significa que tanto o original quanto a cópia podem ser usado como se fossem um repositório central.

##Remotes
No contexto do git, `remotes` representam repositórios remotos. É possível adicionar múltiplos repositórios remotos e operar neles da forma que for mais conveniente: pode-se pegar novas alterações de um repositório e enviá-las para outro, por exemplo.

##Um caso de uso
Sou um usuário relativamente ativo no github e, eventualmente, acabo fazendo contribuições para outros projetos. Para exemplificar, usarei como exemplo o repositório da gem chatterbot (uma gem que auxilia a criação de bots para o twitter).O link do repositório é este aqui: `https://github.com/muffinista/chatterbot`. 

Diferente do que foi feito no post anterior, não basta clonar este repositório e sair trabalhando pois não tenho permissão para commitar diretamente nele. Preciso fazer minhas alterações em algum outro lugar e enviar para que o dono do repositório a aceite. O Github fornece a opção de fork que facilita as coisas, que permitirá que as alterações sejam enviadas pro repositório original (é importante notar que isso é uma feature do github e não tem relação com git em si).

Depois do fork, tenho um novo repositório no github com o mesmo nome do original, [ele pode ser visto aqui][chatterbot_copy]. Para poder trabalhar, preciso de uma cópia local deste repositório. Ao invés de usar `git clone`, vamos configurar o remote manualmente para os dois repositórios.

Em um diretório qualquer, crio um novo repositório git utilizando o commando `git init`. Para os curiosos, este comando cria a estrutura de pastas utilizadas pelo git, este diretório é nomeado `.git/`. Informações sobre como o git funciona internamente podem ser lidas no livro [Pro Git] [progit].

O que se tem até agora é um repositório vazio. Para "linkar" este repositório com o que tenho no github preciso adicionar um `remote`. Neste exemplo, o comando para isto é: `git remote add origin https://github.com/pedrovereza/chatterbot.git`, que criará um remote com alias `origin` e endereço `https://github.com/pedrovereza/chatterbot.git`. Este alias pode ser utilizado para baixar o conteúdo do repositório remoto: `git pull origin master`. Tudo que foi feito até agora é o que acontece quando se executa utiliza o `git clone <url>`.

##Pegando alterações do repositório original
Com o que foi feito até agora e utilizando os exemplos do primeiro post, já posso trabalhar em um repositório local. Mas esse caso é um pouco diferente: o repositório principal do projeto não é a minha cópia no github, mas sim a versão do usuário [muffinista][muffinista_user]. E se eu decidir que quero pegar alterações feitas no repositório dele para poder trabalhar em cima? Preciso fazer o fork novamente?

É aí que os remotes se tornam realmente úteis: basta adicionar outro remote apontando para o repositório `muffinista/chatterbot`. O comando é o mesmo utilizado antes, mas com alias e links diferentes `git remote add muffinista https://github.com/muffinista/chatterbot.git`. Feito isso, é possivel atualizar o meu repositório local com as alterações feitas no remote `muffinista` usando `git pull muffinista master` e enviá-las para o meu repositório no github: `git push origin master` ([É possível ver commits do usuário muffinista no meu histórico de commits][commit_history]. Essas alterações foram enviadas exatamente como explicado aqui). 

PS: É importante lembrar que eu não possuo permissões para alterar o remote `muffinista`

```
 $ git push muffinista master
remote: Permission to muffinista/chatterbot.git denied to pedrovereza.
```

[progit]:http://git-scm.com/book
[muffinista_user]: https://github.com/muffinista
[chatterbot_copy]: https://github.com/pedrovereza/chatterbot
[commit_history]: https://github.com/pedrovereza/chatterbot/commits/master
[git-part-one]: /trabalhando-com-git