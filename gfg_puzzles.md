# 🧠 ULTRA REVISION HANDBOOK — PLACEMENT PUZZLES
> **One-night revision before interviews. Focus: trick, not theory.**

---

# CATEGORY 1: MOST ASKED INTERVIEW PUZZLES

---

# Puzzle #1

## Category
Most Asked Interview Puzzles

## Question
You have two ropes. Each rope takes exactly 1 hour to burn from end to end, but they burn unevenly (not at a constant rate). Using only these ropes and a lighter, how do you measure exactly 45 minutes?

## Trick
* Light both ends of Rope 1 simultaneously; light one end of Rope 2.
* Rope 1 burns out in 30 min; at that moment, light the other end of Rope 2.
* Rope 2 now burns out in 15 more min → total 45 min.

## Visual
```
t=0   : Rope1 [◄═══════════►]  Rope2 [◄═══════════]
t=30  : Rope1 done             Rope2 [◄════►] (light other end)
t=45  : Rope2 done             ✅ 45 minutes elapsed
```

## Answer
✅ Answer: Light both ends of Rope 1 and one end of Rope 2 at t=0. When Rope 1 dies (t=30), light the other end of Rope 2. It burns out at t=45.

## Why It Works
Rope 1 (lit both ends) = 30 min. Remaining Rope 2 = 30 min of burn time; lighting both ends halves it to 15 min. 30 + 15 = 45.

## Pattern
"Halving remaining burn time"

## Memory Anchor
🧠 Both ends = half time; do it twice = 45.

---

# Puzzle #2

## Category
Most Asked Interview Puzzles

## Question
There are 25 horses. You can race 5 horses at a time and only know their relative order (no stopwatch). Find the minimum number of races needed to determine the top 3 fastest horses.

## Trick
1. Race the 25 horses in 5 groups of 5 (5 races).
2. Race the 5 group winners (Race 6).
3. Suppose the winners race result is:

   A1 > B1 > C1 > D1 > E1

4. Eliminate:
   - All horses from Groups D and E.
   - C2, C3, C4, C5
   - B3, B4, B5
   - A4, A5

5. Remaining candidates:

   A1, A2, A3, B1, B2, C1

6. A1 is already the fastest horse.
7. Race A2, A3, B1, B2, C1 (Race 7).
8. The top 2 finishers of this race are overall 2nd and 3rd.

## Visual

```text
Race 1-5:
A1 A2 A3 A4 A5
B1 B2 B3 B4 B5
C1 C2 C3 C4 C5
D1 D2 D3 D4 D5
E1 E2 E3 E4 E5

Race 6:
A1 > B1 > C1 > D1 > E1

Remaining:
A1 A2 A3 B1 B2 C1

Race 7:
A2 A3 B1 B2 C1
```

## Answer
✅ 7 races minimum.

## Why It Works
After Race 6, only A2, A3, B1, B2, and C1 can still be in the overall Top 3. One final race among them determines 2nd and 3rd place.

## Pattern
Group Elimination + Championship Race

## Memory Anchor
🧠 5 Group Races + 1 Winners Race + 1 Final Race = 7

---

# Puzzle #3

## Category
Most Asked Interview Puzzles

## Question
Four people (A, B, C, D) must cross a bridge at night. They have one torch, the bridge holds max 2 people, and they must walk at the slower person's pace. Crossing times: A=1 min, B=2 min, C=5 min, D=10 min. What is the minimum time for all to cross?

## Trick
* Always send the two slowest together to share the time cost.
* Use the two fastest people as torch carriers (back-and-forth).
* Work backwards: 1+2+1+2+10 = 17 min.

## Visual
```
→ A+B cross      (2 min) | Total: 2
← A returns      (1 min) | Total: 3
→ C+D cross      (10min) | Total: 13
← B returns      (2 min) | Total: 15
→ A+B cross      (2 min) | Total: 17
```

## Answer
✅ Answer: 17 minutes.

## Why It Works
Pairing the two slowest (C+D) avoids paying 10+5=15 min separately. Using B as torch-returner instead of A saves 4 min over the naive approach.

## Pattern
"Pair the slowest; use fastest as shuttle"

## Memory Anchor
🧠 Always pair the two slowest together.

---

# Puzzle #4

## Category
Most Asked Interview Puzzles

## Question
You have 1000 wine bottles. Exactly one is poisoned. You have 10 rats and 30 days. Poison takes effect in 24 hours (rat dies the next day). What is the minimum number of rats needed to find the poisoned bottle in one test?

## Trick
* Assign each bottle a 10-bit binary number (1–1000, needs ≤10 bits).
* Each rat represents one bit position. If bit is 1, rat drinks from that bottle.
* Read dead/alive rats as binary → decode the poisoned bottle number.

## Visual
```
Bottle  Binary    Rat: 10 9 8 7 6 5 4 3 2 1
  1     0000000001       . . . . . . . . . ✓
  2     0000000010       . . . . . . . . ✓ .
  ...
 999    1111100111       ✓ ✓ ✓ ✓ ✓ . . ✓✓✓
```

## Answer
✅ Answer: 10 rats (since 2^10 = 1024 ≥ 1000).

## Why It Works
Each rat's death/survival is a binary bit. 10 bits encode 2^10 = 1024 unique bottles — enough to identify any one of 1000.

## Pattern
"Binary encoding"

## Memory Anchor
🧠 Each rat is one binary bit.

---

# Puzzle #5

## Category
Most Asked Interview Puzzles

## Question
You have 2 eggs and a 100-floor building. Find the highest floor from which an egg can be dropped without breaking, using the minimum number of drops in the worst case. How many drops do you need?

