%{
  version: "1.0.3",
  title: "এরল্যাং ইন্টারোপারেবিলিটি",
  excerpt: """
  এরল্যাং ভি এম (BEAM) এর উপর তৈরি হওয়ার অন্যতম সুবিধা হলো, আমাদের কাছে হাজারো লাইব্রেরী ব্যবহার করার সুযোগ করে দেয়া।
  ইন্টারোপারেবিলিটি আমাদের সে সমস্ত লাইব্রেরী ও এরল্যাং এর স্ট্যান্ডার্ড লাইব্রেরী গুলো এলিক্সির কোড থেকে ব্যবহার করতে দেয়।
  এই অধ্যায়ে, আমরা দেখবো কিভাবে স্ট্যান্ডার্ড ও থার্ড পার্টি এরল্যাং প্যাকেজ এর ফাংশনালিটি এক্সেস করতে হয়।
  """
}
---

## স্ট্যান্ডার্ড লাইব্রেরী

আমাদের অ্যাপ্লিকেশানের এলিক্সির কোড থেকে এরল্যাং এর বৃহৎ স্ট্যান্ডার্ড লাইব্রেরী এক্সেস করা যায়।
এরল্যাং এর মডিউল গুলো ছোট হাতের এটম দিয়ে প্রকাশ করা হয় যেমনঃ `:os` এবং `:timer`।

চলুন, তবে `:timer.tc` ব্যবহার করে একটি ফাংশনের এক্সিকিউশন টাইম বের করা যাকঃ 

```elixir
defmodule Example do
  def timed(fun, args) do
    {time, result} = :timer.tc(fun, args)
    IO.puts("Time: #{time} μs")
    IO.puts("Result: #{result}")
  end
end

iex> Example.timed(fn (n) -> (n * n) * n end, [100])
Time: 8 μs
Result: 1000000
```

আরও কি কি মডিউল আছে তার সম্পূর্ণ লিস্ট পেতে দেখুন এরল্যাং এর [রেফারেন্স ম্যানুয়াল](http://erlang.org/doc/apps/stdlib/)।

## এরল্যাং প্যাকেজ

আগের অধ্যায়ে আমরা মিক্স এবং ডিপেন্ডেসী ম্যানেজমেন্ট শিখেছি।
এরল্যাং এর  লাইব্রেরী গুলোও একই ভাবে কাজ করে।
যদি এরল্যাং এর লাইব্রেরীটি [হেক্স](https://hex.pm) এ পুশ করা না থাকে, তাহলে এর পরিবর্তে গিট রিপোজিটরী ও ব্যবহার করা যায়ঃ

```elixir
def deps do
  [{:png, github: "yuce/png"}]
end
```

এখন আমরা আমাদের এরল্যাং লাইব্রেরী এক্সেস করতে পারবোঃ

```elixir
png =
  :png.create(%{:size => {30, 30}, :mode => {:indexed, 8}, :file => file, :palette => palette})
```

## লক্ষণীয় বিষয়

কিভাবে এরল্যাং লাইব্রেরী ব্যবহার করা যায় তা তো শেখা হলো, এবার এরল্যাং ইন্টারোপারেবিলিটি ব্যবহার করার সময় কি কি জিনিস মাথায় রাখা লাগবে তা আলোচনা করা যাক। 

### এটমস

এরল্যাং এর এটমস গুলো অনেকটাই এলিক্সিরের গুলোর মতোই শুধু কোলন (`:`) বাদে।
তাদেরকে ছোট হাতের অক্ষর ও আন্ডারস্কোর ব্যবহার করে লেখা হয়ঃ

এলিক্সিরঃ

```elixir
:example
```

এরল্যাংঃ

```erlang
example.
```

### স্ট্রিংস 

যখন আমরা এলিক্সিরে স্ট্রিং নিয়ে আলোচনা করি তখন আমরা শুধু মাত্র UTF-8 এনকোডেড বাইনারীই বুঝি।
এরল্যাং, এর ক্ষেত্রে স্ট্রিং ডাবল কোট ব্যবহার করলেও কার লিস্টের মতোই আচরণ করেঃ

এলিক্সিরঃ

```elixir
iex> is_list('Example')
true
iex> is_list("Example")
false
iex> is_binary("Example")
true
iex> <<"Example">> === "Example"
true
```

এরল্যাংঃ

```erlang
1> is_list('Example').
false
2> is_list("Example").
true
3> is_binary("Example").
false
4> is_binary(<<"Example">>).
true
```

একটা জিনিস মনে রাখতে হবে, অনেক পুরনো এরল্যাং লাইব্রেরী বাইনারী সাপোর্ট না ও করতে পারে, সেক্ষেত্রে আমাদের এলিক্সির স্ট্রিংকে কার লিস্টে রূপান্তর করতে হবে।
সৌভাগ্যবশত, এটা করা খুবই সহজ আমাদের শুধুমাত্র `to_charlist/1` ফাংশনটি ব্যবহার করতে হবেঃ

```elixir
iex> :string.words("Hello World")
** (FunctionClauseError) no function clause matching in :string.strip_left/2

    The following arguments were given to :string.strip_left/2:

        # 1
        "Hello World"

        # 2
        32

    (stdlib) string.erl:1661: :string.strip_left/2
    (stdlib) string.erl:1659: :string.strip/3
    (stdlib) string.erl:1597: :string.words/2

iex> "Hello World" |> to_charlist() |> :string.words
2
```

### ভ্যারিয়েবলস

এরল্যাং, এ ভ্যারিয়েবল শুরু হয় বড় হাতের অক্ষর দিয়ে এবং পুনঃ বাইন্ডিং করা সম্ভব না। 

এলিক্সিরঃ

```elixir
iex> x = 10
10

iex> x = 20
20

iex> x1 = x + 10
30
```

এরল্যাংঃ 

```erlang
1> X = 10.
10

2> X = 20.
** exception error: no match of right hand side value 20

3> X1 = X + 10.
20
```

ব্যস এতটকুই! আমরা শিখলাম, এলিক্সির এপ্লিকেশন থেকে এরল্যাং এর সকল সুবিধা নেওয়া খুবই সহজ এবং এটি আমাদের কাছে ব্যবহারোপযোগী লাইব্রেরী এর সংখ্যা দুই গুণ করে দেয়।
