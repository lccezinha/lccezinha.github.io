---
layout: post
title:  "Design Patterns: Factory e AbstractFactory em Ruby"
date:   2015-04-09 22:01:28
categories: ruby design-patterns
---

# Factory Pattern

Por definição o [Factory Pattern](http://pt.wikipedia.org/wiki/Factory_Method):

> Factory, é um padrão de projeto que permite as classes delegar para subclasses decidirem que elas devem instanciar.

O Factory Pattern é muito similar ao [Template Method](http://lccezinha.github.io/ruby/design-patterns/2015/03/03/template-method-em-ruby.html) porém é utilizado para criar instâncias de objetos.

Para exemplo iremos criar um classe `Festival` que possui vários estilos musicais, mas para começo terá apenas bandas de `HeavyMetal`.

{% highlight ruby %}
class HeavyMetal; end

class Festival
  attr_accessor :bands

  def initialize(number)
    @bandas = []
    number.times { bands << HeavyMetal.new }
  end
end

###

require 'spec_helper'

describe 'Factory Pattern' do
  it 'its a HeavyMetal festival' do
    festival = Festival.new(3)
    expect(festival.bands.count { |band| band.class == HeavyMetal }).to eq(3)
  end
end
{% endhighlight %}

Até então não haverá problema, mas e se houver a inclusão de bandas de `ThrashMetal` a esse festival ? O método `initialize` da class `Festival` está prepado para apenas adicionar bandas da class `HeavyMetal`, ou seja a implementação tem problemas pois ela não sabe como e quem instanciar nesse caso.

Agora iremos fazer a implementação do Factory Pattern para sanar esse problema, e será possível também notar a similaridade que esse Pattern tem com Template Method.

{% highlight ruby %}
class HeavyMetal; end

class ThrashMetal; end

class Festival
  attr_accessor :bands

  def initialize(number)
    @bandas = []
    number.times { bands << create }
    # essa chamada ao método create que está na subclasse funciona da mesma forma que o Template Method.
  end
end

class HeavyMetalBand < Festival
  def create
    HeavyMetal.new
  end
end

class ThrashMetalBand < Festival
  def create
    ThrashMetal.new
  end
end

###

describe 'Factory Pattern' do
  it 'its a HeavyMetal festival' do
    festival = HeavyMetalBand.new(3)
    expect(festival.bands.count { |band| band.class == HeavyMetal }).to eq(3)
  end

  it 'its a ThrashMetal festival' do
    festival = ThrashMetalBand.new(3)
    expect(festival.bands.count { |band| band.class == ThrashMetal }).to eq(3)
  end
end
{% endhighlight %}

O que aconteceu na implementação foi que utilizando o Template Method, nós implementamos o Factory Pattern. Foi criada uma classe base chamada de `Festival` e essa classe possui uma chamada ao método `create` que será implementado pela subclasse. Ou seja, a chamada ao método `create` é somente um esqueleto de um algoritmo que deverá ser implementado pelas subclasses, e elas decidem quem elas devem criar.

Porém imaginando uma situação onde deveriam existir mais tipos de bandas, deixaria o código muito extenso, criando uma nova classe para cada estilo ou bando, sendo que todas iriam fazer a mesma coisa, apenas retornar um objeto.

Para esses casos é possível alterar um pouco a implementação do Factory Pattern e utilizar parametrização, para decidir quem deverá ser criado.

{% highlight ruby %}
class HeavyMetal; end

class ThrashMetal; end

class Festival
  attr_accessor :bands

  def initialize(number, style)
    @bandas = []
    number.times { bands << create(style) }
  end
end

class BandFactory < Festival
  def create(style)
    if [:heavy, :thrash].include(style)
      self.class.const_get(style.to_s.capitalize).new
    else
      raise 'Unkown!'
    end
  end
end

###

describe 'Factory Pattern' do
  it 'its a HeavyMetal festival' do
    festival = BandFactory.new(3, :heavy)
    expect(festival.bands.count { |band| band.class == HeavyMetal }).to eq(3)
  end

  it 'its a ThrashMetal festival' do
    festival = BandFactory.new(3, :thrash)
    expect(festival.bands.count { |band| band.class == ThrashMetal }).to eq(3)
  end
end
{% endhighlight %}

Agora existe somente uma Factory, a `BandFactory` e ela será a responsável por criar todas as instâncias. Para evitar o uso de vários ifs para cada tipo de classe, utilizou-se metaprogramação do Ruby <3 e como uma forma de garantir que sejam apenas criadas bandas conhecidas, foi feito um `if` de validação da classe passada para a Factory.

# AbstractFactory Pattern

Por definição, [AbstractFactory Pattern](http://pt.wikipedia.org/wiki/Abstract_Factory)

> Abstract Factory é um padrão de projeto que permite a criação de famílias de objetos relacionados ou dependentes por meio de uma única interface e sem que a classe concreta seja especificada.

Seu funcionamento é muito similar a implementação do [Strategy Pattern](http://lccezinha.github.io/ruby/design-patterns/2015/03/06/strategy-em-ruby.html), porém é utilizado para criar famílias de objetos relacionados ou dependentes. Ele irá provar uma interface para a criação de um grupo de objetos sem especificar a sua classe concreta.

{% highlight ruby %}
class HeavyMetal; end

class ThrashMetal; end

class Festival
  attr_accessor :bands, :factory

  def initialize(factory)
    @bands = []
    @factory = factory
  end

  def add_heavy_metal(number)
    number.times { bands << factory.create_heavy_metal }
  end

  def add_thrash_metal(number)
    number.times { bands << factory.create_thrash_metal }
  end
end

class BandFactory
  def create_heavy_metal
    HeavyMetal.new
  end

  def create_thrash_metal
    ThrashMetal.new
  end
end

###

describe 'Factory Pattern' do
  it 'its a HeavyMetal festival' do
    festival = Festival.new(BandFactory.new)
    festival.add_heavy_metal(3)
    expect(festival.bands.count { |band| band.class == HeavyMetal }).to eq(3)
  end

  it 'its a ThrashMetal festival' do
    festival = Festival.new(BandFactory.new)
    festival.add_thrash_metal(3)
    expect(festival.bands.count { |band| band.class == ThrashMetal }).to eq(3)
  end
end
{% endhighlight %}

Com a  implementação agora utilizando `AbstractFactory`, a classe `Festival` recebe como argumento a `Factory` e também possui a chamada para dois métodos `add_heavy_metal` e `add_thrash_metal`, esse método funcionam apenas para o que o objeto criado seja adicionado ao array `bands` por que a chamada para a real criação do objeto fica a cargo da factory, que foi passada na inicialização da classe `Festival`.

Já a classe `BandFactory` possui os métodos responsável por retornar a criação do objeto de fato, funcionando assim como uma fábrica para a criação de objetos relacionados.

A breve explicação de quando utilizar um e quando utilizar outro é a seguinte:

  - Factory: é utilizado para criar objetos, por exemplo instâncias de `HeavyMetal` ou `ThrashMetal`.
  - AbstractFactory: é utilizado para criar fábricas para criação de outros objetos, por exemplo `BandFactory`.

Esse foi o uso de Factory e AbstractFactory em Ruby.

[Refs #1 - Wikipedia](http://pt.wikipedia.org/wiki/Factory_Method)

[Refs #2 - Wikipedia](http://pt.wikipedia.org/wiki/Abstract_Factory)



