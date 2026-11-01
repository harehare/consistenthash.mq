<h1 align="center">consistenthash.mq</h1>

> [!WARNING]
> This project is currently under active development. APIs and features may change without notice.

> [!NOTE]
> This module needs the development version of mq, which is not released yet: it uses the `hash` builtin.

A consistent hash ring for [mq](https://github.com/harehare/mq), as a module: it maps keys to nodes so that adding or removing a node moves only the keys next to it. Use it to shard keys over servers, caches or output files without reshuffling everything when the set changes. With a plain `hash(key) % n`, changing `n` moves almost every key.

The ring is built on `ds::sorted` from [ds.mq](https://github.com/harehare/ds.mq).

## Installation

Clone with submodules, or copy `consistenthash.mq` next to `ds.mq`, and reference them with `-L`.

```sh
git clone --recurse-submodules https://github.com/harehare/consistenthash.mq
```

## Usage

```sh
mq -L consistenthash.mq -L consistenthash.mq/vendor/ds.mq -I null '
  import "consistenthash"
  | let ring = fold(["a", "b", "c"], consistenthash::new(50), fn(r, n): consistenthash::add_node(r, n);)
  | [consistenthash::node_for(ring, "user:1"), consistenthash::nodes_for(ring, "user:1", 2)]
'
# ["a", ["a", "c"]]
```

| Function | Description |
| --- | --- |
| `new(replicas = 100)` | empty ring; each node owns `replicas` points |
| `add_node(ring, node)` / `remove_node(ring, node)` | add or drop a node |
| `node_for(ring, key)` | the node owning `key`, `None` if the ring is empty |
| `nodes_for(ring, key, count)` | up to `count` distinct nodes, owner first, to place replicas |
| `members(ring)` / `size(ring)` / `is_empty(ring)` / `has_node(ring, node)` | inspect the ring |

More replicas spread the keys more evenly and cost more memory. About 100 per node is a common choice.

## Development

```sh
mq-test -L vendor/ds.mq consistenthash_tests.mq
mq-bench -L vendor/ds.mq -L . consistenthash_bench.mq
```
