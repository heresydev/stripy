# Stripy [![hex.pm](https://img.shields.io/hexpm/v/stripy.svg?style=flat-square)](https://hex.pm/packages/stripy) [![hexdocs.pm](https://img.shields.io/badge/docs-latest-green.svg?style=flat-square)](https://hexdocs.pm/stripy)

Stripy is a micro wrapper intended to be
used for sending requests to Stripe's REST API. It is
made for developers who prefer to work directly with the
official API and provide their own abstractions on top
if such are needed.

Stripy takes care of setting headers, encoding the data,
configuration settings, etc (the usual boring boilerplate);
it also makes testing easy by letting you plug your own
mock server (see Testing section below).

Some basic examples:

```elixir
iex> Stripy.req(:get, "subscriptions")
{:ok, %HTTPoison.Response{...}}

iex> Stripy.req(:post, "customers", %{"email" => "a@b.c", "metadata[user_id]" => 1})
{:ok, %HTTPoison.Response{...}}
```

Where `subscriptions` and `customers` are [REST API resources](https://stripe.com/docs/api).

If you prefer to work with a higher-level library, check out
"stripity_stripe" or "stripe_elixir" on Hex.

An optional 4th parameter can be supplied (a Keyword list of options):

```elixir
Stripy.req(:post, "charges", %{amount: 1000, currency: "USD"},
  [stripe_account: "acct_12345678",
   version: "2017-06-05",
   secret_key: "sk_12345679",
   idempotency_key: "123456"])
```

## Installation

Add to your `mix.exs` as usual:
```elixir
def deps do
  [{:stripy, "~> 1.0"}]
end
```
If you're not using [application inference](https://elixir-lang.org/blog/2017/01/05/elixir-v1-4-0-released/#application-inference), then add `:stripy` to your `applications` list.

Then configure the `stripy` app per environment like so:

```elixir
config :stripy,
  secret_key: "sk_test_xxxxxxxxxxxxx", # required
  endpoint: "https://api.stripe.com/v1/", # optional
  version: "2017-06-05", # optional
  httpoison: [recv_timeout: 5000, timeout: 8000] # optional
```

## Testing

You can disable actual calls to the Stripe API like so:

```elixir
# Usually in your test.exs.
config :stripy,
  testing: true
```

All functions that use Stripy would receive response `{:ok, %{status_code: 200, body: "{}"}}`.

To provide your own responses, you need to configure a mock server:

```elixir
config :stripy,
  testing: true,
  mock_server: MyApp.StripeMockServer
```

Here's an example mock server that mocks the `/customer` endpoint and returns a basic
object for a customer with id `cus_test`

```elixir
defmodule MyApp.StripeMockServer do
  @behaviour Stripy.MockServer

  @ok_res %{status_code: 200}

  @impl Stripy.MockServer
  def request(:get, "customers/cus_test", %{}) do
    body = Poison.encode!(%{"email" => "email@email.com"})
    {:ok, Map.put(@ok_res, :body, body)}
  end
end
```

Now let's quickly write a naive function that gets user's billing email:

```elixir
def stripe_email(user) do
  {:ok, res} = Stripy.req(:get, "customers/#{user.stripe_id}")
  res["email"]
end
```

We can test it like so:

```elixir
fake_user = %{stripe_id: "cus_test"}
assert stripe_email(fake_user) == "email@email.com"
```

## About

<img src="http://cdn.heresy.io/media/logo.png" alt="Heresy logo" width=300>

This project is sponsored by [Heresy](http://heresy.io). We're always looking for great engineers to join our team, so if you love Elixir, open source and enjoy some challenge, drop us a line and say hello!

## License

- Stripy: See LICENSE file.
- "Heresy" name and logo: Copyright © 2019 Heresy Software Ltd
