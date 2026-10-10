# Python coding interview questions: the theory round, the coding round, and how to prep for both in six weeks

Search for "python coding interview questions" and you get two completely different things in the same results page. Half of them are flashcard-style lists: 40 questions about `is` versus `==`, decorators, and the GIL. The other half are LeetCode problems with Python solutions and Big-O at the bottom. Novice readers assume one of these is the wrong answer. Both are right, because a modern Python interview usually splits into a language-and-concepts portion and a live coding portion, and each one punishes a different kind of under-preparation.

So this isn't a numbered list of 50 questions. It's a map of what actually gets asked in each part, plus what to do about the parts where memorizing answers stops working.

## What the interview actually tests, in two parts

**The fundamentals portion.** Usually 20 to 45 minutes, either as its own technical screen or folded into the front of the coding call. Questions here are short and deceptively easy: what's the difference between a list and a tuple, what does a decorator do, why does your threading code not speed up a CPU-bound task. The interviewer follows up. A definition gets you one sentence of credit; the follow-up is the real question.

**The live coding portion.** 45 to 60 minutes, one or two problems, you write Python while someone watches, and you narrate. The problems are almost never exotic. They're the standard families: hash maps, two pointers, sliding window, binary search, trees, graphs, dynamic programming.

Level changes the ratio. For a junior role, expect mostly language semantics plus an easy array or string problem. For mid-level, the same coding problems plus concurrency, database behavior, and async. Senior interviews push toward design tradeoffs and the "why did you choose this" questions, and the Python trivia gets thinner.

## The fundamentals questions, with answers worth giving

These are the ones that come up over and over across published interview guides. Short answers, because that's how long you get before the follow-up.

- **List vs tuple.** Lists are mutable, tuples are not. The practical consequence is that a tuple can be a dictionary key or a set member and a list cannot, and tuples use less memory. "Lists are like arrays and tuples are constant" is the answer that gets a follow-up you can't handle.
- **Mutable default arguments.** `def f(items=[])` reuses the same list across every call, because the default is evaluated once when the function object is created. That's why you write `items=None` and assign inside.
- **`is` vs `==`.** `==` calls `__eq__` and compares values. `is` compares identity in memory. Use `is` for `None`, not for integers or strings, where it depends on interning.
- **List comprehension vs generator expression.** Same syntax, different timing. `[x*x for x in data]` builds the whole list in memory. `(x*x for x in data)` yields values on demand, which is why a generator can walk a 10 GB file and a list comprehension cannot.
- **Shallow vs deep copy.** `list(a)`, `a[:]`, and `copy.copy(a)` create a new outer container but share the inner objects, so mutating a nested list shows up in both. `copy.deepcopy` copies recursively and costs more.
- **Removing duplicates while keeping order.** `list(set(items))` is the reflex answer and it destroys order. `list(dict.fromkeys(items))` is the correct one, since dict keys are unique and insertion-ordered.
- **`try`/`except`/`else`/`finally`.** The `else` clause runs only when no exception was raised, which is the part most candidates forget. And a bare `except: pass` that swallows everything is a bug, not a style preference.
- **Reading a huge file.** Iterate the file object line by line or read in fixed-size chunks. `read()` on the whole file is the answer that ends the question.

## The Python-specific traps that separate two kinds of candidates

A lot of the questions on those 40-and-50-item lists are not really Python questions. They're questions about whether you've worked on something that had to survive production traffic.

**The GIL.** In CPython, only one thread executes Python bytecode at a time. The follow-up is always some variation of "so is threading useless?" The answer: threads are fine for I/O (network calls, disk, waiting), `multiprocessing` is what you use for CPU-bound work because each process gets its own interpreter and its own GIL, and `asyncio` is for holding thousands of concurrent connections without a thread per connection. Worth getting right in 2026: Python 3.13's free-threaded mode makes the GIL optional, not gone. Candidates who announce "the GIL was removed" tend to get walked back into the weeds.

**Memory management.** Reference counting handles most objects, freeing immediately when the count hits zero, and a cyclic garbage collector picks up the reference cycles that counting can't see. Mentioning the cycle collector is the difference between a textbook answer and an actual one.

**`__slots__`.** It skips the per-instance `__dict__`, which saves memory on classes you instantiate in large numbers, at the cost of not being able to add attributes dynamically.

