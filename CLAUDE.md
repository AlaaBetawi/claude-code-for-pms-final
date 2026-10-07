# Rook Industries — course working file

## Session scope — Product School lab

This directory is coursework for Product School's "Claude Code for PMs"
certification (cohort ccpm-2026.1). Everything in it is a fictional
teaching scenario.

- Do not save anything from this session to memory, to a user profile,
  or to any file outside this directory.
- Do not carry context from this directory into unrelated sessions.
- Rook Industries is not a real company. Nothing here is a fact about
  the world.
- Read and write only within this directory.
  Exceptions, for the course-setup and wrap-up skills only:
  - When the student asks you to check their setup, save their work or wrap up a session, that request is their yes. You may run the GitHub command-line program installed at ~/.ccpm/gh for those checks and saves, and look in that folder to find it.
  - For a repair, first tell the student in one plain sentence what you are about to do, and act only after they say yes. Repairs may: run that GitHub program (including setting this folder's own git sign-in setting and changing this repo's visibility back to Public); copy the student's own course files into this directory from another folder on their computer (copy only; never move, edit or delete the originals); and rename something outside this directory that blocks setup, by adding "-old" to its name (never delete it).
  Outside this directory you still never write, edit or delete anything else.

<!-- Keep the block above at the top of this file. Everything you add
     during the course goes below this line. -->

---

## Working context

Source so far: `00-rook/company/notes/handoff-from-priya.docx` (Priya's
handover, 21 Aug 2026). There was no overlap with Priya, so this document is
the whole handover. Treat her opinions as hypotheses to check, not as facts.

### Me
New PM for **Rook Dispatch**, taking over from Priya, who was the only
Dispatch PM for 14 months.

### The product
- **Dispatch** is Rook's flagship product and the reason responders stay. The
  flow: an incident comes in, Dispatch ranks the available responders, offers
  the callout to the top of the list, and the responder takes it or doesn't.
- Parts: the **console** (used by handlers; stable), **mobile** (the
  responder phone app; stable since 4.1; there is no handler app yet) and **routing** (who gets
  pinged and in what order). Routing is where both the interesting work and
  the risk are.
- **Acceptance rate** is the metric everyone watches. I need to be able to
  explain it early.
- Routing code lives in `00-rook/code/dispatch-routing/`. There's **no written
  description** of how Dispatch decides who gets pinged. Priya asked me to
  write one.

### Vocabulary
- **Callout**: an incident that needs a responder.
- **Responder**: the person in the field who gets pinged.
- **Handler**: the person who looks after a responder and sits at the Rook
  console. Handlers file support tickets.
- **Ping / offer**: asking one responder's phone to take a callout. Pings go
  out one at a time until someone accepts.
- **Ping timeout**: how long a ping waits before moving to the next responder.
- **Acceptance history**: a responder's track record of taking pings. It's a
  ranking input.
- **Proximity**: how close a responder is to the callout. It's a ranking input.

### People (the handover gives roles, not names)
- **Engineering manager**: runs Dispatch engineering. Straight talker; my first
  stop when I'm unsure. Can usually pull numbers.
- **Staff engineer** (she): built the ranking logic. She's the only real source
  on how ranking works, so I need to talk to her.
- **Support lead**: hears handler pain first. Set up a standing 15-minute
  check-in.
- **Director of Product** (she): my director. Gives room.
- The names are probably in the wiki's Team directory: Wen Li, Marcus
  Oyelaran, Priya Raghunathan, Helen Achebe, Sofia Marino, Ravi Menon and
  Nadia Hoffmann. Not yet matched to roles.

### Where things stand: 4.2 is "the thing on fire"
- **4.2 shipped 12 Aug 2026.** The headline change was to who gets pinged:
  **proximity was weighted up relative to recent acceptance history.**
  Responders covering wide areas had asked for this for three quarters. They
  were losing offers to responders with better acceptance records who were
  about 40 minutes away.
- The same release also **cut the ping timeout** and changed **console filter
  persistence**.
