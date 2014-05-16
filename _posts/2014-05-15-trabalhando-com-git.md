---
layout: post
title: Trabalhando com git
---

Faz um tempo que tenho usado [git][git-link] diariamente, tanto em projetos no trabalho quanto em coisas pessoais (leia: [meu github][github]). Comparado com outros SVC (*Source Version Control*), a curva de aprendizagem do git é muito maior que, por exemplo, [SVN][svn-link] ou [TFS][tfs-link]. A complexidade extra é causada por features que, depois que se aprende a usar, é dificil voltar para os mais simples. Sabendo dessa dificuldade (que passei e que vejo amigos com as mesmas dúvidas), resolvi explicar o (pouco) que sei.

##Commits locais

Trabalhei muito tempo com TFS e, se estivesse implementando algo e precisasse voltar a um estágio de 3 horas atrás (o que claramente já indica um mal dia), a única ferramenta para me salvar disso seria usar `Ctrl+Z` exaustivamente no Visual Studio em todos os arquivos que eu havia alterado (o que só piorava o dia).

Entra o git.

Com git, esse cenário é menos infeliz. Quando preciso voltar a codebase para 3 horas atrás (o que ainda signifca um mal dia), eu posso depender dos commits que tenho **no meu repositório local**. O git é um SVC descentralizado, isso significa que a cópia de um repositório **é um repositório** em si. Para entender as diferenças, um exemplo:

````
$ svn checkout http://umlinkqualquer/svn/trunk/exemplo
$ cd exemplo
$ touch um-arquivo.txt
$ svn add um-arquivo.txt
$ svn commit -m "Adicionando um-arquivo.txt"
````
Esse exemplo começa fazendo uma cópia de um repositório SVN, cria um arquivo `um-arquivo.txt`, adiciona e, por fim, committa esse novo arquivo. Após executar todos os comandos, o repositório em `http://umlinkqualquer/svn/trunk` possui o arquivo `um-arquivo.txt`.

````
$ git clone https://github.com/pedrovereza/exemplo.git
$ cd exemplo
$ touch um-arquivo.txt
$ git add um-arquivo.txt
$ git commit -m "Adicionando um-arquivo.txt"
````

Esse exemplo faz o mesmo que que anterior. A diferença é que o commit **é local**, isto é: o repositório local possui o `um-arquivo.txt`, o repositório em `https://github.com/pedrovereza/exemplo.git` **não possui** `um-arquivo.txt`. Esse comportamento é exatamente o que permite se ter um histórico de mudanças locais que podem ser comparadas, revertidas ou refeitas. É possível fazer vários commits locais e enviá-los de uma vez só usando o comando `$ git push` (que na verdade se traduz, normalmente, para `$git push origin master`, mas vamos devagar).


##Pegando mudanças

Commits locais trazem uma variedade de novas possibilidades, mas nada é de graça nessa vida. É necessário atentar para os problemas que podem trazer.

````
$ svn update
````
É isso que precisamos fazer para atualizar o repositório no SVN. O equivalente no git seria:

````
$ git pull
````
Tudo lindo seria se assim fácil sempre fosse, não é mesmo? O que acontece, normalmente, é que existem conflitos entre a versão que temos e a versão "central". Bom, neste caso:

````
$ svn update
Conflict discovered in 'conflito.txt'.
Select: (p) postpone, (df) diff-full, (e) edit,
        (mc) mine-conflict, (tc) theirs-conflict,
        (s) show all options:
$ e

````
Depois de fazer as alterações necessárias, basta marcar o conflito como resolvido e pronto.

````
Select: (p) postpone, (df) diff-full, (e) edit, (r) resolved,
        (mc) mine-conflict, (tc) theirs-conflict,
        (s) show all options:
$ r
````

E aqui as diferenças do git se tornam mais evidentes. Como temos um repositório local, vamos olhar o log primeiro:

````
$ git log

commit 10f59f3ab2caa387109169495a18135d57740a08
Author: Pedro Vereza <pdrvereza@gmail.com>
Date:   Tue May 13 13:23:16 2014 -0300

    Incluindo exemplo no arquivo conflito.txt

commit 960c34302051e8d1f8a18a5dd77eccbe3b6e3ff1
Author: Pedro Vereza <pdrvereza@gmail.com>
Date:   Tue May 13 09:08:37 2014 -0300

    Adicionando um-arquivo.txt
````
A alteração feita no arquivo `conflito.txt` já está commitada localmente e, bom, existe uma outra alteração no mesmo arquivo no repositório "central". Vamos pegar o commit que também alterou o arquivo `conflito.txt` e fazer merge.

````
$ git pull
CONFLICT (content): Merge conflict in conflito.txt
Automatic merge failed; fix conflicts and then commit the result.
````
O git nos avisa que há um conflito que não pode ser resolvido automaticamente.