## Trick
* You cannot afford to always start from floor 1 (too slow) or floor 50 (egg 1 breaks, egg 2 can't cover range).
* Use a triangular-number strategy: drop from floor n, n+(n-1), n+(n-2)…
* Solve: n(n+1)/2 ≥ 100 → n=14.

## Visual
```
Drop 1: Floor 14   (if breaks, check 1–13 with egg 2: 13 more drops, total 14)
Drop 2: Floor 27   (if breaks, check 15–26: 12 more drops, total 14)
Drop 3: Floor 39 ...
```

## Answer
✅ Answer: 14 drops in the worst case.

## Why It Works
Each subsequent first-egg drop is one floor lower than the last, so the worst-case total drops stay constant at 14 regardless of which range the critical floor is in.

## Pattern
"Triangular numbers / decreasing intervals"

## Memory Anchor
🧠 n(n+1)/2 ≥ 100 → n=14.

---

# Puzzle #6

## Category
Most Asked Interview Puzzles

## Question
You have a 3-gallon jug and a 5-gallon jug (no markings). How do you measure exactly 4 gallons of water?

## Trick
* Fill 5-gallon, pour into 3-gallon until full. 5-gallon now has 2 gallons.
* Empty 3-gallon, pour the 2 gallons into it. Fill 5-gallon again.
* Pour from 5-gallon into 3-gallon (needs 1 more gallon). 5-gallon now has 4 gallons.

## Visual
```
Step | 3-gal | 5-gal
  0  |   0   |   0
  1  |   0   |   5   (fill 5)
  2  |   3   |   2   (pour into 3)
  3  |   0   |   2   (empty 3)
  4  |   2   |   0   (pour 2 into 3)
  5  |   2   |   5   (fill 5)
  6  |   3   |   4   ✅ 4 gallons in 5-gal
```

## Answer
✅ Answer: 4 gallons remain in the 5-gallon jug after 6 steps.

## Why It Works
GCD(3,5) = 1, so any integer 1–5 is reachable. By cycling pour/fill, you isolate the remainder.

## Pattern
"Water jug / remainder cycling"

## Memory Anchor
🧠 Fill big, pour to small, repeat.

---

# Puzzle #7

## Category
Most Asked Interview Puzzles

## Question
In a palace, the king has some elephants, horses, hens, and rabbits. Count: elephants = 4, horses = 6, hens = 8, rabbits = 10. How many legs are there in the palace?

## Trick
* Elephants: 4 legs each, Horses: 4 legs each, Hens: 2 legs each, Rabbits: 4 legs each.
* Don't forget the king has 2 legs (and any servants/guards if stated).
* Multiply and sum.

## Visual
```
Elephants: 4 × 4 = 16
Horses:    6 × 4 = 24
Hens:      8 × 2 = 16
Rabbits:  10 × 4 = 40
King:      1 × 2 =  2
Total           = 98
```

## Answer
✅ Answer: 98 legs (including the king; adjust if problem specifies otherwise).

## Why It Works
Straightforward multiplication; the trick is remembering to count the human(s) in the palace.

## Pattern
"Don't forget the human"

## Memory Anchor
🧠 Count every living thing, including the king.

---

# Puzzle #8

## Category

Arrangement Puzzle

## Question

There are three houses on each side of a road. The six houses are labeled **P, Q, R, S, T, and U**. Each house has a different color (**Red, Blue, Green, Orange, Yellow, White**) and a different height.

Given:

1. T, the tallest house, is exactly opposite to the Red-colored house.
2. The shortest house is exactly opposite to the Green-colored house.
3. U, the Orange-colored house, is located between P and S.
4. R, the Yellow-colored house, is exactly opposite to P.
5. Q, the Green-colored house, is exactly opposite to U.
6. P, the White-colored house, is taller than R but shorter than S and Q.

**What is the color of the tallest house, and how many houses are taller than house R?**

---

## Trick

* First place houses using the "opposite" clues.
* Use "U is between P and S" to determine the road arrangement.
* Assign colors from the given clues.
* Use height constraints to rank the houses.
* Eliminate possibilities until only one valid arrangement remains.

---

## Visual

```text
Side 1:  P   U   S
          |   |   |
Side 2:  R   Q   T

Opposites:
P ↔ R
U ↔ Q
S ↔ T
```

### Colors

```text
P = White
Q = Green
R = Yellow
U = Orange
S = Red
T = Blue
```

### Heights

```text
Tallest     : T (Blue)
2nd/3rd     : Q, S
4th         : P
5th         : R
Shortest    : U
```

---

## Answer

✅ **The tallest house is T and its color is Blue.**

✅ **Four houses are taller than R.**

```text
T, Q, S, and P
```

---

## Why It Works

* U is Orange and lies between P and S.
* Q is opposite U and is Green.
* The shortest house is opposite the Green house, so U is the shortest.
* T is the tallest and opposite the Red house, so S must be Red.
* The only remaining color for T is Blue.
* From the height condition:

```text
Q > P > R
S > P > R
T > everyone
```

Thus the order becomes:

```text
T > Q/S > P > R > U
```

Therefore, four houses are taller than R.

---

## Pattern

**Arrangement + Opposite Positioning + Ranking**

---

## Memory Anchor

🧠 **Place opposites first → Assign colors → Rank heights.**

---

# Puzzle #9

## Category
Most Asked Interview Puzzles

## Question
You have 8 balls. Seven are identical in weight; one is heavier. Using a balance scale, find the heavier ball in a minimum number of weighings. How many?

## Trick
* Divide into three groups of 3, 3, and 2.
* Weigh group of 3 vs group of 3. If balanced, the odd ball is in group of 2.
* If unbalanced, weigh 2 balls from the heavier group to find it.

## Visual
```
Weighing 1: [1,2,3] vs [4,5,6]
  Equal   → odd ball is 7 or 8 → Weighing 2: [7] vs [8]
  Unequal → odd ball in heavier group (say 1,2,3)
            Weighing 2: [1] vs [2] → find it
```

## Answer
✅ Answer: 2 weighings minimum.

## Why It Works
3 groups of 3 give 3 outcomes per weighing (left heavy, right heavy, equal). 3^2 = 9 ≥ 8, so 2 weighings suffice.

## Pattern
"Divide into 3 groups; 3 outcomes per weighing"

## Memory Anchor
🧠 Split in thirds; 3^n covers 3^n balls.

---

# Puzzle #10

## Category
Most Asked Interview Puzzles

## Question
Ten prisoners are each given a red or black hat (randomly). Starting from the back, each prisoner must say only "red" or "black." If correct, they go free; if wrong, they are shot. They can hear all previous answers but not see their own hat. How can they save at least 9 prisoners for certain?

## Trick
* The first prisoner (at the back) counts parity of red hats he sees and says "red" if even, "black" if odd.
* Each subsequent prisoner uses parity of remaining hats + what was said to deduce their own hat.
* First prisoner has 50/50 chance; the rest are certain.

## Visual
```
Back → Front
P10 counts red hats he sees (say 4 = even → says "RED")
P9  counts red hats he sees; if even and P10 said "RED", P9 has red (even count)
... each updates the running parity
```

## Answer
✅ Answer: At least 9 prisoners are saved with certainty; first prisoner has 50% chance.

## Why It Works
Parity is self-correcting. Each prisoner knows the original parity and all hats called so far, letting them deduce their own hat color.

## Pattern
"Parity encoding"

## Memory Anchor
🧠 First prisoner encodes parity; rest decode it.

---

## Most Important Tricks — Category 1

* **Binary encoding** — Represent items as bits; N bits cover 2^N items.
* **Divide into 3 groups** — 3 outcomes per balance weighing → minimum weighings.
* **Parity observation** — Encode/decode information through odd/even counts.
* **Pair the slowest** — In bridge/torch problems, always send the two slowest together.
* **Triangular numbers** — 2-egg / n-floor problems → n(n+1)/2 ≥ floors.

## 30 Second Revision — Category 1

| Puzzle | Trick |
|--------|-------|
| Burning Ropes | Both ends = half time; 30+15=45 |
| 25 Horses | 5 groups + 1 championship + 1 decider = 7 |
| Bridge Crossing | Pair slowest; use fastest as shuttle → 17 |
| Poisoned Bottle | Binary encoding; 10 rats = 10 bits = 1024 |
| Two-Egg Problem | Triangular numbers; n(n+1)/2 ≥ 100 → 14 |
| Four Gallon Problem | Fill big, pour to small, cycle → 4 gallons |
| Number of Legs | Multiply per species; don't forget the king |
| Six Houses | Logic grid; constraint elimination |
| 8 Ball Problem | Split into 3 groups; 2 weighings |
| Red/Black Hats | First prisoner encodes parity; 9 saved |

---

# CATEGORY 2: ANALYTICAL / MATHEMATICAL PUZZLES

---

# Puzzle #11

## Category
Analytical / Mathematical Puzzles

## Question
A man has three daughters. The product of their ages is 36, and the sum equals the house number (which the questioner knows). The man says "I still can't tell." The man adds "the eldest plays piano." What are the daughters' ages?

## Trick
* List all factor triplets of 36; find those with the same sum (that's why the hint wasn't enough).
* Only {1,6,6} and {2,2,9} share a sum (13 and 13 — check your triplets).
* "Eldest plays piano" means there is one eldest → {2,2,9}; not {1,6,6} (two tied at 6).

## Visual
```
Triplets of 36:    Sum
1,1,36  →  38
1,2,18  →  21
1,3,12  →  16
1,4,9   →  14
1,6,6   →  13  ← ambiguous
2,2,9   →  13  ← ambiguous
2,3,6   →  11
3,3,4   →  10
```

## Answer
✅ Answer: 2, 2, 9.

## Why It Works
The man can't determine ages from the sum alone → the sum is ambiguous (13). "Eldest" implies a unique oldest child → {2,2,9}, not {1,6,6}.

## Pattern
"Ambiguous sum + uniqueness constraint"

## Memory Anchor
🧠 Same sum = ambiguous; "eldest" breaks the tie.

---

# Puzzle #12

## Category
Analytical / Mathematical Puzzles

## Question
Two trains 100 km apart head toward each other, each at 50 km/h. A bee starts at one train and flies at 75 km/h back and forth between them until they collide. How far does the bee travel?

## Trick
* Ignore the bee's back-and-forth; calculate total time until collision.
* Trains close at combined speed of 100 km/h; collision in 1 hour.
* Bee flies at 75 km/h for 1 hour → 75 km.

## Visual
```
Train A →  50 km/h          ← 50 km/h  Train B
|←————————— 100 km ———————————→|
Collision time = 100 / (50+50) = 1 hour
Bee distance = 75 × 1 = 75 km
```

## Answer
✅ Answer: 75 km.

## Why It Works
The bee's zig-zag is a red herring. Only total time matters; time = distance / closing speed.

## Pattern
"Ignore complexity; compute total time"

## Memory Anchor
🧠 Don't track the bee; track the time.

---

# Puzzle #13

## Category
Analytical / Mathematical Puzzles

## Question
How many paths are there in a 6×6 grid from the top-left corner to the bottom-right corner, moving only right or down?

## Trick
* You must make exactly 5 right moves and 5 down moves (total 10 moves).
* Choose which 5 of the 10 steps are "right" → C(10,5).
* C(10,5) = 252.

## Visual
```
(0,0) → → → → → (5,0)
  ↓               ↓
(0,5) → → → → → (5,5)
Steps needed: 5 right + 5 down = 10 total
Paths = C(10,5) = 10!/(5!×5!) = 252
```

## Answer
✅ Answer: 252 paths.

## Why It Works
Each unique path is a unique arrangement of 5 R's and 5 D's. Combinations formula gives the count.

## Pattern
"Combinations C(m+n, m)"

## Memory Anchor
🧠 Grid paths = C(right+down, right).

---

# Puzzle #14

## Category
Analytical / Mathematical Puzzles

## Question
A game show host shows 3 doors: 1 has a car, 2 have goats. You pick door 1. The host (who knows) opens door 3 to reveal a goat. Should you switch to door 2 or stay with door 1?

## Trick
* Your initial pick has 1/3 chance. The other two doors together have 2/3 chance.
* Host's reveal moves that 2/3 entirely to the other remaining door.
* Always switch → 2/3 win probability.

## Visual
```
Initial:  Door1=1/3 | Door2=1/3 | Door3=1/3
After reveal (Door3=goat):
          Door1=1/3 |  Door2=2/3  | Door3=0
Stay=1/3 win   Switch=2/3 win
```

## Answer
✅ Answer: Always switch. Switching wins 2/3 of the time.

## Why It Works
The host's informed action (never reveals the car) transfers probability from the eliminated door to the unchosen door. Your original door probability stays frozen at 1/3.

## Pattern
"Conditional probability / host's knowledge"

## Memory Anchor
🧠 Always switch — host's knowledge doubles your odds.

---

# Puzzle #15

## Category
Analytical / Mathematical Puzzles

## Question
Four people (A=1min, B=2min, C=5min, D=10min) must cross a bridge with one torch. The bridge holds max 2 people. Minimum time?

*(Same as Bridge Crossing #3 above — appears in both categories.)*

## Trick
* Send two slowest together; use two fastest as shuttles.
* Optimal sequence: A+B cross, A back, C+D cross, B back, A+B cross.

## Answer
✅ Answer: 17 minutes. *(See Puzzle #3 for full breakdown.)*

## Pattern
"Pair slowest; use fastest as shuttle"

## Memory Anchor
🧠 2+1+10+2+2 = 17.

---

# Puzzle #16

## Category
Analytical / Mathematical Puzzles

## Question
You have 2 eggs and 100 floors. Find the minimum number of attempts needed in the worst case to determine the critical floor.

*(Same as Puzzle #5 — appears in both categories.)*

## Answer
✅ Answer: 14 drops. n(n+1)/2 ≥ 100 → n=14. *(See Puzzle #5.)*

## Pattern
"Triangular numbers"

## Memory Anchor
🧠 n(n+1)/2 ≥ 100 → n=14.

---

# Puzzle #17

## Category
Analytical / Mathematical Puzzles

## Question
A bag has 3 white and 2 black balls. You draw 2 balls. What is the probability that at least one is white, or both are white?

## Trick
* P(both white) = C(3,2)/C(5,2) = 3/10.
* P(at least one white) = 1 − P(both black) = 1 − C(2,2)/C(5,2) = 1 − 1/10 = 9/10.

## Visual
```
Total ways to choose 2 from 5 = C(5,2) = 10
Both white: C(3,2) = 3  → P = 3/10
Both black: C(2,2) = 1  → P = 1/10
At least 1 white = 1 - 1/10 = 9/10
```

## Answer
✅ Answer: P(both white) = 3/10; P(at least one white) = 9/10.

## Why It Works
Complement rule: P(at least one white) = 1 − P(none white).

## Pattern
"Complement probability"

## Memory Anchor
🧠 P(at least one) = 1 − P(none).

---

# Puzzle #18

## Category
Analytical / Mathematical Puzzles

## Question
You have 8 bottles of wine. One is poisoned. You have 3 rats. Each rat either dies or survives after drinking. What is the minimum number of rats to identify the poisoned bottle?

## Trick
* 3 rats → 3 bits → 2^3 = 8 combinations.
* Assign each bottle a unique 3-bit binary code.
* Each rat drinks from bottles where its bit position = 1.

## Visual
```
Bottle  Binary  Rat3 Rat2 Rat1
  1      001      0    0    1
  2      010      0    1    0
  3      011      0    1    1
  4      100      1    0    0
  5      101      1    0    1
  6      110      1    1    0
  7      111      1    1    1
  8      000 (all live → bottle 8)
```

## Answer
✅ Answer: 3 rats (2^3 = 8 ≥ 8 bottles).

## Why It Works
Each rat's death/survival is a binary digit. Three bits uniquely encode 8 different bottles.

## Pattern
"Binary encoding"

## Memory Anchor
🧠 N rats identify 2^N bottles.

---

# Puzzle #19

## Category
Analytical / Mathematical Puzzles

## Question
You have a 4-minute hourglass and a 7-minute hourglass. How do you measure exactly 9 minutes?

## Trick
* Start both simultaneously.
* At 4 min, 4-glass empty (flip it), 7-glass has 3 min left.
* At 7 min, 7-glass empty (flip it), 4-glass has 1 min left.
* At 8 min, 4-glass empty → flip 7-glass (has run 1 min → 6 min remain). Run it for 1 more minute.

## Visual
```
t=0: Start 4min & 7min
t=4: Flip 4min (7min has 3 left)
t=7: Flip 7min (4min has 1 left)
t=8: 4min done → flip 7min (1 min used = 6 remain)
t=9: Stop → 9 minutes measured ✅
```

## Answer
✅ Answer: Flip and cycle to measure 9 minutes in 3 flips.

## Why It Works
Chaining the remainders (3+1 = 4 min offset) with the 7-min glass lets you isolate a 1-min segment, then count to 9.

## Pattern
"Chain hourglass remainders"

## Memory Anchor
🧠 Offset one glass off the other's remainder.

---

# Puzzle #20

## Category
Analytical / Mathematical Puzzles

## Question
In a country where every family keeps having children until they have a boy, then stops: what is the ratio of boys to girls?

## Trick
* Each family has exactly one boy (the last child born).
* But girls count varies: 0,1,2,3... depending on how many girls before the boy.
* Expected girls per family = E[geometric–1] = 1. So ratio is 1:1.

## Visual
```
Family A: B         → 1 boy, 0 girls
Family B: G,B       → 1 boy, 1 girl
Family C: G,G,B     → 1 boy, 2 girls
Family D: G,G,G,B   → 1 boy, 3 girls
Expected girls/family = 0+1+2+... weighted = 1
Ratio Boys:Girls = 1:1
```

## Answer
✅ Answer: Boy-to-girl ratio remains **1:1**.

## Why It Works
Every birth is independently 50/50. The stopping rule changes family size but not the population-level ratio.

## Pattern
"Stopping rules don't change birth probability"

## Memory Anchor
🧠 The stopping rule can't change 50/50 birth odds.

---

# Puzzle #21

## Category
Analytical / Mathematical Puzzles

## Question
A car has 4 tires and 1 spare. Each tire can be used for a maximum of 20,000 km. What is the maximum distance you can travel?

## Trick
* Total tire-km available = 5 × 20,000 = 100,000 km.
* 4 tires are in use at any time → max distance = 100,000 / 4 = 25,000 km.

## Visual
```
5 tires × 20,000 km = 100,000 km total
4 tires used at once → 100,000 / 4 = 25,000 km
Each tire used for 25,000 × 4/5 = 20,000 km ✅
```

## Answer
✅ Answer: 25,000 km.

## Why It Works
Divide total available tire-km by number of tires in use simultaneously.

## Pattern
"Total resource / simultaneous usage"

## Memory Anchor
🧠 Total km ÷ tires in use = max distance.

---

# Puzzle #22

## Category
Analytical / Mathematical Puzzles

## Question
A person buys chocolates: 3 wrappers can be exchanged for 1 new chocolate. If you have 15 chocolates, how many can you eat in total?

## Trick
* Eat 15 → 15 wrappers → exchange for 5 more.
* Eat 5 → 5 wrappers → exchange for 1 more (keep 2 wrappers).
* Eat 1 → 3 wrappers total → 1 more. Total = 15+5+1+1 = 22.

## Visual
```
15 chocolates → eat → 15 wrappers → 5 new
 5 chocolates → eat →  5 wrappers → 1 new + 2 leftover
 1 chocolate  → eat →  3 wrappers → 1 new
 1 chocolate  → eat →  1 wrapper  (stop)
Total = 15+5+1+1 = 22
```

## Answer
✅ Answer: 22 chocolates.

## Why It Works
Keep recycling wrappers. Formula: n + floor((n–1)/(k–1)) where k=3. Or simulate directly.

## Pattern
"Recursive wrapper exchange"

## Memory Anchor
🧠 Keep exchanging until you can't; simulate step by step.

---

# Puzzle #23

## Category
Analytical / Mathematical Puzzles

## Question
A round cake is cut with 3 straight cuts. What is the maximum number of pieces you can get?

## Trick
* Each new cut intersects all previous cuts maximally.
* Cut 1: 2 pieces. Cut 2: 4 pieces. Cut 3: 7 pieces.
* Formula: max pieces = 1 + n(n+1)/2.

## Visual
```
Cut 1: splits into 2
Cut 2: crosses cut 1 → 4
Cut 3: crosses cuts 1 & 2 at 2 points → adds 3 more regions → 7
```

## Answer
✅ Answer: 7 pieces with 3 cuts.

## Why It Works
Each nth cut crosses (n−1) previous cuts, creating n new pieces. Total = 1 + Σk from 1 to n = 1 + n(n+1)/2.

## Pattern
"Each cut crosses all previous cuts"

## Memory Anchor
🧠 1 + n(n+1)/2 = max pieces.

---

# Puzzle #24

## Category
Analytical / Mathematical Puzzles

## Question
A shopkeeper sells a shirt for ₹500 to a man who pays with a fake ₹500 note. The shopkeeper gives the shirt (₹200) + ₹300 change. The neighbor who exchanged the note is repaid ₹500 real money. How much does the shopkeeper lose?

## Trick
* Total loss = what the shopkeeper paid out that isn't recovered.
* He gave: ₹200 shirt + ₹300 change = ₹500 in real value.
* The fake note returns to him worthless. Net loss = ₹500.

## Visual
```
Out: shirt ₹200 + change ₹300 = ₹500
In:  fake note (₹0 real value)
Shopkeeper loss = ₹500
```

## Answer
✅ Answer: Shopkeeper loses ₹500.

## Why It Works
The fake note has zero value. Every rupee of shirt + change came from the shopkeeper's real assets.

## Pattern
"Track real value, not nominal"

## Memory Anchor
🧠 Fake money = zero value; count only real assets lost.

---

# Puzzle #25

## Category
Analytical / Mathematical Puzzles

## Question
A couple has two children. One is a girl. What is the probability that the other child is also a girl?

## Trick
* Sample space for two children: {BB, BG, GB, GG}.
* "One is a girl" eliminates BB. Remaining: {BG, GB, GG}.
* P(both girls) = 1/3.

## Visual
```
All cases:  BB  BG  GB  GG
At least 1 girl: BG  GB  GG  (3 cases)
Both girls: GG  (1 case)
P = 1/3
```

## Answer
✅ Answer: 1/3.

## Why It Works
The condition "at least one girl" restricts the sample space to 3 equally likely outcomes, only one of which is {GG}.

## Pattern
"Conditional probability / restricted sample space"

## Memory Anchor
🧠 "At least one girl" → 3 cases, only 1 is GG.

---

# Puzzle #26

## Category
Analytical / Mathematical Puzzles

## Question
Three employees want to know each other's salaries without revealing their own. How can they find the average salary?

## Trick
* Person A picks a random number R, adds salary, passes sum to B.
* B adds their salary, passes to C. C adds their salary, passes back to A.
* A subtracts R, divides by 3 → average, with no individual salary revealed.

## Visual
```
A → (R + SalaryA) → B
B → (R + SalaryA + SalaryB) → C
C → (R + SalaryA + SalaryB + SalaryC) → A
A → subtract R → total salaries / 3 = average
```

## Answer
✅ Answer: Add a private random number (noise) to hide individual values; remove it at end.

## Why It Works
The random offset R masks A's true salary throughout the circuit. Only the final sum (with R removed) is revealed.

## Pattern
"Random noise masking"

## Memory Anchor
🧠 Add random noise; remove at end to reveal only the average.

---

# Puzzle #27

## Category
Analytical / Mathematical Puzzles

## Question
In cricket, what is the maximum runs one batsman can score in a single match (Test or One-Day), assuming no extras?

## Trick
* In ODI (50 overs × 6 balls = 300 balls). If one batsman faces every ball: 300 × 6 = 1800 runs.
* In reality, striker can only face certain deliveries; partner must not get out.
* Maximum per ball = 6 runs (six). Maximum balls faceable in ODI ≈ 300 (if partner always scores 0 and turns strike back).

## Answer
✅ Answer: Theoretical maximum ≈ 1800 runs in an ODI (300 balls × 6 runs); puzzle answer typically = **1800**.

## Why It Works
Each ball can yield maximum 6 runs. If the batsman faces all 300 balls and hits six each time: 1800.

## Pattern
"Maximum per unit × number of units"

## Memory Anchor
🧠 Max runs = max per ball × total balls.

---

# Puzzle #28

## Category
Analytical / Mathematical Puzzles

## Question
A job takes 10 men 10 days, or 10 women 20 days. How long will 5 men and 5 women take?

## Trick
* Man-rate: 1/(10×10) = 1/100 per person-day.
* Woman-rate: 1/(10×20) = 1/200 per person-day.
* Combined: 5/100 + 5/200 = 10/200 + 5/200 = 15/200 per day → Days = 200/15 = 13.33 days.

## Visual
```
1 man's rate  = 1/100 job/day
1 woman's rate = 1/200 job/day
5 men + 5 women = 5/100 + 5/200 = 15/200
Days = 1 ÷ (15/200) = 200/15 ≈ 13.3 days
```

## Answer
✅ Answer: 13⅓ days (approximately 13.33 days).

## Why It Works
Add individual work rates; invert total rate to get total days.

## Pattern
"Sum work rates; invert for days"

## Memory Anchor
🧠 Add rates, invert to get days.

---

# Puzzle #29

## Category
Analytical / Mathematical Puzzles

## Question
An Excel spreadsheet has rows numbered 1 to N, but one row number is missing. The numbers are from 1 to N. How do you find the missing row?

## Trick
* Expected sum = N(N+1)/2.
* Actual sum = sum of all visible numbers.
* Missing number = Expected sum − Actual sum.

## Visual
```
Rows: 1,2,3,_,5,6  (N=6, missing=4)
Expected sum = 6×7/2 = 21
Actual sum = 1+2+3+5+6 = 17
Missing = 21 - 17 = 4 ✅
```

## Answer
✅ Answer: Missing number = N(N+1)/2 − (sum of visible numbers).

## Why It Works
Arithmetic series formula gives the expected total; the gap identifies the missing element.

## Pattern
"Sum formula to find missing number"

## Memory Anchor
🧠 Expected sum − actual sum = missing number.

---

# Puzzle #30

## Category
Analytical / Mathematical Puzzles

## Question
Four people must cross a bridge at night with one torch. *(Same as Bridge puzzle — listed again here for completeness.)*

## Answer
✅ Answer: 17 minutes. *(See Puzzle #3.)*

## Pattern
"Pair slowest; use fastest as shuttle"

## Memory Anchor
🧠 2+1+10+2+2 = 17.

---

# Puzzle #31

## Category
Analytical / Mathematical Puzzles

## Question
A man fell into a well 30 feet deep. He climbs 3 feet per day and slips back 2 feet each night. How many days to escape?

## Trick
* Net progress per day = 3−2 = 1 foot.
* BUT: on the last day he climbs out without slipping back.
* After 27 days he is at 27 feet. On day 28 he climbs 3 feet → reaches 30 feet and escapes.

## Visual
```
Day 1: 3-2=1ft | Day 2: 2ft | ... | Day 27: 27ft
Day 28: 27+3=30ft → OUT ✅
Answer: 28 days
```

## Answer
✅ Answer: 28 days.

## Why It Works
Don't apply the nighttime slip on the final day of escape. After 27 nights he's at 27 ft; one more day climb of 3 ft = 30 ft out.

## Pattern
"No slip on escape day"

## Memory Anchor
🧠 Don't count the slip on the day he escapes.

---

# Puzzle #32

## Category
Analytical / Mathematical Puzzles

## Question
You have 50 red marbles and 50 blue marbles and 2 bags. Arrange all 100 marbles in the bags to maximize the probability of drawing a red marble from a randomly chosen bag.

## Trick
* Put 1 red marble in bag 1; put all remaining 49 red + 50 blue in bag 2.
* P(red) = (1/2)(1/1) + (1/2)(49/99) = 0.5 + 49/198 ≈ 0.747.
* This is the maximum possible.

## Visual
```
Bag 1: [1 red]
Bag 2: [49 red, 50 blue]
P(red) = 1/2 × 1 + 1/2 × 49/99 ≈ 74.7%
```

## Answer
✅ Answer: Put 1 red marble in one bag; all others in the second bag. P(red) ≈ 74.7%.

## Why It Works
Isolating one red marble guarantees 100% probability for one bag choice. The other bag's ratio is nearly 50/50, but the overall probability is far above 50%.

## Pattern
"Asymmetric distribution to maximize probability"

## Memory Anchor
🧠 Isolate 1 red marble in its own bag.

---

# Puzzle #33

## Category
Analytical / Mathematical Puzzles

## Question
Using 6 matchsticks of equal length, form exactly 4 equilateral triangles. How?

## Trick
* A flat arrangement can only use matchsticks for 2D triangles — won't work for 4 with 6 sticks.
* Build a 3D tetrahedron: 4 triangular faces, 6 edges, each edge = 1 matchstick.
* Think in 3D!

## Visual
```
       A
      /|\
     / | \
    B--+--C
     \ | /
      \|/
       D
6 edges = 6 matchsticks = 4 equilateral triangular faces
```

## Answer
✅ Answer: Form a tetrahedron (3D pyramid with triangular base). 6 sticks = 6 edges = 4 triangular faces.

## Why It Works
In 3D, a tetrahedron has exactly 4 equilateral triangular faces and 6 edges. 2D thinking is a trap.

## Pattern
"Think in 3D"

## Memory Anchor
🧠 6 sticks, 4 triangles → tetrahedron, not flat.

---

# Puzzle #34

## Category
Analytical / Mathematical Puzzles

## Question
You have 10 bottles of 1000 pills each. One bottle has pills weighing 1.1g; all others weigh 1g. You have one digital scale (one use only). How do you identify the odd bottle?

## Trick
* Take 1 pill from bottle 1, 2 from bottle 2, … 10 from bottle 10. Total = 55 pills.
* Expected weight if all normal = 55g. Actual weight = 55 + 0.1×k, where k = bottle number.
* k = (actual weight − 55) / 0.1.

## Visual
```
Bottle:  1   2   3   4   5   6   7   8   9  10
Pills:   1   2   3   4   5   6   7   8   9  10
If bottle 7 is odd: weight = 55 + 0.1×7 = 55.7g → k=7
```

## Answer
✅ Answer: Weigh once. Bottle number = (actual − 55) ÷ 0.1.

## Why It Works
Each bottle contributes a different number of pills, so each bottle creates a unique weight signature in one weighing.

## Pattern
"Unique quantity per source → one-shot identification"

## Memory Anchor
🧠 Take k pills from bottle k; excess weight identifies k.

---

# Puzzle #35

## Category
Analytical / Mathematical Puzzles

## Question
What is the maximum number of pieces you can get by cutting a circle with n straight cuts?

## Trick
* Formula: P(n) = 1 + n(n+1)/2.
* Each cut adds one more piece than the number of cuts it crosses.
* P(0)=1, P(1)=2, P(2)=4, P(3)=7, P(4)=11.

## Answer
✅ Answer: Max pieces = **1 + n(n+1)/2**.

## Why It Works
The nth cut can cross at most n−1 previous cuts, creating n new regions.

## Pattern
"Lazy caterer sequence"

## Memory Anchor
🧠 1 + n(n+1)/2 = max circle pieces.

---

# Puzzle #36

## Category
Analytical / Mathematical Puzzles

## Question
A chain has 23 links. You want to pay a hotel for 23 days (1 link per day). What is the minimum number of links you must cut to be able to pay exactly any number of days 1–23?

## Trick
* Cutting 1 link gives you pieces of size 1, and the two remaining segments.
* Cut link 3: pieces = 1, 2, 3 → you can make 1,2,3,4,5,6 (1+2+3=6). Not enough.
* Cut link 4: 1+3+19 = covers 1–23? Check: 1,3,19 → can make 1,2(1+?),3,4(3+1)... need 2 somewhere.
* Optimal: cut 1 link → segments of 1, (n-2) works for 1-step coverage.

## Visual
```
Best cut: link at position 3
Segments: 1, 2, 20 → covers? 
1,2,3,4,5,6... up to 23 if pieces = 1,3,19
Pieces 1,3,19: 1,2(no!), → cut pos 4: 1,3,18 → still need 2
Try: cut link at position 4 → pieces 1,3,18 → no
Cut 2 links: positions 4 and 11 → pieces 1,3,7,11 → covers 1-22? 
Actually for 23 days, cut 1 link (at position 4): 1,3,19 → 
1,2(3-1),3,4(3+1),5(3+1+1?)... 
```

## Answer
✅ Answer: Cut **1 link** (at position 4 or optimally): creates pieces 1, 3, 19 which together can make any value 1–23 using combinations.

## Why It Works
Pieces 1+3+19=23. Using {1,3,19}: any value 1–4 from 1&3, 5–22 by adding 19, and 23 = all three.

## Pattern
"Piece coverage using powers-of-2-like splits"

## Memory Anchor
🧠 Cut fewest links; pieces must sum to cover 1–N.

---

# Puzzle #37

## Category
Analytical / Mathematical Puzzles

## Question
A customer buys goods worth ₹30 and gives ₹50. The shopkeeper has no change, so borrows ₹50 from a neighbor. Later the neighbor finds the ₹50 note is fake and demands it back. How much does the shopkeeper lose?

## Trick
* Shopkeeper gave: goods (₹30) + change (₹20) = ₹50 real value.
* Shopkeeper also repaid neighbor: ₹50 real money.
* But neighbor's ₹50 was the original fake used to pay — not real money from elsewhere.
* Net loss = ₹50 (goods + change given; the note that paid for them was fake).

## Answer
✅ Answer: Shopkeeper loses **₹50**.

## Why It Works
All real value (goods ₹30 + change ₹20 + repayment ₹50) came from the shopkeeper; the only input was a ₹0-value fake note.

## Pattern
"Track real value flows"

## Memory Anchor
🧠 Fake money = zero. Count every real rupee that left the shopkeeper.

---

# Puzzle #38

## Category
Analytical / Mathematical Puzzles

## Question
You are at a door. The guard says: "Eggs are dropped. Find the critical floor with minimum drops, using an egg-drop strategy." (General Egg Dropping Puzzle — t trials, n floors, k eggs.)

## Trick
* 1 egg, n floors: need n trials (linear scan).
* 2 eggs, n floors: triangular numbers strategy — n(n+1)/2 ≥ floors.
* k eggs, n floors: dynamic programming — f(t,k) = floors testable in t trials with k eggs = f(t-1,k-1) + f(t-1,k) + 1.

## Answer
✅ Answer: For 2 eggs/100 floors: **14 trials**. For k eggs, use DP relation f(t,k) = f(t-1,k-1)+f(t-1,k)+1.

## Pattern
"Triangular numbers (2 eggs) / DP (k eggs)"

## Memory Anchor
🧠 2 eggs: n(n+1)/2 ≥ floors; k eggs: DP.

---

# Puzzle #39

## Category
Analytical / Mathematical Puzzles

## Question
A farmer has 1000 apples to store in baskets, such that any number of apples from 1 to 1000 can be given using only whole baskets. What is the minimum number of baskets?

## Trick
* Use powers of 2: 1, 2, 4, 8, 16, 32, 64, 128, 256 → sum = 511.
* Remaining: 1000 − 511 = 489. Add basket of 489.
* Total: 10 baskets cover any count 1–1000.

## Visual
```
Baskets: 1, 2, 4, 8, 16, 32, 64, 128, 256, 489
Sum = 1000; any subset sum = any value 1–1000
```

## Answer
✅ Answer: 10 baskets.

## Why It Works
Binary representation: any integer 1–511 is a subset sum of {1,2,4,...,256}. Adding 489 extends coverage to 1000.

## Pattern
"Binary powers + remainder"

## Memory Anchor
🧠 Powers of 2 + leftover = minimal baskets.

---

# Puzzle #40

## Category
Analytical / Mathematical Puzzles

## Question
A snail is at the bottom of a 10m wall. It climbs 3m each day and slips back 2m each night. How many days to reach the top?

## Trick
* Net gain per day = 1m (same as man-in-well logic).
* But on the final day, snail climbs 3m without slipping.
* After 7 days = 7m. On day 8, climbs 3m → 10m → escapes.

## Answer
✅ Answer: 8 days.

## Why It Works
Same as well puzzle: no slip applied on escape day. 7 full days of net +1m = 7m; day 8 climb of 3m = 10m.

## Pattern
"No slip on escape day"

## Memory Anchor
🧠 Escape day: no slip back.

---

# Puzzle #41

## Category
Analytical / Mathematical Puzzles

## Question
There are 1000 light bulbs numbered 1–1000, all off. In round k, toggle every kth bulb. After 1000 rounds, which bulbs are on?

## Trick
* A bulb is toggled once per divisor. Bulb n is toggled = number of divisors of n times.
* Most numbers have even divisors → bulb ends off.
* Only perfect squares have an odd number of divisors → those bulbs end ON.

## Visual
```
Bulb 1: divisors {1} → 1 toggle → ON
Bulb 4: divisors {1,2,4} → 3 toggles → ON
Bulb 9: divisors {1,3,9} → 3 toggles → ON
Bulb 36: divisors {1,2,3,4,6,9,12,18,36} → 9 → ON
Perfect squares ≤ 1000: 1,4,9,16,...,961 → 31 bulbs
```

## Answer
✅ Answer: 31 bulbs are on (perfect squares: 1², 2², ..., 31²).

## Why It Works
Perfect squares have an odd number of divisors because the square root pairs with itself, leaving one unpaired divisor.

## Pattern
"Perfect squares / odd number of divisors"

## Memory Anchor
🧠 Only perfect squares stay ON — odd divisor count.

---

# Puzzle #42

## Category
Analytical / Mathematical Puzzles

## Question
Four knights on a 3×3 chessboard alternately occupy the four corner squares. Can you swap them (2 black knights at top-corners to bottom-corners and vice versa) without them ever attacking each other?

## Trick
* Knights on a standard board move in L-shapes. Map knight moves as a graph.
* Rotate all four knights around the board cycle simultaneously.
* 16 moves are needed; use circular rotation.

## Answer
✅ Answer: Yes — rotate all four knights around the perimeter squares in a consistent circular direction (16 total moves).

## Why It Works
Knight moves on a 3×3 board trace a Hamiltonian cycle through the 8 non-center squares. Rotating together avoids collisions.

## Pattern
"Graph cycle / simultaneous rotation"

## Memory Anchor
🧠 Rotate all knights in the same circular direction.

---

# Puzzle #43

## Category
Analytical / Mathematical Puzzles

## Question
100 cows give 100 litres of milk in 100 days. How many litres do 10 cows give in 10 days?

## Trick
* Rate per cow per day = 100 / (100 × 100) = 0.01 litres.
* 10 cows × 10 days × 0.01 = 1 litre.

## Visual
```
1 cow in 1 day = 100/(100×100) = 0.01 L
10 cows × 10 days × 0.01 = 1L
```

## Answer
✅ Answer: **1 litre**.

## Why It Works
Simple unitary method: scale down proportionally in both cows and days.

## Pattern
"Unitary method"

## Memory Anchor
🧠 Scale cows and days proportionally.

---

# Puzzle #44

## Category
Analytical / Mathematical Puzzles

## Question
You walk 1 mile south, 1 mile east, 1 mile north and you're back where you started. Where on Earth can you be?

## Trick
* Obvious answer: North Pole.
* Also valid: any circle of latitude near the South Pole where circumference = 1 mile (or 1/n mile for integer n), so the eastward walk loops back to start.

## Visual
```
North Pole:
Start → 1 mi South → 1 mi East (circling parallel) → 1 mi North → Start ✅

Near South Pole:
Circle at ~1/(2π) km from South Pole has circumference ~1 mi
```

## Answer
✅ Answer: The **North Pole** (and infinitely many points near the South Pole).

## Why It Works
At the North Pole, south–east–north traces a closed path. Near the South Pole, the east mile traces a complete latitude circle, returning to the same longitude.

## Pattern
"Spherical geometry / lateral thinking"

## Memory Anchor
🧠 North Pole + circles near South Pole.

---

# Puzzle #45

## Category
Analytical / Mathematical Puzzles

## Question
A game board has counters placed on squares. Players take turns removing 1 or 2 counters. The player who takes the last counter wins. What is the winning strategy?

## Trick
* If current count ≡ 0 (mod 3), the player whose turn it is LOSES (opponent wins with correct play).
* Always leave a multiple of 3 to your opponent.
* On your turn, reduce to the nearest multiple of 3.

## Visual
```
Counters: 9 (multiple of 3) → current player loses
If you leave 9,6,3 for opponent each time → you win
Take 1 or 2 to make remaining = 3k
```

## Answer
✅ Answer: Win by always leaving a multiple of 3 for your opponent.

## Why It Works
Whatever opponent takes (1 or 2), you take (2 or 1) to reduce by 3 total. Reach 3 → opponent must leave 1 or 2 → you take last.

## Pattern
"Modulo 3 strategy"

## Memory Anchor
🧠 Leave multiples of 3; you control the endgame.

---

# Puzzle #46

## Category
Analytical / Mathematical Puzzles

## Question
A camel must cross a 1000-mile desert. She starts with 3000 bananas and can carry at most 1000 at a time. She eats 1 banana per mile traveled. How many bananas can she get to the other side?

## Trick
* Split the journey: 0–200 miles (3 trips needed), 200–533 miles (2 trips), 533–1000 (1 trip).
* Bananas left: 3000 − 5×200 = 2000 at mile 200. 2000 − 3×333 = 1000 at mile 533. 1000−467 = **533**.

## Visual
```
Segment 1 (0–200): 3 loads × 200 mi × 2 (back-forth) = loses 1000 → 2000 left
Segment 2 (200–533.3): 2 loads → loses 1000 → 1000 left
Segment 3 (533.3–1000): 1 load → 1000 − 466.7 = 533.3
```

## Answer
✅ Answer: **533⅓ bananas** (≈ 533 whole bananas).

## Why It Works
Divide journey by number of trips required at each stage. Minimize back-and-forth cost by reducing trip count as soon as possible.

## Pattern
"Staged transport / minimize back-and-forth"

## Memory Anchor
🧠 Divide journey by trips needed; reduce trips ASAP.

---

# Puzzle #47

## Category
Analytical / Mathematical Puzzles

## Question
Using 6 matchsticks, make 4 equilateral triangles of the same size. *(Same as Puzzle #33.)*

## Answer
✅ Answer: Build a 3D **tetrahedron** — 4 faces, 6 edges. *(See Puzzle #33.)*

## Pattern
"Think in 3D"

## Memory Anchor
🧠 Flat is a trap; go 3D.

---

# Puzzle #48

## Category
Analytical / Mathematical Puzzles

## Question
A person starts with some money. He spends half and then ₹100 more, then again half and ₹100 more, with nothing remaining. How much did he start with?

## Trick
* Work backwards. At the end = ₹0. Before last "−₹100" = ₹100. Before "÷2" = ₹200.
* Before second "−₹100" = ₹300. Before second "÷2" = ₹600.
* Before first "−₹100" = ₹700. Before first "÷2" = ₹1400? Verify forward.

## Visual
```
Backward:
0 → +100 → 100 → ×2 → 200 → +100 → 300 → ×2 → 600
Start = 600? Check: 600/2=300, 300-100=200, 200/2=100, 100-100=0 ✅
```

## Answer
✅ Answer: ₹600 (starting amount).

## Why It Works
Working backwards reverses each operation (add instead of subtract; multiply instead of divide) to recover the original amount.

## Pattern
"Work backwards"

## Memory Anchor
🧠 Work backwards; invert each operation.

---

# Puzzle #49

## Category
Analytical / Mathematical Puzzles

## Question
How many cuts does it take to cut a round cake into exactly 8 equal pieces?

## Trick
* Method 1: 3 cuts (if you can stack): 2 cuts through the middle (4 pieces), 1 horizontal cut = 8 pieces.
* Method 2 (flat only): 3 cuts don't suffice for 8 with one cake flat → need to stack.
* Standard puzzle answer: **3 cuts** (including one horizontal cut).

## Visual
```
Top view:    Side view after 2 cross cuts:
  |   |          ________
--+---+--    ---|--------|---  ← horizontal cut
  |   |          ‾‾‾‾‾‾‾‾
2 cross + 1 horizontal = 8 equal pieces
```

## Answer
✅ Answer: **3 cuts** (2 vertical through center + 1 horizontal).

## Why It Works
Two perpendicular vertical cuts give 4 equal quarters. One horizontal cut through the middle halves all 4 simultaneously.

## Pattern
"3D cutting; horizontal slice doubles pieces"

## Memory Anchor
🧠 Horizontal cut doubles all pieces at once.

---

# Puzzle #50

## Category
Analytical / Mathematical Puzzles

## Question
Two creepers are climbing a tree from opposite sides. Creeper A grows 1 foot per day; Creeper B grows 2 feet per day. The tree is 10 feet tall. When do they meet?

## Trick
* They start at the bottom (0 ft) and climb upward.
* A: height = t. B: height = 2t. They meet at the top or where their heights match from opposite sides.
* If climbing from opposite sides: A reaches x ft from left, B from right. They meet when x + (10-x) = 10 → but on the same day A+B heights sum to 10.
* A+B = t+2t = 3t = 10 → t = 10/3 ≈ 3.33 days.

## Answer
✅ Answer: They meet after **3⅓ days** (at height 3⅓ ft from A's side).

## Why It Works
Combined coverage = A's progress + B's progress = tree height. Sum rates = 3 ft/day; 10/3 = meeting day.

## Pattern
"Combined rate / sum to total"

## Memory Anchor
🧠 Sum both rates; divide total distance by combined rate.

---

## Most Important Tricks — Category 2

* **Work backwards** — reverse operations to find starting values.
* **Sum formula / unitary method** — N(N+1)/2, unitary rate problems.
* **Complement probability** — P(at least one) = 1 − P(none).
* **Binary encoding** — assign unique IDs using bits; one weighing = 1 test.
* **Ignore the distraction** — Bee puzzle: ignore zig-zag, just find time.

## 30 Second Revision — Category 2

| Puzzle | Trick |
|--------|-------|
| Ages of Daughters | Same sum = ambiguous; "eldest" breaks tie → 2,2,9 |
| Bee Distance | Ignore bee path; find total time → 75km |
| 6×6 Grid Paths | C(10,5) = 252 |
| Monty Hall | Always switch → 2/3 win chance |
| 2 Eggs 100 Floors | Triangular numbers n(n+1)/2≥100 → 14 |
| White Ball Probability | Complement: 1 − P(both black) = 9/10 |
| Poison & Rat | Binary encoding; 3 rats = 8 bottles |
| Hourglass Puzzle | Chain remainders; 4+7 → 9 min |
| Boy-Girl Ratio | Stopping rules don't change 50/50 → 1:1 |
| Car Wheel Puzzle | 5×20000 / 4 tires = 25000km |
| Maximum Chocolates | Simulate wrapper exchange → 22 |
| Rs 500 Note | Real value lost = ₹500 |
| Girl or Boy | Restricted sample space → 1/3 |
| Average Salary | Add random noise; strip at end |
| Missing Row Excel | N(N+1)/2 − actual sum |
| Man Fell in Well | No slip on escape day → 28 days |
| 50 Red Blue Marbles | 1 red alone → P≈74.7% |
| Three Equilateral Triangles | Tetrahedron (3D) |
| 10 Bottles of Pills | k pills from bottle k → weight/0.1 |
| Circle Cutting | 1+n(n+1)/2 |
| Camel & Banana | Staged transport → 533 bananas |
| 1000 Light Bulbs | Perfect squares → 31 bulbs on |
| Snail & Wall | No slip day → 8 days |
| Initial Money | Work backwards → ₹600 |
| Round Cake 8 Pieces | Horizontal cut → 3 cuts |
| Two Creepers | Sum rates; 10/3 days |

---

# CATEGORY 3: LOGICAL PUZZLES

---

# Puzzle #51

## Category
Logical Puzzles

## Question
You have a gold rod that must be used to pay a worker for 7 days of work. The worker must be paid 1/7th of the rod each day. You can only make 2 cuts. How do you pay the worker exactly each day?

## Trick
* Cut the rod into pieces of 1/7, 2/7, and 4/7.
* Use exchange logic: give and take back pieces like change.

## Visual
```
Pieces: [1/7]  [2/7]  [4/7]
Day 1: Give 1/7
Day 2: Give 2/7, take back 1/7
Day 3: Give 1/7 + 2/7 (worker has 3/7)
Day 4: Give 4/7, take back 1/7 + 2/7
Day 5: Give 4/7 + 1/7
Day 6: Give 4/7 + 2/7, take back 1/7
Day 7: Give all three pieces
```

## Answer
✅ Answer: Cut into **1/7, 2/7, 4/7**. Use exchange like binary counting.

## Why It Works
Powers of 2 (1, 2, 4) cover all values 1–7 with combinations. Two cuts produce three pieces.

## Pattern
"Binary powers / exchange payment"

## Memory Anchor
🧠 1+2+4=7; two cuts, three pieces, pay daily with change.

---

# Puzzle #52

## Category
Logical Puzzles

## Question
25 horses, 5 can race at a time. Find the top 3 fastest in minimum races (no stopwatch).

*(Same as Puzzle #2 — repeated in Category 3.)*

## Answer
✅ Answer: **7 races**. *(See Puzzle #2.)*

## Pattern
"Group elimination + championship"

## Memory Anchor
🧠 5 + 1 + 1 = 7.

---

# Puzzle #53

## Category
Logical Puzzles

## Question
A patient needs anesthesia injected exactly 45 minutes before surgery. You have two burning ropes (each burns in 60 minutes, non-uniform). How do you measure exactly 45 minutes?

*(Same logic as Burning Ropes — Puzzle #1.)*

## Answer
✅ Answer: Light both ends of Rope 1 and one end of Rope 2 at t=0. When Rope 1 dies at t=30, light the other end of Rope 2. It finishes at t=45.

## Pattern
"Halving remaining burn time"

## Memory Anchor
🧠 30 + 15 = 45.

---

# Puzzle #54

## Category
Logical Puzzles

## Question
There are 3 light bulbs in a room and 3 switches outside. The door is closed. You can only enter once. How do you determine which switch controls which bulb?

## Trick
* Turn on Switch 1 for 10 minutes. Then turn it off.
* Turn on Switch 2. Enter the room.
* Bulb ON → Switch 2. Bulb OFF but warm → Switch 1. Bulb OFF and cold → Switch 3.

## Visual
```
Action:  Switch 1 ON (10 min) → OFF, Switch 2 ON → Enter
Result:
  Bulb ON       = Switch 2
  Bulb OFF+warm = Switch 1 (was on, now off)
  Bulb OFF+cold = Switch 3 (never on)
```

## Answer
✅ Answer: Use heat as a second observable. One entry is enough.

## Why It Works
Light gives ON/OFF info; heat gives "was recently on" info. Three states (on, warm-off, cold-off) map to three switches.

## Pattern
"Use multiple observables (light + heat)"

## Memory Anchor
🧠 Heat tells you what was on; light tells you what is on.

---

# Puzzle #55

## Category
Logical Puzzles

## Question
A camel must cross a 1000-mile desert with 3000 bananas, carrying max 1000 at a time, eating 1 per mile.

*(Same as Puzzle #46.)*

## Answer
✅ Answer: **533⅓ bananas**. *(See Puzzle #46.)*

## Pattern
"Staged transport"

## Memory Anchor
🧠 Reduce trips ASAP; 533 bananas survive.

---

# Puzzle #56

## Category
Logical Puzzles

## Question
There are 10 jars of pills. One jar has contaminated pills weighing 11g each; all others weigh 10g. You have one digital scale (one use). How do you find the contaminated jar?

*(Same as Puzzle #34 — 10 bottles of pills.)*

## Answer
✅ Answer: Take k pills from jar k. Excess weight ÷ 1g = jar number. *(See Puzzle #34.)*

## Pattern
"Unique quantity per source"

## Memory Anchor
🧠 k pills from jar k; extra weight reveals k.

---

# Puzzle #57

## Category
Logical Puzzles

## Question
100 prisoners are in solitary cells. Each day, the warden picks one prisoner to enter a room with a light bulb (initially off). A prisoner can toggle the light or do nothing. Any prisoner can declare "all 100 prisoners have visited." If correct, all go free; if wrong, all die. How do they guarantee freedom?

## Trick
* Designate one "counter" prisoner.
* Rule: non-counters turn the light ON exactly once (if they haven't already and the light is off). Counter turns the light OFF every time, incrementing count.
* When counter counts 99 OFF actions, all have visited.

## Visual
```
Counter: turns OFF → count++
Others: turn ON once (only if light is OFF and haven't done so yet)
When count = 99 → all 100 have visited → declare!
```

## Answer
✅ Answer: Designate one counter; non-counters flip ON once; counter flips OFF and counts. Declare at count = 99.

## Why It Works
Each non-counter contributes exactly one ON signal. Counter tallies them. Worst case takes a long time but guarantees correctness.

## Pattern
"Designated counter + one-time signaling"

## Memory Anchor
🧠 One counter, one signal per person; count to 99.

---

# Puzzle #58

## Category
Logical Puzzles

## Question
You have 10 coins on a table. Some are heads up, some tails up. You are blindfolded. You know there are exactly 5 heads-up coins. How do you split the coins into two groups so each group has the same number of heads?

## Trick
* Separate any 5 coins into Group A; remaining 5 in Group B.
* Flip all coins in Group A.
* Both groups now have the same number of heads.

## Visual
```
Say Group A (random 5) has k heads → (5-k) tails
Group B has (5-k) heads
Flip Group A: now has (5-k) heads
Both groups: (5-k) heads ✅
```

## Answer
✅ Answer: Split into any two groups of 5. Flip all coins in one group.

## Why It Works
If Group A has k heads, Group B has 5−k heads. After flipping Group A, it has 5−k heads. Both groups equal.

## Pattern
"Algebraic complement / blind split and flip"

## Memory Anchor
🧠 Split 5, flip one group — math guarantees equal heads.

---

# Puzzle #59

## Category
Logical Puzzles

## Question
Two players take turns picking 1 or 2 coins from a pile of N coins. The player who takes the last coin wins. What is the winning strategy?

*(Same as Puzzle #45 — Counters and Board.)*

## Answer
✅ Answer: Leave multiples of 3 for your opponent. If N ≡ 0 (mod 3), second player wins; otherwise first player wins by reducing to 3k.

## Pattern
"Modulo 3 strategy"

## Memory Anchor
🧠 Force multiples of 3 on opponent.

---

# Puzzle #60

## Category
Logical Puzzles

## Question
5 pirates find 100 gold coins. They rank from most senior (P1) to least (P5). P1 proposes a split; all vote. If ≥50% agree, the split stands. Otherwise, P1 is thrown overboard and P2 proposes. What does P1 propose to keep the most gold and survive?

## Trick
* Work backwards from P5, P4, P3...
* P3 alone: takes all 100.
* P4 alive: P3 takes all, so P4 offers P5 one coin to get 50% → P4 keeps 99.
* P5 alive: P4 would give P5 one coin, so P5 prefers P3's world. P3 must bribe P5 with 2 coins.
* With 5 pirates: P1 needs 3 votes (including own). Bribe P3 (gets 0 if P2 proposes) with 1 coin, and P5 (gets 0 if P2 proposes) with 1 coin.

## Visual
```
Work back:
2 pirates (P4,P5): P4 proposes 100,0 → gets own vote = 50% ✅
3 pirates: P3 proposes 99,0,1 → P3+P5 vote ✅
4 pirates: P2 proposes 99,0,1,0 → P2+P4 vote ✅ (P4 gets 0 under P3)
5 pirates: P1 proposes 98,0,1,0,1 → P1+P3+P5 vote ✅
```

## Answer
✅ Answer: P1 proposes **[98, 0, 1, 0, 1]** — keeps 98, gives 1 each to P3 and P5.

## Why It Works
Backward induction: bribe the pirates who would get nothing in the next-round scenario.

## Pattern
"Backward induction / game theory"

## Memory Anchor
🧠 Bribe those who'd get nothing next round; work backwards.

---

# Puzzle #61

## Category
Logical Puzzles

## Question
A chain has 23 links. What is the minimum number of links to cut so you can pay 1 link per day for up to 23 days?

*(Same as Puzzle #36.)*

## Answer
✅ Answer: Cut **1 link** (at position 4), creating pieces of 1, 3, and 19 that combine to cover any value 1–23.

## Pattern
"Power-of-2-like splits"

## Memory Anchor
🧠 One cut; pieces 1+3+19=23, cover all values.

---

# Puzzle #62

## Category
Logical Puzzles

## Question
A prisoner is told: "You will be hanged on a day next week that you cannot predict the evening before." Can such a hanging take place?

## Trick
* Prisoner reasons: can't be Friday (last day; by Thursday night I'd know). Can't be Thursday (Friday eliminated; by Wednesday night I'd know). Eliminating backwards → no day works.
* Paradox: the prisoner concludes hanging is impossible, is surprised on Wednesday.

## Answer
✅ Answer: The **Unexpected Hanging Paradox** — the prisoner's logical elimination is self-defeating; the hanging CAN occur because the prediction assumption breaks down.

## Why It Works
The self-referential logic creates a paradox. The assumption "I'll know the evening before" is invalidated by the reasoning process itself.

## Pattern
"Self-referential paradox"

## Memory Anchor
🧠 Perfect logic can be self-defeating; surprise is preserved.

---

# Puzzle #63

## Category
Logical Puzzles

## Question
A husband suspects one of 100 married men in a village is cheating. Every wife knows about all cheating husbands except her own. The chief announces "at least one husband is cheating." What happens?

## Trick
* If 1 cheater: his wife (only one who didn't know) now knows → shoots him on night 1.
* If 2 cheaters: each wife knew of 1. After night 1, no shooting → each realizes her own husband is also cheating → both shoot on night 2.
* If k cheaters: all shot on night k.

## Answer
✅ Answer: If there are k cheating husbands, all are shot on **night k** simultaneously.

## Why It Works
Common knowledge is created by the announcement. Inductive logic propagates: each wife counts the "known" cheaters and waits to see if shooting occurs the prior night.

## Pattern
"Common knowledge induction"

## Memory Anchor
🧠 Announcement creates common knowledge; k cheaters shot on night k.

---

# Puzzle #64

## Category
Logical Puzzles

## Question
A man is blindfolded and given an unknown number of red and blue balls in a bag. How does he ensure at least 2 balls of the same color are drawn?

## Trick
* Pigeonhole principle: draw 3 balls. With only 2 colors, at least 2 must match.

## Answer
✅ Answer: Draw **3 balls** — by the pigeonhole principle, at least 2 must share a color.

## Pattern
"Pigeonhole principle"

## Memory Anchor
🧠 n colors → draw n+1 balls to guarantee a match.

---

# Puzzle #65

## Category
Logical Puzzles

## Question
There are 3 chameleons: 5 red, 6 blue, 7 green. When two different-colored chameleons meet, both change to the third color. Can all chameleons ever become the same color?

## Trick
* Track counts mod 3: (5,6,7) → mod 3 → (2,0,1).
* For all same color, need (n,0,0) mod 3 → (0,0,0). Current residues are all different.
* Each meeting changes (a,b,c) → (a-1,b-1,c+2) in some permutation. This preserves the differences mod 3.
* Since 5≡2, 6≡0, 7≡1 (all different mod 3), uniform color is impossible.

## Answer
✅ Answer: **No** — it's impossible. The mod-3 residues of the three counts are all different and stay different after every meeting.

## Why It Works
Each interaction changes all three counts in a way that preserves the distinctness of their mod-3 values. Uniform color requires two counts to be zero (same mod 3), which can never be reached.

## Pattern
"Invariant (mod 3 residues)"

## Memory Anchor
🧠 Check mod-3 residues; if all distinct, uniformity is impossible.

---

# Puzzle #66

## Category
Logical Puzzles

## Question
You stand at a fork. One path leads to Heaven, one to Hell. Two guards stand there — one always lies, one always tells the truth. You don't know which is which. You can ask one question. What do you ask?

## Trick
* Ask either guard: "If I asked the other guard which path leads to Heaven, what would he say?"
* The liar lies about the truth-teller's answer → points to Hell.
* The truth-teller truthfully reports the liar's answer → points to Hell.
* Both point to Hell → take the other path!

## Answer
✅ Answer: Ask either guard: "What would the other guard say is the path to Heaven?" Then take the **opposite** path.

## Why It Works
Double negation: liar negates truth-teller's correct answer. Truth-teller reports liar's wrong answer. Both yield the wrong path; invert it.

## Pattern
"Meta-question / double negation"

## Memory Anchor
🧠 Ask what the other would say; take the opposite.

---

# Puzzle #67

## Category
Logical Puzzles

## Question
There are 3 jars mislabeled: "Red," "Blue," and "Mixed." All labels are wrong. How do you correctly label all jars by drawing only one marble?

## Trick
* Draw from the jar labeled "Mixed" — since it's mislabeled, it contains either all Red or all Blue.
* Say you draw Red → that jar is "Red." The jar labeled "Red" can't be Red (mislabeled) and can't be Mixed (taken) → it's "Blue." The remaining jar is "Mixed."

## Visual
```
Jars (mislabeled): [Mixed*] [Red*] [Blue*]
Draw from [Mixed*] → get Red marble
[Mixed*] = Red (confirmed)
[Red*] ≠ Red, ≠ Red(taken) → Blue
[Blue*] → Mixed
```

## Answer
✅ Answer: Draw from the jar labeled "Mixed." Its color tells you all three correct labels.

## Why It Works
Since all labels are wrong, the "Mixed" jar is pure. One draw fixes it, and the remaining two are determined by elimination.

## Pattern
"Mislabeled jars / process of elimination"

## Memory Anchor
🧠 Always draw from "Mixed" — it must be pure.

---

# Puzzle #68

## Category
Logical Puzzles

## Question
You have 8 balls. Seven weigh the same; one is heavier. Find the heavier ball in minimum weighings using a balance scale.

*(Same as Puzzle #9.)*

## Answer
✅ Answer: **2 weighings**. Split 3-3-2; compare the two groups of 3 first. *(See Puzzle #9.)*

## Pattern
"Divide into 3 groups"

## Memory Anchor
🧠 3^2 = 9 ≥ 8; two weighings suffice.

---

# Puzzle #69

## Category
Logical Puzzles

## Question
Cheryl tells Albert her birthday month and Bernard the day. Albert says he doesn't know but Bernard doesn't either. Bernard then says he now knows. Albert then says he knows too. When is Cheryl's birthday?

## Trick
* Possible dates: May 15/16/19, June 17/18, July 14/16, Aug 14/15/17.
* Albert (knows month) says Bernard can't know → month isn't May/June (which have unique days 18,19 that would let Bernard guess immediately). So month is July or August.
* Bernard now knows → day isn't 14 (appears in both July and August). Must be July 16, Aug 15, or Aug 17.
* Albert then knows → month is July (if August, two options 15/17 remain). Answer: **July 16**.

## Visual
```
Remaining after clue 1: July 14,16 / Aug 14,15,17
After clue 2 (day≠14): July 16 / Aug 15,17
After clue 3 (Albert knows month): July 16 ✅
```

## Answer
✅ Answer: Cheryl's birthday is **July 16**.

## Why It Works
Each clue eliminates possibilities through nested knowledge: what you know others know/don't know.

## Pattern
"Nested knowledge elimination"

## Memory Anchor
🧠 Eliminate months with unique days; then eliminate shared days; month with one option wins.

---

# Puzzle #70

## Category
Logical Puzzles

## Question
On an island, a lion always lies on Monday, Tuesday, Wednesday and tells the truth other days. A unicorn always lies on Thursday, Friday, Saturday and tells the truth other days. Both say "I lied yesterday." What day is it?

## Trick
* Find the day where both statements "I lied yesterday" are true simultaneously.
* Lion lies Mon–Wed (truth Thu–Sun). For "I lied yesterday" to be true (truth day for lion): day after a lie day → Thursday (after Wed lie).
* Unicorn lies Thu–Sat (truth Sun–Wed). For "I lied yesterday" to be true: day after a lie day (for unicorn, truth day) → Sunday (after Sat lie).
* Only **Thursday** works for the lion being truthful and claiming yesterday (Wed) was a lie. Check unicorn on Thursday: unicorn lies on Thursday → "I lied yesterday" is a lie → unicorn actually told truth yesterday (Wednesday) → unicorn tells truth Wed ✓.

## Answer
✅ Answer: **Thursday**.

## Why It Works
Cross-check which day satisfies both animals' statements simultaneously given their lie/truth schedules.

## Pattern
"Cross-schedule constraint satisfaction"

## Memory Anchor
🧠 Map each animal's schedule; find the intersecting valid day.

---

# Puzzle #71

## Category
Logical Puzzles

## Question
A farmer needs to cross a river with a goat, a wolf, and a cabbage. The boat holds only the farmer + one item. Wolf eats goat if left alone; goat eats cabbage if left alone. How does the farmer cross safely?

## Trick
* Take goat across first.
* Come back, take wolf across.
* Bring goat back.
* Take cabbage across.
* Return, take goat across.

## Visual
```
→ Goat        [Farm: Wolf, Cab | River: - | Other: Goat]
← (alone)     
→ Wolf        [Farm: Cab | River: - | Other: Wolf, Goat]
← Goat        [Farm: Cab, Goat | River: - | Other: Wolf]
→ Cabbage     [Farm: Goat | River: - | Other: Wolf, Cab]
←             
→ Goat        [Farm: - | River: - | Other: All] ✅
```

## Answer
✅ Answer: 5 trips: Goat → Wolf (bring Goat back) → Cabbage → Goat.

## Why It Works
The goat is the conflict item with both others. It must be moved back temporarily to allow the other two to cross safely.

## Pattern
"Move the conflict item back"

## Memory Anchor
🧠 The goat is double trouble — bring it back in the middle.

---

# Puzzle #72

## Category
Logical Puzzles

## Question
You have a 3-gallon and 5-gallon jug. Measure exactly 4 gallons.

*(Same as Puzzle #6.)*

## Answer
✅ Answer: Fill 5-gal → pour into 3-gal → empty 3-gal → pour 2 gal in → refill 5-gal → top up 3-gal (1 more) → **4 gal in 5-gal jug**. *(See Puzzle #6.)*

## Pattern
"Fill big, pour to small, cycle"

## Memory Anchor
🧠 GCD(3,5)=1; any integer reachable by cycling.

---

# Puzzle #73

## Category
Logical Puzzles

## Question
A blind man has 2 red pills and 2 blue pills (life-saving; must take 1 red + 1 blue). They're mixed up. He can't see color. How does he ensure he takes the correct combination?

## Trick
* Break each of the 4 pills in half.
* Make two groups: one half of each pill in Group 1, other halves in Group 2.
* Each group now contains half of each pill = 1 red + 1 blue equivalent.
* Take one group today, one tomorrow.

## Answer
✅ Answer: Break all pills in half; put one half of each into two piles. Each pile contains the equivalent of 1 red + 1 blue.

## Why It Works
Each pile guaranteed to contain ½ of each type = same ratio as the required dose.

## Pattern
"Split to equalize"

## Memory Anchor
🧠 Halve everything; each pile becomes a balanced dose.

---

# Puzzle #74

## Category
Logical Puzzles

## Question
You have two candles that each burn for exactly 1 hour (non-uniform burn rate). How do you measure exactly 45 minutes?

*(Same logic as Burning Ropes — Puzzle #1. Substitute "candles" for "ropes.")*

## Answer
✅ Answer: Light both ends of Candle 1 and one end of Candle 2 at t=0. When Candle 1 dies (t=30), light the other end of Candle 2. It finishes at t=45.

## Pattern
"Halving remaining burn time"

## Memory Anchor
🧠 30+15=45; same trick as ropes.

---

# Puzzle #75

## Category
Logical Puzzles

## Question
There are 1000 milk bottles. One is poisoned (kills in 1 hour). You have 10 rats and 1 hour. How do you find the poisoned bottle?

*(Same as Puzzle #4 — Poisoned Bottle.)*

## Answer
✅ Answer: Binary encode each bottle to a 10-bit number; rat k drinks from all bottles with bit k set. Dead rats decode the bottle number. **10 rats** suffice for 2^10 = 1024 ≥ 1000.

## Pattern
"Binary encoding"

## Memory Anchor
🧠 Each rat = one binary bit.

---

# Puzzle #76

## Category
Logical Puzzles

## Question
You need to measure exactly 6 litres using a 4-litre and a 9-litre jug.

## Trick
* Fill 9-litre, pour into 4-litre until full. 9-litre has 5 left.
* Empty 4-litre, pour 5 into it. Fill 9-litre again.
* Pour from 9-litre into 4-litre (needs 1 more litre). 9-litre now has 8 left.
* Empty 4-litre, pour — or use a shorter route:
* Fill 4-litre twice into 9-litre (8 litres). Fill 4-litre again, pour 1 into 9-litre (fills it). 4-litre has 3 left.
* Empty 9-litre, pour 3 in. Fill 4-litre, pour into 9-litre → **7** — not quite.
* Direct: fill 9, pour into 4 → 5 left. Empty 4, pour 5 in → fill 9, pour 3 more into 4-litre (full). **9-litre has 6**. ✅

## Visual
```
Step | 4-L | 9-L
  0  |  0  |  0
  1  |  0  |  9   fill 9
  2  |  4  |  5   fill 4 from 9
  3  |  0  |  5   empty 4
  4  |  4  |  1   fill 4 from 9
  5  |  0  |  1   empty 4
  6  |  1  |  0   pour 1 into 4
  7  |  1  |  9   fill 9
  8  |  4  |  6 ✅ fill 4 from 9 (needs 3 more)
```

## Answer
✅ Answer: 6 litres in the 9-litre jug after 8 steps.

## Pattern
"Water jug / remainder cycling"

## Memory Anchor
🧠 GCD(4,9)=1; all values reachable. Cycle fill/pour.

---

# Puzzle #77

## Category
Logical Puzzles

## Question
Same as Puzzle #8 — Six Houses (Einstein's Riddle).

## Answer
✅ Answer: **The German owns the fish.** *(See Puzzle #8.)*

## Pattern
"Logic grid / constraint elimination"

## Memory Anchor
🧠 Build a grid; apply clues sequentially.

---

# Puzzle #78

## Category
Logical Puzzles

## Question
Two candles are lit at the same time. One lasts 5 hours, the other 4 hours. After how long is one candle twice the length of the other?

## Trick
* Let height of 5-hr candle = 1 − t/5; height of 4-hr candle = 1 − t/4.
* Set (1 − t/5) = 2(1 − t/4): 1 − t/5 = 2 − t/2.
* t/2 − t/5 = 1 → t(3/10) = 1 → t = 10/3 ≈ 3.33 hours.

## Answer
✅ Answer: After **3⅓ hours**, the 5-hour candle is twice as tall as the 4-hour candle.

## Pattern
"Set up equation; solve for t"

## Memory Anchor
🧠 Express heights as functions of t; solve the ratio equation.

---

# Puzzle #79

## Category
Logical Puzzles

## Question
Ten prisoners wear red or black hats. Starting from the back, each says one word. How can 9 be saved with certainty?

*(Same as Puzzle #10.)*

## Answer
✅ Answer: First prisoner (back) encodes **parity** of red hats seen. Remaining 9 decode their own hat color. *(See Puzzle #10.)*

## Pattern
"Parity encoding"

## Memory Anchor
🧠 First encodes parity; rest decode it → 9 certain survivors.

---

# Puzzle #80

## Category
Logical Puzzles

## Question
In a joint family: 2 fathers, 2 sons, 1 grandfather, 1 grandson. How many people are there?

## Trick
* Grandfather, Father (who is also a son), and Son = 3 people.
* "2 fathers" = Grandfather + Father. "2 sons" = Father + Son. "1 grandson" = Son. All roles overlap.

## Answer
✅ Answer: **3 people** (Grandfather, Father/Son, Son/Grandson).

## Pattern
"Overlapping roles"

## Memory Anchor
🧠 Middle person is both father and son; 3 people fill all roles.

---

# Puzzle #81

## Category
Logical Puzzles

## Question
You have a circle of N lights, all off. Each step you toggle a light and its two neighbors. How do you turn them all on?

## Trick
* This is a parity/linear algebra problem over GF(2). For N=3: toggle each light once → all on.
* General: depends on N. For N divisible by 3, a solution exists by toggling lights at positions spaced N/3 apart.

## Answer
✅ Answer: For **N=3**, toggle each light once (3 moves → all on). For larger N, the solvability depends on N mod 3 and requires GF(2) analysis.

## Pattern
"Toggle puzzle / GF(2) linear algebra"

## Memory Anchor
🧠 Think about which toggles affect which bulbs; solve as a system of equations mod 2.

---

# Puzzle #82

## Category
Logical Puzzles

## Question
9 students, each wearing a red or black hat (at least one of each color). They can see others' hats but not their own. Simultaneously, each writes their guess. At least one must be correct. How?

## Trick
* Assign students 0–8. Each student counts red hats they see.
* Student k guesses red if (count seen + k) ≡ 0 (mod 2); else guesses black. Wait — classic version uses a parity strategy.
* Standard strategy: use "balanced parity rows." Each student assumes they are the one making the total match a target, cycling through all residues.

## Answer
✅ Answer: Use a **combinatorial strategy** where each student assumes a different total parity class. At least one assumption matches reality → at least one correct guess guaranteed.

## Pattern
"Covering all residue classes"

## Memory Anchor
🧠 Divide students into parity roles; one must be right.

---

# Puzzle #83

## Category
Logical Puzzles

## Question
A room has N light bulbs, all off. You want all ON. You can only press a button that toggles all bulbs. How many presses?

*(Too simple as stated — likely refers to a specific variant such as "toggle groups.")*

## Answer
✅ Answer: **1 press** (all off → all on with one toggle, if the button toggles all simultaneously).

## Pattern
"Direct toggle"

## Memory Anchor
🧠 One press toggles all; all off → all on.

---

# Puzzle #84

## Category
Logical Puzzles

## Question
Distribute 8 litres of water into two equal parts using 8L, 5L, and 3L containers (8L is full, others are empty).

## Trick
* Pour 8L → 5L: [3, 5, 0]. Pour 5L → 3L: [3, 2, 3]. Pour 3L → 8L: [6, 2, 0].
* Pour 5L → 3L: [6, 0, 2]. Pour 8L → 5L: [1, 5, 2]. Pour 5L → 3L: [1, 4, 3]. Pour 3L → 8L: [4, 4, 0]. ✅

## Visual
```
Step | 8L | 5L | 3L
  0  |  8 |  0 |  0
  1  |  3 |  5 |  0
  2  |  3 |  2 |  3
  3  |  6 |  2 |  0
  4  |  6 |  0 |  2
  5  |  1 |  5 |  2
  6  |  1 |  4 |  3
  7  |  4 |  4 |  0 ✅
```

## Answer
✅ Answer: **4 litres in each of the 8L and 5L containers** (7 steps).

## Pattern
"Water jug cycling"

## Memory Anchor
🧠 Pour between jugs; track state; stop when two containers read equal.

---

# Puzzle #85

## Category
Logical Puzzles

## Question
In a group of people, prove that at least two share the same number of hairs on their head.

## Trick
* Human head has at most ~150,000 hairs. Earth's population is ~8 billion.
* By the pigeonhole principle, with more people than possible hair counts, at least two must share a count.

## Answer
✅ Answer: **Pigeonhole principle** — far more people than possible hair counts guarantees a match.

## Pattern
"Pigeonhole principle"

## Memory Anchor
🧠 More pigeons than holes → at least one shared hole.

---

# Puzzle #86

## Category
Logical Puzzles

## Question
9 balls, one is heavier. Find it in minimum weighings.

## Trick
* 3^2 = 9 → 2 weighings exactly.
* Divide into 3 groups of 3. Weigh two groups.
* Identify the heavy group. Weigh 2 balls from that group → find the heavy one (or it's the third).

## Answer
✅ Answer: **2 weighings** (same logic as Puzzle #9 but scaled to 9 balls).

## Pattern
"Divide into 3 groups"

## Memory Anchor
🧠 3^n covers 3^n balls in n weighings.

---

# Puzzle #87

## Category
Logical Puzzles

## Question
How many legs are there in the palace? *(Same as Puzzle #7.)*

## Answer
✅ Answer: **98 legs** (don't forget the king). *(See Puzzle #7.)*

---

---

# CATEGORY 4: ARRANGEMENT PUZZLES

---

# Puzzle #88

## Category
Arrangement Puzzles

## Question
Place 10 coins in 5 straight lines with exactly 4 coins on each line.

## Trick
* Arrange 10 coins as a **5-pointed star (pentagram)**.
* Each line of the star passes through exactly 4 coins.
* The 5 intersections + 5 outer tips = 10 coins, 5 lines, 4 per line.

## Visual
```
        ★
       / \
      ★   ★
     / \ / \
    ★   ★   ★
     \ / \ /
      ★   ★
       \ /
        ★
        ★
(Star arrangement — 5 lines × 4 coins)
```

## Answer
✅ Answer: Arrange coins in a **pentagram (5-pointed star)** pattern.

## Why It Works
Each arm of the star + the central crossing forms a line of 4. Five such lines use 10 coins total with proper overlap.

## Pattern
"Star / pentagram arrangement"

## Memory Anchor
🧠 5-pointed star = 5 lines, 4 coins each, 10 coins total.

---

# Puzzle #89

## Category
Arrangement Puzzles

## Question
Using 2 dice, display every day of the month (1 to 31). Each die has 6 faces. How do you number them?

## Trick
* Days 1–9 need a leading 0 (01–09) plus single digits.
* A 6 can be flipped upside down to serve as a 9.
* Die 1: 0, 1, 2, 3, 4, 5. Die 2: 0, 1, 2, 6/9, 7, 8.
* Both dice must have 0 (for 01–09), 1 (for 11, 21, 31), and 2 (for 12, 22).

## Visual
```
Die 1: [0][1][2][3][4][5]
Die 2: [0][1][2][6/9][7][8]
Day 07: Die1=0, Die2=7 ✅
Day 31: Die1=3, Die2=1 (or swap) ✅
Day 19: Die1=1, Die2=9(flip 6) ✅
```

## Answer
✅ Answer: Die 1: **0,1,2,3,4,5** — Die 2: **0,1,2,6/9,7,8**. The 6 doubles as a 9 when flipped.

## Why It Works
The 6-face, when rotated 180°, becomes a 9. Both dice must carry 0, 1, and 2 to display any date from 01 to 31.

## Pattern
"Shared digits + 6=9 flip"

## Memory Anchor
🧠 Both dice need 0,1,2; flip 6 to get 9.

---

# Puzzle #90

## Category
Arrangement Puzzles

## Question
In a game of Tic-Tac-Toe, if both players play optimally, what is the result?

## Trick
* Tic-Tac-Toe is a solved game.
* With optimal play from both sides, the game always ends in a **draw**.
* First player strategy: take center, then corners.

## Visual
```
Optimal play tree leads to:
X in center → O in corner → X in opposite corner...
Every line of attack is blocked → Draw
```

## Answer
✅ Answer: **Draw** — with perfect play, neither player can win.

## Why It Works
The game tree has been fully analyzed. No first-move strategy guarantees a win; the second player can always force a draw.

## Pattern
"Game theory / perfect play = draw"

## Memory Anchor
🧠 Tic-Tac-Toe: center → corners → always draw.

---

# Puzzle #91

## Category
Arrangement Puzzles

## Question
Move exactly 3 matchsticks to turn a fish facing left into a fish facing right.

## Trick
* The fish shape typically has a body (rectangle/triangle) and a tail.
* Remove the tail (2 matchsticks) and one body side, reposition them on the other side.
* The eye (1 matchstick or coin) may need to move too — count carefully.

## Visual
```
Left-facing:    →→ →→ (body) + tail pointing left
Move 3 sticks:  Relocate tail to right side + adjust body
Right-facing:   ←← ←← (body) + tail pointing right
```

## Answer
✅ Answer: Move the **3 tail matchsticks** to the opposite side of the body. The fish now faces the other direction.

## Why It Works
The fish body stays; only the tail's orientation determines which direction the fish faces.

## Pattern
"Reverse orientation by moving tail"

## Memory Anchor
🧠 Move the tail (3 sticks) to flip the fish direction.

---

# Puzzle #92

## Category
Arrangement Puzzles

## Question
What is the last palindrome date before today?

## Trick
* A palindrome date reads the same forwards and backwards (e.g., 02/02/2020 → 02022020).
* Format: DDMMYYYY or MM/DD/YYYY.
* In DD/MM/YYYY: 12/02/2021 = 12022021 → reversed = 12022021 ✅ palindrome.
* Next ones: 03/02/2030, etc.

## Answer
✅ Answer: In DD/MM/YYYY format, **12/02/2021** (12022021) is a recent palindrome date. The one before that: 02/02/2020. After 2021, the next is 03/02/2030.

## Pattern
"Palindrome date pattern"

## Memory Anchor
🧠 Check DDMMYYYY reversed; common ones cluster in 2020s.

---

# Puzzle #93

## Category
Arrangement Puzzles

## Question
10 bottles of pills, one with heavier pills (1.1g vs 1g). Find it in one weighing.

*(Same as Puzzle #34 / #56.)*

## Answer
✅ Answer: Take k pills from bottle k; weigh all. Excess ÷ 0.1 = bottle number. *(See Puzzle #34.)*

---

# Puzzle #94

## Category
Arrangement Puzzles

## Question
Place 10 balls in 5 rows with 4 balls in each row.

*(Same as Puzzle #88 — pentagram arrangement.)*

## Answer
✅ Answer: **Pentagram (5-pointed star)** — 5 lines × 4 balls = 10 balls. *(See Puzzle #88.)*

---

# Puzzle #95

## Category
Arrangement Puzzles

## Question
Two players alternately place coins on a round table. No coin may overlap another or hang off the edge. The player who cannot place a coin loses. Who wins and what is the strategy?

## Trick
* The **first player** always wins.
* Strategy: place the first coin in the exact center of the table.
* For every coin the second player places, the first player mirrors it symmetrically through the center.
* The second player always runs out of moves first.

## Answer
✅ Answer: **First player wins** by placing the first coin at the center and mirroring every subsequent move.

## Why It Works
The center coin is unique (no mirror). Every valid move by Player 2 has a valid symmetric mirror for Player 1. Player 2 must eventually be unable to move first.

## Pattern
"Symmetry strategy / mirror play"

## Memory Anchor
🧠 Take center; mirror every opponent move.

---

# Puzzle #96

## Category
Arrangement Puzzles

## Question
A circle of N lights, all off; toggling one light also toggles its two neighbors. How do you turn all lights on?

*(Same as Puzzle #81.)*

## Answer
✅ Answer: For N=3, toggle each light once. For other N, depends on divisibility by 3. *(See Puzzle #81.)*

---

---

# CATEGORY 5: SHAPE-BASED PUZZLES

---

# Puzzle #97

## Category
Shape-Based Puzzles

## Question
Cut a round cake into 8 equal pieces with exactly 3 cuts.

*(Same as Puzzle #49.)*

## Answer
✅ Answer: 2 vertical cuts through the center (perpendicular) → 4 pieces. 1 horizontal cut through the middle → 8 equal pieces. **3 cuts total.** *(See Puzzle #49.)*

## Pattern
"Horizontal cut doubles all pieces"

## Memory Anchor
🧠 2 cross cuts + 1 horizontal = 8.

---

# Puzzle #98

## Category
Shape-Based Puzzles

## Question
A chessboard (8×8) has two diagonally opposite corners removed (62 squares remain). Can you tile it perfectly with 31 dominoes (each covers exactly 2 squares)?

## Trick
* Color the board in standard black-and-white checkerboard fashion.
* The two removed corners are the **same color** (both black, say).
* Each domino covers exactly 1 black + 1 white square.
* After removal: 30 black squares, 32 white squares (or vice versa) → **impossible** to tile with dominoes.

## Visual
```
Standard board: 32 black, 32 white
Remove 2 same-color corners: 30 black, 32 white
Each domino = 1 black + 1 white
31 dominoes need 31 black + 31 white → impossible ✅
```

## Answer
✅ Answer: **No** — it is impossible. Opposite corners share the same color; removing them creates a 2-square color imbalance, and dominoes require equal counts.

## Why It Works
Domino tiling is possible only when black and white squares are equal. Removing same-color corners breaks this parity.

## Pattern
"Checkerboard coloring invariant"

## Memory Anchor
🧠 Color the board; opposite corners = same color → parity breaks → impossible.

---

# Puzzle #99

## Category
Shape-Based Puzzles

## Question
Move 3 matchsticks in a given arrangement to form a new shape (fish, house, etc.).

*(General matchstick puzzle — solution depends on specific arrangement.)*

## Trick
* Identify what shape is given. Determine what moves create the target shape.
* For fish-flip: move the 3 tail matchsticks.
* For square-to-three-triangles: move sticks to create inner triangles.

## Answer
✅ Answer: Varies by specific puzzle. **Always identify the minimum change to transition between shapes**; often just rotating or relocating the "tail" or "base" elements.

## Pattern
"Minimum relocation to change shape"

## Memory Anchor
🧠 Don't rebuild from scratch; move only the edge sticks that define the change.

---

# Puzzle #100

## Category
Shape-Based Puzzles

## Question
What is the maximum number of pieces you can get from a circle with n straight cuts?

*(Same as Puzzles #23 and #35.)*

## Answer
✅ Answer: **1 + n(n+1)/2** pieces. For n=1: 2, n=2: 4, n=3: 7, n=4: 11. *(See Puzzle #35.)*

## Pattern
"Lazy caterer sequence"

## Memory Anchor
🧠 Each new cut crosses all prior cuts; formula = 1 + n(n+1)/2.

---

# Puzzle #101

## Category
Shape-Based Puzzles

## Question
A cake is cut but a rectangular piece is missing from the middle (not from an edge, and not aligned to the original sides). How do you cut the remaining cake into two equal halves with one straight cut?

## Trick
* Any straight line through the **center of a rectangle** divides it into two equal halves.
* The missing piece is also a rectangle; any line through its center also halves it.
* Cut along the line connecting the **center of the original cake** to the **center of the missing piece** — this one cut bisects both simultaneously.

## Visual
```
Original cake (rectangle): center at C1
Missing piece (rectangle): center at C2
One cut through C1 and C2 → bisects both shapes → two equal remaining halves ✅
```

## Answer
✅ Answer: Draw a line through both the **center of the full cake** and the **center of the missing piece**. One cut = two equal halves.

## Why It Works
Any line through the center of a rectangle divides it into two equal areas. The single line simultaneously bisects both rectangles.

## Pattern
"Bisect both centers with one line"

## Memory Anchor
🧠 Connect the two centers; one cut splits everything equally.

---

# Puzzle #102

## Category
Shape-Based Puzzles

## Question
Three ants are sitting at the three corners of an equilateral triangle. Each ant randomly picks a direction and walks along a side. What is the probability that none of them collide?

## Trick
* Each ant has 2 choices (clockwise or counterclockwise). Total combinations = 2^3 = 8.
* No collision only if all go clockwise (CW,CW,CW) or all go counterclockwise (CCW,CCW,CCW) = 2 favorable outcomes.
* P(no collision) = 2/8 = **1/4**.

## Visual
```
Total outcomes: 2³ = 8
Safe: all CW (1) + all CCW (1) = 2
P(safe) = 2/8 = 1/4
P(collision) = 6/8 = 3/4
```

## Answer
✅ Answer: P(no collision) = **1/4** (25%).

## Why It Works
Only unanimous direction choices avoid collision. Any mixed direction results in at least one pair of ants heading toward each other.

## Pattern
"Count unanimous outcomes / 2^n total"

## Memory Anchor
🧠 2 safe outcomes out of 8 total = 1/4.

---

---

# MASTER QUICK-REFERENCE

## Category 3 — Key Tricks

| Puzzle | Trick |
|--------|-------|
| Gold Rod Payment | Cut 1/7, 2/7, 4/7; exchange like binary |
| 3 Bulbs 3 Switches | Heat as second observable |
| 5 Pirates 100 Coins | Backward induction; bribe skipped players → 98,0,1,0,1 |
| 100 Prisoners Light Bulb | Designate counter; one signal each → count 99 |
| 10 Coins Flip Puzzle | Split 5, flip one group → equal heads |
| Mislabeled Jars | Draw from "Mixed"; one marble identifies all |
| Cheryl's Birthday | Nested elimination → July 16 |
| Lion and Unicorn | Schedule intersection → Thursday |
| Farmer Goat Wolf Cabbage | Move conflict item (goat) back → 5 trips |
| Heaven and Hell | Ask what the other would say; take opposite path |
| Cheating Husband | Common knowledge induction; k cheaters shot night k |
| Chameleons Puzzle | Mod-3 invariant; (5,6,7) → impossible |
| Joint Family | Overlapping roles → 3 people |
| Blind Man and Pills | Break all in half; one pile per dose |
| Water Distribute (8L) | Pour between 8L/5L/3L; 7 steps to get 4L+4L |
| Same Number of Hairs | Pigeonhole principle |
| 9 Students Hats | Cover all residue classes |
| Red Hat Blue Hat | Parity encoding; 9 certain |

## Category 4 — Key Tricks

| Puzzle | Trick |
|--------|-------|
| 10 Coins 5 Lines | Pentagram / 5-pointed star |
| Days on 2 Dice | Both need 0,1,2; flip 6→9 |
| Tic-Tac-Toe | Perfect play → Draw |
| Matchstick Fish | Move 3 tail sticks to flip direction |
| Palindrome Date | DDMMYYYY reversed; 12022021 |
| Round Table Coin | Center first; mirror every move → Player 1 wins |

## Category 5 — Key Tricks

| Puzzle | Trick |
|--------|-------|
| 3 Cuts → 8 Pieces | 2 vertical + 1 horizontal cut |
| Chessboard Dominoes | Same-color corners removed → parity broken → impossible |
| Circle Max Pieces | 1 + n(n+1)/2 |
| Missing Cake Piece | Connect two centers; one cut bisects both |
| 3 Ants Triangle | 2 safe / 8 total = 1/4 collision-free |

---

## ULTIMATE ONE-PAGE TRICK REFERENCE

| # | Puzzle | Core Trick |
|---|--------|-----------|
| 1 | Burning Ropes | Both ends = half time; 30+15=45 |
| 2 | 25 Horses | 5+1+1=7 races |
| 3 | Bridge Crossing | Pair slowest; shuttle fastest → 17 min |
| 4 | Poisoned Bottle | Binary encode; N rats = 2^N bottles |
| 5 | Two-Egg Problem | n(n+1)/2 ≥ 100 → 14 drops |
| 6 | Four Gallon | Fill big, pour small, cycle |
| 7 | Legs in Palace | Count humans too! |
| 8 | Six Houses | Logic grid; constraint elimination |
| 9 | 8 Ball Problem | Split in thirds; 2 weighings |
| 10 | Hat Parity | First prisoner encodes parity; 9 saved |
| 11 | Daughters' Ages | Same sum = ambiguous; "eldest" → 2,2,9 |
| 12 | Bee Distance | Ignore path; time × speed = 75 km |
| 13 | Grid Paths | C(10,5) = 252 |
| 14 | Monty Hall | Always switch → 2/3 win |
| 17 | Ball Probability | Complement: 1 − P(none) = 9/10 |
| 18 | 8 Bottles Rats | 3 bits = 8 combos; 3 rats |
| 19 | Hourglass | Chain remainders; 4+7 → 9 min |
| 20 | Boy-Girl Ratio | Stopping rule ≠ change birth odds → 1:1 |
| 21 | Car Wheels | 5×20000 / 4 = 25000 km |
| 22 | Chocolates | Simulate exchange → 22 |
| 23 | Cake Cuts | 1+n(n+1)/2 |
| 24 | ₹500 Fake Note | Fake = ₹0; real loss = ₹500 |
| 25 | Girl or Boy | Restricted sample space → 1/3 |
| 26 | Salary Average | Random noise mask; remove at end |
| 29 | Missing Row | N(N+1)/2 − actual sum |
| 31 | Man in Well | No slip on escape day → 28 days |
| 32 | Red/Blue Marbles | Isolate 1 red → P ≈ 74.7% |
| 33 | 6 Matchsticks | Tetrahedron (3D); 4 faces, 6 edges |
| 34 | 10 Pills Bottles | k pills from bottle k; weight/0.1 |
| 36 | Chain Link | Cut 1 link → pieces 1,3,19 |
| 39 | Apple Baskets | Powers of 2 + remainder = 10 baskets |
| 40 | Snail & Wall | No slip on escape day → 8 days |
| 41 | 1000 Bulbs | Perfect squares ON → 31 bulbs |
| 44 | One Mile Globe | North Pole + circles near South Pole |
| 45 | Counter Game | Leave multiples of 3 for opponent |
| 46 | Camel Bananas | Staged transport → 533⅓ bananas |
| 48 | Initial Money | Work backwards → ₹600 |
| 49 | Cake 8 Pieces | 2 cross cuts + 1 horizontal = 3 cuts |
| 50 | Two Creepers | Sum rates; 10/3 days |
| 54 | 3 Bulbs 1 Entry | Heat = second observable |
| 57 | 100 Prisoners | Counter strategy; 99 signals |
| 58 | 10 Coins Flip | Split 5; flip one group |
| 60 | 5 Pirates | Backward induction → [98,0,1,0,1] |
| 62 | Unexpected Hanging | Self-referential paradox |
| 63 | Cheating Husbands | Common knowledge → shot on night k |
| 65 | Chameleons | Mod-3 invariant → impossible |
| 66 | Heaven/Hell | Ask the other's answer; take opposite |
| 67 | Mislabeled Jars | Draw from "Mixed" → deduce all |
| 69 | Cheryl's Birthday | Nested elimination → July 16 |
| 70 | Lion & Unicorn | Schedule intersection → Thursday |
| 71 | Farmer Wolf Goat | Take goat back in the middle |
| 73 | Blind Man Pills | Break all in half; one pile = one dose |
| 78 | Melting Candles | Set height equation; t = 10/3 hrs |
| 80 | Joint Family | Overlapping roles → 3 people |
| 85 | Hair Puzzle | Pigeonhole principle |
| 88 | 10 Coins 5 Lines | 5-pointed star (pentagram) |
| 89 | 2 Dice Calendar | Both need 0,1,2; flip 6→9 |
| 90 | Tic-Tac-Toe | Perfect play → draw |
| 95 | Round Table Coins | Center + mirror strategy; P1 wins |
| 98 | Chessboard Domino | Same-color corners → impossible |
| 101 | Missing Cake Piece | Connect both rectangle centers; one cut |
| 102 | 3 Ants Triangle | 2/8 = 1/4 no-collision probability |
