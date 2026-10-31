# onlymove-data

The tsumego corpus behind [onlymove.net](https://onlymove.net): one SGF per problem at `<size>/<id>.sgf` for board sizes 9 and 13, plus `retired_ids.txt`, the ids that must never be reissued. Every tool that reads or writes the corpus lives in the code repository, [YoseTsumego](https://github.com/typohh/YoseTsumego).

## Layout

Clone it beside the code repository, which looks for `../onlymove-data` by default:

```
Repositories/
  onlymove/        # git@github.com:typohh/YoseTsumego.git
  onlymove-data/   # this repository
```

To keep it anywhere else, set `ONLYMOVE_TSUMEGOS_DIR` to its path (resolved by `utils/paths.py` in the code repository).

## Invariants

- **An id is symmetry-aware**: SHA-256 over all 8 board symmetries, take the minimum, truncate to 8 hex. No two problems are rotations or reflections of each other.
- **The answer is the decision node's first child**, by construction. Nothing reorders it.
- **Ids in `retired_ids.txt` count as taken.** A retired problem keeps its attempt history in the database, so a reused id would silently inherit someone else's record.
- **Branch comments are `"<winrate>%/<score>/<visits>"`** and are read directly by the client. Don't strip them.

Details are in `docs/corpus_and_engine.md` in the code repository.

## Untracked subtrees

`new/<size>/` is generator staging (problems are adopted into the corpus by an explicit `mv`), `broken/` is quarantine, and `fixed/` holds repaired copies awaiting merge. `broken/` and `fixed/` are gitignored.
