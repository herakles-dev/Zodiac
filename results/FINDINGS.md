# Research log

Methods work on the bot itself: what was tried to make Z13/Z32 testable, what was measured,
and what was concluded. Written by hand, unlike REPORT.md. Newest first.

## 2026-11-21: Sharper scoring, Zodiac priors, and why the ciphers resist

**Goal.** Raise the bot's *power* (how often genuine text, enciphered with the real cipher's
exact repeat pattern, beats shuffled copies) so that its verdicts on Z13/Z32 mean something.

**Method.** `scripts/power_study.py` runs the bot's own test machinery (real cipher, 19 shuffled
nulls, 40 genuine-text controls) for each scoring method. ±0.07 is roughly one standard error
on a power of 0.2. Many settings were tried, so the best-looking number is optimistic.

### 1. The decisive result: the search is not the bottleneck

On 12 Z32-like test ciphers (genuine English, homophonic key, Z32's repeat profile) the solver
found a reading that scores **higher than the true plaintext in 12 of 12** cases, under both
letter-pattern and word scoring, typically by 20-40 nats (e.g. truth `BYTHATTIMEBARCOHADDISPOSEDOFITAT`
at -82, solver's `ITWASAPERSOFTHELANDABOUTTHEYCAMP` at -47). Z13-like ciphers under a one-to-one
key: 11-12 of 12.

So with a free key, many readings are *more* English than the real message. No scoring tweak or
extra computation can recover it. **Only constraints on the key** (or new ciphertext) can. This
agrees with winchxyz (2026) and nibzard (2026), now measured directly.

### 2. Power by scoring method (whole-word controls, 40 each)

| cipher, key | letter 5-grams | word scoring | word scoring, gentler unknown-letter cost | letters + words |
|---|---|---|---|---|
| Z32, homophonic | 0.07 | 0.20 | **0.28** | 0.17 |
| Z13, one symbol per letter | 0.40 | 0.20 | 0.45 | 0.10 |
| Z13, homophonic | 0.10 | 0.25 | 0.07 | – |

- Word scoring is the best Z32 detector (0.07 → ~0.2-0.28). It is **not** a good long-cipher
  solver: word-only scoring recovers 20% of Z408, letters+words 97.5%. Each word-scored test
  measures its own power, so its verdicts still say how far to trust them.
- For Z13 nothing beats letter patterns reliably.
- No real cipher stood out under any method (lowest p: Z32 0.05 and 0.10, among ~20 settings,
  so not significant).

### 3. Second statistic: uniqueness of the best reading

Gap between the best reading and the best clearly different reading (>30% of letters differ),
judged against nulls and controls like the score. Power 0.05-0.30; best for Z32 under letter
scoring (0.23). Now reported on every result page as a secondary test (not in the Holm correction).

### 4. Zodiac's habits as key constraints

- **Strict homophone cycling: rejected.** In Z408/Z340 Zodiac reused a symbol for a letter before
  all its other symbols ~42-44% of the time, so a strict-cycling rule would reject his real keys.
- **Key size per letter** (no more symbols per letter than he used in Z408/Z340, +1 slack;
  `zbot/zkey.py`, key model `zodiac`): Z32 power 0.00-0.15, no gain.
- **Key reuse.** Z32 read with the Z408 key: `.IF.LNI.WHDA.NGOAOESNBXLT.ETDIEI`; with the Z340 key:
  `YNSETAI.DOBP.AORTRDFNOPTIYRGAIWN`. Full reuse is clearly ruled out (winchxyz tested partial reuse:
  no evidence).

### 5. Zodiac's vocabulary (his letters, `zbot/words.py`)

Words from his letters (incl. his misspellings, e.g. PARADICE) boosted in the word model. Measured
with a split: vocabulary from half the letters, test texts from the other half.
Z32 with word-only scoring: 0.12 → 0.30 (boost 6). With letters+words: 0.10 → 0.05, no gain.
Fragile; one built-in test uses it. Note: these runs used a vocabulary that still included the
Z408 plaintext file; it is now excluded (Z408 calibration would otherwise be graded on its own words).

### 6. Measurement fixes made along the way

- Controls are now runs of **whole words** (Z13 and Z32 are complete messages); before, they were
  windows cut mid-word, which unfairly penalised word scoring.
- Earlier: controls reproduce the cipher's exact repeat profile (2026-11-19 fix).

### Conclusion and where effort should go

The ciphers resist because of information, not search. The bot now uses the best measured detector
for Z32 and reports power with every verdict. What could actually move things:

1. **New information**: better scans (Z13's three circles; Z32's glyph identities), more ciphertext
   under the same key, Zodiac's worksheets.
2. **Key-constraining hypotheses with a real basis**: a specific key-construction method, a crib
   tied to the map ("radians and inches"), or a link between Z13/Z32 and a known key. Each should
   be queued as one targeted test, not a broad search, to keep the multiple-testing cost low.