````
$git status
...
Changed but not updated:
   (use "git add <file>..." to update what will be committed)
   (use "git checkout -- <file>..." to discard changes in working directory)
   
   	unmerged:   conflito.txt
````
Feitas as devidas alterações para que o arquivo fique no estágio correto:

````
$ git add conflito.txt
$ git commit -m"Merge do arquivo conflito.txt"
   
````

Bom, resolvido! Vamos rever o log:


````
$ git log

commit 1451db723f8e101e43de1056c6ac174bde20afbf
Author: Pedro Vereza <pdrvereza@gmail.com>
Date:   Tue May 13 15:25:37 2014 -0300

    Merge do arquivo conflito.txt

commit 080fca97ac0839132ee93d81de6933dceb507d91
Author: Outro Desenvolvedor <outro@email.com>
Date:   Tue May 13 14:32:45 2014 -0300

    Incluindo outras informações no arquivo conflito.txt

commit 10f59f3ab2caa387109169495a18135d57740a08
Author: Pedro Vereza <pdrvereza@gmail.com>
Date:   Tue May 13 13:23:16 2014 -0300

    Incluindo exemplo no arquivo conflito.txt

commit 960c34302051e8d1f8a18a5dd77eccbe3b6e3ff1
Author: Pedro Vereza <pdrvereza@gmail.com>
Date:   Tue May 13 09:08:37 2014 -0300

    Adicionando um-arquivo.txt

````

Quando fizemos o merge, acabamos tendo que criar um commit apenas para isso. O conteúdo do arquivo `conflito.txt` após o merge era diferente dos outros dois commits (o local e o que recebemos), isso nos obrigou a commitar essa nova versão. Existe uma outra opção:

````
$ git pull --rebase
Auto-merging conflito.txt
CONFLICT (content): Merge conflict in conflito.txt
Failed to merge in the changes.
...

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".

````

Da mesma forma que antes, há um arquivo que o merge automático não conseguiu resolver. Note que o git já indica o que deve ser feito após a resolução do conflito.

````
$git status
...
rebase in progress; onto a42f17c
You are currently rebasing branch 'master' on 'a42f17c'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

	both modified:      conflito.txt
	
````
Feita as alterações, ainda é preciso adicionar o arquivo, mas note que não é necessário fazer um novo commit, apenas continuar o rebase.
 
````
$ git add conflito.txt
$ git rebase --continue
Applying: Incluindo exemplo no arquivo conflito.txt
   
````

Olhando o log agora:

````
$ git log
commit 78cdca7c8e4dec62261103d7c3e46f061a75cc02
Author: Pedro Vereza <pdrvereza@gmail.com>
Date:   Tue May 13 13:23:16 2014 -0300

    Incluindo exemplo no arquivo conflito.txt

commit 080fca97ac0839132ee93d81de6933dceb507d91
Author: Outro Desenvolvedor <outro@email.com>
Date:   Tue May 13 14:32:45 2014 -0300

    Incluindo outras informações no arquivo conflito.txt

commit 960c34302051e8d1f8a18a5dd77eccbe3b6e3ff1
Author: Pedro Vereza <pdrvereza@gmail.com>
Date:   Tue May 13 09:08:37 2014 -0300

    Adicionando um-arquivo.txt

````

Dessa vez não temos commit de merge! Para entender o porquê, é preciso analisar as diferenças entre `git pull` e `git pull --rebase`.

##git pull##

O `git pull` funciona da forma mais óbvia: pega as mudanças do repositório "central" e aplica por cima dos commits do repositório local. Ao fazer isso, no exemplo anterior, foi necessário um terceiro commit, contendo a versão desejada do arquivo conflitante.

##git pull --rebase##
Já o `git pull --rebase` tem um truque diferente: esse comando pega as mudanças do repositório "central" e **aplica por baixo dos commits que existem apenas no repositório local**. Na prática, é como se a alteração feita no arquivo `conflito.txt` tivesse ocorrido após as alterações do outro desenvolvedor no mesmo arquivo. Analisando atentamente os logs, percebe-se que o *hash* do commit alterou de `10f59f3ab2caa387109169495a18135d57740a08` para `78cdca7c8e4dec62261103d7c3e46f061a75cc02`, isso ocorre porque o *hash* é calculo com base no commit anterior (e outras coisas).



Espero que isso tenha esclarecido algumas coisas, ou ao menos levantado perguntas interessantes.

[git-link]: http://git-scm.com/
[svn-link]: http://subversion.apache.org/
[tfs-link]: http://www.visualstudio.com/en-us/products/tfs-overview-vs.aspx
[github]: https://github.com/pedrovereza