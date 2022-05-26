# *This note is from  [Ruby Programming](https://en.wikibooks.org/wiki/Ruby_Programming)*

# Getting started

# Basic Ruby

直接上ide了，RubyMine

## Hello World

```ruby
puts("Hello World") # this is how ruby comments
puts('Hello World')
=begin
or you can comments like
this
=end
```

## Strings

1. Escape sequences \n \r \t …单双引号转义规则不同
2. `puts()`: same as `System.out.println()`, prints out a newline after the text
3. `print`: same as `System.out.print()`

## Alternate quotes

%q or %Q  + delimiter + str + delimiter  == 'str' or "str"

eg:  `%q(abc) == 'abc'`, `%q{iceRabbit} == "iceRabbit"`

当分隔符是括号( (),<>,{} )的时候不需要转义

感觉很多时候没必要用到这种替代语法，不美观也不易读，也没讲什么特殊的应用场景

## Here documents

```ruby
puts <<heredoc
this is a heredoc
------------
1. Salad mix.
2. Strawberries.*
3. Cereal.
4. Milk.*

* Organic
heredoc
```

<< + identifier

with puts()

```ruby
puts 'Grocery list', '------------', <<GROCERY_LIST, '* Organic'
1. Salad mix.
2. Strawberries.*
3. Cereal.
4. Milk.*

GROCERY_LIST
```

if have to indent the terminator, use <<-

```ruby
puts 'Grocery list', '------------', <<-GROCERY_LIST
    1. Salad mix.
    2. Strawberries.
    3. Cereal.
    4. Milk.
    GROCERY_LIST
```

terminator不要加引号

## ASCII

str.ord

## Encoding

UTF-8 after Ruby 2.0

to change encoding, use magic comment

```ruby
#encoding: UTF-8
#coding: UTF-8
#blah blah coding: US-ASCII
```

```ruby
#encoding: ISO-8859-1
"Olé!".encode("UTF-8") #Valid
"Olé!".encode("US-ASCII") #Error

#encoding: ISO-8859-1
"\u27d0".force_encoding("UTF-8")
```

## Introduction to objects

ruby is a pure object-oriented language

```ruby
str = "test"
str.upcase! # modify the object
str.upcase # return upcase str but does'nt modify the original object
```

## Ruby basics

if, unless(感觉很不适应)

if else 在一行：`if a < 5 then puts "#{a} less than 5" end `, use then

if/unless在ruby里可以作为判断条件放在statement之后:`puts "#{a} less than 5" if a < 5`

while/until

case: 

```ruby
case a
when 0..5
  puts "#{a}: Low"
when 6
  puts "#{a}: Six"
else
  puts "#{a}: Cheese toast!"
end
```



functions

```ruby
def my_function( a )
  puts "Hello, #{a}"
  return a.length
end

len = my_function( "Giraffe" )
puts "My secret word is #{len} characters long"
```

支持参数的默认值和可变长度的参数列表



blocks

Code blocks in Ruby are defined either with the keywords `do..end` or the curly brackets `{..}`

*the block of code behaves like a 'partner' to which the function occasionally hands over control*

*Functions interact with blocks through the `yield`. Every time the function invokes `yield` control passes to the block. It only comes back to the function when the block finishes*

```ruby
def simpleFunction
  yield 
  yield
end

simpleFunction { puts "Hello!" }
#Hello!
#Hello!

def animals
  yield "Tiger"
  yield "Giraffe"
end

animals { |x| puts "Hello, #{x}" }
#Hello, Tiger
#Hello, Giraffe
```

## Data types

1. Constants
   1. Constants start with capital letters. 
   2. You can change the values of constants, but Ruby will give you a warning
2. Symbols