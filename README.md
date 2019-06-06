# mustache

[Mustache templating](https://mustache.github.io) for Carp.

## Installing

```clojure
(load "git@github.com:carpentry-org/mustache@0.0.4")
```

## Usage

The module `Mustache` only exports one function: `template`. This function
accepts a string value to be templated, and a map of values that are the
template context. That map must be from `String` to `Mustache`, where `Mustache`
is either a `Str` or a `Lambda`.

```clojure
(Mustache.template
  "this is a super {{ adjective }} library"
  &{@"adjective" (Mustache.Str @"cool")}
)
```

More comprehensive docs can be found [online](https://veitheller.de/mustache).

<hr/>

Have fun!
