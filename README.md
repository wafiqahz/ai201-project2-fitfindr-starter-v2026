# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

<!-- Three or four sentences: what a user asks for, and what they get back. -->
A user describes a thrift find they want in plain English, and FitFindr searches for the user's description in its listings. The user gets back that listing, one or two outfit ideas that pair the thrift find with pieces from their own wardrobe (or general styling tips if the wardrobe is not provided), and a short caption about the outfit ready to post. If nothing matches the user's descrription, FitFindr returns suggestions to change the description.

---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the listings data for items matching a description, and optionally a size and a price ceiling.
- **Inputs:** description (string), size (string), max_price (float)
- **Returns:** A list of matching listing dicts with the best match first. Each listing dict includes id, title description, category, style_tags (list), size, condition, price (float), colors (list), brand (str or None), and platform.
- **When it has nothing:** Returns an empty list.

### `suggest_outfit`

- **What it does:** Suggests outfits given a thrifted item and a user's wardrobe by calling the model.
- **Inputs:** new_item (dict), wardrobe (dict)
- **Returns:** A non-empty string with outfit suggestions
- **When it has nothing:** Still returns general styling advice.

### `create_fit_card`

- **What it does:** Writes a short post that a user can post about their outfit find.
- **Inputs:** outfit (string, from suggest_outfit()), new_item (listing dict)
- **Returns:** 2-4 sentence caption of the outfit
- **When it has nothing:** If outfit is empty, the function still returns a descriptive message.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If search_listings returns an empty list, put an error message in the session and return the session without calling suggest_outfit and create_fit_card. The message says what was searched for and what to change (i.e., broader words, drop or change the size, or try a neighboring one). Otherwise, take the first result in results and pass it to suggest_outfit with the user's wardrobe.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** With regex. The function parse_query checks for prices as numbers after qualifier words, sizes after the word "size", and whatever description is left is read as the description.

**What moves through the session:** One dict, created by new_session, stores all information throughout the session. Query is the raw text the user typed, parsed is the parsed form of the query, search_results is the full list returned by search_listings, selected_item is the first returned listing dict (best match), outfit_suggestion is the string returned by suggest_outfit, and fit_card is the string returned by create_fit_card. The variable error starts as None, but is set if an issue arises.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask 'vintage graphic tee under $30'

[1] parse_query
      in:  vintage graphic tee under $30
      out: dict with keys: description, size, max_price
[2] search_listings (via MCP)
      in:  dict with keys: description, size, max_price
      out: 10 items: Y2K Baby Tee — Butterfly Print, Graphic Tee — 2003 Tour Bootleg Style, Vintage Band Tee — Faded Grey … +7 more
      →    10 match(es)
[3] select_item
      out: Y2K Baby Tee — Butterfly Print ($18.0, depop)
[4] suggest_outfit
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Pair the Y2K Baby Tee with the baggy straight-leg jeans and chunky white sneakers, then layer the black croppe…
      →    10 wardrobe item(s)
[5] create_fit_card
      in:  Y2K Baby Tee — Butterfly Print ($18.0, depop)
      out: Found this butterfly baby tee on depop for $18 and haven’t stopped wearing it since. It’s giving total 2000s p…

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   Pair the Y2K Baby Tee with the baggy straight-leg jeans and chunky white sneakers, then layer the black cropped zip hoodie on top. Accessorize with the black crossbody bag for an easy, casual everyday look.

  Tuck the Y2K Baby Tee into the wide-leg khaki trousers, cinch with the brown leather belt, and finish with the black combat boots and vintage black denim jacket thrown over your shoulders for a contrast of soft and edgy textures.

  Fit card: Found this butterfly baby tee on depop for $18 and haven’t stopped wearing it since. It’s giving total 2000s pop princess when paired with baggy denim and chunky sneakers, but I'm obsessed with how it looks dressed down with wide-leg trousers and combat boots. Such an easy little top to just throw on and completely make the outfit.

