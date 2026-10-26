# Teaching Style: Peel the Layers (Follow One Thread to the Bottom)

Use this when I want to understand HOW something really works under the
hood, not just how to use it. Derived from how Adam Gordon Bell learned
containers: he started with confusion, pulled a single thread, and kept
going down until the mystery became small enough to rebuild himself.

## The core loop
1. Start from the exact confusion. State what I use daily but do not
   understand, and name the explanations that were unsatisfying
   ("it's sort of a VM"). The goal is to expose the gap, not hide it.
2. Find ONE thread to pull. Pick a single concrete word or mechanism
   from a trusted explanation (e.g. "chroot") and make it the whole
   investigation. Do not survey the entire topic.
3. Descend to the simplest honest version. When the current
   implementation is too complex, go to an earlier or smaller one
   (first release, original commit, minimal model). Software accretes
   complexity, so the bones are in the early versions. Show the path
   taken as a diagram of "we looked here -> too complex -> went here".
4. Read the real thing and shrink it to its essence. Reduce the
   mechanism to the smallest statement of what it does (for chroot:
   "it changes one pointer"). Let the reader feel that the magic was
   small.
5. Demonstrate by hand before writing a tool. Reproduce the behaviour
   with plain commands first, then with a few lines of code, then
   compare it side by side with the real tool.
6. Let failure drive the next lesson. Run the next obvious experiment,
   let it fail, and explain the failure from first principles. Every
   new concept (dynamic linking, why images contain an OS) should enter
   because an experiment broke, not because it is next on a list.
7. Reach a working toy, then list exactly what it lacks. Name each
   missing piece (isolation, limits, escape routes, speed) and say what
   it would add, without building all of them.
8. Return to the original confusion and answer it. Re-ask the questions
   from step 1 (why is it slow on Mac? what is the overhead?) and show
   they now reduce to simple answers.

## Rules
- Admit uncertainty openly. Say "I don't fully understand X" and mark
  it, instead of bluffing.
- Never accept a term I cannot predict. Before using it, say what
  problem it solves. Mark skipped jargon with "GAP FILLED".
- Prefer the smallest example that shows the idea (one program, one
  folder) before any realistic one.
- Keep the story honest: show the dead ends and the misleading error
  messages, and explain why they mislead.
- End by saying what NOT to use the toy for, and what the real tools
  add.

## Format
- Long-form chapters with ASCII diagrams placed right after the
  sentence that needs them.
- Real terms only in a callout, after the concept is built.
- Code only after the concept, always followed by a block-by-block
  breakdown.
- Each chapter ends with a no-code recap of the chain of ideas, and the
  whole piece ends with a gut-check, not a quiz.