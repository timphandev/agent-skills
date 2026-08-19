# Prose in the repository

Everything in the repository that is not code but is read as if it were: comments, error and log messages, README files, architecture notes, decision records, docs directories, module and package documentation. All of it is written once and read many times, by someone who does not have the author available and did not see any earlier version.

**The governing rule: prose in the repository describes the current state.** Not how it got here, not what it used to do, not who changed it or when. Version control already holds that history with a timestamp and a diff attached, and holds it accurately — a narrative in prose goes stale the moment someone edits the thing it describes without editing the prose. A reader who never saw the earlier version, which is eventually every reader, gets a description of something they are not looking at.

Prose has no compiler. Nothing fails when it drifts, which is why it drifts, and why the discipline has to be deliberate.

---

## Part 1 — Comments

### What earns a comment

Comments are worth their maintenance cost when they carry something the code cannot.

**Why, when the why is not recoverable from the code.** A constraint from the domain, a business rule, a deliberate trade-off, an ordering that matters for a reason the call site does not show. This is the highest-value comment and the most commonly missing one.

**Unidiomatic code.** When something looks redundant, roundabout, or plainly wrong but is deliberate, say why. Without that, the next reader "simplifies" it and reintroduces whatever it was avoiding — and their change looks like a cleanup, so it passes review.

**A link to an external source that governs the code.** A specification, an RFC, a standard, a provider's documented behavior, a known bug in a dependency. Put it directly above the code it explains, not in a header block far away. Where code was adapted from an outside source, link it: the reader gets context and any subsequent corrections, and some licenses require the attribution regardless.

**A boundary you could not verify.** The skill's rule against asserting unobserved behavior applies inside comments too, and a comment is where it is easiest to forget: mark the assumption rather than stating it flat, so the next reader knows what is established and what is inference.

### What does not

**A restatement of the code.** `i = i + 1; // add one to i` costs a line, adds visual noise, and becomes a lie the first time the code changes and the comment does not. If the code says it, the comment should not.

**A patch onto an older comment.** Rewrite the comment whole against the current code instead of appending a clause — the skill's "rewrite the unit whole" rule, applied to prose.

**A changelog, a ticket number, a date, or a name.** These belong to the commit. In a comment they age badly and answer a question nobody reading the code is asking. Linking a *specification* is different from citing a *ticket*: the specification explains what the code must do and stays true; the ticket explains why someone once opened a task, and reading it usually costs a login for context that is no longer relevant.

**A commented-out block.** Delete it. Version control is where code goes to stay recoverable, and a commented-out block is ambiguous in a way deleted code is not — no reader can tell whether it is a work in progress, a rollback waiting to happen, or something forgotten years ago.

**Cleverness aimed at another author.** A joke or an in-reference costs the reader time and returns nothing, and reads differently to someone who does not share the context.

### When a comment is a signal to fix the code

**A comment that exists to explain an unclear name is a naming bug.** Rename the thing and delete the comment — the better name helps at every call site, while the comment helps only where it sits.

**If a clear comment is hard to write, suspect the code.** Difficulty explaining a block usually means the block is doing too much or its logic is tangled, not that the right words are missing. Restructuring is the cheaper fix, and it removes the need for the comment.

### Marking work that is not finished

The report's **Left alone** and **Needs your call** sections are how unfinished work reaches the person who decides priorities. A marker in a file does not — nobody greps a repository to find out what a task left behind.

So a `TODO` is not a substitute for reporting. It is only acceptable alongside the report, where the code genuinely needs a pointer at the exact line, and it must name the condition that resolves it — `TODO: switch to the batch endpoint once it supports partial failures` — so a later reader can tell whether it is still live. A bare `TODO: fix this` is unresolvable by anyone but its author, who will not remember either.

Never use a marker to make an incomplete implementation read as intentional. A stub that looks finished is the failure mode "no half-done work" exists to prevent; leaving a `TODO` on it does not change what the next reader sees.

---

## Part 2 — Error messages

An error message is the only prose in the repository that gets read at the worst possible moment, by someone who is already blocked. It is also the only prose with two distinct audiences, and conflating them is the usual failure.

**Say what happened, what the consequence is, and what to do next.** Most error messages manage the first and stop. "Invalid date" leaves the reader guessing which field, which format, and whether anything was saved; "Start date must be before end date — nothing was saved" answers all three in the same breath. The consequence clause matters most when the operation was partial or was not performed at all, because that is what the reader is actually trying to work out.

**Name the specific thing, not the category.** The value that was rejected, the file that was not found, the field that was missing, the limit that was exceeded and what it is. An error that omits the identifier forces a search that the code could have skipped.

**Never blame the reader, and never editorialize.** No exclamation marks, no "oops", no "you forgot to". A message reaching a user is a failure of the software, whatever caused it, and jokes read badly to someone whose work just failed.

**Write for the audience that actually reads it.** A message a developer will see — a panic, a startup failure, a log line — carries the internals: values, identifiers, the state that was violated. A message crossing to a client does not, and `security.md` governs where that line falls. Where both are needed, they are two separate strings, not one string reused in both places.

**A wrapped error adds context, not narration.** Each layer prepends what *it* was trying to do — `loading user config: opening /etc/app.toml: permission denied` — so the chain reads as a path. What it must not do is restate the layer below it (`failed to load config: config loading failed`) or narrate the code's structure rather than the operation's purpose.

**Do not put a resolution in the message if the code can just do it.** "Try again" belongs in a retry, not in a sentence. Suggest an action only where the action is genuinely the reader's to take.

---

## Part 3 — Documents

A README, an architecture note, a decision record, anything under a docs directory. The governing rule still holds — and "rewrite whole" bites hardest here, because two paragraphs that disagree leave the reader working out which is current with no way to. What follows is what a document needs beyond a comment, being longer-lived and further from the code that would contradict it.

### Write for the reader who arrives cold

**Say what the thing is before saying anything about it.** A document that opens mid-topic assumes context its most important reader — the one who has none — does not have.

**State the current design, not the path to it.** No migration narratives, no "recently changed" sections, no comparison against how it used to work. A decision record is the one exception, and only within its own frame: it records a decision made at a point in time, so it is allowed to say "we chose X over Y because Z". That is not a changelog; it is the content. What it must not do is get edited later to describe a newer decision — supersede it with a new record and mark the old one superseded in one line.

**Every reference must be verified before it is written.** File paths, command names, config keys, environment variables, URLs. Run the command; open the path. A confidently-wrong reference is worse than an absent one, because the reader trusts it and spends time proving it wrong.

### Keep it true, or delete it

**Update the document in the same change that invalidates it.** A doc corrected in a later pass is wrong in the interim, and the interim is exactly when someone reads it and acts on it. This is the same rule as "batch what belongs together": the doc is part of the change, not follow-up work.

**Prefer deleting a stale document to leaving it.** An absent document sends the reader to the code, which is always true. A confidently wrong one stops them looking, which is why it does more damage than nothing at all. If deleting it is beyond the task's scope, that is a **Left alone** line in the report, not a silent pass.

**Do not document what the repository already records.** The file tree, what a function does, the list of endpoints when they are generated from the code. Anything derivable from the source will diverge from it, and the copy is always the one that is wrong. Documents earn their place with what the code cannot say about itself: the why, the constraint, the trap, the alternative that was rejected.

### Length is not a virtue

A document is read under time pressure by someone looking for one thing. Sections that exist for completeness rather than because someone needs them push the useful part further down and make the whole thing feel unmaintained. When a section has no reader you can name, cut it.
