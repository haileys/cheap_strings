# Cheap Strings

Cheap Strings is a simple gem that makes it convenient to use frozen strings in your code.

Since Ruby strings are mutable, every string literal must allocate a new string whenever it is evaluated. Often this is a waste of time, as most strings in Ruby programs are never mutated.

Cheap Strings freezes your string literals and then reuses the same object every time - making your program faster and reducing GC pressure by not creating huge amounts of objects.

## Benchmark

The benchmarks below use this common setup code:

```ruby
require "cheap_strings"

module A
  extend CheapStrings

  def self.create_lots_of_strings
    1_000_000.times do
      "hello world"
    end
  end

  def self.create_lots_of_cheap_strings
    1_000_000.times do
      `hello world`
    end
  end
end
```

### Time benchmark

```ruby
require "benchmark"

Benchmark.bmbm do |b|
  b.report "regular strings" do
    A.create_lots_of_strings
  end
  b.report "cheap strings" do
    A.create_lots_of_cheap_strings
  end
end
```

**Results:**

```
Rehearsal ---------------------------------------------------
regular strings   0.240000   0.000000   0.240000 (  0.242331)
cheap strings     0.140000   0.000000   0.140000 (  0.135519)
------------------------------------------ total: 0.380000sec

                      user     system      total        real
regular strings   0.240000   0.000000   0.240000 (  0.235980)
cheap strings     0.140000   0.000000   0.140000 (  0.139030)
```

### Object allocation benchmark

```ruby
def object_bench(name)
  before, after = {}, {}

  GC.start
  GC.disable
  ObjectSpace.count_objects(before)

  yield

  ObjectSpace.count_objects(after)
  GC.enable

  printf "%20s %6d -> %6d (+%8d)\n", name, before[:T_STRING], after[:T_STRING], after[:T_STRING] - before[:T_STRING]
end

object_bench "regular strings" do
  A.create_lots_of_strings
end

object_bench "cheap strings" do
  A.create_lots_of_cheap_strings
end
```

**Results:**

```
     regular strings     5629 ->  1005655 (+ 1000026)
       cheap strings     5657 ->     5657 (+       0)
```
