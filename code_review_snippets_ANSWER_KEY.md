# Answer Key: code_review_snippets.py

**Interviewer only. Do not share.**

Based on quantifiedcode/python-anti-patterns (`examples/new_patterns.py`), with the answers removed from the candidate file. Some of the original "bad" examples contain extra real bugs. They were kept on purpose and are marked **Bonus** below. Severity: **Bug** = wrong behavior or crash, **Style** = works but should change.

| # | Main issue | Fix | Bonus finds |
|---|---|---|---|
| 1 | Style: indexing with `range(len(l))` | `for i, le in enumerate(l):` | `l` is a bad name (looks like `1`); `range(0, ...)` start is redundant |
| 2 | Style/design: returns a list or `None`, so every caller must check | Raise an exception instead of returning `None` | `"foo" in e` is clearer than `e.find("foo") != -1`; names `l`, `r` are unclear |
| 3 | **Bug:** `i` is never incremented, so the loop never ends | Add `i += 1`, or use `for i in range(10):` | |
| 4 | Style: looking up `d[key]` inside the loop | `for key, value in d.items():` | Note: the original repo says `.iteritems()`, which is Python 2 only. `.items()` is correct in Python 3. |
| 5 | Style: index loop over two lists | `for l1v, l2v in zip(l1, l2):` | **Bug:** `range(l1)` raises `TypeError` (should be `range(len(l1))`). Also: `zip` silently stops at the shorter list; `zip(..., strict=True)` in 3.10+ |
| 6 | Performance: `in` on a list is O(n) | Use a `set` if you check membership many times | **Nuance:** for a single check on 4 items the list is fine. Building a set is itself O(n). A strong candidate says "it depends". |
| 7 | Style: flag variable `found` | `for ... else:` (the `else` runs if no `break`) | Many teams find `for/else` less readable; accepting either with a reason is fine. `any(i == 4 for i in l)` also works. |
| 8 | Style: manual check-then-create | `d.setdefault("foo", []).append("bar")` or `defaultdict(list)` | `not "foo" in d` should be `"foo" not in d` (PEP 8) |
| 9 | Style: manual default | `foo = d.get("foo", "default")` | |
| 10 | Style: `map`/`filter` with `lambda` | `[x * 2 for x in values]`, `[x for x in values if x < 2]` | **Bug risk:** in Python 3 these return lazy iterators, not lists. They can only be read once and `len()` fails. `True if x < 2 else False` is just `x < 2`. |
| 11 | Style: manual counter init | `defaultdict(int)` or `collections.Counter` | `not "count" in d` (same as 8) |
| 12 | Design: magic status code tuple `(-1, "not found")` | Raise an exception, or return a `NamedTuple`/`dataclass` | Returning error codes is C style; Python prefers exceptions |
| 13 | Style: index-by-index assignment | `l0, l1, l2 = l` | Unpacking also fails loudly if the length is wrong, which is a plus |
| 14 | **Bug (likely):** `_t = x` saves the old `x` but is never used, so `y` is computed from the **new** `x` (result `y = 0`). The author probably meant the old value. | Ask the author's intent. If old values were meant: `x, y = y + 2, x - 4` (gives `x = 4, y = -3`) | **Note:** the original repo presents this fix as equivalent, but it changes the result. A strong candidate spots this. |
| 15 | **Bug risk:** if `f.read()` raises, the file is never closed | `with open("file.txt") as f: content = f.read()` | The original repo's "good" version reads from `f` instead of `input_file`, which is its own bug |
| 16 | **Bug (race condition):** the file can disappear between `exists` and `unlink` | `try: os.unlink(...)` / `except FileNotFoundError: pass`, or `Path("file.txt").unlink(missing_ok=True)` | "Easier to ask forgiveness than permission" (EAFP) |
| 17 | Style: `dict()` around a list of tuples | `{n: n * 2 for n in l}` | |
| 18 | Style: string concatenation with `str()` | `f"there were {n_errors} errors."` | |
| 19 | Style: type in the variable name (Hungarian notation) | Descriptive names, plus type hints if needed: `count: int = 4` | |
| 20 | Design: Java-style getters and setters | Use a plain attribute, or `@property` if logic is needed | **Bugs:** `__init__(a)` is missing `self`, so `self` is undefined (NameError). `get_a` returns `a` instead of `self._a`. `(object)` is unnecessary in Python 3. |
| 21 | Style: long `if/elif` chain | Dict of functions, e.g. `{"+": operator.add, ...}` | **Bugs:** an unknown operator silently returns `None`; `/` raises `ZeroDivisionError` with no handling. Using `operator.add` etc. beats lambdas. Also: the parameter `operator` shadows the stdlib module name. |
| 22 | Design: class used only as a namespace for a static method | Plain module-level function | **Bugs:** in Python's `weekday()`, 5 is **Saturday**, not Friday (Friday is 4). The dict is missing 5 of 7 days, so most inputs raise `KeyError`. Better: `calendar.day_name[weekday]`. Typo `nameds_weekdays`. |

## Scoring (22 snippets, about 30 to 40 min)

- **The real bugs** (the hardest to spot, worth the most): 3, 5 (`range(l1)`), 10 (lazy iterators), 14, 15, 16, 20, 21, 22 (wrong weekday).
- **Strong:** finds most style issues quickly, plus 5 or more of the real bugs, and explains *why* (not only "use enumerate").
- **Good:** finds most style issues and 2 to 4 real bugs.
- **Weak:** only renames variables or reformats; misses the infinite loop (3) or the crashes (5, 20).

## Good follow-up questions

- Snippet 6: "When would you keep the list?" (One-off check, small data, need order or duplicates.)
- Snippet 7: "Would you actually use `for/else` on your team?" (Tests judgment, not memorization.)
- Snippet 10: "What happens if I call `len(doubled_values)`?" (`TypeError` on a `map` object.)
- Snippet 14: "What should you do when the intent is unclear?" (Ask the author, don't assume.)
- Snippet 16: "Why is `try/except` safer here than checking first?" (Race condition.)
