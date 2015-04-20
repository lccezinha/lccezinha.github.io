---
layout: post
title:  "Design Patterns: Observer em Ruby"
date:   2015-03-17 22:01:28
categories: ruby design-patterns
comments: true
---

Por definição o [Observer](http://pt.wikipedia.org/wiki/Observer):

> O Observer é um padrão de projeto de software que define uma dependência um-para-muitos entre objetos de modo que quando um objeto muda o estado, todos seus dependentes são notificados e atualizados automaticamente. Permite que objetos interessados sejam avisados da mudança de estado ou outros eventos ocorrendo num outro objeto. O padrão Observer é também chamado de Publisher-Subscriber, Event Generator e Dependents.

Em outras palavras, Observer irá nos ajudar quando precisamos propagar uma determinada informação, para outras partes do sistema.

Para o exemplo, teremos uma classe `Hero`, e uma classe `Tile` caso o `Hero` encontre uma classe `Tile` que esteja com o atributo `cursed: true` ele irá sofrer um dano e perder um valor do atributo `damage`.

{% highlight ruby %}
# Behavior Pattern

class Hero
  def initialize
    @cursed = false
  end

  def cursed?
    @cursed
  end

  def discover(tile)
    @cursed = true if tile.cursed?
  end
end

class Tile
  def initialize(attrs = {})
    @cursed = attrs.fetch(:cursed, false)
  end

  def cursed?
    @cursed
  end
end

###

require 'spec_helper'

describe 'Observer Pattern' do

  describe Hero do
    it 'its cursed then discovers cursed Tile' do
      hero = Hero.new
      tile = Tile.new cursed: true
      hero.discover(tile)

      expect(hero.cursed?).to be_truthy
    end

    it 'its not cursed when discovers title without curse' do
      hero = Hero.new
      tile = Tile.new
      hero.discover(tile)

      expect(hero.cursed?).to be_falsy
    end
  end

  describe Tile do
    it 'its not cursed by default' do
      tile = Tile.new

      expect(tile.cursed?).to be_falsy
    end

    it 'can create as cursed' do
      tile = Tile.new cursed: true

      expect(tile.cursed?).to be_truthy
    end
  end
end
{% endhighlight %}

O próximo passo é cria a implementação para que quando um `Hero` use o método `discover` e encontre algum `Tile` que seja `cursed: true`, o método `damage` seja invocado e faça com que o `Hero` perca valores no atributo `health`. A primeira implementação não será muito bonita, mas servirá para o caso.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :health

  def initialize
    @cursed = false
    @health = 10
  end

  def cursed?
    @cursed
  end

  def damage(hit)
    @health -= hit
  end

  def discover(tile)
    @cursed = true if tile.cursed?
  end
end

class Tile
  def initialize(attrs = {})
    @cursed = attrs.fetch(:cursed, false)
    @hero = attrs.fetch(:hero, nil)
  end

  def cursed?
    @cursed
  end

  def activate_curse
    @hero.damage(4)
  end
end

###

require 'spec_helper'

describe 'Observer Pattern' do

  describe Hero do
    let(:hero) { Hero.new }

    it 'its cursed then discovers cursed Tile' do
      tile = Tile.new cursed: true
      hero.discover(tile)

      expect(hero.cursed?).to be_truthy
    end

    it 'its not cursed when discovers title without curse' do
      tile = Tile.new
      hero.discover(tile)

      expect(hero.cursed?).to be_falsy
    end

    it 'it has default health eq to 10' do
      expect(hero.health).to eq(10)
    end

    it 'can be damaged' do
      hero.damage(6)
      expect(hero.health).to eq(4)
    end
  end

  describe Tile do
    it 'activetes curse' do
      hero = Hero.new
      tile = Tile.new cursed: true, hero: hero

      tile.activate_curse
      expect(hero.health).to eq(6)
    end

    it 'its not cursed by default' do
      tile = Tile.new

      expect(tile.cursed?).to be_falsy
    end

    it 'can create as cursed' do
      tile = Tile.new cursed: true

      expect(tile.cursed?).to be_truthy
    end
  end
end

{% endhighlight %}

O problema aparece pois agora estamos injetando um `Hero` dentro de `Tile` para que ele ative o método `activate_curse`, e essa situação não é boa, pois o acoplamento entre as classes fica muito alto, e também por que impossibilita a tentativa de usar o método `activate_curse` para vários `Hero` ao mesmo tempo.

Faremos uma primeira "implementação" do Observer Patterns, removendo a injeção de uma instância de `Hero` em `Tile`. Na nova implementação, a classe `Tile` terá um array de objetos a serem notificados para que o método `damage` seja chamado a essa lista. A alteração na classe `Hero` será que agora quando `Hero` descobrir um `Tile.new cursed: true` ele irá se adicionar a essa lista de objetos a serem notificados.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :health

  def initialize
    @cursed = false
    @health = 10
  end

  def cursed?
    @cursed
  end

  def damage(hit)
    @health -= hit
  end

  def discover(tile)
    if tile.cursed?
      @cursed = true
      tile.add_cursed(self)
    end
  end
end

class Tile
  attr_reader :cursed_creatures

  def initialize(attrs = {})
    @cursed = attrs.fetch(:cursed, false)
    @cursed_creatures = []
  end

  def cursed?
    @cursed
  end

  def add_cursed(creature)
    @cursed_creatures << creature
  end

  def activate_curse
    cursed_creatures.each { |creature| creature.damage(4) }
  end
end

###

require 'spec_helper'

describe 'Observer Pattern' do

  describe Hero do
    let(:hero) { Hero.new }

    it 'its cursed then discovers cursed Tile' do
      tile = Tile.new cursed: true
      hero.discover(tile)

      expect(hero.cursed?).to be_truthy
    end

    it 'its not cursed when discovers title without curse' do
      tile = Tile.new
      hero.discover(tile)

      expect(hero.cursed?).to be_falsy
    end

    it 'it has default health eq to 10' do
      expect(hero.health).to eq(10)
    end

    it 'can be damaged' do
      hero.damage(6)
      expect(hero.health).to eq(4)
    end
  end

  describe Tile do
    it 'activetes curse' do
      hero = Hero.new
      tile = Tile.new cursed: true

      hero.discover(tile)
      tile.activate_curse

      expect(hero.health).to eq(6)
    end

    it 'activate curse in several heroes' do
      hero_one = Hero.new
      hero_two = Hero.new
      tile = Tile.new cursed: true

      hero_one.discover(tile)
      hero_two.discover(tile)
      tile.activate_curse

      expect(hero_one.health).to eq(6)
      expect(hero_two.health).to eq(6)
    end

    it 'its not cursed by default' do
      tile = Tile.new

      expect(tile.cursed?).to be_falsy
    end

    it 'can create as cursed' do
      tile = Tile.new cursed: true

      expect(tile.cursed?).to be_truthy
    end
  end
end
{% endhighlight %}

Essa é a implementação do Observer Pattern, porém ela não está muito visível pois o nome dos métodos não estão ajudando, para isso será feita uma alteração nos nomes afim de deixar o Observer mais visível.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :health

  def initialize
    @cursed = false
    @health = 10
  end

  def cursed?
    @cursed
  end

  def damage(hit)
    @health -= hit
  end

  def update
    damage(4)
  end

  def discover(tile)
    if tile.cursed?
      @cursed = true
      tile.add_observer(self)
    end
  end
end

class Tile
  attr_reader :observers

  def initialize(attrs = {})
    @cursed = attrs.fetch(:cursed, false)
    @observers = []
  end

  def cursed?
    @cursed
  end

  def add_observer(observer)
    @observers << observer
  end

  def activate_curse
    notify_observers
  end

  def notify_observers
    observers.each { |observer| observer.update }
  end
end

{% endhighlight %}

Agora a implementação está mais clara, e toda vez que um `Hero` invocar o método `discover(tile)` e esse `Tile` estiver com `cursed: true` a instância de `Hero` será adicionada a lista de `observers` de `Tile`. Quando `Tile` invocar o método `activate_curse` ele irá notificar a todos aqueles que estão na sua lista, que eles devem invocar o método `update`, esse que deve ser implementado dentro da classe que será adicionada a lista de observers.

Para deixar a implementação mais clara, podemos remover os métodos que são pertinentes ao `Observer` em um módulo e utlizar como um `mixin` que será incluído na classe `Tile`, assim como a documentação do módulo `Observer` recomenda.

{% highlight ruby %}
module Observable
  attr_reader :observers

  def initialize(attrs = {})
    @observers = []
  end

  def add_observer(observer)
    @observers << observer
  end

  def notify_observers
    observers.each { |observer| observer.update }
  end
end

###

class Tile
  include Observable

  def initialize(attrs = {})
    super
    @cursed = attrs.fetch(:cursed, false)
  end

  def cursed?
    @cursed
  end

  def activate_curse
    notify_observers
  end
end
{% endhighlight %}

Vale lembrar também que o próprio Ruby já tem a implementação do módulo Observer.

Para utlizá-lo nessa implementação, é preciso:

  + Fazer `require "observer"`
  + Adicionar `include Observable` na classe `Tile`
  + Remover a chamada de `super` no `initialize` de `Tile`
  + Adicionar uma chamada o método `changed` do módulo `Observer` ao método que deverá notificar os observers


{% highlight ruby %}
class Tile
  include Observable

  def initialize(attrs = {})
    @cursed = attrs.fetch(:cursed, false)
  end

  def cursed?
    @cursed
  end

  def activate_curse
    change
    notify_observers
  end
end
{% endhighlight %}

Esse foi o uso de Observer Pattern em Ruby.

[Refs #1 - Reefpoints](http://reefpoints.dockyard.com/2013/08/20/design-patterns-observer-pattern.html)

[Refs #2 - Edapx](http://edapx.com/2013/12/08/observer-pattern/)

[Refs #3 - Anderson Leite](https://andersonleiteblog.wordpress.com/2009/10/25/design-patterns-observer-em-ruby/)

[Refs #4 - Wikipedia](http://pt.wikipedia.org/wiki/Observer)



