# onlymove-data

The tsumego corpus behind [onlymove.net](https://onlymove.net): one SGF per problem at `<size>/<id>.sgf` for board sizes 9 and 13, plus `retired_ids.txt`, the ids that must never be reissued. Every tool that reads or writes the corpus lives in the code repository, [onlymove](https://github.com/typohh/onlymove).

## Layout

Clone it beside the code repository, which looks for `../onlymove-data` by default:

```
Repositories/
  onlymove/        # git@github.com:typohh/onlymove.git
  onlymove-data/   # this repository
```

To keep it anywhere else, set `ONLYMOVE_TSUMEGOS_DIR` to its path (resolved by `utils/paths.py` in the code repository).

## How the problems were made

Every problem is a position reached by play, not composed: a [KataGo](https://github.com/lightvector/KataGo) network played a game from a few randomly placed stones, steered toward positions where exactly one move wins. Black is always the side to move, under Japanese rules with a komi of 6.5.

The variations in each file and the validation of each problem come from KataGo's transformer network `kata1-tf3-b10c512-s3203M-d5937M`, the one the generation pipeline in the code repository runs. A problem is in the collection only if it passes three checks:

- **A deep read of the decision.** At 1000 playouts the best move must hold a win probability of at least 80%. A second 1000-playout search with that move excluded must find nothing better than 20%, and behind on score.
- **Both lines played to the end.** The answer and the strongest alternative are each played out to a double pass. The answer must stay ahead on win probability and score at every step, and the alternative must stay behind.
- **Every legal first move evaluated.** The answer's continuation and the refutations are searched at 1000 playouts, and every remaining legal first move at 30. Exactly one move may win; a second winner or an ambiguous move discards the position.

Each branch comment records the playouts behind its own estimate as its third field, so the depth of any evaluation in a file can be read off the file.

These are engine verdicts, not proofs. A small share of positions (unresolved life-and-death or ko) evaluate differently from one search to the next at the same budget, and problems found to be unstable or wrong are retired. The full method is in `docs/methodology.md` §1.1 in the code repository.

## Invariants

- **An id is symmetry-aware**: SHA-256 over all 8 board symmetries, take the minimum, truncate to 8 hex. No two problems are rotations or reflections of each other.
- **The answer is the decision node's first child**, by construction. Nothing reorders it.
- **Ids in `retired_ids.txt` count as taken.** A retired problem keeps its attempt history in the database, so a reused id would silently inherit someone else's record.
- **Branch comments are `"<winrate>%/<score>/<visits>"`** and are read directly by the client. Don't strip them.

Details are in `docs/corpus_and_engine.md` in the code repository.

## Untracked subtrees

`new/<size>/` is generator staging (problems are adopted into the corpus by an explicit `mv`), `broken/` is quarantine, and `fixed/` holds repaired copies awaiting merge. `broken/` and `fixed/` are gitignored.

## License

The problems are licensed under [Creative Commons Attribution 4.0 International](LICENSE) (CC BY 4.0): reuse them freely, with credit to onlymove.net. The code that generates and serves them is in the code repository under the MIT license.
