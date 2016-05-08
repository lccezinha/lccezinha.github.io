---
layout: post
title:  "Dicas de como começar com Ruby e Rails"
date:   2015-05-11 22:01:28
categories: ruby rails aprendizado dicas
---

Algumas dicas pra quem pretende começar com Ruby e consequentemente Rails, dicas essas que digo por experiência e que se talvez alguém tivesse me dado, meu aprendizado poderia ter sido melhor, mais rápido e me ajudaria e não apanhar tanto nesse caminhos da pedras inicial.

> Posso começar com Rails direto, sem saber Ruby ?

Bom, a resposta pra essa pergunta seria: SIM.

Eu mesmo comecei assim, direto com o Rails, na verdade estudei Ruby antes, porém estudei pouco e sim, sofri um pouco para começar a enteder como o Rails funcionava e que eram aquelas coisas do tipo `var[:x] = 1`, `:some => 10`, nessa linguagem maluca.

Porém a medida que você irá começar a se aprofundar no Rails e tentar fazer coisas além do básico dentro do framework, vai sentir falta de uma boa base em Ruby.

Então a dica é, se vai começar com Rails, antes dedique um tempo a aprender o básico de Ruby. Aprenda sobre suas estruturas condicionais, estruturas de decisão, tipos de dados, módulos/mixins, orientação a objetos e até mesmo o básico sobre metaprogramação (não precisa virar um ninja, eu mesmo sei pouca coisa), porém saiba que ela exista e que é possível utilizar determinados "truques" com ela.

> Como posso começar com o Rails ?

Se você já sabe o básico de Ruby, terá uma ajuda muito grande na hora de começar com o Rails.

A dica aqui seria: **Comece devagar**.

Não tente começar logo de cara com algo gigante, ou até mesmo algo para algum "cliente", pode ser que por falta de experiência você cometa alguns erros e poderá acabar se frustrando e se desmotivando com o framework.

Uma aplicação boa para se começar seria um bom e velho `Blog`.

Comece criando um `scaffold` de um `Post`, entenda o que aquele "comando mágico" está fazendo pra você, entenda como funciona o MVC do Rails, a forma mágica como ele cria suas rotas e adicione algumas validações.

Depois disso, aprimore seu exemplo, adicione algum outro `model`, por exemplo `Comment` e comece a aprender como o Rails trabalha na parte de relacionamento entre `models` e `tabelas do banco de dados`.

Melhore sua view e permita um usuário qualquer comentar um post.

Aprimore ainda mais, adicionando agora um upload de imagem ao seu post e exiba a foto na view.

Ainda existe muito que se possa fazer na aplicação, tente adicionar uma autenticação de usuário para que somente usuários logados possam cadastrar ou comentar alguns posts.

Feito isso, tente melhorar a parte de comentários fazendo com que ele funcione utilizando `AJAX`.

A grande saca nisso tudo é: **Aprenda aos poucos**, foque seu estudo em cima de um tema específico e estude-o.

É muito melhor focar em um tema e aprender por partes, do que tentar abraçar o todo e começar a se perder e não aprender, pior de tudo, é muito fácil se desmotivar dessa forma.

Bom essas são as dicas que eu dou para quem está interessado em começar com Ruby/Rails.

Alguns links úteis para aprender e tirar dúvidas sobre Ruby:

- [Ruby-Lang](https://www.ruby-lang.org/pt/)
- [Code Cademy - Ruby](http://www.codecademy.com/pt/tracks/ruby)
- [Net Guru](https://netguru.co/blog/top-free-online-ruby-tutorials)

Pra quem quiser aprender e tirar dúvidas de Rails, aqui você encontra QUASE tudo que precisa:

- [Rails Guides](http://guides.rubyonrails.org/) <3
- [API DOCK](http://apidock.com/) <3

Bom é isso ae, espero que as dicas possam ser úteis para quem precisar, valeu!
