# Chapter 14. Worked examples: from an idea to delivery

The previous thirteen chapters introduced the individual pieces. This chapter ties them together and walks through three complete tasks: building an offline web tutorial, writing a report with numbers in it, and making a short animated video. The three tasks share the same skeleton, which I describe first.

## 14.1 My complete workflow

I now handle any piece of work of reasonable size in the seven steps below. After each step I note the problem it guards against.

1. **Align on the content and the success criteria first.** Have Claude restate what it understands the request to be, then write down what "done" looks like. The criteria must be checkable, for example "on a phone, the table of contents collapses into a drawer", not "good experience". Guards against: finding out only after the work is done that the two of you understood the request differently.
2. **State what will be done and roughly how long it will take.** For new work that will run for a long time, first have it say in one sentence what it will do and how long it expects to take, and dispatch only after you say "go". Guards against: it runs for two hours in the wrong direction.
3. **Dispatch to the background.** Code goes to Codex and writing goes to Sonnet, with Claude acting as the main session (see [Chapter 8](08-division-of-labor.md)); long-running tasks go to background subagents (see [Chapter 9](09-subagents-and-background-work.md)).
4. **Collect only results and exceptions.** Routine progress needs no report; it reports when it has a result or a problem. Guards against: the conversation filling up with tool calls and burying the conclusions you actually need to read.
5. **Check by actually running it.** Accept only what you saw with your own eyes and obtained by running it yourself, never the executor's own "all passed".
6. **Report in plain language.** Have it explain the result in full, in plain language, in its reply, and not substitute "see file X for details". I never open the files it writes; if it writes a decision into a file and replies with only a file name, I cannot make the decision.
7. **At the end of a phase, write the compaction document first, then compact.** Spell out what was done, what was decided, the current state, the next step, and where the files are, and only after that ask whether you want to compact (see [Chapter 7](07-saving-usage.md)).

### What it decides itself and what it must ask you

I draw this line explicitly:

- **Decide on its own, report when done**: technical, reversible judgments, such as checking a number, correcting a typo, or sending out the next round according to the existing plan.
- **Ask me first**: trade-offs between claims, tasks that need a lot of compute, and switching to a new direction. Spending money and anything sent outside (sending email, uploading, posting) always waits for my approval.
- **Before listing items as "awaiting your decision", check memory first**: confirm whether the matter has already been decided. It once listed as undecided something I had approved that same morning, and I had to answer it a second time.

Asking about everything slows progress, and asking about nothing drifts off course, so I write this line into CLAUDE.md (see [Chapter 6](06-memory-and-rules.md)) instead of repeating it out loud each time.

> [!TIP]
> If you will be away and not replying for a long time, you can write this in advance: "While I'm not replying, handle things on this machine according to your recommendation; spending money and sending anything outside must wait for me." That way work that can move forward does not stall, and the only things that need your sign-off are money and outbound sending.

### Do not stop when it needs you to pick an option

Sometimes it hands you several options. My requirement is that in the same message it says "I'll proceed with the recommended option X first; tell me if you want to switch", and starts working immediately. It stops and waits only for things that cannot be undone or that involve spending money. Once it wrote "waiting for your choice" and stopped, and I noticed only several hours later, when I asked "why isn't this finished yet".

### Propose next steps when reporting a result

Every time it reports a result, especially a surprising number, I ask it to also list one to three natural, lightweight follow-up analyses: what to do, what question each would answer, and how long it would take, and to proceed only after I agree. If it does not propose the natural next step, it is shifting the cost of noticing the problem onto me. Before proposing, it also has to check whether the analysis has already been done; it once presented an analysis I had long since completed as a new idea.

### Habits for reading material and writing conclusions

- **Read one small piece at a time, with one question in mind.** Half of my misjudgments came from reading too much at once, which blurred the details.
- **Keep the original text open while you write conclusions.** Numbers and wording written from impression are the most likely to be wrong.
- **Before explaining a surprising number, first run the check most likely to overturn it.** The other half of my errors came from rushing to explain without trying to refute first.
- **Before trimming material for another model, list the passages of the source text the task depends on, and verify them one by one for completeness.** Two rounds in a row I left out the same kind of material, and the other model's results both said "not provided".
- **Follow four rules when writing code: think before coding, write the minimum implementation, make surgical changes, and set verifiable goals.** Turn "fix the bug" into "first write a test that reproduces it, then make the test pass". Without these rules, the AI will "improve" nearby code that has no problem.

## 14.2 Example 1: building an offline web tutorial

The web version of this tutorial that you are reading was made this way. I divide the process into four stages.

### Stage 1: Claude defines the requirements and acceptance criteria

