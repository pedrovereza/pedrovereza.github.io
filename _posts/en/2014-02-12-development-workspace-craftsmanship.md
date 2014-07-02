---
layout: post
lang: "en"
trans: /artesao-de-maquina-de-desenvolvimento
title: Development Workspace Craftsmanship 
---

Craftsmen are usually known as people that produces something manually, without an industrial established process. Normally, the price of the products are higher than the manufactured equivalent, part of it is due to the originality and exclusivity. In theory, no two pieces look exactly the same.

Dev workspace craftsmen work similarly: they don't use automatized process and, **rarely, two development workspace behave the same**. The effects of it come as headaches and the same questions echoing across the room weekly: "*Did I have to install it?*", "*Can I take a look on the configurations you're using?*", "*Who said the plugin is not working? It's running fine in my machine*". It's a painful process that is normally ignored. 

But the truth is: 90% of what needs to be done to have a properly configured workspace is like a cake receipt. Download this, install that, change some environment variables, run a script, restart (in case it's Windows) and done: now it should compile. It's common sense to automatize the build, but why don't we do it when creating new workspaces? There's a bunch of tools available.

Among these tools (e.g [Vagrant][vagrant], [Chef][chef], etc.) one that called my attention, mainly because its simplicity, was [Babushka][babushka]. Imagine a world where everything that you need to do to have a development workspace set up and ready to go are a few ``.rb`` files in a repository somewhere and run a command like ``babushka dev-workspace``. Babushka has a nice DSL that is very  easy to learn. It works beautifully with ``homebrew`` and ``apt`` and, basically, any thing can be done. More details about babushka can be found in [The 8th edition of P2 Magazine, by **Thought**Works][p2-babushka]. You can also check [my github][babushka-repo] for an always-improving example.

Another option is to use the plain old bash scrpit. It's better than doing everything manually, no doubt about it, but the first disadvantage is the bizarre code. The second one is that it may be harder to maintain in the long run (part of if due to the first). Still, it's worth trying and will probably save time in the future.  

In the end of the day, it doesn't matter which tool will be used. The important thing is to have an automatized way to create new development environments that avoid pain and works for everyone. If a  new member of your team needs eight hours to configure everything, maybe he's being forced to be a dev workspace craftsman.

[vagrant]:http://www.vagrantup.com/
[chef]:http://www.getchef.com/
[babushka]:https://babushka.me/
[p2-babushka]:http://thoughtworks.github.io/p2/issue08/babushka/
[babushka-repo]:https://github.com/pedrovereza/babushka-deps
