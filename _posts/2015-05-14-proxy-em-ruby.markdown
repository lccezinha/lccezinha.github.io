---
layout: post
title:  "Design Patterns: Proxy em Ruby"
date:   2015-05-14 22:01:28
categories: ruby design-patterns
comments: true
---

Por definição [Proxy](http://pt.wikipedia.org/wiki/Proxy_%28padr%C3%B5es_de_projeto%29)

> Um proxy, em sua forma mais geral, é uma classe que funciona como uma interface para outra classe. A classe proxy poderia conectar-se a qualquer coisa: uma conexão de rede, um objeto grande em memória, um arquivo, ou algum recurso que é difícil ou impossível de ser duplicado.

Em outras palavras, o padrão `Proxy` fornece uma maneira de criar uma classe substituta para controlar o acesso a classe real. Além disso fornece uma maneira simples de implementar `Lazy loading`.

Para o exemplo, imagine que temos uma classe `Computer` que executa determinados comandos, e nesse caso precisamos fornecer uma maneira de dar autorização de executação a esses comandos.

A class `ComputerProxy`, é uma classe simples, que na sua inicialização, recebe um objeto `real_object` e apenas está fazendo a delegação dos comandos executados para a classe real.

{% highlight ruby %}
class ComputerProxy
  def initialize(real_object)
    @real_object = real_object
  end

  def add(command)
    @real_object.add(command)
  end

  def execute
    @real_object.execute
  end
end


class Computer
  attr_reader :queue

  def initialize
    @queue = []
  end

  def add(command)
    @queue << command
  end

  def execute
    queue.inject("\n") { |result, command| result + command.execute }
  end
end

###

require 'spec_helper'
require_relative '../lib/proxy'

describe 'Proxy Pattern' do

  it 'delegates all to real object' do
    computer = double('computer', queue: [], add: [], execute: true)
    proxy = ComputerProxy.new(computer)

    allow(computer).to receive(:add)
    proxy.add(double('command'))

    allow(computer).to receive(:execute)
    proxy.execute
  end
end
{% endhighlight %}

Como vimos no exemplo do [Decorator](http://lccezinha.github.io/ruby/design-patterns/2015/05/11/decorator-em-ruby.html) podemos simplificar essa delegação utilizando o módulo `Forwardable` do Ruby

{% highlight ruby %}

require 'forwardable'

class ComputerProxy
  extend Forwardable

  def_delegators :@real_object, :add, :execute

  def initialize(real_object)
    @real_object = real_object
  end
end


class Computer
  attr_reader :queue

  def initialize
    @queue = []
  end

  def add(command)
    @queue << command
  end

  def execute
    queue.inject("\n") { |result, command| result + command.execute }
  end
end
{% endhighlight %}

Até aqui não fizemos nada demais, apenas encapsulamos o objeto real e fizemos um proxy delegar as chamadas aos métodos da classe.

Agora vamos codificar uma maneira de autorização de execução de comandos com a nossa classe `Proxy`. Para isso iremos criar uma nova classe chamada `Hero`, a idéia que é `Hero` só poderá executar o método `executar`se ele tiver dentro de `keywords` a keyword `computer`. Ou seja, para executar esse método ele precisa estar autorizado.

{% highlight ruby %}
require 'forwardable'

class Hero
  attr_accessor :keywords

  def initialize
    @keywords = []
  end
end

class ComputerProxy
  extend Forwardable

  def_delegators :@real_object, :add

  def initialize(real_object, hero)
    @real_object = real_object
    @hero = hero
  end

  def execute
    check_access
    @real_object.execute
  end

  private

  def check_access
    raise 'You have no access' unless @hero.keywords.include?(:computer)
  end
end

class Computer
  attr_reader :queue

  def initialize
    @queue = []
  end

  def add(command)
    @queue << command
  end

  def execute
    queue.inject("\n") { |result, command| result + command.execute }
  end
end

###

require 'spec_helper'
require_relative '../lib/proxy'

describe 'Proxy Pattern' do

  it 'delegates all to real object' do
    computer = double('computer', queue: [], add: [], execute: true)
    hero = Hero.new
    proxy = ComputerProxy.new(computer, hero)

    allow(computer).to receive(:add)
    proxy.add(double('command'))

    hero.keywords << :computer
    allow(computer).to receive(:execute)
    proxy.execute
  end

  it 'You have no access' do
    computer = double('computer', queue: [], add: [], execute: true)
    hero = Hero.new #without :computer in keywords
    proxy = ComputerProxy.new(computer, hero)

    expect { proxy.execute }.to raise_error
  end
end
{% endhighlight %}

Nossa implementação mudou, a classe `ComputerProxy` agora além de receber um `real_object`, também irá receber um `hero`, existe agora antes da execução do método `execute` a chamada para um método `check_access` que irá verificar a condição para que o `Hero` possa executar o método.

Repare que até os testes mudaram para atender essa nova implementação, aqui está a implementação do padrão `Proxy`

Porém existe algo que pode ser melhorado, a implementação atual recebe um `real_object` na classe `ComputerProxy`, porém poderia ser alterado para que ele só chame `real_object` quando ele realmente for necessário, ou seja, `Lazy loading`.

{% highlight ruby %}

require 'forwardable'

class Hero
  attr_accessor :keywords

  def initialize
    @keywords = []
  end
end

class ComputerProxy
  extend Forwardable

  def_delegators :real_object, :add

  def initialize(hero)
    @hero = hero
  end

  def execute
    check_access
    @real_object.execute
  end

  private

  def real_object
    @real_object ||= Computer.new
  end

  def check_access
    raise 'You have no access' unless @hero.keywords.include?(:computer)
  end
end

class Computer
  attr_reader :queue

  def initialize
    @queue = []
  end

  def add(command)
    @queue << command
  end

  def execute
    queue.inject("\n") { |result, command| result + command.execute }
  end
end

###

require 'spec_helper'
require_relative '../lib/proxy'

describe 'Proxy Pattern' do

  it 'delegetas all to real object' do
    pending
    computer = double('computer', queue: [], add: [], execute: true)
    hero = Hero.new
    proxy = ComputerProxy.new(computer, hero)

    allow(computer).to receive(:add)
    proxy.add(double('command'))

    hero.keywords << :computer
    allow(computer).to receive(:execute)
    proxy.execute
  end

  it 'delegetas all to real object with lazy' do
    computer = double('computer', queue: [], add: [], execute: true)
    allow(Computer). to receive(:new).and_return(computer)
    hero = Hero.new
    proxy = ComputerProxy.new(hero)

    allow(computer).to receive(:add)
    proxy.add(double('command'))

    hero.keywords << :computer
    allow(computer).to receive(:execute)
    proxy.execute
  end

  it 'You have no access' do
    pending
    computer = double('computer', queue: [], add: [], execute: true)
    hero = Hero.new #without :computer in keywords
    proxy = ComputerProxy.new(computer, hero)

    expect { proxy.execute }.to raise_error
  end
end
{% endhighlight %}

Para realizar essa implementação, removemos a variável de instância `@real_object` do método `def_delegators` e utilizamos agora a achamada para um método `real_object`, que irá verificar se o objeto já existe, caso não exista irá criar um objeto novo. Dessa forma `real_object` só será criado quando realmente for chamado.

[Refs #1 - Wikipedia](http://pt.wikipedia.org/wiki/Proxy_%28padr%C3%B5es_de_projeto%29)

[Refs #2 - Tutorials Point](http://www.tutorialspoint.com/design_pattern/decorator_pattern.htm)

[Refs #3 - Sourcemaking](https://sourcemaking.com/design_patterns/proxy)

[Refs #4 - Devmedia](http://www.devmedia.com.br/conheca-o-pattern-proxy-gof-gang-of-four/4066)

Até a próxima!.
