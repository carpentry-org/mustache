# mustache

A [Mustache](https://mustache.github.io) templating library for Carp.

Supports the full spec: variables, HTML-escaped and unescaped output,
sections, inverted sections, object and list contexts, dotted names,
lambdas, partials, comments, and set-delimiter tags.

## Installing

```clojure
(load "git@github.com:carpentry-org/mustache@0.2.0")
```

## Usage

`Mustache.template` takes a template string and a `(Map String Mustache)`
context. The `Mustache` sumtype has four value kinds:

- `Str` — a plain string. Empty strings are falsy for section purposes.
- `Lst` — an `(Array (Box Mustache))`. Empty arrays are falsy; non-empty
  arrays cause their enclosing section to iterate once per item.
- `Mp` — a `(Map String (Box Mustache))`. Used as a section value it
  pushes its entries onto the context for the section body; outer
  bindings still fall through.
- `Lambda` — a `(Fn [String] String)` applied to the raw section body; its
  result is re-parsed and rendered as a template.

### Variable substitution

```clojure
(Mustache.template
  "this is a super {{ adjective }} library"
  &{@"adjective" (Mustache.Str @"cool")})
; => "this is a super cool library"
```

`{{name}}` HTML-escapes its value. `{{{name}}}` and `{{& name}}` leave it
raw.

### Sections

A section on a truthy `Str` renders its body once. On a falsy value
(empty `Str`, empty `Lst`) the whole section is skipped:

```clojure
(Mustache.template
  "{{#visible}}shown{{/visible}}{{^visible}}hidden{{/visible}}"
  &{@"visible" (Mustache.Str @"")})
; => "hidden"
```

### Lists

A section on an `Lst` iterates. List items that are `Mp` push their fields
onto the context; other values are bound as `{{.}}`:

```clojure
(Mustache.template
  "<ul>{{#users}}<li>{{name}}</li>{{/users}}</ul>"
  &{@"users" (Mustache.Lst
              [(Box.init (Mustache.Mp {@"name" (Box.init (Mustache.Str @"Ada"))}))
               (Box.init (Mustache.Mp {@"name" (Box.init (Mustache.Str @"Grace"))}))])})
; => "<ul><li>Ada</li><li>Grace</li></ul>"
```

### Object contexts

A section on an `Mp` renders its body once with the map on top of the
context stack. Outer keys fall through:

```clojure
(Mustache.template
  "{{#user}}{{name}} ({{role}}){{/user}}"
  &{@"user" (Mustache.Mp {@"name" (Box.init (Mustache.Str @"Ada"))})
    @"role" (Mustache.Str @"admin")})
; => "Ada (admin)"
```

### Dotted names

A dotted name walks nested `Mp` values. The first segment is looked up on
the context stack; each further segment must be a key of the previous
segment's map:

```clojure
(Mustache.template
  "{{user.address.city}}"
  &{@"user" (Mustache.Mp
             {@"address" (Box.init
                          (Mustache.Mp {@"city" (Box.init (Mustache.Str @"Berlin"))}))})})
; => "Berlin"
```

Dotted names work in interpolation, sections (`{{#a.b}}…{{/a.b}}`), and
inverted sections. If any segment is missing or is not a map, the name is
treated as absent — interpolation renders empty, a section is skipped, and
an inverted section renders. A bare `{{.}}` remains the implicit iterator.

### Lambdas

A lambda receives the raw (unrendered) section body as input. Its return
value is parsed as a Mustache template and rendered against the current
context, so any tags in the body — or tags the lambda emits — are
interpolated:

```clojure
(Mustache.template
  "{{# shout }}hello{{/ shout }}"
  &{@"shout" (Mustache.Lambda (fn [s] (String.append &s "!!")))})
; => "hello!!"

(Mustache.template
  "{{# wrap }}{{ name }}{{/ wrap }}"
  &{@"wrap" (Mustache.Lambda (fn [s] (String.append &s "!")))
    @"name" (Mustache.Str @"world")})
; => "world!"
```

The output is re-parsed with the delimiters in effect at the section tag,
per the Mustache spec.

### Partials

`{{> name}}` loads `name.mustache` from disk and renders it with the
current context. Missing files render as empty.

### Set delimiters

`{{=< >=}}` changes the open/close delimiters for the rest of the
template (or section body):

```clojure
(Mustache.template
  "{{=& &=}}hello, &name&"
  &{@"name" (Mustache.Str @"world")})
; => "hello, world"
```

## Notes

- Names inside tags are trimmed: `{{ thing }}` looks up `"thing"`.
- Opening and closing section markers must use consistent names —
  `{{/thing}}` does not close `{{^ thing }}`.
- Templates are processed at byte level, which is correct for ASCII and
  UTF-8 input as long as the delimiters themselves are ASCII.

<hr/>

Have fun!
