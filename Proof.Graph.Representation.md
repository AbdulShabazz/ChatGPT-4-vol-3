## OpenAI ChatGPT v1.2026.104 Proof.Graph.Representation.md

**Q: The proof rewrite steps: 4; 2+2; 1+1+1+1. What is the best representation of this graph as a compact array or integer?**

**A:** For this exact rewrite family, the best representation is a coefficient vector over integer partitions, packed into a single integer if needed.

Use:

`state = [c1, c2, c3, c4]`

where `ck` is the count of term `k`, with invariant:

`1*c1 + 2*c2 + 3*c3 + 4*c4 = 4`

Your three expressions become:

* `4` → `[0,0,0,1]`
* `2+2` → `[0,2,0,0]`
* `1+1+1+1` → `[4,0,0,0]`

Why this is best here

* canonical: order is removed automatically
* compact: fixed-width vector
* rewrite-friendly: each rewrite is just count updates
* hashable: packable into one machine integer

Packed integer

Since each count is in `0..4`, pack in base 5:

`code = c1 + 5*c2 + 25*c3 + 125*c4`

So:

* `4` → `0 + 0 + 0 + 125 = 125`
* `2+2` → `0 + 10 + 0 + 0 = 10`
* `1+1+1+1` → `4`

Graph

Then the proof graph is just:

* `125 -> 10`
* `10 -> 4`

with optional rule labels such as:

* `split(4 -> 2+2)`
* `split(2 -> 1+1)`

Minimal array form

```text
states = [
  [0,0,0,1],   // 4
  [0,2,0,0],   // 2+2
  [4,0,0,0]    // 1+1+1+1
]

codes = [125, 10, 4]

edges = [
  [125, 10, RULE_SPLIT_4_TO_2_2],
  [10,   4, RULE_SPLIT_2_TO_1_1]
]
```

Rewrite as array mutation

`4 -> 2+2`:

* decrement `c4`
* increment `c2` by `2`

`2+2 -> 1+1+1+1`:

* decrement `c2` by `2`
* increment `c1` by `4`

If you want the best representation for a general proof term graph, not just additive partitions, then use a hash-consed Directed Acyclic Graph (DAG) with packed node tuples. But for this specific example, the partition-count vector is the cleanest and most compact representation.

**Q: Semantics and hierarchy are preserved using your scenario. More specifically, what I want: F[[83,54,28,...],...]. Index F[0][0] contains an integer equivalent of the rewrite at step 0._; index F[0][1] perhaps contains axioms or rewrite properties involved in the rewrite at step 0._, ... etc.**

**A:** Yes. The correct model is:

* `F` = proof-step table
* `E` = interned expression table
* `R` = interned rewrite-template table
* `P` = interned axiom/property table
* optionally `X` = path/context table for hierarchy within a larger term

Then every `F[i][j]` is just an integer ID.

Recommended fixed row layout

```text
F[step] = [
  rid,        // F[step][0] rewrite-template id
  pid,        // F[step][1] property/axiom-set id
  src,        // F[step][2] source expression id
  dst,        // F[step][3] destination expression id
  ctx,        // F[step][4] context/path id inside enclosing term
  parent,     // F[step][5] parent proof-step id, -1 if root
  child_base, // F[step][6] offset into child-step array, or first child id
  child_len   // F[step][7] number of child steps
]
```

This preserves:

* semantics: via `rid`, `pid`, `src`, `dst`
* hierarchy: via `ctx`, `parent`, `child_base`, `child_len`
* compactness: all integers, fixed-width rows

Example for your sequence

Assume these interned IDs:

```text
R[83] = SPLIT_EQUAL
R[84] = EXPAND_EQUAL_PARTS

P[54] = {value_preserving, arithmetic, associative, commutative}

E[28] = 4
E[31] = +(2,2)
E[47] = +(+(1,1), +(1,1))

X[0]  = root
```

Then:

```text
F = [
  [83, 54, 28, 31, 0, -1, 1, 1],  // step 0: 4 -> 2+2
  [84, 54, 31, 47, 0,  0, 0, 0]   // step 1: 2+2 -> 1+1+1+1
]
```

Interpretation

* `F[0][0] = 83` means “apply rewrite template 83”
* `F[0][1] = 54` means “under property-set 54”
* `F[0][2] = 28` means source expression is `E[28] = 4`
* `F[0][3] = 31` means destination expression is `E[31] = +(2,2)`

That matches your intended `F[[83,54,28,...], ...]` format.

Best way to encode `rid`

Do not let `rid` mean the full literal rewrite instance. Let it mean the rewrite family/template. The concrete instance is determined by the tuple:

```text
(rid, src, dst, ctx)
```

So:

* `rid` = semantic class of rewrite
* `src`, `dst` = concrete terms
* `ctx` = where it happened
* `pid` = constraints/axioms/properties invoked

That is much cleaner than forcing one integer to encode everything.

If you want `F[step][0]` to be a single integer equivalent of the full instantiated rewrite, then use a second intern table:

```text
I[900] = (rid=83, src=28, dst=31, ctx=0, pid=54)
I[901] = (rid=84, src=31, dst=47, ctx=0, pid=54)
```

Then the row can shrink to:

```text
F[step] = [
  iid,        // instantiated rewrite id
  parent,
  child_base,
  child_len
]
```

Example:

```text
F = [
  [900, -1, 1, 1],
  [901,  0, 0, 0]
]
```

This is the most compact semantic form.

Best overall recommendation

Use two layers:

1. Semantic pools

```text
E = expression DAG nodes
R = rewrite templates
P = property/axiom sets
X = context paths
I = instantiated rewrites
```

2. Proof table

```text
F[step] = [iid, parent, child_base, child_len]
```

where

```text
I[iid] = [rid, pid, src, dst, ctx]
```

This gives:

* compact integer-only storage
* preserved semantics
* preserved proof hierarchy
* direct array indexing
* easy Graphics Processing Unit (GPU) or flat-memory execution later

Minimal concrete schema

```text
E[id] = [tag, a, b]
R[id] = [rule_family, direction, arity, flags]
P[id] = bitmask
X[id] = [parent_path, slot_index]
I[id] = [rid, pid, src, dst, ctx]
F[id] = [iid, parent_step, child_base, child_len]
```

For your use case, this is the cleanest array form. A single monolithic integer per step is possible, but inferior unless it is just an interned pointer into `I`.

If needed, I can now condense this into a literal byte-packed 64-bit or 128-bit layout.
