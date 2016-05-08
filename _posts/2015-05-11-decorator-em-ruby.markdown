---
layout: post
title:  "Design Patterns: Decorator em Ruby"
date:   2015-05-11 22:01:28
categories: ruby design-patterns
---

Por definição [Decorator](http://pt.wikipedia.org/wiki/Decorator)

> Decorator ou wrapper, é um padrão de projeto de software que permite adicionar um comportamento a um objeto já existente em tempo de execução, ou seja, agrega dinamicamente responsabilidades adicionais a um objeto.

Em outras palavras, o `Decorator` permite que em tempo de execução sejam adicionados novos comportamentos a determinada classe.

Para exemplificar, imagine a criação de uma classe `Item`, que tem os atributos `price`, `description` e o método `use`.

{% highlight ruby %}
class Item
  attr_reader :price, :description

  def initialize
    @price = 10
    @description  = 'Item.'
  end

  def use; end
end

###

require 'spec_helper'
require_relative '../lib/decorator'

describe 'Decorator Pattern' do
  let(:common_item) { Item.new }

  context 'common_item' do
    it 'has prince' do
      expect(common_item.price).to eq(10)
    end

    it 'has description' do
      expect(common_item.description).to eq('Item.')
    end

    it 'can be used' do
      expect(common_item).to respond_to(:use)
    end
  end
end
{% endhighlight %}

Agora iremos adicionar um novo tipo de item `MagicItem`, porém esse não será uma nova classe, mas apenas iremos utilizar a já criada classe `Item` e adicionar um parâmetro ao seu seu `initialize` para dizer se é ou não um `MagicItem`.

{% highlight ruby %}
class Item
  attr_reader :price, :description

  def initialize(is_magic = false)
    @price = 10
    @description  = 'Item.'

    if is_magic
      @price *= 3
      @description += 'Magic.'
    end
  end

  def use; end
end

###

require 'spec_helper'
require_relative '../lib/decorator'

describe 'Decorator Pattern' do
  let(:common_item) { Item.new }

  context 'magic item' do
    let(:magic_item) { Item.new true }

    it 'has price 3 times more expansive' do
      expect(magic_item.price).to eq(common_item.price * 3)
    end

    it 'has description' do
      expect(magic_item.description).to eq("#{common_item.description}Magic.")
    end
  end

  context 'common_item' do
    it 'has prince' do
      expect(common_item.price).to eq(10)
    end

    it 'has description' do
      expect(common_item.description).to eq('Item.')
    end

    it 'can be used' do
      expect(common_item).to respond_to(:use)
    end
  end
end
{% endhighlight %}

A princípio, o uso dessa verificação parece "bom", porém agora e se quisermos adicionar um novo tipo de item ? Vamos adicionar um novo tipo, chamado `MasterpieceItem` e para isso vamos usar a mesma técnica, passando um novo parâmetro no método `initialize` de `Item` para fazer a verificação.

{% highlight ruby %}
class Item
  attr_reader :price, :description

  def initialize(is_magic = false, is_masterpiece = false)
    @price = 10
    @description  = 'Item.'

    if is_magic
      @price *= 3
      @description += 'Magic.'
    elsif is_masterpiece
      @price *= 2
      @description += 'Masterpiece.'
    end
  end

  def use; end
end

###

require 'spec_helper'
require_relative '../lib/decorator'

describe 'Decorator Pattern' do
  let(:common_item) { Item.new }

  context 'magic item' do
    let(:magic_item) { Item.new true }

    it 'has price 3 times more expansive' do
      expect(magic_item.price).to eq(common_item.price * 3)
    end

    it 'has description' do
      expect(magic_item.description).to eq("#{common_item.description}Magic.")
    end
  end

  context 'magic item' do
    let(:masterpiece_item) { Item.new false, true  }

    it 'has price 2 times more expansive' do
      expect(masterpiece_item.price).to eq(common_item.price * 2)
    end

    it 'has description' do
      expect(masterpiece_item.description).to eq("#{common_item.description}Masterpiece.")
    end
  end

  context 'common_item' do
    it 'has prince' do
      expect(common_item.price).to eq(10)
    end

    it 'has description' do
      expect(common_item.description).to eq('Item.')
    end

    it 'can be used' do
      expect(common_item).to respond_to(:use)
    end
  end
end
{% endhighlight %}

Olhando agora, aquela implementação que a primeiro momento parecia boa, já não parece tão boa assim, a inicialização do objeto já começa a fica complexa com as condições, embora funcione não está bonito. Vamos PIORAR e criar um novo tipo de `Item`, o `MagicMasterpieceItem`.

{% highlight ruby %}
class Item
  attr_reader :price, :description

  def initialize(is_magic = false, is_masterpiece = false)
    @price = 10
    @description  = 'Item.'

    if is_magic
      @price *= 3
      @description += 'Magic.'
    end
    if is_masterpiece
      @price *= 2
      @description += 'Masterpiece.'
    end
  end

  def use; end
end

###

require 'spec_helper'
require_relative '../lib/decorator'

describe 'Decorator Pattern' do
  let(:common_item) { Item.new }

  context 'magic item' do
    let(:magic_item) { Item.new true }

    it 'has price 3 times more expansive' do
      expect(magic_item.price).to eq(common_item.price * 3)
    end

    it 'has description' do
      expect(magic_item.description).to eq("#{common_item.description}Magic.")
    end
  end

  context 'Masterpiece item' do
    let(:masterpiece_item) { Item.new false, true  }

    it 'has price 2 times more expansive' do
      expect(masterpiece_item.price).to eq(common_item.price * 2)
    end

    it 'has description' do
      expect(masterpiece_item.description).to eq("#{common_item.description}Masterpiece.")
    end
  end

  context 'magic masterpiece item' do
    let(:full_item) { Item.new true, true  }

    it 'has price 6 times more expansive' do
      expect(full_item.price).to eq(common_item.price * 6)
    end

    it 'has description' do
      expect(full_item.description).to eq("#{common_item.description}Magic.Masterpiece.")
    end
  end

  context 'common_item' do
    it 'has prince' do
      expect(common_item.price).to eq(10)
    end

    it 'has description' do
      expect(common_item.description).to eq('Item.')
    end

    it 'can be used' do
      expect(common_item).to respond_to(:use)
    end
  end
end
{% endhighlight%}

A implementação é simples, só trocamos um `elsif` por um `if`, porém o código em si está muito feito e precisa ser melhorado e agora para melhorar isso começaremos a utilizar o `Decorator Pattern`.

Criaremos classes chamadas `MagicItemDecorator` e `MasterpieceItemDecorator`, ambas irão receber `Item` como parâmetro na sua inicialização e irão "decorar" o objeto de acordo com suas necessidades.

Implementando dessa forma, estamos favorecendo a composição de objetos ao invés de herença direta.

{% highlight ruby %}
class MagicItemDecorator
  def initialize(item)
    @item = item
  end

  def price
    @item.price * 3
  end

  def description
    "#{@item.description}Magic."
  end

  def use
    @item.use
  end
end

class MasterpieceItemDecorator
  def initialize(item)
    @item = item
  end

  def price
    @item.price * 2
  end

  def description
    "#{@item.description}Masterpiece."
  end

  def use
    @item.use
  end
end

class Item
  attr_reader :price, :description

  def initialize
    @price = 10
    @description  = 'Item.'
  end

  def use; end
end

###

require 'spec_helper'
require_relative '../lib/decorator'

describe 'Decorator Pattern' do
  let(:common_item) { Item.new }

  context 'magic item' do
    let(:magic_item) { MagicItemDecorator.new common_item }

    it 'has price 3 times more expansive' do
      expect(magic_item.price).to eq(common_item.price * 3)
    end

    it 'has description' do
      expect(magic_item.description).to eq("#{common_item.description}Magic.")
    end
  end

  context 'Masterpiece item' do
    let(:masterpiece_item) { MasterpieceItemDecorator.new common_item }

    it 'has price 2 times more expansive' do
      expect(masterpiece_item.price).to eq(common_item.price * 2)
    end

    it 'has description' do
      expect(masterpiece_item.description).to eq("#{common_item.description}Masterpiece.")
    end
  end

  context 'magic masterpiece item' do
    pending
    let(:full_item) { Item.new true, true  }

    it 'has price 6 times more expansive' do
      expect(full_item.price).to eq(common_item.price * 6)
    end

    it 'has description' do
      expect(full_item.description).to eq("#{common_item.description}Magic.Masterpiece.")
    end
  end

  context 'common_item' do
    it 'has prince' do
      expect(common_item.price).to eq(10)
    end

    it 'has description' do
      expect(common_item.description).to eq('Item.')
    end

    it 'can be used' do
      expect(common_item).to respond_to(:use)
    end
  end
end
{% endhighlight%}

É possível notar que os dois Decorators, possuem comportamentos em comum, como a sua inicialização, e o caso do método `use`. Importante notar que o método `use` dentro de cada `Decorator` não faz nada, além de delegar a sua chamada para o objeto `Item`. Para remover essa duplicação, iremos criar um decorator base, e faze as duas classes decorator herdarem dela.

{% highlight ruby %}
class ItemDecorator
  def initialize(item)
    @item = item
  end

  def use
    @item.use
  end
end

class MagicItemDecorator < ItemDecorator
  def price
    @item.price * 3
  end

  def description
    "#{@item.description}Magic."
  end
end

class MasterpieceItemDecorator < ItemDecorator
  def price
    @item.price * 2
  end

  def description
    "#{@item.description}Masterpiece."
  end
end

class Item
  attr_reader :price, :description

  def initialize
    @price = 10
    @description  = 'Item.'
  end

  def use; end
end
{% endhighlight%}

Como foi dito acima o método `use` dentro do `ItemDecorator` apenas está fazendo uma delegação para o método que realmente implementa o comportamento, então ao invés de reescrever o método, podemos fazer uso do módulo `Forwardable` do Ruby e assim realizar essa chamada.

{% highlight ruby %}
require 'forwardable'

class ItemDecorator
  extend Forwardable

  def_delegator :@item, :use

  def initialize(item)
    @item = item
  end
end
{% endhighlight%}

O método `def_delegator` recebe como argumentos, a instância do objeto no qual se deseja fazer a delegação e quais o métodos ele deverá delegar, para o nosso caso, apenas o método `use` irá bastar.

Usando decorator também é possível criar composição de decorator para criação de novos objetos, como no exemplo abaixo onde `full_item` pode ser composto de duas formas, através do uso de dois decorators.

{% highlight ruby %}
describe 'Decorator Pattern' do
  let(:common_item) { Item.new }

  context 'magic masterpiece item' do
    let(:full_item) { MasterpieceItemDecorator.new(MagicItemDecorator.new(common_item))  }

    it 'has price 6 times more expansive' do
      expect(full_item.price).to eq(common_item.price * 6)
    end

    it 'has description' do
      expect(full_item.description).to eq("#{common_item.description}Magic.Masterpiece.")
    end
  end

  context 'masterpiece magic item' do
    let(:full_item) { MagicItemDecorator.new(MasterpieceItemDecorator.new(common_item))  }

    it 'has price 6 times more expansive' do
      expect(full_item.price).to eq(common_item.price * 6)
    end

    it 'has description' do
      expect(full_item.description).to eq("#{common_item.description}Masterpiece.Magic.")
    end
  end
end
{% endhighlight%}

Usando decorators dessa forma, conseguimos extender o comportamento de uma classe inicial de uma maneira flexível e mais simples.

Essa foi a implementação de uso do `Decorator Pattern` em Ruby.

[Refs #1 - Wikipedia](http://pt.wikipedia.org/wiki/Decorator)

[Refs #2 - Tutorials Point](http://www.tutorialspoint.com/design_pattern/decorator_pattern.htm)

Até a próxima!.
