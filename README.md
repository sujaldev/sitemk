# Site Maker

A framework for creating static-site generators.

## Design

I'm writing this section mostly for myself, to figure out what I intend to build before I actually do. The goal is to
create a general enough model that mostly whatever build pipeline one would like to design for a static website, can be
accomplished neatly.

A good model to design around seems to be to picture a static-site generator as a transformation from a source tree $S$
to an output tree $O$. One or many node(s) in $S$ may transform into one or many node(s) in $O$. An example of a
many-to-one relationship is transforming a `my-post.md` file that relies on a `post.jinja` template, this also implies
that `post.jinja` has a one-to-many relationship with all generated post files.

_Transformers_ are responsible for transforming nodes in $S$ (or intermediate trees) to either nodes in $O$ or
generating some state. A transformer $T$ is defined by the tuple:

$$
\langle D,~R,~P,~Q\rangle
$$

Where,

<div align="center">

$D$ is a set containing transformers that must execute before this transformer (dependencies).

$R$ is an evaluation function that defines what nodes a transformer operates on (rules).

$P$ is a function that takes a node as input and produces one or many nodes and/or state as output.

$Q$ is a queue of nodes that should be processed with $P$ (to avoid cycles we could disallow $P$ from appending to its
own $Q$).

</div>

A rough procedure I have in mind that uses this model as a build pipeline:

1. Resolve dependencies, and build an execution order for transformers (we could possibly do cycle detection here,
   but that's a problem for future me).
2. Walk the input tree $S$, use every transformer's $R$ function to check for matches at each node.
3. If a node matches the $R$ function of a transformer $T$, store that node in $T$'s work queue $Q$.
4. After walking the entire tree $S$, pick the highest priority transformer with a non-empty work queue in the execution
   order. Execute $P$ of that transformer with the first entry in $Q$. Repeat step 3 for nodes returned by $P$ in this
   step.
5. Repeat step 4 until no transformers have any work left (empty mappings).