**Immutability and sharing.** Tuples as dict keys, strings built with `join` instead of `+=` in a loop, lists passed into functions and mutated in place. These are all the same underlying question: do you know what your object is doing in memory.

**Typing and modern syntax.** Across several 2026 interview guides, the recurring complaint is that candidates over-prepare syntax and under-prepare async and typing. `Protocol` versus abstract base classes, `TypedDict`, `ParamSpec`, `asyncio.TaskGroup` instead of `gather`, `asyncio.timeout` instead of `wait_for`. If your target role lists Python as a primary language and the codebase is type-hinted, expect at least one of these.

## The live coding questions are pattern questions in Python clothes

Here's the useful reframe: when you're handed a problem in the coding round, nobody is testing whether you can write a `for` loop in Python. They're testing how fast you classify the problem. Five examples that cover a huge share of what gets asked.

**Two Sum.** Return indices of the two numbers that add to a target. Brute force is O(n²). The answer is one pass with a hash map:

python
def two_sum(nums, target):
    seen = {}
    for i, x in enumerate(nums):
        if target - x in seen:
            return [seen[target - x], i]
        seen[x] = i


**Palindrome check.** `s == s[::-1]` is correct, O(n) time and O(n) extra space for the reversed copy. Two pointers from each end is O(n) time and O(1) space. Say both, then say which you'd ship, and mention whether case, spaces, and punctuation should be normalized. That last part is often the actual question.

**Contains duplicates.** Hash set gives O(n) time and O(n) space. Sorting first gives O(n log n) time and O(1) extra space. Naming the tradeoff beats picking one silently.

**Sliding window.** Longest substring without repeating characters, or longest subarray with a sum constraint. Maintain a window, move the right edge, and shrink the left edge when the invariant breaks.

**Binary search, BFS/DFS, and DP.** Sorted input, minimum or maximum answer, or a search space you can halve → binary search, sometimes on the answer rather than the array. Shortest path in a grid or graph → BFS. Count the ways, or find the optimum where each choice affects the next → dynamic programming, usually starting from a recursive definition and adding memoization before converting to a bottom-up table.

Interviewers pick from these families for a reason: each one has a recognizable trigger in the problem statement. "Contiguous subarray", "minimum", "sorted", "how many ways". Recognizing the trigger in the first two minutes is what buys you the remaining 43 minutes.

## What gets graded in the live round

Two things consistently separate people who finish from people who don't, and neither is implementation skill.

The first is time spent on the wrong approach. Fifteen minutes lost to a solution that doesn't fit the problem structure is the most common way a prepared candidate fails a screen. Pattern selection speed is the bottleneck.

The second is the follow-up. After you get it working, expect: what's the time complexity now, what if the input arrives as a stream instead of a list, what if the numbers can be negative, what happens at the boundaries. Those questions test whether you understand the tradeoffs of what you just wrote, or whether you reproduced something that happened to pass.

Mock pressure is the part you cannot get from study alone. Timed practice with someone watching, talking out loud the whole time, is uncomfortable exactly because it's the thing being tested.

## Where a question list stops helping

