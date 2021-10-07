---
timeToRead: 3
authors: ["Chip Keyes"]
title: Drink The Elixir
excerpt: 'Think of programming languages as positions on a clock where similar languages are closer together on the clock face. When you go to learn a new language, try to...'
date: 2017-04-25
hero: ''
draft: false

---
> Think of programming languages as positions on a clock where similar languages are closer together on the clock face. When you go to learn a new language, try to choose one that’s at least 60 degrees around the clock face from one you already know.

> — Paraphrased from Dave Thomas,

> Elixir Fountain Episode (2015–09–25)

### What is Elixir?

This is certainly not the first discussion on the Elixir programming language. Nor is it the most comprehensive or revolutionary. I don’t plan to go into language syntax or detailed code examples on how to solve a specific problem with Elixir.

What I would like to discuss in this article is how I’ve been able to grow my understanding of programming by diving into the world of Elixir and learning a language that challenged my current programming paradigm.

I believe the way that Elixir forces you to think about solving problems can carry over to how you solve problems outside of Elixir. I also think it’s a pretty awesome language that is great for solving a variety of problems.

#### Summary of Elixir

To give a brief summary, Elixir is a language built on the Erlang Virtual Machine. It’s a dynamically typed, functional language that compiles to Erlang bytecode.

Erlang was created in 1986 to solve problems in the telecommunications industry. It was mainly focused on handling massive concurrency as well as providing a fault tolerant environment where individual processes could fail without affecting the whole system. Incidentally, these are the same problems that web systems have to handle today.

Elixir was developed by Jose Valim in 2014\. He created it in response to his struggles with concurrency and scalability in Ruby, and he saw that Erlang offered solutions to these problems. Thus Elixir was born with both a more eloquent syntax than the more industrial looking Erlang and the ability to take advantage of Erlang’s powerful ecosystem.

It is a fast and powerful language that is being used in production in many different scenarios. Because it’s built on Erlang, its virtual machine is battle tested and runs most of our telecom software today.

Despite all this, my focus, as I mentioned, is on the principles enforced by Elixir that can carry over even if you’re not using it in your personal or professional projects.

* * *

### What can I take away from Elixir?

#### Immutable Data

The first and one of the most fundamental concepts you learn in Elixir is the idea of data being **immutable**. This is a concept that has become re-popularized by major front end frameworks — like React in combination with Redux. The idea is that when we think about the state of our application, and ultimately the data that our application holds, we should think of it as being unchangeable.

More specifically, if we create a piece of data in our application, anytime we reference that piece of data we should expect that it’s the same as when we initialized it. The benefit this has over languages that don’t enforce immutability is that when a variable references a piece of data, we can be confident that it will always return the same value. In contrast, if data is mutable and one variable reference alters the data, other variables will also reflect those changes.

In this first example, you can see that in a mutable programming environment, changing data in a variable that is referenced by other variables causes all variables to reference the new data. In most cases, this side effect is unintended and can cause run-time errors.

```js
// Javascript example showing how mutable data can have side effects
var firstVariable = [1,2,3,4];
var secondVariable = firstVariable;

secondVariable[2] = 10000;

console.log(firstVariable);

// OUTPUT
[ 1, 2, 1000, 4 ]
```
In Elixir, data is immutable so even an attempt to manipulate data in this way causes a compiler error and suggests the correct function for accessing and altering data.

```elixir
# Elixir example show how immutability prevents side effects

firstVariable = [1,2,3,4]

secondVariable = firstVariable

secondVariable[3] = 1000

def foo do
end

** (CompileError) iex:3: cannot invoke remote function Access.get/2 inside match
```

Thinking about data as immutable — even in a mutable language like Javascript — protects you from unanticipated errors. **Allowing your application to mutate data willy-nilly is a sure way to make code hard to test and errors hard to debug.**

