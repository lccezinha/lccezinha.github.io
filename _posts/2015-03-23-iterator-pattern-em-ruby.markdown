---
layout: post
title:  "Design Patterns: Iterator em Ruby"
date:   2015-03-23 22:01:28
categories: ruby design-patterns
---

Por definição o [Iterator](http://pt.wikipedia.org/wiki/Iterator):

> Um iterador é um objeto que permite a um programador examinar um container, particularmente listas. Vários tipos de iteradores são frequentemente fornecidos através de uma interface de container. Apesar da interface e semântica de um determinado iterador serem fixas, iteradores são frequentemente implementados em termos de estruturas subjacentes a uma implementação de container e são muitas vezes ligados intimamente ao container para permitir a semântica operacional do iterador.

Ou seja, a implementação do Iterator em um determinado objeto irá dar a ele a flexibidade necessária para ter comportamento que fazem parte da implementação de coleções.

Para exemplificar o uso de Iterator em Ruby, iremos implementar seu uso através da interação entre as classes `Item` e `Inventory`, onde a classe `Item` possui um atributo chamado `cost` e a classe `Inventory` possu um método `add(item)`

{% highlight ruby %}
# Behavior Pattern
class Inventory
  attr_reader :items

  def initialize
    @items = []
  end

  def add(item)
    @items << item
  end

end

class Item
  attr_accessor :cost

  def initialize
    @cost = 0
  end
end

###

require 'spec_helper'
require_relative '../lib/interator'

describe 'Interator Pattern' do

  describe Inventory do
    it 'add items do collection' do
      item_one = Item.new
      item_one.cost = 20

      item_two = Item.new
      item_two.cost = 10

      inventory = Inventory.new
      inventory.add(item_one)
      inventory.add(item_one)

      expect(inventory.items.size).to eq(2)
    end
  end

  describe Item do
    let(:item) { Item.new }

    it 'has cost' do
      expect(item.cost).to eq(0)
    end
  end

end
{% endhighlight %}

A partir dessa implementação temos a necessidade de iterar sobre os items dentro de `Inventory`, para isso iremos implementar um `External Iterator`

{% highlight ruby %}
# Behavior Pattern

class InventoryIterator

  def initialize(inventory)
    @items = inventory.items
    @index = 0
  end

  def has_next?
    @index < @items.size
  end

  def next
    value = @items[@index]
    @index +=1
    value
  end

end

class Inventory
  attr_reader :items

  def initialize
    @items = []
  end

  def add(item)
    @items << item
  end

end

class Item
  attr_accessor :cost

  def initialize
    @cost = 0
  end
end

###

require 'spec_helper'
require_relative '../lib/iterator'

describe 'Interator Pattern' do

  describe Inventory do
    let(:inventory) { Inventory.new }

    before do
      item_one = Item.new
      item_one.cost = 20

      item_two = Item.new
      item_two.cost = 10

      inventory.add(item_one)
      inventory.add(item_one)
    end

    it 'add items do collection' do
      expect(inventory.items.size).to eq(2)
    end

    it 'can iterated and get total' do
      iterator = InventoryIterator.new(inventory)
      result = 0
      while iterator.has_next?
        result += iterator.next.cost
      end

      expect(result).to eq(30)
    end
  end

  describe Item do
    let(:item) { Item.new }

    it 'has cost' do
      expect(item.cost).to eq(0)
    end
  end

end

{% endhighlight %}

A nova class `InventoryIterator` implementa um tipo de `Iterator` chamado de `External Iterator` ou seja, onde a lógica de percorrer a coleção e iterar sobre seus elementos fica a critério de uma outra classe.

Uma outra maneira de realizar essa implementação é através do uso de `Internal Iterator` onde a própria classe será responsável por implementar a iteração pela lista, para o exemplo, bastar fazer com que a classe `Inventory` implemente um método `each` e que esse delegue seu argumento para um método `each` de uma classe array, nesse caso esse array será o próprio atributo `@items`.

{% highlight ruby %}
# Behavior Pattern

class Inventory
  attr_reader :items

  def initialize
    @items = []
  end

  def add(item)
    @items << item
  end

  def each(&block)
    @items.each(&block)
  end
end

class Item
  attr_accessor :cost

  def initialize
    @cost = 0
  end
end

###

require 'spec_helper'
require_relative '../lib/iterator'

describe 'Interator Pattern' do

  describe Inventory do
    let(:inventory) { Inventory.new }

    before do
      item_one = Item.new
      item_one.cost = 20

      item_two = Item.new
      item_two.cost = 10

      inventory.add(item_one)
      inventory.add(item_two)
    end

    it 'add items do collection' do
      expect(inventory.items.size).to eq(2)
    end

    it 'can iterated and get total' do
      result = 0
      inventory.each { |item| result += item.cost }
      expect(result).to eq(30)
    end
  end

  describe Item do
    let(:item) { Item.new }

    it 'has cost' do
      expect(item.cost).to eq(0)
    end
  end

end

{% endhighlight %}

Porém apenas a implementação do método `each` deixa o uso limitado, para dar uma maior flexibidade e ainda fazer com que a classe `Inventory` tenha métodos como `sort`, `max` e `min`, sem precisar implementá-los, Ruby nos permite expandir o comportamento dessa classe, através de um mixin com o módulo `Enumerable`, onde basta que a classe Iterator inclua o módulo e implemente o método `each`, e a classe que será listada dentro do laço, deverá implementar o operador `<=>` de comparação para garantir que os métodos `sort`, `max` e `min` funcionem corretamente.

{% highlight ruby %}
class Inventory
  include Enumerable

  attr_reader :items

  def initialize
    @items = []
  end

  def add(item)
    @items << item
  end

  def each(&block)
    @items.each(&block)
  end
end

class Item
  attr_accessor :cost

  def initialize
    @cost = 0
  end

  def <=>(other)
    cost <=> other.cost
  end
end

###

require 'spec_helper'
require_relative '../lib/iterator'

describe 'Interator Pattern' do

  describe Inventory do
    let(:inventory) { Inventory.new }

    before do
      item_one = Item.new
      item_one.cost = 20

      item_two = Item.new
      item_two.cost = 10

      inventory.add(item_one)
      inventory.add(item_two)
    end

    it 'add items do collection' do
      expect(inventory.items.size).to eq(2)
    end

    it 'most expansive item' do
      expansive = Item.new
      expansive.cost = 100
      inventory.add(expansive)

      expect(inventory.max).to eq(expansive)
    end

    it 'can iterated and get total' do
      result = inventory.inject(0) { |sum, item| sum + item.cost }
      expect(result).to eq(30)
    end
  end

  describe Item do
    let(:item) { Item.new }

    it 'has cost' do
      expect(item.cost).to eq(0)
    end
  end

end

{% endhighlight %}

Esse foi o uso de Iterator Pattern em Ruby.

[Refs #1 - Chicken Rice Platter](http://chickenriceplatter.github.io/blog/2013/04/07/internal-vs-external-iterators/)

[Refs #2 - Wikipedia](http://pt.wikipedia.org/wiki/Iterator)



