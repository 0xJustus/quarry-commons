# quarry-commons

A content-addressed commons of verified crash abstracts, queryable offline.

Each artifact is one crash abstract addressed by the hash of its content. A Bloom
digest answers "novel or already known?" locally, so you pull a few kilobytes and
check membership without cloning the whole tree.

## Layout

- `artifacts/` one JSON abstract per crash, at `artifacts/<id[:2]>/<id>.json`
- `keys/` behavioral-key index (sharded JSONL), mapping key to artifact id
- `digest/keys.bloom` Bloom filter of every indexed key
- `views/` by-class and by-project id lists for sparse checkout
- `commons.json` manifest (shard prefix, counts)

## Use

```sh
quarry commons-verify .               # audit the tree
quarry query --tree . --keys bk:...   # look up prior art by behavioral key
```

Contributions are gated in CI: `commons-verify` re-derives the index, digest, and
views from the audited artifacts and rejects anything that does not match.

## License

[Apache 2.0](LICENSE).