> One of my favorite Javascript libraries that helps enforce this concept is [Immutable.js](https://facebook.github.io/immutable-js/)

#### Stateless Functions

So you must be thinking, “But wait, my application needs to change its state over its lifecycle…how can I do that if data is immutable!?”.

I’m glad you asked.

The second concept that follows on the heels of immutable data is the idea that application state (data) should only be mutated using pure stateless functions. A pure function refers to a function that takes everything it needs as a parameter and returns a **new copy** of data as its output.

This snippet shows an example of this concept in Javascript. The first section shows a standard implementation of a Javascript class that provides a constructor and a function to manipulate the data.

The second section shows how you can accomplish the same functionality by creating a module of functions that act on lists and simply return copies of the data that have been manipulated.

```js
// SECTION 1
// Example of an impure function within a javascript class
function List() {
  this.list = []
  
  this.addToList = function(item) {
    this.list.push(item);
  }
}

var newList = new List();
console.log(newList.list) // Output = [];
newList.addToList(42);
console.log(newList.list) // Output = [42];

// SECTION 2
// Example of a list module that manipulates data via pure functions
function ListModule = function() {
  this.new = function() {
    return [];
  }
  
  this.addToList = function(list, item) {
    return list.concat(item);
  }
}

var PureModule = new ListModule();
var newList = PureModule.new();
console.log(newList); // Output = [];

var copyList = PureModule.addToList(newList, 42);
console.log(newList); // Output = [];
console.log(copyList); // Output = [42];
```

The module is much clearer because it’s apparent which data we want to manipulate and how we intend to manipulate. And it comes with the benefit that **we will never accidentally change data that another part of the application is expecting to stay the same.**

Also, these functions are **much easier to test** (especially as the application gets more complex) because they don’t rely on any external data.

Current Data in --> Function Copies and Changes Data --> New Data out.

**No matter how many times you call it with the same data, you always get the same output**. Simple as that.

One of the great implications of this is that you can start thinking of your data manipulations as a set of messages that you can dispatch.

Essentially, you know your current state and the message you want to pass and your message receiver knows exactly how to change the data based on the message…no need to worry about side effects.

This is a much bigger topic, which I’m not going to go into here. Elixir takes this to heart by encapsulating data in concurrent processes and using message passing to make changes to a fully isolated state. **The only way to manipulate data in an Elixir process is to send it a message.**

> One of the best examples of this in Javascript is [Redux](http://redux.js.org/) which acts as a state manager for React applications. Changes to your application state are funneled through your Redux reducer which is just a collection of pure functions that return a new copy of the state, altered based on the message you sent.

#### Data Transformation

The final concept I wanted to bring up is really a result of the first two. When we start enforcing immutable data and then manipulate our data via pure stateless functions, it allows us to think about the application data flow in terms of what transformations need to occur to get to the final state.

This is a much more straightforward way to think about the application flow as opposed to serializing data into the correct data encapsulating class, then determining all the methods that may need to be called on the data object, then making sure that object is passed around correctly and only manipulated in the prescribed ways.

Instead, we look at what data is being passed in, determine what we want our data to look like at the end, and then engineer all the pure functions along the way that will result in the desired data output.

Again, Elixir takes this to an extreme by even including a pipe operator which pipes the result of the previous function into the first parameter of the next function.

```elixir

# Simple example of a module that has a two private functions and uses those to take a range of integers
# and output the sum of all the even numbers.

defmodule Play do
  defp is_odd(num) do
    case rem(num, 2) do
      0 -> false
      _ -> true
    end
  end

  defp reducer(current, accumulation) do
      accumulation + current
  end

  def main(range) do
    range
    |> Enum.map(&(&1*3))
    |> Enum.filter(&(is_odd(&1)))
    |> Enum.reduce(&reducer/2)
  end
end
```
In this really basic example, we can think about the main problem as “I want to give you a list of numbers, and I want you to return the sum of all the even numbers after you’ve multiplied all of them by three.” So, data comes in as a range of numbers.

The steps of transformations we need to take are as follows:

1.  Multiply every number by 3 (line 18)
1.  Filter out all the odd numbers (line 19 --> lines 5–10)
1.  Sum all the remaining numbers together and return to caller (line 20 --> lines 12–14)

While this is a really basic example, you can see how this method of thinking can simplify even the more complex application workflows.

1.  Data comes in i.e. from an HTTP request
1.  Server maps the data to the correct request handler
1.  Handler passes data through one or more transformation functions
1.  Insert as many transformations as you want
1.  Server responds with new and/or updated data

No matter what steps we have in a data flow, we can always boil it down to a pipeline of data transformations. Then our entire application becomes a large function that takes input, manipulates data and returns a new copy of the altered data (sound familiar?).

Sure, this is an oversimplification, but if we start out building the simple logic this way and slowly work our way up, then the end product will be that much easier to reason about, and we’ll have time to focus on the more complex problems.

* * *

### A word of caution

Not every concept in Elixir is fully portable into other programming languages, and I’m certainly not making that claim. These concepts are ones that are enforced in Elixir but are portable (to an extent) outside of the world of Elixir. **When you decide to work in a specific language, be it Javascript, Elixir, Java, or C++, you agree to a contract between you and that language.**

Sometimes there is overlap between languages, sometimes not. Be careful that you don’t try to force a concept from one language into another language that it doesn’t fit in. _Something about a square peg and a round hole comes to mind here._

* * *

**One Final Note**

I write about these concepts because they’re ones that I’ve personally benefited from while developing in Javascript. I find that when I adhere to these principles, it makes my programs easy to approach, easy to reason about, easy to test, and more predictable during runtime.

Elixir has a lot more to offer regarding concurrency and scalability that was outside the scope of this article (although may well be within the scope of a future article…*cough*…OTP…*cough, cough*). I hope this teaser encourages you to go out and start exploring more about the language, in addition, challenging your current programming approaches.

* * *

### Resources

*   **Elixir Language Homepage/Docs — **[http://elixir-lang.org/](http://elixir-lang.org/)
*   **Elixir Fountain Podcast —** [http://elixirfountain.com/](http://elixirfountain.com/)
*   **Programming Elixir — **[https://pragprog.com/book/elixir/programming-elixir](https://pragprog.com/book/elixir/programming-elixir)
*   **Little Elixir & OTP Guidebook — **[https://www.manning.com/books/the-little-elixir-and-otp-guidebook](https://www.manning.com/books/the-little-elixir-and-otp-guidebook)
*   **Elixir in Action — **[https://www.manning.com/books/elixir-in-action](https://www.manning.com/books/elixir-in-action)

[sc name="post-signature"]