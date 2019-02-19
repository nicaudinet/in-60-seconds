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

⇒
&#x21d2;

---

### What is a graph?

A graph &#120126; ( *V*, *E* ) is a pairing of:
* a set of vertices *V* &#x2286; &#120141;
* a set of edges *E* ⊆ ( *V* × *V* )

where &#120141; is the set of all possible vertices

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

You can represent things that are not graphs!

e.g. G [1] [(1,2)]

since *E* &#x2286; (&#120141; × &#120141;)

---

### Algebraic Graphs

```haskell
data Graph a
  = Empty
  | Vertex a
  | Overlay (Graph a) (Graph a)
  | Connect (Graph a) (Graph a)
```

+++

**Empty**

&#x03b5; = (&#x2205;, &#x2205;)

![Empty image](assets/img/empty.png)

+++

**Vertex *v***

( {*v*}, &#x2205;) where *v* &#x2208; &#120141;

![Vertex image](assets/img/vertex.png)

+++

**Overlay g g**

(*V*<sub>1</sub> , *E*<sub>1</sub> ) + (*V*<sub>2</sub> , *E*<sub>2</sub> ) = (*V*<sub>1</sub> &#x222a; *V*<sub>2</sub> , *E*<sub>1</sub> &#x222a; *E*<sub>2</sub> )

![Overlay image](assets/img/overlay.png)

+++

**Connect g g**

(*V*<sub>1</sub> , *E*<sub>1</sub> ) \* (*V*<sub>2</sub> , *E*<sub>2</sub> ) = (*V*<sub>1</sub> &#x222a; *V*<sub>2</sub> , *E*<sub>1</sub> &#x222a; *E*<sub>2</sub> &#x222a; ( *V*<sub>1</sub> × *V*<sub>2</sub>)

![Connect image](assets/img/connect.png)

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

## Soundness

+++

**Absorption theorem**

(x \* y) + x + y = x \* y

+++

Proof using decomposition:

x \* y

= x \* y \* &#x03b5;

= x \* y + x \* &#x03b5; + y \* &#x03b5;

= x \* y + x + y

+++

> *edges are inseparable from their vertices*

Therefore we cannot construct an incorrect graph.

**proof of soundness**  

---

### The Typeclass

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

vertices :: Graph g => [Vertex g] -> g
vertices = foldr overlay empty . map vertex

edges :: Graph g => [(Vertex g, Vertex g)] -> g
edges = foldr overlay empty . map (uncurry edge)

graph :: Graph g => [Vertex g] -> [(Vertex g, Vertex g)] -> g
graph vs es = overlay (vertices vs) (edges vs)
```

**proof of completeness**

---

### Subgraph

If **x + y = y** then x is a subgraph of y

+++

x + y = y

(*V*<sub>x</sub>, *E*<sub>x</sub>) + (*V*<sub>y</sub>, *E*<sub>y</sub>) = (*V*<sub>y</sub>, *E*<sub>y</sub>)

therefore

*V*<sub>x</sub> &#x222a; *V*<sub>y</sub> = *V*<sub>y</sub>   ⇒   *V*<sub>x</sub> &#x2286; *V*<sub>y</sub>

*E*<sub>x</sub> &#x222a; *E*<sub>y</sub> = *E*<sub>y</sub>   ⇒   *E*<sub>x</sub> &#x2286; *E*<sub>y</sub>

+++

```haskell
isSubgraphOf :: (Graph g, Eq g) => g -> g -> Bool
isSubgraphOf x y = overlay x y == y
```

---

## Equality

+++

A binary relation

```haskell
data Relation a = R
  { domain   :: Set a
  , relation :: Set (a,a)
  } deriving Eq
```

+++

```haskell
import           Data.Set (Set, singleton, union, elems, fromAscList)
import qualified Data.Set as S (empty)

instance Ord a => Graph (Relation a) where
  type Vertex (Relation a) = a
  empty = R S.empty S.empty
  vertex x = R (singleton x) S.empty
  overlay x y = R (domain x `union` domain y) (relation x `union` relation y)
  connect x y = R (domain x `union` domain y) (relation x `union` relation y `union`
    fromAscList [ (a,b) | a <- elems (domain x), b <- elems (domain y) ]
```

+++

This representation is unique!
Can be used to test for equality.

+++

Converting the Graph data type

```haskell
fold :: Graph g => Graph ( Vertex g) -> g
fold Empty         = empty
fold ( Vertex x  ) = vertex x
fold ( Overlay x y) = overlay (fold x) (fold y)
fold ( Connect x y) = connect (fold x) (fold y)
```

+++

Equality

```haskell
instance Ord a => Eq (Graph a) where
  x == y = fold x == (fold y :: Relation a)
```

+++

Compactness