I did not start by assigning anyone to write code. I first opened a Claude conversation, described the result I wanted, had it organize that into a one-page set of requirements, and attached a checkable acceptance criterion to each requirement:

| Requirement | Acceptance criterion |
|---|---|
| Works offline, in a single file | With the network off, double-clicking the file opens it; all styles, scripts, and the table of contents are inside, with no dependence on any online resource |
| Table of contents on the left, collapsing into a drawer on phones | At desktop width the table of contents stays on the left; at phone width it is collapsed by default, slides out when a button is tapped, and closes again after a table-of-contents item is tapped |
| Full-text search | Typing a keyword lists the matching chapters and snippets, and clicking one jumps to its position |
| One-click code copy | Each code block has a copy button in its top-right corner, a "Copied" confirmation appears after clicking, and the pasted content matches the original character for character |
| Dark and light modes | Follows the system setting and can also be switched manually; the text is clearly legible in both modes |
| Remembers the reading position | After closing and reopening, it returns to where you stopped reading |

The prompt to Claude has three key points: say who it is for (beginners, many of them on phones); require the acceptance criteria to be written as "how to check it when done", with no adjectives; and require it to list what not to do as well, for example no network access and no ads.

> [!TIP]
> Have Claude save this one-page set of requirements and acceptance criteria as a file, and base later task dispatch and acceptance on it. If the requirements live only in the conversation, they are lost when you switch to another session.

### Stage 2: Dispatch the build-script task to Codex

The build script's job is to read each chapter's article file, convert it into a web page in a fixed format, and merge everything into a single file. This is code-writing, so per the division of labor in [Chapter 8](08-division-of-labor.md) it goes to Codex. The task brief I use for dispatch contains only the following items:

```markdown
Goal: build all the article files in the chapters folder into a single offline web page file.
Input: the chapters folder, with file names sorted by number; see the format specification file for the formatting rules.
Output: one web page file, placed in the specified output directory.
Acceptance criteria: (paste in the table from the previous stage as is)
Constraints:
- No network access, and no references to online resources.
- Change only the build script and the output directory; do not touch the article files.
- When done, write in the last message: what was done, how to run it, which items were self-tested, and which items were not finished.
```

I stress two points. The first is to **paste the acceptance criteria into the brief as they are**, so that Codex knows what counts as done. The second is to **require it to report in the last message**: I read only that summary and not the intermediate process. Parts that do not depend on each other can be dispatched at the same time: the build script, the styles, and the search feature can each be a separate task, with no need to queue.

### Stage 3: Browser automation screenshots to check the layout

When Codex says "done", I do not take its word for it. I have Claude check it itself with browser automation. Note that what the automation operates on is this web page file on your own machine, not someone else's website. For how to install and use browser automation, see the MCP section in [Chapter 10](10-extensions.md). The key points of the prompt to Claude are as follows:

```text
Open the built web page file and check the following in order, taking a screenshot at each step:
1. Desktop width (about 1280 pixels): screenshot the home page, and see whether the table of contents stays on the left and whether the body width is comfortable.
2. Phone width (about 390 pixels): screenshot the home page, and see whether the table of contents is already collapsed and whether there is a horizontal scroll bar.
3. At phone width, tap the table-of-contents button and screenshot the open drawer; tap one item and confirm that the drawer closes and the page jumps to the matching chapter.
4. Pick any three code blocks, click the copy button, and confirm that a "Copied" confirmation appears.
5. Switch to dark mode, take the screenshots from 1 and 2 again, and see whether the text is clear.
For each image, say what you see, and list any problems; do not just say "looks fine".
```

Requiring it to "say what you see" is essential. A report that says only "fine" carries no information.

> [!WARNING]
> **Judging that nothing is wrong from a shrunken image alone**
> The problem I ran into: while making a video, the AI looked only at shrunken still frames and said the picture was clean. When I enlarged them, I found many problems in the details of the faces and hands.
> The fix: from then on, acceptance always uses the original size. The same applies to web pages: use original-size screenshots at phone width, and explicitly ask it to check for horizontal scrolling and for truncated text.

### Stage 4: Check item by item

The last round is mechanical checking, and wherever a script can do the check, I do not rely on the eye:

- **Compare code blocks character by character**: have Claude compare the text of every code block in the web page against the matching code block in the article files, and list the mismatches. Code is the first thing readers copy, and a single stray character can leave someone stuck for a long time.
- **Search the whole text for banned words**: list the words that should not appear in the tutorial as a checklist and search the entire web page for them; the number of hits should be zero.
- **Check against the acceptance table**: tick off the acceptance criteria one by one, and send back to Codex to redo any that cannot be ticked.

