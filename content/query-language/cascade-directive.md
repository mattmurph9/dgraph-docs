+++
date = "2017-03-20T22:25:17+11:00"
title = "Cascade Directive"
weight = 15
[menu.main]
    parent = "query-language"
+++

With the `@cascade` directive, nodes that don't have all predicates specified in the query are removed. This can be useful in cases where some filter was applied or if nodes might not have all listed predicates.


Query Example: Harry Potter movies, with each actor and characters played.  With `@cascade`, any character not played by an actor called Warwick is removed, as is any Harry Potter movie without any actors called Warwick.  Without `@cascade`, every character is returned, but only those played by actors called Warwick also have the actor name.
{{< runnable >}}
{
  HP(func: allofterms(name@en, "Harry Potter")) @cascade {
    name@en
    starring{
        performance.character {
          name@en
        }
        performance.actor @filter(allofterms(name@en, "Warwick")){
            name@en
         }
    }
  }
}
{{< /runnable >}}

You can apply `@cascade` on inner query blocks as well.
{{< runnable >}}
{
  HP(func: allofterms(name@en, "Harry Potter")) {
    name@en
    genre {
      name@en
    }
    starring @cascade {
        performance.character {
          name@en
        }
        performance.actor @filter(allofterms(name@en, "Warwick")){
            name@en
         }
    }
  }
}
{{< /runnable >}}

## Parameterized `@cascade`

The `@cascade` directive can optionally take a list of fields as an argument.
This changes the default behavior, considering only the supplied fields as mandatory instead of all the fields for a type.
Listed fields are automatically cascaded as a required argument to nested selection sets.
A parameterized cascade works on levels (e.g. on the root function or on lower levels), so
you need to specify `@cascade(param)` on the exact level where you want it to be applied.

{{% notice "tip" %}}
The rule with `@cascade(predicate)` is that the predicate needs to be in the query at the same level `@cascade` is.
{{% /notice %}}

Take the following query as an example:

{{< runnable >}}
{
  nodes(func: allofterms(name@en, "jones indiana")) {
    name@en
    genre @filter(anyofterms(name@en, "action adventure")) {
      name@en
    }
    produced_by {
      name@en
    }
  }
}
{{< /runnable >}}

This query gets nodes that have all the terms _"jones indiana"_ and then traverses to `genre` and `produced_by`.
It also adds an additional filter for `genre`, to only get the ones that either have _"action"_ or _"adventure"_ in the name.
The results include nodes that have no `genre` and nodes that have no `genre` and no `producer`.

If you apply a regular `@cascade` without a parameter, you'll loose the ones that had `genre` but no `producer`.

To get the nodes that have the traversed `genre` but posibly no `produced_by`, you can parameterize the cascade:

{{< runnable >}}
{
  nodes(func: allofterms(name@en, "jones indiana")) @cascade(genre) {
    name@en
    genre @filter(anyofterms(name@en, "action adventure")) {
      name@en
    }
    produced_by {
      name@en
    }
    written_by {
      name@en
    }
  }
}
{{< /runnable >}}

If you want to check for multiple fields, just comma separate them. For example, to cascade over `produced_by` and `written_by`:

{{< runnable >}}
{
  nodes(func: allofterms(name@en, "jones indiana")) @cascade(produced_by,written_by) {
    name@en
    genre @filter(anyofterms(name@en, "action adventure")) {
      name@en
    }
    produced_by {
      name@en
    }
    written_by {
      name@en
    }
  }
}
{{< /runnable >}}

### Nesting and parameterized cascade

The cascading nature of field selection is overwritten by a nested `@cascade`.

The previous example can be cascaded down the chain as well, and be overridden on each level as needed.

For example, if you only want the _"Indiana Jones movies that were produced by the same person who produced a Jurassic World movie"_:

{{< runnable >}}
{
  nodes(func: allofterms(name@en, "jones indiana")) @cascade(produced_by) {
    name@en
    genre @filter(anyofterms(name@en, "action adventure")) {
      name@en
    }
    produced_by @cascade(producer.film) {
      name@en
      producer.film @filter(allofterms(name@en, "jurassic world")) {
        name@en
      }
    }
    written_by {
      name@en
    }
  }
}
{{< /runnable >}}

Another nested example: _"Find the Indiana Jones movie that was written by the same person who wrote a Star Wars movie and was produced by the same person who produced Jurassic World"_:

{{< runnable >}}
{
  nodes(func: allofterms(name@en, "jones indiana")) @cascade(produced_by,written_by) {
    name@en
    genre @filter(anyofterms(name@en, "action adventure")) {
      name@en
    }
    produced_by @cascade(producer.film) {
      name@en
      producer.film @filter(allofterms(name@en, "jurassic world")) {
        name@en
      }
    }
    written_by @cascade(writer.film) {
      name@en
      writer.film @filter(allofterms(name@en, "star wars")) {
        name@en
      }
    }
  }
}
{{< /runnable >}}
