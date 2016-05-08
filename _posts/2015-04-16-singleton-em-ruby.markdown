---
layout: post
title:  "Design Patterns: Singleton Ruby"
date:   2015-04-16 22:01:28
categories: ruby design-patterns
---

Por definição o [Singleton](http://pt.wikipedia.org/wiki/Singleton):

> Singleton é um padrão que garante a existência de apenas uma instância de uma classe, mantendo um ponto global de acesso ao seu objeto.

Em outras palavras, implementar Singleton em uma classe irá garantir que exista apenas uma instância dela por todo o sistema, sendo essa instância acessada por apenas um ponto, esse ponto de acesso geralmente é um método chamado `instance` que como o nome diz, será o responsável por retornar a instância da classe. A aplicação desse pattern é comumente utilizada em classes de acesso a banco de dados, loggers e classes de configuração.

Para a primeira parte do exemplo iremos criar um singleton "na mão".

Usando como exemplo as classes do exemplo anterior do pattern [Factory e AbstractFactory](http://lccezinha.github.io/ruby/design-patterns/2015/04/09/factory-e-abstract-factory-em-ruby.html), queremos garantir que a classe `FestivalFactory` seja uma classe singleton, para garantir que exista apenas uma instância de fabríca de bandas.

{% highlight ruby %}
class HeavyMetal; end
class ThrashMetal; end

class FestivalFactory

  @@instance = nil

  def self.instance
    @@instance ||= FestivalFactory.new
  end

  def create_heavy_metal
    HeavyMetal.new
  end
  def create_thrash_metal
    ThrashMetal.new
  end
end

###

require 'spec_helper'
require_relative '../lib/singleton_pattern'

describe 'Singleton Pattern' do
  let(:factory) { FestivalFactory.instance }

  it 'needs to be a FestivalFactory instance' do
    expect(factory).to be_instance_of(FestivalFactory)
  end

  it 'needs to be the same instance for another object' do
    another_factory = FestivalFactory.instance

    expect(another_factory).to eq(factory)
  end
end
{% endhighlight %}

Essa é a implementação do `Singleton` em Ruby, na sua forma mais básica, o que fizemos foi criar um variável de instância `@@instance` que será a variável que armazenará a instância da classe, e implementamos o método de classe `self.instance` que será o responsável por retornar essa instância.

Porém essa implementação ainda apresenta um problema, pois se tentarmos executar o código `FestivalFactory.new` ele irá retornar uma nova instância da factory, não fazendo sentido assim o uso de `Singleton`. Para resolver esse problema, temos que deixar a chamada para o método `new` da classe `FestivalFactory` privada, utilizando o método `private_class_method`.

{% highlight ruby %}
class HeavyMetal; end
class ThrashMetal; end

class FestivalFactory

  @@instance = nil

  def self.instance
    @@instance ||= FestivalFactory.new
  end

  def create_heavy_metal
    HeavyMetal.new
  end
  def create_thrash_metal
    ThrashMetal.new
  end

  private_class_method :new
end

###

require 'spec_helper'
require_relative '../lib/singleton_pattern'

describe 'Singleton Pattern' do
  let(:factory) { FestivalFactory.instance }

  it 'needs to be a FestivalFactory instance' do
    expect(factory).to be_instance_of(FestivalFactory)
  end

  it 'needs to be the same instance for another object' do
    another_factory = FestivalFactory.instance

    expect(another_factory).to eq(factory)
  end

  it 'deny call .new method' do
    expect { FestivalFactory.new }.to raise_exception
  end
end
{% endhighlight %}

Pronto, a implementação do singleton está feita.

A implementação desse módo possui uma "vantagem", pois como a variável `@@instance` inicialmente possui o valor `nil` ela só será preenchida com a instância da classe no momento em que a chamada `FestivalFactory.instance` for executada, ou seja, ela é `lazy initialization` só ocupando memória quando for chamado.

Porém em Ruby ainda é possível simplificar ainda mais essa criação, utilizando o módulo `Singleton`, já fornecido pela linguagem, apenas fazendo `require` do módulo.

{% highlight ruby %}
require 'singleton'

class HeavyMetal; end
class ThrashMetal; end

class FestivalFactory
  include Singleton

  def create_heavy_metal
    HeavyMetal.new
  end
  def create_thrash_metal
    ThrashMetal.new
  end
end

###

require 'spec_helper'
require_relative '../lib/singleton_pattern'

describe 'Singleton Pattern' do
  let(:factory) { FestivalFactory.instance }

  it 'needs to be a FestivalFactory instance' do
    expect(factory).to be_instance_of(FestivalFactory)
  end

  it 'needs to be the same instance for another object' do
    another_factory = FestivalFactory.instance

    expect(another_factory).to eq(factory)
  end

  it 'deny call .new method' do
    expect { FestivalFactory.new }.to raise_exception
  end
end
{% endhighlight %}

Essa foi a implementação de uso do `Singleton Pattern` em Ruby.

[Refs #1 - Wikipedia](http://pt.wikipedia.org/wiki/Singleton)

Até a próxima!.
