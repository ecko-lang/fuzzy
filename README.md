# fuzzy - Ecko Std Lib Package

String similarity: Levenshtein, Jaro-Winkler, Dice and Jaccard, plus best-match
selection over a list.

Pure computation - no capabilities.

## Install

```bash
ecko get github.com/ecko-lang/fuzzy
```

```ecko
import fuzzy
```

## Usage

```ecko
fuzzy.levenshtein("kitten", "sitting")     # 3 edits
fuzzy.ratio("kitten", "sitting")           # 0.571 - that distance, normalised
fuzzy.jaro_winkler("MARTHA", "MARHTA")     # 0.961
fuzzy.dice("night", "nacht")               # 0.25

fuzzy.best_match("instal", commands)
# { value: "install", score: 0.966... }

fuzzy.top_matches("instal", commands, 3)
```

## API

### Measures

| function | returns | good for |
|---|---|---|
| `levenshtein(a, b)` | edit count | "how many changes" |
| `ratio(a, b)` | 0.0-1.0 | that distance, normalised by the longer string |
| `hamming(a, b)` | differing positions | fixed-width codes; raises unless lengths match |
| `jaro(a, b)` | 0.0-1.0 | short strings with transpositions |
| `jaro_winkler(a, b)` | 0.0-1.0 | typos, where a shared prefix matters |
| `dice(a, b)` | 0.0-1.0 | names and titles, via bigram overlap |
| `jaccard(a, b)` | 0.0-1.0 | order-blind character overlap, cheapest |

### Selection

| function | what it does |
|---|---|
| `best_match(needle, candidates)` | closest as `{ value, score }`, or `null` |
| `best_match_by(needle, candidates, scorer)` | the same with your own scorer |
| `top_matches(needle, candidates, limit)` | the best `limit`, ranked |
| `top_matches_by(...)` | the same with your own scorer |

### Helpers

`bigrams(s)`, `scored(value, score)`, `max2`, `min2`.

## Notes

**They disagree, and that is why there are four.**

| a | b | lev | ratio | jaro-w | dice |
|---|---|---|---|---|---|
| kitten | sitting | 3 | 0.571 | 0.746 | 0.364 |
| MARTHA | MARHTA | 2 | 0.667 | 0.961 | 0.400 |
| apple | appel | 2 | 0.600 | 0.953 | 0.500 |

`MARTHA`/`MARHTA` is two edits apart, but Jaro-Winkler scores it 0.961: a
transposition early in a short word is almost certainly a typo. If you are
correcting user input, that is the measure you want. If you are counting how
much a document changed, it is not.

**The reference values are the tests.** `MARTHA`/`MARHTA`, `DWAYNE`/`DUANE` and
`DIXON`/`DICKSONX` are the pairs from Winkler's papers, asserted against their
published scores rather than against whatever this implementation produced.

**Jaro-Winkler's prefix bonus is capped at four characters**, and only applies
when the base Jaro score is already 0.7 or better. Without both limits a long
shared prefix would drag every comparison toward 1.0.

**Cost.** Levenshtein is `O(n*m)` time but only `O(min(n,m))` memory, using two
rows rather than the full matrix. Jaro is `O(n*m)` in the worst case. These are
built for words and short phrases, not for diffing files.

## Testing

```bash
ecko test
```

Offline and deterministic.

## License

MIT
