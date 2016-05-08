---
layout: post
title:  "Design Patterns: Adapter em Ruby"
date:   2015-05-07 22:01:28
categories: ruby design-patterns
---

Por definição o [Adapter](http://pt.wikipedia.org/wiki/Adapter):

> Adapter, também conhecido como Wrapper, é um padrão de projeto de software (do inglês design pattern). Este padrão é utilizado para 'adaptar' a interface de uma classe. O Adapter permite que classes com interfaces incompatíveis possam interagir. Adapter permite que um objeto cliente utilize serviços de outros objetos com interfaces diferentes por meio de uma interface única.

Ou seja, o `Adapter` permite sanar as diferenças entre a interface que os objetos precisam para a interface atual que temos. Converte uma interface de uma classe em outra interface que o cliente espera, permite as classes trabalharem juntas de uma forma que não poderia ser feita antes por causa de suas incompatibilidades de interface.

Para o exemplo imagine que temos uma classe `Quest` e uma class `Hero`. A classe `Quest` ao ser instanciada recebe como argumento seu nível de dificuldade, essa dificuldade será utilizada internamente por ela para calcular a quantidade de experiência adquirida por `Hero` ao terminar essa tarefa.

Já a class `Hero` possui os métodos `take_quest` e `finish_quest` responsáveis por iniciar e terminar uma task respectivamente.

{% highlight ruby %}
class Quest
  attr_accessor :difficulty, :hero

  def initialize(difficulty)
    @difficulty = difficulty
    @hero = nil
  end

  def finish
    hero.exp += calculate_exp
  end

  private

  def calculate_exp
    # Regra de da cálculo de experiência = difficulty * 50 / hero.level
    difficulty * 50 / @hero.level
  end

  # complex ...
end

class Hero
  attr_accessor :level, :exp, :quests

  def initialize
    @level  = 1
    @exp    = 0
    @quests = []
  end

  def take_quest(quest)
    @quests << (quest.hero = self)
  end

  def finish_quest(quest)
    quest.finish
    @quests.delete(quest)
  end
end

###

require 'spec_helper'
require_relative '../lib/adapter'

describe 'Adapter Pattern' do

  describe 'Finish quert' do
    it 'rewards hero with xp points' do
      # Regra de da cálculo de experiência = difficulty * 50 / hero.level
      hero = Hero.new
      quest = Quest.new(5)

      hero.take_quest(quest)
      hero.finish_quest(quest)

      expect(hero.exp).to eq(250)
    end
  end

end
{% endhighlight %}

Mas agora digamos que exista uma classe de terceiros ou uma API que possui uma determinada classe que não temos acesso ao código fonte, porém precisamos muito utilizar aquele código, mas ela não possui a mesma interface na qual estamos trabalhando, nesse caso teremos que fazer uma "adaptação" p/ poder utilizá-la, no exemplo essa classe se chama `OldQuest`. Essa classe não recebe nada na inicialização e também não possui o método `finish`.

Para solucionar o problema iremos construir um objeto intermediário `AdapterQuest` que irá conectar `OldQuest` a nossa `Quest`. Essa nova class `AdapterQuest` irá receber um objeto de `OldQuest` e irá ter a mesma interface comum a nossa interface atual, ou seja, ela irá implementar um método `finish`. A class `AdapterQuest` irá receber `OldQuest` na sua instanciação e irá fazer uma adaptação do método `done` de `OldQuest` para o método `finish` da nossa interface comum.

{% highlight ruby %}
class AdapterQuest
  attr_accessor :hero

  def initialize(old_quest, difficulty)
    @old_quest = old_quest
    @old_quest.difficulty = difficulty
    @hero = nil
  end

  def finish
    @hero.exp += @old_quest.done
  end
end

class OldQuest
  attr_accessor :exp, :difficulty

  def initialize
    @difficulty = 3
    @exp        = 10
  end

  def done
    difficulty * exp
  end
end

class Quest
  attr_accessor :difficulty, :hero

  def initialize(difficulty)
    @difficulty = difficulty
    @hero = nil
  end

  def finish
    hero.exp += calculate_exp
  end

  private

  def calculate_exp
    difficulty * 50 / @hero.level
  end

  # complex ...
end

class Hero
  attr_accessor :level, :exp, :quests

  def initialize
    @level  = 1
    @exp    = 0
    @quests = []
  end

  def take_quest(quest)
    @quests << (quest.hero = self)
  end

  def finish_quest(quest)
    quest.finish
    @quests.delete(quest)
  end
end

###

require 'spec_helper'
require_relative '../lib/adapter'

describe 'Adapter Pattern' do

  describe 'Finish Quest' do
    it 'rewards hero with xp points' do
      # difficulty * 50 / hero.level
      hero = Hero.new
      quest = Quest.new(5)

      hero.take_quest(quest)
      hero.finish_quest(quest)

      expect(hero.exp).to eq(250)
    end
  end

  describe 'Finish OLD Quest' do
    it 'rewards hero with xp points' do
      hero = Hero.new
      quest = AdapterQuest.new(OldQuest.new, 5)

      hero.take_quest(quest)
      hero.finish_quest(quest)

      expect(hero.exp).to eq(50)
    end
  end

end
{% endhighlight %}

Essa foi a implementação de uso do `Adapter Pattern` em Ruby.

[Refs #1 - Wikipedia](http://pt.wikipedia.org/wiki/Adapter)

[Refs #2 - Tutorials Point](http://www.tutorialspoint.com/design_pattern/adapter_pattern.htm)

[Refs #3 - Sourcemaking](https://sourcemaking.com/design_patterns/adapter)

Até a próxima!.
