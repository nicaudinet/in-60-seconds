## Algebraic Graphs

Nicolas Audinet, RELEX

February 2019

+++

Some fancy set unicode shenanigans:

ε
&#03b5;

&#120141; this is a cool V

&#120124; this is a cool E

&#120126;

&#8712;
&#x2208;
∈

∅
&#x2205;

⊆
&#x2286;

×
&#x00d7;

∪
&#x222a;
---

## Graph Basics

+++

A graph &#120126; ( &#120141;, &#120124; ) is a pairing of:
* a set of vertices &#120141;
* a set of edges &#120124; ⊆ ( &#120141; × &#120141; )

+++

In Haskell:

```haskell
data G a = G
  { vertices :: [a]
  , edges    :: [(a,a)]
  }
```

+++

The Problem:

You can represent things that are not graphs

e.g. G [1] [(1,2)]

---

```haskell
data Graph a
  = Empty
  | Vertex a
  | Overlay (Graph a) (Graph a)
  | Connect (Graph a) (Graph a)
```

+++

Empty

![Empty image](assets/img/empty.png)

+++

Vertex a

![Vertex image](assets/img/vertex.png)

+++

Overlay (Graph a) (Graph a)

![Overlay image](assets/img/overlay.png)

+++

Connect (Graph a) (Graph a)

![Connect image](assets/img/connect.png)

---

## Set Semantics

+++

Empty = &#x03b5; = (&#x2205;, &#x2205;)

Vertex *v* = ( {*v*}, &#x2205;) where *v* &#x2208; &#120141;

+++

**Overlay**

(*V*<sub>1</sub> , *E*<sub>1</sub> ) + (*V*<sub>2</sub> , *E*<sub>2</sub> ) = (*V*<sub>1</sub> &#x222a; *V*<sub>2</sub> , *E*<sub>1</sub> &#x222a; *E*<sub>2</sub> )

+++

**Connect**

(*V*<sub>1</sub> , *E*<sub>1</sub> ) \* (*V*<sub>2</sub> , *E*<sub>2</sub> ) = (*V*<sub>1</sub> &#x222a; *V*<sub>2</sub> , *E*<sub>1</sub> &#x222a; *E*<sub>2</sub> &#x222a; ( *V*<sub>1</sub> × *V*<sub>2</sub>)

---

## Axioms

+++

**\+ is commutative and associative**

x + y = y + x

x + (y + z) = (x + y) + z

+++

**(G , \* , &#x03b5;) is a monoid**

x \* &#x03b5; = x

&#x03b5; \* x = x

x \* (y \* z) = (x \* y) \* z

+++

**\* distributes over +**

x \* (y + z) = (x \* y) + (x \* z)

(x + y) \* z = (x \* z) + (y \* z)

![Distributivity](assets/img/distributivity.png)

+++

**Decomposition**

x \* y \* z = (x \* y) + (x \* z) + (y \* z)

![Decomposition](assets/img/decomposition.png)

---

```haskell
class Graph g where
  type Vertex g
  empty   :: g
  vertex  :: Vertex g -> g
  overlay :: g -> g -> g
  connect :: g -> g -> g
```

+++

```haskell
edge :: Graph g => Vertex g -> Vertex g -> g
edge x y = connect (vertex x) (vertex y)
```

```haskell
vertices :: Graph g => [Vertex g] -> g
vertices = foldr overlay empty . map vertex
```
