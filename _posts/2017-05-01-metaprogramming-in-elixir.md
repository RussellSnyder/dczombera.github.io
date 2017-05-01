---
layout: post
title:  Metaprogramming in Elixir.
keywords: elixir, metaprogramming 
tags: [elixir, metaprogramming] 
comments: true
---

Metaprogramming is an essential part of Elixir. 
It makes the language extremely extensible and gives the developer a lot of freedom (and responsibility).
The metaprogramming aspect of Elixir in combination with its functional programming paradigm provided me with new and refreshing ways of thinking about and solving problems.
It felt like I discovered a whole new planet where a lot of my old thinking patterns were just not valid anymore. I've got to admit, this new discover felt kind of uncomfortable at first. 
However, once I got used to it I was hooked.  
  
In order to metaprogram in Elixir, you have to understand macros and the abstract syntax tree (AST). After you understood both concepts, you have to become good friends with them. **Really** good friends.  
In a nutshell, the AST consists of three-element tuples, which represent the expressions you write in Elixir.   
For example, adding two numbers looks like this under the hood: 
```elixir
iex> quote do: 4 + 2
{:+, [context: Elixir, import: Kernel], [4, 2]}
```
The first element represents the function call as an atom, the second element is some necessary metadata and the last one is a list of function arguments.

The macro system in Elixir makes extensively use of the AST. In fact, both have a relationship as deep and strong as cat pictures and the Internet.
Furthermore, macros also represent the building blocks of Elixir. So, as you can see, both the AST and macros are pretty freaking important in Elixir's world of metaprogramming.

When you start diving into Elixir, you might notice at one point that there is no `while` loop in the language.
Even worse, after a while tinkering around you might start to miss your old buddy. Never say die! Macros come to save the day:

```elixir
defmodule OldBuddy do
  defmacro while(expression, do: block) do
    quote do
      try do
        for _ <- Stream.cycle([:ok]) do
          if unquote(expression) do
            unquote(block)
          else
            OldBuddy.break
          end
        end
      catch 
        :break -> :ok
      end
    end
  end

  def break, do: throw :break
end
```

With just a few couple of lines, we created the module `OldBuddy`, which defines our dear old friend the `while` loop as a macro.
The probably most difficult part of solving this problem was to find a way to loop infinitely. Here, we took advantage of Elixir's `Stream` module in the form of `Stream.cycle([:ok])]`, which never ends cycling until we explicitly break out of the loop. 

## A MIME type conversation module
To show the power of metaprogramming in Elixir, let's create a module that helps us  finding the proper file extensions given a MIME type and vice versa.  
However, let's do things a little bit differently than usual. Instead of defining the types and extensions within the module, wouldn't it be cool if we could use a plain old text file to save our MIME types and theirs extensions?
And wouldn't it be even cooler if we then  would only have to update the text file to support new MIME types?
Yes, this would be indeed really awesome.  
  
Instead of dreaming about it, let's make it happen! Luckily, Elixir gives us all the tools we need for this kind of endeavour.
To put it more precisely, Elixir provides us with the ability to use external data to create functions. 
Let's see how we can create a MIME type conversation module with just a dozen of lines using an external file to define our functions:
```elixir
defmodule Mime do
  for line <- File.stream!(Path.join([__DIR__, "mimes.txt"]), [], :line) do
    [type, rest] = line |> String.split("\t") |> Enum.map(&String.strip(&1)) 
    extensions = String.split(rest, ~r/,\s?/)

    def exts_from_type(unquote(type)), do: unquote(extensions)
    def type_from_ext(ext) when ext in unquote(extensions), do: unquote(type) 
  end

    def exts_from_type(_type), do: []
    def type_from_ext(_ext), do: nil
    def valid_type?(type), do: exts_from_type(type) |> Enum.any?
end
```
That's it! That's all what it takes to create a module that can handle MIME type conversations. And it's easily extensible.  
One line 2, we first read an external file holding all MIME types we want to support. 
After parsing each individual line to find the MIME type and its extensions on lines 3 and 4, we define the necessary functions to convert the type to its extensions and vice versa.
Let's open iex and convert some types:
```elixir
iex> Mime.exts_from_type("application/javascript")
[".js"]
iex> Mime.exts_from_type("image/jpeg")
[".jpeg", ".jpg"]
iex> Mime.type_from_ext(".html")
"text/html"
iex>Mime.valid_type("image/hologram")
false
```
Our MIME type converter works like a charm! And its creation only took us a dozen of lines. However, this is just the beginning. 
To make it even more powerful, we could inject the generated functions into the context of the caller's module and give him the possibility to extend the support of MIME types on the fly. Let's have a look how this could look like from the perspective of the caller:

```elixir
defmodule CallersModule do
  use Mime, "text/jawaese":      [".jws"],
            "text/wookieespeak": [".wks"]
end
```
To build a feature like this, both [Kernel.use/2](https://hexdocs.pm/elixir/Kernel.html#use/2) and the [@before_compile](https://hexdocs.pm/elixir/Module.html#module-compile-callbacks) hook are pretty handy tools. You should get your hands dirty and give it a shot.  
  
This concludes my small adventure into the world of metaprogramming in Elixir.
As you might have guessed, it barely scratched the tip of the iceberg. However, I really hope I could spark you interest.  
If you want to learn more and deepen your metaprogramming skills, I highly recommend you Chris McCord's book [Metaprogramming Elixir](https://pragprog.com/book/cmelixir/metaprogramming-elixir) (who is also the creator of the [Phoenix web framework](http://www.phoenixframework.org)).
As a matter of fact, this blog post and its examples were heavily inspired by this book (I hope Chris is not to angry about that :O ) and I learnt __a ton__ of valuable stuff while working through it. 
