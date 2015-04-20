---
layout: post
title:  "Design Patterns: Strategy em Ruby"
date:   2015-03-06 22:01:28
categories: ruby design-patterns
comments: true
---

Por definição o [Strategy](http://pt.wikipedia.org/wiki/Strategy):

> O objetivo é representar uma operação a ser realizada sobre os elementos de uma estrutura de objetos. O padrão Strategy permite definir novas operações sem alterar as classes dos elementos sobre os quais opera. Definir uma família de algoritmos e encapsular cada algoritmo como uma classe, permitindo assim que elas possam ser trocados entre si. Este padrão permite que o algoritmo possa variar independentemente dos clientes que o utilizam.

Em outras palavras, define uma família algoritmos e os encapsula de forma isolada, fazendo que eles possam ser trocados entre si, fazendo a o algoritmo independente dos clientes que ela usa. É semelhante ao [Template Method](http://lccezinha.github.io/ruby/design-patters/2015/03/03/template-method-em-ruby.html), porém não faz o uso de herança e sim prefere uma estratégia de composição.

Para iniciar o exemplo, teremos uma classe `Parser` que será responsável por realizar o parser de textos para vários formatos, incialmente `json` e `xml`.

{% highlight ruby %}
# Behavior Pattern

class TextParser
  attr_accessor :text, :parser

  def initialize(text, parser)
    @text   = text
    @parser = parser
  end

  def parse
    if parser == :json
      "{ text: #{text} }"
    elsif parser == :xml
      "<text>#{text}</text>"
    end
  end
end

###

require 'spec_helper'
require_relative '../lib/strategy'

describe 'Strategy' do
  context 'parse text to xml' do
    let(:text) { 'Foo Bar' }
    subject(:parser) { TextParser }

    it '.parser to xml' do
      expect(parser.new('Foo Bar', :xml).parse).to eq("<text>Foo Bar</text>")
    end

    it '.parser to json' do
      expect(parser.new('Foo Bar', :json).parse).to eq("{ text: Foo Bar }")
    end
  end
end

{% endhighlight%}

A solução funciona, mas faz o uso de `ifs` o que não é bom a medida que novos parsers serão adicionados, o que irá ocorrer é que para se adequar ao novo parser será preciso mais um `if` e assim por diante gerando um código de difícil manutenção e pouca flexibilidade.

Mas e para resolver essa solução, qual seria a saída ideal ?

O `Template Pattern`, poderia ser uma solução para resolver o problema, porem tentando evitar o uso de herança, iremos dar preferência ao uso de composição usando o `Strategy Pattern`

Inicialmente para solucionar o problema e dar uma maior flexibilidade ao algoritmo o método `parse` da classe `TextParser` irá apenas delegar uma chamada ao método `parser` de uma classe `JsonParser` ou `XmlParser` que terá a real implementação. E cada uma dessas classes terá sua própria implementação do método `parser`. As classes `JsonParser` e `XmlParser` serão injetadas na inicialização de `TextParser` para deixar o algoritmo flexível.

{% highlight ruby %}
# Behavior Pattern

class TextParser
  attr_accessor :text, :parser

  def initialize(text, parser)
    @text   = text
    @parser = parser
  end

  def parse
    parser.parse(text)
  end
end

class BaseParser
  def parse(text)
    raise 'Must Implement !'
  end
end

class JsonParser < BaseParser
  def parse(text)
    "{ text: #{text} }"
  end
end

class XmlParser < BaseParser
  def parse(text)
    "<text>#{text}</text>"
  end
end

###

require 'spec_helper'
require_relative '../lib/strategy'

describe 'Strategy' do
  context 'parse text to xml' do
    let(:text) { 'Foo Bar' }
    let(:xml_parser) { XmlParser.new }
    let(:json_parser) { JsonParser.new }
    subject(:parser) { TextParser }

    it '.parser to xml' do
      expect(parser.new('Foo Bar', xml_parser).parse).to eq("<text>Foo Bar</text>")
    end

    it '.parser to json' do
      expect(parser.new('Foo Bar', json_parser).parse).to eq("{ text: Foo Bar }")
    end
  end
end
{% endhighlight %}

Dessa maneira, sempre que for preciso adicionar um novo tipo de parser, basta apenas criar uma nova classe que herde de `BaseParser` e implemente o método `parse` da sua própria maneira, para utilizar basta injetá-lo na criação da classe `TextParser`.

Esse foi o uso de Strategy Pattern em Ruby.

[Refs #1 - Reefpoints](http://reefpoints.dockyard.com/2013/07/25/design-patterns-strategy-pattern.html)

[Refs #2 - Edapx](http://edapx.com/2013/11/27/strategy-pattern/)

[Refs #3 - Anderson Leite](https://andersonleiteblog.wordpress.com/2009/10/22/design-patterns-template-method-e-strategy-em-ruby/)

[Refs #4 - Wikipedia](http://pt.wikipedia.org/wiki/Strategy)

[Refs #5 - Open/Closed](https://robots.thoughtbot.com/back-to-basics-solid)

[Refs #5 - Dependency Inversion](https://robots.thoughtbot.com/back-to-basics-solid)