Reading 50 Q&A pairs teaches you the questions. It doesn't sort them into a system, and the coding half of the interview isn't a question list at all, it's a recognition problem. That's the gap [AlgoMonster](https://algo.monster?fpr=li69) was built to close. It's an interview prep course put together by ex-Google engineers and competitive programmers, organized around patterns rather than a pile of problems.

What's inside, as described on the official site and in the plan features:

- **Pattern-based structure.** The curriculum is built around roughly 48 patterns, each with a conceptual explanation, a reusable code template, and practice problems that apply it.
- **Decision flowcharts.** Questions like "is the input sorted?", "is the subarray contiguous?", "is this a tree or a graph?" walk you from a problem statement to a likely pattern. This is the closest thing to a cheat sheet for the step where most candidates stall.
- **Six programming languages, Python included**, alongside Java, JavaScript, C++, Go, and Haskell. You can work the same problem in whichever language your interview uses.
- **Interactive code editor in the browser**, plus Speedrun mode for timed, clock-running practice and an in-browser AI assistant that explains selected content or gives hints when you're stuck.
- **Content volume**, per the official page: 500+ lessons across 6 courses, 1,000+ illustrations, and 3,000+ editorials, with questions drawn from top tech companies and refreshed monthly.
- **Bonus courses** bundled with a subscription: a Foundation course that rebuilds CS fundamentals, and a Build an LLM course covering tokenization, attention, and training.
- **Text and diagrams, no video.** Every explanation is written. If you learn by reading and immediately coding, that's efficient. If lecture-style walkthroughs are how things click for you, it's a real mismatch.

👉 [See the current AlgoMonster plans and pricing](https://algo.monster?fpr=li69)

The honest limits, since they matter for planning: the problem bank is in the hundreds, not thousands, so volume practice usually means supplementing with LeetCode. System design coverage is introductory, which is fine for junior and mid-level loops and not enough for senior rounds. And nothing inside replaces a live mock interview with another human.

## All AlgoMonster plans, compared

AlgoMonster sells three tiers and no monthly option. Pricing rotates with promotions, and the checkout page has displayed different figures in different windows, so treat the numbers below as the promotional range and confirm on the official checkout.

| Plan | What you get | Price | Billing | Start |
| --- | --- | --- | --- | --- |
| Free | Limited access to sample patterns, lessons, and problems. Enough to judge the text-and-diagram format. | $0 | None | [Open the free access](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons, and problems; unlimited AI assistant; unlimited Speedrun; real questions from top tech companies updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps; 12 months of access | List $198/year, discounted to roughly $99/year ($8.25/month), with lower promotional windows seen | Billed annually, cancel or upgrade anytime | [Get Pro Yearly access](https://algo.monster?fpr=li69) |
| Pro — Lifetime | Everything in the yearly plan plus lifetime updates, one payment | List $378, discounted to roughly $189, with lower promotional windows seen | One-time payment | [Get Pro Lifetime access](https://algo.monster?fpr=li69) |

Two notes on the price. There's no monthly billing, so this is annual or nothing. And because the discount changes, the number you see can differ from the one above; the checkout page is the only figure that matters when you're actually buying.

## Which plan fits which situation

If you have an interview loop in the next two or three months and a single job search in front of you, the yearly plan is the cheaper way in. Twelve months covers the search plus the follow-up loops, and you can drop the subscription after.

The lifetime plan is the one to think about if you expect to interview again, which most engineers do every two to three years. That's also roughly the break-even point against renewing annually, so it's a straight calculation rather than a leap of faith. If you're prepping for promotions and outside offers on a recurring cycle, the one-time payment stops being a stretch.

One practical caution: the promotional price makes the annual plan look trivial, but a subscription you don't use is more expensive than no subscription. Start with the free material, check whether the pattern-first, written explanations actually match how you study, and then pick.

## A six-week plan that uses both halves of the question list

**Weeks 1–2.** Fundamentals plus arrays, strings, and hash maps. Answer the language questions out loud, not in your head. Write the Two Sum and duplicate-detection solutions from scratch, then explain the complexity to an empty room. It feels ridiculous and it's the point.

**Weeks 3–4.** Two pointers, sliding window, binary search, then trees and graphs. Work pattern by pattern rather than topic by topic, and after each problem, ask what in the statement would have told you to use that pattern. That question is the actual skill.

**Week 5.** Dynamic programming, which is where most people's confidence collapses. Start from recursion, add memoization, then convert to a table. Use timed sessions here; Speedrun mode in AlgoMonster is built for exactly this, cycling problems with a clock running.

**Week 6.** Company-specific question sets, then mocks under real conditions: 45 minutes, timer visible, narrating every step, one or two problems, and no pausing to look things up. Do the follow-up questions too. If you finish early, ask yourself the streaming version and the negative-numbers version.

## When AlgoMonster is the wrong purchase

Three cases, stated plainly.

You already know the patterns and your gap is raw volume. Someone who has solved 300 problems and wants 300 more should be on LeetCode, not here.

You learn from video. Every explanation on AlgoMonster is text and diagrams by design, and a candidate who needs a person on screen drawing an array is going to retain less, no matter how good the written lessons are.

You're targeting senior loops with a dedicated system design round. AlgoMonster's system design material is a starting point. L5-and-above rounds need 20 to 40 hours of dedicated distributed systems, database, and scaling prep that this platform isn't built to provide.

For everyone else, the pitch is straightforward: fewer questions, sorted by pattern, with a flowchart for the part of the interview where you'd otherwise burn fifteen minutes. The question list is the easy half. The recognition reflex is the half that gets you hired.