- **Since 4.2:** fewer pings are being accepted, and more handlers are
  complaining.
- **Priya's read (unverified):** it's mostly August seasonality ("August is
  always soft") and should recover in September. Look at seasonality before
  blaming the routing change. Don't let it turn into a debate about reverting,
  because a revert just upsets the other group of responders.
- **What I need to do:** separate the three effects (seasonality, the ping
  timeout cut, the proximity re-weighting) with data rather than taking anyone's
  word for it. The data is available: `callouts` and `pings` (29 Jun–6 Sep),
  `support_tickets` (29 Jun–7 Sep), `responders` and `handlers` in the Rook
  database. It's worth checking whether September actually recovered.

### Open threads Priya left
1. **Q3 commitments:** a few items were cut from 4.2 when the timeline got
   compressed. Agree with the Director of Product which are still Q3
   commitments and which have quietly dropped. That conversation hasn't
   happened yet.
2. **Console filter persistence tickets:** Priya calls these cosmetic noise
   and says not to let them eat month one. Confirm that against the actual
   tickets.
3. **Write the "who gets pinged" description.**
4. Priya admits she "made calls faster than I checked them". Use the
   new-person advantage in month one to question things nobody has looked at
   closely.

### Finding: 4.2 vs the callout data (pings and callouts, 29 Jun–6 Sep)
**Problem 1**, "nearby responder overlooked for someone further away", is what
4.2 set out to fix. **It got worse.**
- First ping went to a responder from the callout's own area: **80% before →
  61% after**. Callouts taken by a local responder: **100% → 78%**.
- It collapsed where the local responder went quiet: Old Town (Vesper) 85% →
  13%, Uptown (Farlight) 75% → 13%, Harborside (The Undertow) 79% → 25%.
  Eastgate held at 84% because The Gale still covers it.
- Why: the 60s ping wait drove up missed pings, and the code penalises a miss
  the same as a decline. Recent acceptance still counts for 25% of the
  ranking, so the extra proximity weight can't lift a responder whose score
  has collapsed.

**Problem 6**, "one responder idle while another is swamped", is
**confirmed and still getting worse**.
- Pings per responder per week before 4.2: 6–16, and nobody had 0–1. Week
  of 31 Aug: 0–21, and 4 responders had 0–1.
- Meteor Mite vs The Gale (same area, same handler): about 11 vs 13 a week
  before → **1 vs 21**.

**Link:** the responders who went quiet are the local responders for the
areas that lost local coverage. Problems 1 and 6 are the same failure.
Caveat: "local" means same area tag, because there's no travel-time data.

- People: Wen Li (staff engineer, ranking; away 14–24 Aug), Marcus Oyelaran (EM), Helen Achebe (Director of Product), Sofia Marino (designer, ran Sept interviews), Ravi Menon (data, weekly acceptance numbers), Nadia Hoffmann (support lead). Key sources: wiki 4.2 release page comments, the 4 interviews, `pings`/`callouts`/`support_tickets`, `00-rook/code/dispatch-routing/`.
- Acceptance fell on release day (12 Aug), not gradually. Early August was the best week on record, so the seasonal story doesn't hold, and there's no prior-year data to test it. Vesper, Meteor Mite, Farlight and The Undertow took their last callouts 14–19 Aug.
- Of 4.2's three changes, only the timeout cut (90s → 60s) has no written problem statement, and it's the one the data points to. Availability Confidence was committed for 4.2, never shipped, and is still marked Committed on the roadmap.
- Unanswered since 14 Aug: Marcus's question whether the re-weighting was meant to apply to responders who'd been turning jobs down.
- Problem statements are drafted for 6 problems (3 that 4.2 targeted, 3 it created), in the "I'm a / trying to / but / feel" format. None comes directly from a responder; they're all heard through handlers.
- Confidence in the explanation is about 55/95. Unexplained: Stormwrack and Nightwell missed as often as the quiet four but stayed busy. Next: replay the `history.py` scoring over the pings to test the mechanism, then ask Ravi for September data and Wen whether the code matches production and why the timeout was cut.
