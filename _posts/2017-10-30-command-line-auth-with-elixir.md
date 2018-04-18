---
layout: post
title: "Command Line Interface Auth with Elixir / Erlang"
description: "Auth0 is not just good for securing web apps, we can secure non-interactive clients just as easily!  We're going to build and authenticate a CLI in Elixir, with Auth0."
date: 2017-10-30 08:30
---

**TL;DR:**  Part **one** of a **three** part series on non-interactive Auth0 clients.  Today, we're going to build a Command Line Interface in Elixir that authenticates a user, using the _Resource Owner Password Grant_  flow, demonstrating using Auth0 in a **non**-web app manner.   The completed app can be [found on GitHub](https://github.com/rbnpercy/padlock).


## Introduction
**Elixir is my favourite language**.  A bold way to open an article (pun intended), but bear with me on this one.  As a polyglot dev, I've dabbled in pretty much all of the major languages, and naturally, I now know what I like and what I don't like.

My first _real_ programming language was Ruby, and it still has a place in my heart.  When I took an interest in lower-level systems, distribution, concurrency, and parallelism, naturally (but reluctantly) I realised I had to look elsewhere.  There is a special place in hell reserved for threading in Ruby.

When I moved into the Database industry, I quickly became acquainted with [Erlang](http://learnyousomeerlang.com/), and I instantly bonded with it.  The idea of a _functional_ programming language was at the time, more than a little bit alien to me.  I had been using object-oriented languages almost exclusively, and they were all I knew.  Although the syntax of Erlang was more difficult to get to grips with than Ruby, I still very much enjoyed my time learning it, and the sheer **power** that it delivered was absolutely astounding to me.  (And still is.)

One of the things I always **loved** about Ruby was the friendly syntax.  I loved how easy it was to learn, how easy it was to read and how easy it was to write.  I always wished that Erlang had a more friendly, and readable syntax.  So, you can imagine my unrequited joy and excitement when in 2014 (I know I was late to the party), I got introduced to [Elixir](https://elixir-lang.org/).  I got introduced through a colleague who described it as:

> "Ruby on the Erlang VM."

A description that excited me for sure, but was in reality, not _quite_ the case.

Elixir is a _functional,_ dynamically-typed language, that does indeed sit atop of the Erlang VM.  Elixir is fundamentally different to Ruby, not only in its paradigm but also in the fact that it was built with the aims of delivering low-latency, fault tolerant, distributed systems.  Elixir allows for huge vertical scaling, as it runs its code inside incredibly lightweight threads of execution (similar to [_Goroutines_ in Golang,](https://tour.golang.org/concurrency/1)  and [threads in Crystal](https://auth0.com/blog/an-introduction-to-crystal-lang/).) These threads are all entirely isolated, to allow for independent garbage collection and reduction of system-wide pauses.  This uses all of the machine's resources more efficiently.  Elixir also allows for massive _horizontal_ scaling, as those threads can communicate with other process threads running on separate machines.

As Elixir is based on distribution and concurrency, it relies on its _functional_ programming paradigms to ensure thread safety and allow itself to run effectively under its expected constraints.  Threading is **infinitely** more complex in object-oriented systems, and works a remarkable amount better inside functional systems like Elixir and Erlang.

When my colleague introduced Elixir to me, he was correct **almost only** in the terms of _syntax._  Elixir takes its syntax inspiration rather heavily from Ruby to ensure ease of use and readability.  Take, for example, the following code in Elixir then Ruby:

```ruby
defmodule Times do
  def double(n) do
    n * 2
  end

  def triple(n), do: n * 3

  def quad(n) do
    double(n) * 2
  end
end
```
And in Ruby:
```ruby
class Times
  def double(n)
    n * 2
  end

  def triple(n)
    n * 3
  end

  def quad(n)
    double(n * 2)
  end
end
```

As you can see, they are both very similar, the biggest difference being evident in the _Class_ definition in Ruby, and the _Module_ definition in Elixir.  The idea of classes in Ruby belongs in an object-oriented environment and doesn't apply in Elixir.  In _Elixir_, we define our functions, and namespace them inside _Modules_.

There is excellent tooling built-in to Elixir, that makes development a true joy.  The testing framework `ExUnit` makes writing detailed test suites simple, and very familiar to any developers coming from a Ruby/Rails or a Golang background.  On the same note, if you do come from a Ruby background; [IEx (Interactive Elixir Shell)](https://hexdocs.pm/iex/) offers a very similar programming experience to _IRB_ in Ruby.  Also on offer is an excellent build tool named [_Mix,_](https://hexdocs.pm/mix/Mix.html) which integrates nicely with the [Hex package manager.](https://hex.pm/)  Hex provides package management, dependency resolution, and the ability to remotely fetch packages; a similar concept to [RubyGems](https://rubygems.org/) and [NPM](https://www.npmjs.com/).

Elixir also has fantastic advanced features, that I've had the joy of using in past production systems.  Meta-programming protocols such as Macros that allow us to inject code into our application at compile-time, and generate functions on the fly.  Joining _Macros_ in the awesome advanced Elixir features, are **_protocols_** (which allows you to add behaviours to existing modules), and **_use_** (which allows you to add capabilities to a module).  One of my favourite native features of Elixir, is the Interoperability with the entire ecosystem of Erlang functionality, including the epic, [advanced OTP features](https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html).

If you would like to learn Elixir, I would recommend the [Programming Elixir](https://pragprog.com/book/elixir/programming-elixir) book available from the Pragmatic Bookshelf.  I purchased this book a few years ago and it really did aid my Elixir education, making the learning process simple, and offering practical guides and programming tasks that show-off the features of the language really rather well.


## Overview of our Application
[The completed sample app GitHub repo can be found here.](https://github.com/rbnpercy/padlock)

As mentioned previously, the idea behind the app we're going to build today revolves around:

 - Using Auth0 in a **non-interactive** (non-web-app) way,
 - Utilising Auth0 in an Elixir / Erlang ecosystem,
 - Following the **_Resource Owner Password Grant_** authentication flow.

In this tutorial, we will build a simple Elixir app that allows a user to authenticate themselves through a CLI,  and view account details of themselves in a command line environment, utilising the [Resource Owner Password Grant flow with Auth0](https://auth0.com/docs/api-auth/grant/password).

>Highly trusted applications can use this flow to access APIs. In this case, the end-user is asked to fill in credentials (username/password), typically using an interactive form. This information is sent to the backend and from there to Auth0.

The reason we are using this authorisation flow, and the main reason for the article, is that we are not offering an interactive login form, such as we would offer in a traditional web application.  As we are not offering a standard log in flow, we need to utilise one of Auth0's alternative authorisation strategies.  The _Resource Owner Password Grant_ suits our needs, as it allows us to still take a username and password, but without the interactivity of a web client.   This grant type can eliminate the need for the client to store the user credentials for future use, by exchanging the credentials with a long-lived access token or refresh token.

In this non-interactive auth article, we are still imagining we have an end-user that needs to log in.  In the next article in the series, we will focus on the [Client Credentials Grant,](https://auth0.com/docs/api-auth/grant/client-credentials) in which our backend service will take the place of our user, and require authentication for itself.

I often speak to developers who believe that Auth0 can only really be used in a web application context, and this simply _isn't_ the case.  In reality, there are many  authorisation flows that can be utilised for a huge range of use cases.  In today's case, we're going to follow the aforementioned [Resource Owner Password Grant flow](https://auth0.com/docs/api-auth/grant/password).

The flow for our authentication will be as follows:

- The end user enters the credentials into the CLI application.
- The client forwards the credentials to Auth0.
- Auth0 validates the information and returns an access_token, and optionally a refresh_token.
- Our application decodes the access_token and utilises it.

![Resource Owner Password Grant flow](https://cdn.auth0.com/blog/elixir-cmd/password-grant.png)


## Pre-code Auth0 Setup
Before we write any code, there are a few things we need to set up in our [Auth0 Management Dashboard](https://manage.auth0.com/#/).  Firstly, create a New Client with the type of **_Non Interactive Client._**  Click into the _Settings_ tab of your created client, and scroll down to the _Advanced Settings_.  Expand those settings and click into the _Grant Types_ tab.  Ensure that the **_Password_** grant box is selected, then save your settings. This step is highly important, as here we are allowing our client to follow the Client Password Access Grant flow.

Next, click into the **_APIs_** section, and create an API.  I gave mine the same name as my Client, as it's not hugely important.  There are two things to note here:  Firstly, the _Identifier_ should be a URL but does not need to be publicly available.  For my API, I used `https://padlockapp.rbin`, even though such a domain doesn't exist.  Secondly, the _Signing Algorithm_ should be set to `HS256`.  After creation, click into the **_Non Interactive Clients_** tab of the API settings.  Scroll down the list of Clients until you find the relevant Client that you created for this app.  Switch the toggle to _Authorised_, so that our Client will be able to exchange information with our API.  When creating an _API_ we would usually define _Scopes_, but for this application, there is no need.

Now that we have a Client setup, and an API for it to communicate with, we need a data source for our registered users.  Click into the **_Connections > Database_** section from the left-hand navigation, and _create a new DB connection._  I named mine `non-interactive`, but you may call yours whatever you wish.  Once created, click into the **_Clients_** tab in the database settings.  We need to authorise our Client to connect to this database, so scroll down to locate your new Client, and just the same as with the API, switch the toggle to the active position.

Lastly, now that we have our Auth0 connections setup, we need to set our `default_directory` value for _Password Grant_ exchanges.  Head into your [Tenant Settings](https://manage.auth0.com/#/tenant), and scroll down to the _API Authorization Settings_.  In the **_Default Directory_** setting, enter the name of the DB Connection you created for this application i.e. `non-interactive`.  Once again, this is an important step as the _Password Grant_ relies on a connection capable of authenticating users via username and password. In order to indicate which connection the _Password Grant_ should use, we need to have set the value of the `default_directory` tenant setting.

Understandably, when we use Non-Interactive Clients, there are a few more settings than with standard clients.  The reasons for this lie in security and isolation.  Now that we have the above settings taken care of, we can get on with coding our CLI app.


## Building our CLI App (Padlock)

The completed sample application for this article can be found in [this GitHub repo.](https://github.com/rbnpercy/padlock)

Assuming one has Erlang and Elixir installed (`brew install erlang, brew install elixir-lang` if not), we can go ahead and scaffold the application.  Run:

~~~ bash
mix new padlock
~~~

In doing this, Mix has created an application skeleton that we can immediately set to work on.  Head into the app directory and open the `mix.exs` file.  This file will allow us to enter our app dependencies, and tell Elixir exactly how to run our application.  Elixir applications, unlike other languages, are built to run in _Supervised_ conditions on the OTP platform, and aren't often built as executable binary command-line applications.  Because of this, Elixir apps usually wouldn't have a `main()` function to run.  As we _are_ creating a command-line app, we need to tell Elixir that there will be a `main()` func, and that it should be called at runtime.

In the `project()` function, we need to set an extra attribute.  Add in:

~~~ elixir
escript: [main_module: Padlock],
~~~

This line is telling the _escript_ runtime to compile the app to a binary file, running the `main()` function found in the `Padlock` module.  The next thing to do is to add in our application dependencies.  Update the `deps()` func to reflect the following:

``` elixir
defp deps do
  [
    {:table_rex, "~> 0.10"},
    {:httpoison, "~> 0.13"},
    {:poison, "~> 3.1"},
    {:joken, "~> 1.5"}
  ]
end
```

Finally, have the `application()` func reflect this:

~~~ elixir
def application do
  [
    extra_applications: [:logger, :table_rex]
  ]
end
~~~

The dependencies we have called include a CLI table formatter, a HTTP library to call Auth0's API, a JSON library, and a JWT library.  Once inputted, save the `mix.exs` file, and run the following command to fetch and install the app dependencies:

~~~ bash
mix deps.get
~~~

So with the application setup, and dependencies installed, we can start writing some Elixir.  Open up the main `lib/padlock.ex` file, and add the following function:

``` elixir
@doc """
  Main function to run when app is called.
"""
def main(args \\ []) do
  args |> parse_args |> process_args
end
```

By defining this `main()` function, our application now has something to run when executed from it's compiled binary.  The function is going to accept an array of arguments.  These arguments are the command-line arguments (flags) that we will call the binary with.  One thing to note in this function is the use of Elixir's **excellent** function chaining ability.  We are taking the arguments supplied, passing them onto a `parse_args()` function, then passing the result onto a `process_args()` function.  Both of which we are about to define.

Let's define our `parse_args()` function:

``` elixir
@doc """
  Use Elixir's OptionParser to parse the arguments.
"""
def parse_args(args) do
  opts = OptionParser.parse(args)

  case opts do
    {[email: email, password: password], _, _} -> [email, password]
    _ -> :no_args
  end
end
```

Elixir has a built-in `OptionParser` module that we can utilise to deal with our command-line arguments.  We are using a _case_ statement to deal with the function being called either with arguments, or without.  The first scenario in the case statement will deal with an email address and password supplied, extracting and returning them.  By using the `_ ->` case, we can catch any other scenarios, without having to write them all out.  We only want this function to work if an email address and password is supplied.  If incorrect arguments are supplied, the function will return a `:no_args` atom.

Finally, we need to define our `process_args()` func:

``` elixir
defp process_args([email, password]) do
  Auth.process_auth(email, password)
end
defp process_args(:no_args) do
  IO.puts "Insufficient arguments supplied..."
end
```

If you are new to Elixir, the first thing you may notice here, is that we are defining the same function twice.  In Elixir, we do this almost as a control flow.  The function body changes depending on the arguments supplied.  In our funcs, if an email and password are supplied, we are calling `Auth.process_auth()` (which we will define shortly), and if incorrect arguments are supplied, we are returning an error.

Before moving onto building the Auth module, add the following to the top of the file, just beneath the module declaration:

``` elixir
defmodule Padlock do
  @moduledoc """
    CLI client, takes in arguments of Username and Password, and calls Auth0.
  """

  alias Padlock.Auth
```

By using the _alias_ command, we can include the Auth module we are about to create, allowing us to call `Auth.func()` natively.

Now then, create the file for our Auth module.  Create the folder `lib/padlock`, where the top-level `padlock.ex` file is, and create the file `lib/padlock/auth.ex`.  Initially, have it reflect this:

``` elixir
defmodule Padlock.Auth do
  @moduledoc """
    Padlock.Auth handles the call to Auth0 with the inputted User.
  """
  import Joken

  @doc """
    Function chain the process together of signing-in, gathering and verifying the JWT.
  """
  def process_auth(email, password) do
    token = do_auth(email, password)
    |> get_id_token
    |> verify_token

    token.claims
    |> Padlock.Output.output_user_table
  end
```

The first thing of note here is the import statement of `Joken`.  This is a JWT library and was one of the dependencies we installed in our `mix.exs` file.  Inside our `process_auth()` func, we have chained together three further functions: `do_auth(), get_id_token() and verify_token()`.  The result of which will be assigned to the variable `token`.  After being returned from our processing functions, the verified and decoded JWT will then have it's `claims` extracted, and will be passed to our table outputting function. (Again, which we will define shortly.)

We will next define the first func called in the above chain; `do_auth()`.  This is the function that will take the supplied email address and password, and send it to Auth0's API to receive the _Password Grant_.

``` elixir
@doc """
  Make the initial call to Auth0 Authentication API with Credentials supplied in Args.
"""
def do_auth(email, password) do
  body = Poison.encode!(%{"grant_type" => "password", "username" => "#{email}", "password" => "#{password}", "audience" => "[API AUDIENCE]", "scope" => "openid profile", "client_id" => "[CLIENT_ID]", "client_secret" => "[CLIENT_SECRET]"})

  url = "https://[YOUR_DOMAIN].auth0.com/oauth/token"
  headers = [{"Content-type", "application/json"}]
  HTTPoison.post(url, body, headers, [])
end
```

In this function, we are using `Poison` to encode a natural Elixir map of data into a JSON blob that we can send to the Auth0 API.  In the Elixir data map, we have set the `grant_type` to `password`.  This is the key piece of information that will let Auth0 know about our requested authentication strategy.  The `username` and `password` fields are being string interpolated from the arguments supplied when the function is called.  The `audience` item is the _Identifier_ set when you created your _API_ in the Auth0 Dashboard.  The next setting, `scope`, is a list of the [scopes](https://auth0.com/docs/scopes) we want to include in our authentication call, each separated by a whitespace.  In our case, we want the user's profile returned, so we have set `openid profile`.  The next settings are the familiar `client_id` and `client_secret` from the Client settings in our Auth0 Dashboard.

We then set the `url` to which our request is to be posted, set the `content-type` header to accept _JSON_, and used the [_HTTPoison_](https://hexdocs.pm/httpoison/HTTPoison.html) library to POST our request to the Auth0 API.

The second func in the `process_auth` function chain is `get_id_token()`.  Define it as follows:

``` elixir
@doc """
  Strip the ID Token from the JSON response.
"""
def get_id_token(response) do
  {:ok, res} = response
  r = Poison.decode!(res.body)
  if r["id_token"] do
    r["id_token"]
  else
    raise "Incorrect Username / Password"
  end
end
```

This function takes as an argument the JSON response sent back from Auth0.  The response we get back is an Elixir data Map with the JSON blob contained within.  Therefore, we must use Elixir's awesome [Pattern Matching](https://elixir-lang.org/getting-started/pattern-matching.html) defining a tuple `{:ok, res}` to assign the data to.  We then use the [Poison JSON library](https://github.com/devinus/poison) to decode the response body.  As we already know that the response will only contain an `id_token` if the authentication was successful, we can add in some control flow here, to prevent an incorrect username / password response going any further.  If the authentication was all good, and an `id_token` was present, we simply return this to be passed along the chain to the next function.

The next step of the function chain is the `verify_token()` function.  It is in this function that our earlier `import Joken` statement will be utilised:

``` elixir
@doc """
  Verify the JWT from Auth0.
"""
def verify_token(tkn) do
  tkn
  |> token
  |> with_signer(hs256("[CLIENT_SECRET]"))
  |> verify
end
```

Our `verify_token()` func takes as an argument the encoded `id_token` value from the JWT returned from Auth0.  The [_Joken_](https://github.com/bryanjos/joken) library allows us to input the token, and verify it against our secret key, using the HS256 protocol.  In our case, the signing _secret key_ is actually the _Client Secret_ from our Auth0 application Client.  The very same _Client Secret_ that we submitted in the original authentication API call.  This function returns the decoded and verified JSON Web Token information.  So, with this final function in the chain, we have verified our JWT and have returned the decoded information contained within it.

Finally, let's focus on the last part of our initial `process_auth()` function chain.  

``` ruby
token.claims
|> Padlock.Output.output_user_table
```

As we assigned the return value of our function chain to the `token` variable, we can use the native Map access functionality of Elixir to grab just the `claims` section of the returned, decoded JWT.  This section contains the user details that we want.  We then send the `claims` section to another function: `Padlock.Output.output_user_table()`.

We need to create this module, and the contained function.  Create the file `lib/padlock/output.ex`.  This module and function will take care of outputting the user data into the table format to be displayed in the user's command line.  Once the file has been created, have it reflect the following:

``` elixir
defmodule Padlock.Output do

  @doc """
    Generate a table to output and display the user's details in the CLI.
  """
  def output_user_table(usr) do
    user = usr["name"]
    header = ["Email Address", "Nickname", "ID"]
    rows = [
      [usr["name"], usr["nickname"], usr["sub"]]
    ]

    TableRex.quick_render!(rows, header, user)
    |> IO.puts
  end
end
```

This function takes in an Elixir data map as its argument, and can access attributes of that map within the function body.  For example, we can extract the user's name by accessing the `usr["name"]` attribute of the map supplied as the function argument.  We also access attributes of this data map when setting the rows of our output table.

Utilising the `TableRex.quick_render!()` function, we can output a table with the attributes we wish to be included.  We then forward the results onto an `IO.puts`  command, to return the table to the top-level function, which will print it in the command line.  And with that, our coding is complete!


## Compile Time
Now that we have our app coded, we can compile it to an executable binary.  Ensure all files are saved, head into the app's root directory and run:

``` bash
mix escript.build
```

This command tells the `escript` protocol in Mix to build a binary.  We now have a binary that we can call, with the email address and password of a registered user as arguments.  **_Before_** you do that, make sure you have actually registered a test user from within the [Auth0 Management Dashboard](https://manage.auth0.com/#/users).  Once you've created a test user, you can execute the binary:

``` bash
./padlock --email robin@percy.test --password pa55word
```

Providing  the correct login credentials were supplied, the output will look like this:

![Auth0 Non-Interactive Client Output](https://cdn.auth0.com/blog/elixir-cmd/binary-output.png)


## Conclusion
So, there we have it.  A non-conventional, _non-interactive_ use case for Elixir and Auth0.  The point of this article, and the overall point of the series, is to demonstrate how Auth0 can be used not just for interactive web applications, but can be used in almost **any** scenario in which authentication is required.  As stated above, the authentication flow used in this article is the [Resource Owner Password Grant](https://auth0.com/docs/api-auth/tutorials/password-grant), but there are many other flows available to use in a huge variety of scenarios.

Elixir is a remarkably dynamic language (pun most definitely intended).  I love so many different aspects of it, and ironically, we haven't explored any of those concepts in this article.  However, from this article, I have spawned another sample application and tutorial that I will publish as soon as possible.  _That_ tutorial will focus more on the features of Elixir that I **really** love: Distribution, Concurrency, and the OTP platform.  A couple of the features I am happy to have been able to show off in this tutorial include Mix, the wonderful syntax, and the functional paradigm of function chaining.  All of which are features I am a big fan of, but that only scratch the surface of the amazing language that is Elixir.

This article is just part one in a three-part series.  In the next article, we will be exploring the [Client Credentials Grant](https://auth0.com/docs/api-auth/grant/client-credentials) authentication flow, as we dive back into [**_Crystal Lang_**](https://auth0.com/blog/building-your-first-crystal-app-with-jwt-authentication) and **_C_**, to build a low-level processing system.  I hope you will join me back on the Auth0 Blog for that!

[ **_- @rbin_**](https://twitter.com/rbin)
