---
layout: post
title:  "Design Patterns: Template Method em Ruby"
date:   2015-03-03 22:01:28
categories: ruby design-patterns
---

Por definição o [Template Method](http://pt.wikipedia.org/wiki/Template_Method):

> Um Template Method auxilia na definição de um algoritmo com partes do mesmo definidos por métodos abstratos. As subclasses devem se responsabilizar por estas partes abstratas, deste algoritmo, que serão implementadas, possivelmente de várias formas, ou seja, cada subclasse irá implementar à sua necessidade e oferecer um comportamento concreto construindo todo o algoritmo.

> Define um esqueleto de um algoritmo, deixando certar partes para serem implementadas pelas subclasses, essas subclasses podem sobreescrever os comportamentos sem alterar a estrutura do algoritmo principal.

Em outras palavras, o Template Method fornece uma estrutura "esqueleto" em um método na superclasse, cabendo as classes filhas a responsabilidade de implementar e preencher esse "esqueleto" de acordo com suas necessidades.

Para iniciar o exemplo, existe uma classe `Hero` com um método `#attack`, `Hero` também pode ter uma `occupation` definida como `hero` ou `warrior`, o que apenas aumenta o valor do attributo `damage`.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :damage

  def initialize(occupation = nil)
    @damage = occupation == :warrior ? 15 : 10
  end

  def attack
    "Attacked dealing #{damage} damage"
  end

end

###

require 'spec_helper'
require_relative '../lib/template_method'

describe 'Template Method Pattern' do
  context 'Default Hero' do
    let(:hero) { Hero.new }

    it 'has default damage rating 10' do
      expect(hero.damage).to eq(10)
    end

    it 'can attack' do
      expect(hero.attack).to eq('Attacked dealing 10 damage')
    end
  end

  context 'Default Warrior' do
    let(:hero) { Hero.new :warrior }

    it 'has default damage rating 15' do
      expect(hero.damage).to eq(15)
    end

    it 'can attack' do
      expect(hero.attack).to eq('Attacked dealing 15 damage')
    end
  end
end
{% endhighlight %}

Embora não seja a melhor alternativa, para o modelo inicial a implementação com um `if` dentro do método `initialize` resolve o problema.

Agora além de poder ser um `Warrior` ou um `Hero`, a classe `Hero` também pode ser um `Mage` e tambem terá um novo atributo `abilities` que é um atributo tipo `array`.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :damage, :abilities

  def initialize(occupation = nil)
    if occupation == :warrior
      @damage = 15
      @abilities = [:strike]
    elsif occupation == :mage
      @damage = 7
      @abilities = [:magic_arrow]
    else
      @damage = 10
      @abilities = []
    end
  end

  def attack
    "Attacked dealing #{damage} damage"
  end

end

###

require 'spec_helper'
require_relative '../lib/template_method'

describe 'Template Method Pattern' do
  context 'Default Hero' do
    let(:hero) { Hero.new }

    it 'has default damage rating 10' do
      expect(hero.damage).to eq(10)
    end

    it 'can attack' do
      expect(hero.attack).to eq('Attacked dealing 10 damage')
    end
  end

  context 'Warrior' do
    let(:hero) { Hero.new :warrior }

    it 'has default damage rating 15' do
      expect(hero.damage).to eq(15)
    end

    it 'has Strike special ability' do
      expect(hero.abilities).to include(:strike)
    end
  end

  context 'Mage' do
    let(:hero) { Hero.new :mage }

    it 'has default damage rating 7' do
      expect(hero.damage).to eq(7)
    end

    it 'has Magic Arrow special ability' do
      expect(hero.abilities).to include(:magic_arrow)
    end
  end
end
{% endhighlight %}

O problema da implementação atual é que o uso exagerado de `if/elsif/else` para esse tipo de seleção aumenta muito a complexidade e a lógica dentro do método `initialize`, deixando o modelagem engessada e dificultando possíveis novas implementações.

A primeira solução que surge é fazer o uso de herança, onde cada um dos tipos (`Warrior` e `Mage`) tenha sua própria classe que herdem de `Hero`, apenas sobreescrevendo o método `initialize` para atribuir os valores corretos dos atributos `damage` e `abilities`.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :damage, :abilities

  def initialize
    @damage = 10
    @abilities = []
  end

  def attack
    "Attacked dealing #{damage} damage"
  end
end

class Warrior < Hero
  def initialize
    @damage = 15
    @abilities = [:strike]
  end
end

class Mage < Hero
  def initialize
    @damage = 7
    @abilities = [:magic_arrow]
  end
end

###

require 'spec_helper'
require_relative '../lib/template_method'

describe 'Template Method Pattern' do
  context 'Default Hero' do
    let(:hero) { Hero.new }

    it 'has default damage rating 10' do
      expect(hero.damage).to eq(10)
    end

    it 'can attack' do
      expect(hero.attack).to eq('Attacked dealing 10 damage')
    end
  end

  context 'Warrior' do
    let(:hero) { Warrior.new }

    it 'has default damage rating 15' do
      expect(hero.damage).to eq(15)
    end

    it 'has Strike special ability' do
      expect(hero.abilities).to include(:strike)
    end
  end

  context 'Mage' do
    let(:hero) { Mage.new }

    it 'has default damage rating 7' do
      expect(hero.damage).to eq(7)
    end

    it 'has Magic Arrow special ability' do
      expect(hero.abilities).to include(:magic_arrow)
    end
  end
end
{% endhighlight %}

Dessa maneira a modelagem funciona, mas a partir do momento que seja necessário alguma lógica comum entre as classes na inicialização, como por exemplo carregar alguns `initialize_stats` de cada tipo de `Hero` surgem alguns problemas na implementação.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :damage, :abilities

  def initialize
    initialize_stats

    @damage = 10
    @abilities = []
  end

  def attack
    "Attacked dealing #{damage} damage"
  end

  def initialize_stats; end
end

class Warrior < Hero
  def initialize
    @damage = 15
    @abilities = [:strike]
  end
end

class Mage < Hero
  def initialize
    @damage = 7
    @abilities = [:magic_arrow]
  end
end
{% endhighlight %}

Com a implementação acima, todas as subclasses deveriam chamar `initialize_stats` ou chamar o `super` no `initialize` da superclasse e sobreescrever os valores de `damage` e `abilities`, porém essas duas saídas são propensas a erros no futuro.

A alternativa então será utilizar Template Method no método `initialize`. No exemplo o método `initialize` irá servir como template, ou seja, ele será apenas o esqueleto de um algoritmo que será construido de chamadas para outros métodos. Então no caso do exemplo ao invés de os atributos receberem os valores diretamente, essa atribuição será feita através de chamada de métodos que irão retornar os valores aos atributos.

Esses métodos são chamados de `hook methods` e eles definem um valor default e opcionalmente podem ser sobreescritos por subclasses para informar detalhes especifícos da implementação de cada subclasse.

Para o exemplo, podemos remover a atribuição dentro do método `initialize` de cada subclasse e sobreescrever os `hook methods` (`damage_rating` e `occupation_abilities`) em cada uma delas.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :damage, :abilities

  def initialize
    initialize_stats

    @damage    = damage_rating
    @abilities = occupation_abilities
  end

  def attack
    "Attacked dealing #{damage} damage"
  end

  def initialize_stats; end

  def damage_rating
    10
  end

  def occupation_abilities
    []
  end
end

class Warrior < Hero
  def damage_rating
    15
  end

  def occupation_abilities
    [:strike]
  end
end

class Mage < Hero
  def damage_rating
    7
  end

  def occupation_abilities
    [:magic_arrow]
  end
end
{% endhighlight %}

Agora será adicionado um novo comportamento a classe `Hero`, ela terá um método `greet` responsável por uma saudação única de cada tipo de `Hero`.

  + `Warrior`, deverá saudar `Hello, Warrior is ready to fight!`
  + `Mage`, deverá saudar `Hello, Mage is ready to fight!`

Para resolver esse problema, pois todas as classes terão esse comportamento em comum, porém cada uma terá sua própria saudação única, também será utilizado o Template Method.

O método `greet` implementa o Template Method pois ele apenas serve de base para a implementação final em cada classe, que será feita sobreescrevendo o método `unique_greeting_ling`.

O importante aqui, é que como o método `greet` em `Hero` é apenas o Template Method, ele não implementa ação nenhuma, ele apenas faz chamadas para outros métodos, no caso `unique_greeting_ling` que devem ser implementados pelas subclasses, para terem seu comportamento especifíco esperado.

{% highlight ruby %}
# Behavior Pattern

class Hero
  attr_reader :damage, :abilities

  def initialize
    initialize_stats

    @damage    = damage_rating
    @abilities = occupation_abilities
  end

  def attack
    "Attacked dealing #{damage} damage"
  end

  def initialize_stats; end

  def damage_rating
    10
  end

  def occupation_abilities
    []
  end

  def greet
    greeting = ['Hello']
    greeting << unique_greeting_ling
    greeting
  end

  def unique_greeting_ling
    raise 'You must define unique_greeting_ling'
  end
end

class Warrior < Hero
  def damage_rating
    15
  end

  def occupation_abilities
    [:strike]
  end

  def unique_greeting_ling
    'Warrior is ready to fight!'
  end
end

class Mage < Hero
  def damage_rating
    7
  end

  def occupation_abilities
    [:magic_arrow]
  end

  def unique_greeting_ling
    'Mage is ready to fight!'
  end
end

###

require 'spec_helper'
require_relative '../lib/template_method'

describe 'Template Method Pattern' do
  context 'Default Hero' do
    let(:hero) { Hero.new }

    it 'has default damage rating 10' do
      expect(hero.damage).to eq(10)
    end

    it 'can attack' do
      expect(hero.attack).to eq('Attacked dealing 10 damage')
    end

    it 'requires subclass to implemente unique_greeting_ling' do
      expect { hero.greet }.to raise_exception
    end
  end

  context 'Warrior' do
    let(:hero) { Warrior.new }

    it 'has default damage rating 15' do
      expect(hero.damage).to eq(15)
    end

    it 'has Strike special ability' do
      expect(hero.abilities).to include(:strike)
    end

    it 'greet other characters' do
      expect(hero.greet).to eq(['Hello', 'Warrior is ready to fight!'])
    end
  end

  context 'Mage' do
    let(:hero) { Mage.new }

    it 'has default damage rating 7' do
      expect(hero.damage).to eq(7)
    end

    it 'has Magic Arrow special ability' do
      expect(hero.abilities).to include(:magic_arrow)
    end

    it 'greet other characters' do
      expect(hero.greet).to eq(['Hello', 'Mage is ready to fight!'])
    end
  end
end
{% endhighlight %}

Esse é o uso de Template Method em Ruby.

[Refs #1 - Reefpoints](http://reefpoints.dockyard.com/ruby/2013/07/10/design-patterns-template-pattern.html)

[Refs #2 - Edapx](http://edapx.com/2013/10/27/template-pattern/)

[Refs #3 - Wikipedia](http://pt.wikipedia.org/wiki/Template_Method)

Até a próxima!.