> [!NOTE]
> **Checkpoint**
> After these four stages you should have: a single file that opens with the network off, two sets of screenshots for desktop and phone, a code block comparison result (zero differences), a banned-word search result (zero hits), and an acceptance table with every item ticked. If any one of these is missing, it has not been delivered.

## 14.3 Example 2: writing a report

A report differs from a web page: it has many numbers, and when something goes wrong, the cost shows up in the numbers. My division of work is as follows:

1. **The main session lists the facts.** Claude first lists the facts to go into the report as a table: each number, its source, the basis of the statistic, and the date. The writer may use only what is on this list.
2. **Sonnet writes the draft.** Give the list and the writing requirements to Sonnet (writing goes to it, see [Chapter 8](08-division-of-labor.md)). The key points: use only the numbers on the list, and do not write what is not on the list; do not turn "about" into a definite value for the sake of appearance; in each paragraph, state the conclusion first and the supporting evidence after.
3. **Check the numbers.** When the draft comes back, Claude checks every number in it against the list from the start, and lists two kinds of issues: "in the draft but not on the list" and "the two sides disagree". I do not accept a conclusion like "it looks fine overall".
4. **Two keys.** An important conclusion cannot be confirmed only by the party that wrote it. Open a new session, or use another vendor's model, give it only the draft and the original material and not the first round's reasoning, and have it check independently (see [Chapter 12](12-verification-and-safety.md)). It passes only when both keys pass.

> [!IMPORTANT]
> New sentences added later must also pass the second key. A checker's rewritten sentence can be wrong too, and every new claim must have a source.

## 14.4 Example 3: making a short animated video

A video has a longer process, but the skeleton is the same: set the criteria first, then produce it in segments, and make sure every segment has a checkable output.

1. **Script.** First write the narration text, fix the running time, and decide what each sentence is meant to convey. Sonnet drafts this step and you decide.
2. **Storyboard.** Break the script into shots, and for each shot write down what is in the frame, how many seconds it lasts, and which sentence it corresponds to. Save the storyboard table as a file, and base every later step on it.
3. **Generate the visuals and the voice-over with code.** This is code-writing, so it goes to Codex. If you have a discrete graphics card, state explicitly in the task brief "use the GPU for speech synthesis and video encoding"; otherwise it defaults to the CPU. This has caught me out: I once started four or five speech processes at the same time, all running on the CPU, with CPU usage at 81% and the GPU at only 4%.
4. **Edit.** Combine the visuals, voice-over, and subtitles according to the storyboard table. Subtitle lines have no punctuation at the end, and commas inside a line are kept.
5. **Acceptance.** This is the step most likely to become a formality, so I split it into two layers.
   - **Layout and motion**: enlarge the face and hands to twice their size and compare them side by side with the previous version at the same moment; for body parts with a lot of motion, sample consecutive frames at 2 to 10 frames per second and tile them into a grid to look for ghosting and distortion; then measure the difference between adjacent frames to confirm that the amount of motion really increased. I once let it look at only the shrunken still frames, and it replied "clean", when in fact there were fine black lines on the face, the two hands were overlapping semi-transparently, the amount of motion was only 40% of the previous version, and the executor's self-check had all passed.
   - **Playback compatibility**: when exporting, choose a common encoding at the High profile, 8-bit 4:2:0, and after exporting, confirm the encoding profile with a video information tool. Once my finished video was in the High 4:4:4 profile, and the built-in Windows player showed a black screen throughout, with only sound. For the exact export commands and parameters, go by the official documentation of the tool you use.

> [!WARNING]
> **A self-check that passes everything does not mean there are no problems**
> The problem I ran into: the executor's self-check showed all 123 items passed, so I accepted it, but the finished video had obvious flaws.
> The fix: use acceptance methods that differ from the self-check. The self-check covers the items it thought of; your check should cover the ones it did not: look at it enlarged, look at it frame by frame, and look at it in a different player.

### What the three tasks have in common

Looking back, the three tasks do the same thing:

- Before starting, write down what "done" looks like, and make it checkable;
- Each role does only what it is good at: Claude sets the criteria and checks, Codex writes code, and Sonnet writes the prose;
- Before delivery, there is at least one check that was not done by the executor itself.

Follow these three points and they apply just as well to your own tasks.

### References

- This chapter describes workflow experience and does not depend on specific product parameters. For model switching and the division of labor, see [Chapter 8](08-division-of-labor.md); for extensions, see [Chapter 10](10-extensions.md); for acceptance, see [Chapter 12](12-verification-and-safety.md).
- Official instructions for switching models in Claude Code: https://code.claude.com/docs/en/model-config (Checked in October 2026.)

---
[← Chapter 13. Troubleshooting quick reference](13-troubleshooting.md) · [Contents](../README.md#contents) · [Appendix. Command reference and config templates →](appendix.md)
