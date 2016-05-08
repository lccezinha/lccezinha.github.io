---
layout: post
title:  "Sincronizando repositórios forkeados do github"
date:   2015-02-23 22:01:28
categories: github
---

Quando se participa de projetos open source é muito comum após um tempo surgir a necessidade de atualizar o código de seu repositório forkeado com o remoto, mas como fazer isso ?

Tomando como exemplo um fork meu do projeto [Ruby Jobs](http://github.com/ruby-jobs/ruby-jobs) e executando o comando `git remote -v` temos a seguinte saída:

    origin  git@github.com:lccezinha/ruby-jobs.git (fetch)
    origin  git@github.com:lccezinha/ruby-jobs.git (push)

O primeiro passo é adicionar um novo endereço `remote` ao repositório local com o endereço do repositório remoto, no exemplo o novo `remote` se chamará `upstream` e o repositório será o do [Ruby Jobs](http://github.com/ruby-jobs/ruby-jobs).

    git remote add upstream https://github.com/ruby-jobs/ruby-jobs.git

Agora ao executar o comando `git remote -v` novamente, temos uma saída diferente:

    origin  git@github.com:lccezinha/ruby-jobs.git (fetch)
    origin  git@github.com:lccezinha/ruby-jobs.git (push)
    upstream  https://github.com/ruby-jobs/ruby-jobs.git (fetch)
    upstream  https://github.com/ruby-jobs/ruby-jobs.git (push)

O próximo passo é executar o `fetch` do novo `remote` adicionado, para baixar os novos commits:

    git fetch upstream

E por fim fazer o `merge` da branch `master` do `upstream` com a sua branch `master` local:

    git merge upstream/master

Pronto, o repositório local está atualizado com o repositório remoto e agora novas features e pull requests podem ser trabalhados sem o risco de estar com código desatualizado.