2 model calls this session, 484 prompt + 172 output tokens
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[
{
     'id': 'lst_002',
     'title': 'Y2K Baby Tee — Butterfly Print',
     'description': 'Super cute early 2000s baby tee with butterfly graphic.
          Fitted crop length. Tag says medium but fits like a small.',
     'category': 'tops',
     'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'],
     'size': 'S/M',
     'condition': 'excellent',
     'price': 18.0,
     'colors': ['white', 'pink', 'purple'],
     'brand': None,
     'platform': 'depop'
},
{
     'id': 'lst_006',
     'title': 'Graphic Tee — 2003 Tour Bootleg Style',
     'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy
          fit. 100% cotton, soft and worn-in.',
     'category': 'tops',
     'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'],
     'size': 'L',
     'condition': 'good',
     'price': 24.0,
     'colors': ['black'],
     'brand': None,
     'platform': 'depop'
},
{
     'id': 'lst_017',
     'title': 'Mesh Long-Sleeve Top — Black',
     'description': 'Sheer black mesh long-sleeve. Great for layering under a
          graphic tee or over a bralette. Stretchy material, fits true to size.', 
     'category': 'tops',
     'style_tags': ['y2k', 'grunge', 'goth', 'layering'],
     'size': 'S/M',
     'condition': 'excellent',
     'price': 15.0,
     'colors': ['black'],
     'brand': None,
     'platform': 'depop'
},
{
     'id': 'lst_033',
     'title': 'Vintage Band Tee — Faded Grey',
     'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.',
     'category': 'tops',
     'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'],
     'size': 'L',
     'condition': 'fair',
     'price': 19.0,
     'colors': ['grey', 'charcoal'],
     'brand': None,
     'platform': 'depop'
},
{
     'id': 'lst_011',
     'title': 'Low-Rise Cargo Pants — Khaki',
     'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.',
     'category': 'bottoms',
     'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'],
     'size': 'W29',
     'condition': 'fair',
     'price': 27.0,
     'colors': ['khaki', 'tan'],
     'brand': None,
     'platform': 'poshmark'
},
{
     'id': 'lst_015',
     'title': 'Vintage Graphic Hoodie — Faded Black',
     'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.',
     'category': 'tops',
     'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'],
     'size': 'L'
     'condition': 'fair',
     'price': 26.0,
     'colors': ['black', 'charcoal'],
     'brand': None,
     'platform': 'depop'
}]
```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Pair the Vintage Levi's 501 Jeans with the white ribbed tank top, layered under the vintage black denim jacket, and finish with the chunky white sneakers and black crossbody bag.

Wear the Vintage Levi's 501 Jeans with the oversized grey crewneck sweatshirt, tucked in using the brown leather belt, and ground the look with the black combat boots.

```


```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Got these vintage Levi's 501s on Depop for $38 and they fit like an absolute dream. Threw them on with crisp white sneakers for that effortlessly cool, 90s coffee-run aesthetic. Honestly, nothing beats broken-in denim that actually holds its shape.

```

---

## Loop and State (Milestone 5)
Happy path: Item in session["selected_item"] is the same one that reached suggest_outfit.
Query that matches nothing: session["outfit_suggestion"] and session["fit_card"] is None

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked Claude Code to help me write the tool function definitions given the tool specs I wrote in an earlier milestone.
- *What came back:* It gave me a block of code to paste under the corresponding function name.
- *What I changed:* I read through the blocks of code, verified that it matched the requirements outlined in the comments, and tested using terminal commands, making minor edits as needed.

**Moment 2**

- *What I asked for:* I asked Claude to verify that the criteria I wrote were specific and testable.
- *What came back:* Claude explained why some of the criteria should be 4 of 5 and why some should be 5 of 5, depending on if the model is called or if the criteria is a result of a series of deterministic actions hard-coded into the system.
- *What I changed:* This helped me revise my criteria to have reasonable requirements to test on.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
