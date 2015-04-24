---
layout: post
title:  "Design Patterns: Builder em Ruby"
date:   2015-04-23 22:01:28
categories: ruby design-patterns
comments: true
---

Por definição o [Builder](http://pt.wikipedia.org/wiki/Builder):

> Builder é um padrão de projeto que permite a separação da construção de um objeto complexo da sua representação, de forma que o mesmo processo de construção possa criar diferentes representações.

Em outras palavras o que o `Builder` faz é facilitar a construção de objetos que tenham muitas dependências relacionadas a sua criação, também tem o objeto de esconder os detalhes dessa criação e validar se a criação desse objeto está construindo um objeto válido ou não.

Para o exemplo, vamos utilizar um classe `Computer` que deverá ser composta de vários outros componentes, como `Motherboard` e uma série de `Drive`.

{% highlight ruby %}
class Computer
  attr_accessor :display, :motherboard, :drives

  def initialize(display, motherboard, drives)
    @display     = display
    @motherboard = motherboard
    @drives      = drives
  end
end

class CPU; end
class BasicCPU < CPU; end
class TurboCPU < CPU; end

class Motherboard
  attr_accessor :cpu, :memory_size

  def initialize(cpu = BasicCPU.new, memory_size = 1000)
    @cpu = cpu
    @memory_size = memory_size
  end
end

class Drive
  attr_accessor :type, :size

  def initialize(type, size)
    @size = size
    @type = type
  end
end
{% endhighlight %}

Esses são todos os componentes básicos da classe para criar um `Computer`, porém a criação desse objeto se torna complexa ou até chata de se fazer:

{% highlight ruby %}
motherboard = Motherboard.new(BasicCPU.new, 1000)

drives = []
drives << Drive.new(:hard_drive, 1000)
drives << Drive.new(:cd, 1000)
drives << Drive.new(:dvd, 1000)

computer = Computer.new(:lcd, motherboard, drives)
{% endhighlight %}

Para cada vez que se precisa criar um objeto `Computer` será preciso toda essa configuração antes, em casos onde esse cenário ocorre, a implementação de uma classe `Builder` é o ideal, por que além de facilitar a criação, irá esconder a criação dos componentes e encapsular em apenas uma classe.

A classe `Builder` deve ter uma interface que permite a criação de objeto parte por parte.

{% highlight ruby %}
class ComputerBuilder
  attr_reader :computer

  def initialize
    @computer = Computer.new
  end

  def basic_computer
    @computer.motherboard.cpu = BasicCPU.new
  end

  def display=(display)
    @computer.display = display
  end

  def memory_size=(size)
    @computer.memory_size = memory_size
  end

  def add_cd
    @computer.drives << Drive.new(:cd, 1000)
  end

  def add_dvd
    @computer.drives << Drive.new(:dvd, 1000)
  end

  def computer
    @computer
  end
end

###

describe 'Builder Pattern' do
  it 'creates a computer with builder' do
    builder  = ComputerBuilder.new
    builder.basic_computer
    builder.display = :lcd
    builder.memory_size = 1000
    builder.add_cd
    computer = builder.computer

    expect(computer.drives.map(&:type)).to include(:cd)
    expect(computer.display).to eq(:lcd)
    expect(computer.memory_size).to eq(1000)
    expect(computer.motherboard.cpu).to be_a(BasicCPU)
  end
end
{% endhighlight %}

Com essa implementação, o `Builder` esconde toda a lógica de criação de um objeto `Computer` e cria uma interface simples e obtenção do objeto criado, através do método `computer`.

Outra importante utilização do `Builder` é que ele pode validar a criação do objeto, nesse caso bastaria criar as validações do objeto antes de ele retornar o objeto final criado, para o nosso exemplo as validações seriam feitas no método `computer`, antes de ele retornar a instância.

{% highlight ruby %}
class ComputerBuilder
  attr_reader :computer

  # todos os outros métodos

  def computer
    raise 'Invalid' unless valid?
    @computer
  end

  private

  def valid?
    @computer.display.present? && @computer.motherboard.cpu.present? && @computer.drives.any?
  end
end

###

describe 'Builder Pattern' do
  # outros specs

  it 'raise error when invalid' do
    builder  = ComputerBuilder.new

    expect { builder.computer }.to raise_exception
  end
end
{% endhighlight %}

Essa foi a implementação de uso do `Builder Pattern` em Ruby.

[Refs #1 - Wikipedia](http://pt.wikipedia.org/wiki/Builder)

Até a próxima!.
