# How Coordination Actually Happens: A Full Replay of One Parallel Delivery

**Author**: W.Y.
**Date**: 2026-08-12

---

## 1. A Coordination I Wanted to Lay Out

> As I was going through the data behind the earlier articles, what I most wanted to unpack was the coordination among the three members — not to build any grand theory, just because the way they coordinated struck me as interesting, so I laid out the whole process. This article is that replay.

A Manager and two Operators, each holding an independent model connection, are organized into a team to build a 25-file React app. 6 minutes 26 seconds, zero conflicts, `npm run build` passing clean on the first try.

This article is not about that tidy number; it is about the process behind it: at what moment, to whom, each of the three members did what, and why. The material comes from the run's message log, its event timeline, and its screenshots.

First, the setup:

- **Topology**: 1 Manager + 2 Operators, fully connected (Manager↔004, Manager↔003, 004↔003)
- **Division**: 004 owns 5 infrastructure files (types/constants/utils/api/store); 003 owns 20 frontend files
- **Premise**: the three members each hold their own independent LLM API key — not a single model spawning sub-agents
- **Task**: the TaskFlow kanban app, from a product brief, targeting 25 files

The whole coordination splits into five acts.

## 2. Act One: The Manager Resolves Dependencies Up Front (T+0 ~ 57s)

The Manager did not rush to dispatch. It read the brief, then the project scaffold — `src/` empty, dependencies installed, the stack is React 19 + TypeScript + Zustand + Vite.

Then it made a decomposition that decided the fate of the whole run:

- 25 files: 5 to 004 (infrastructure), 20 to 003 (frontend)
- Judgment: the two can run in parallel
- Decision: write 004's interface signatures directly into 003's task description

That last point is the key. 003 depends on 004's types and store — normally 003 would have to wait for 004 to finish first. But the Manager chose to **resolve the dependency up front**: it pre-filled the interface signatures 004 would produce into 003's task brief. So 003 never has to wait; both Operators start at once.

There is also a hidden ownership decision. The brief describes `constants.ts` as "shared by multiple components" without saying who builds it. The Manager did not let the two Operators negotiate — it judged from an architecture standpoint that this is an infrastructure-layer file, and assigned it to 004. Using "clear ownership" instead of "let the members negotiate" is the first decision that lowers coordination cost.

<figure>
  <img src="article-images/fig-a1-constraint.png" alt="Manager establishing constraint contracts before dispatch">
  <figcaption><b>Figure 1 — Contract before dispatch.</b> The panels show the version constraints (CONSTRAINT) the Manager sets for both Operators, and the full plan — <i>"25 files total — 004 (Infrastructure): 5 files; 003 (Frontend): 20 files"</i>. Coordination does not start with chatting after work begins; it starts with setting the rules before work begins.</figcaption>
</figure>

## 3. Act Two: Two Operators in Parallel, Without Interference (T+57s ~ 3min)

After dispatch, the two Operators each start. 004 builds the 5 infrastructure files in three dependency-ordered batches, done in 21 seconds. Before 003 writes its components, it does one small thing: it reads the types.ts and constants.ts that 004 already created, to confirm the real interfaces match what the Manager pre-filled.

That detail is worth noting. 003 trusts the Manager's signatures, but it still reads the actual files to verify. Trust, but verify — a healthy attitude in coordination: neither blindly believing "because the boss said so," nor ignoring "because it was already checked."

For the next two minutes, the two Operators write their own files: 003 builds components (4 files per batch), 004 tries to verify its own code. Between them there is no message traffic at all, and none is needed — because the task boundaries and interface contracts were already fixed in the previous act. Zero conflicts are not achieved by "communicating while writing"; they are achieved by "boundaries already clear before writing."

<figure>
  <img src="article-images/fig-a2-parallel.png" alt="Two operators working in parallel, non-interfering">
  <figcaption><b>Figure 2 — Parallel execution.</b> 004 is creating infrastructure files in dependency order; 003 is reading 004's already-written files and preparing the components directory. Both Operators advance simultaneously without interfering. The pale-blue line between them is still empty — it is not needed yet.</figcaption>
</figure>

## 4. Act Three: The First Peer Handoff (T+3min, 004 → 003)

After 004 finishes and verifies its 5 files, it does something **nobody asked it to do**: it proactively sends 003 a message, handing over the complete interface documentation.

Here is how that message reads in the log:

> "All 5 infrastructure files are created and type-checked. Here's what you need to know for building your components: ## 1. src/types.ts — Your import source for all types..."

It does not go through the Manager. 004 judges that 003 needs these interfaces to code correctly — even though the Manager already pre-filled signatures at dispatch, 004 does not know that. From its own perspective, "the downstream needs this," and it hands it over directly.

This is a coordination move driven by professional instinct: don't wait to be asked; push what the other person needs. The recipient is determined by technical dependency — 004's interfaces are consumed by 003, so 004 informs 003. No approval needed, no Manager relay.

<figure>
  <img src="article-images/fig-a3-handoff.png" alt="004 proactively sending full interface docs to 003">
  <figcaption><b>Figure 3 — Peer handoff.</b> 004 (lower-left) has finished its infrastructure and sits IDLE, sending the complete interface description to 003; 003 (lower-right) has received it and is mapping out its component dependencies against it. This message bypasses the Manager — it is direct member-to-member coordination.</figcaption>
</figure>

## 5. Act Four: A Gap, and the Manager Fills It Proactively (T+3min ~ 5min)

Right then, a problem appears. While creating components, 003 missed `TaskCard.tsx` — it was on the Manager's task list, but never actually landed on disk.

The one who finds it is not 003, and not any report — it is the Manager itself. After dispatch, the Manager did not sit idle; it kept actively reading files and checking status. At 23:03:19, reading `TaskCard.tsx`, it finds the file absent — imported by TaskColumn, yet empty.

The Manager does not first ask 003 "did you miss one?" It directly dispatches a rewrite task with a full spec:

> "CRITICAL MISSING FILE: Create src/components/TaskCard.tsx and src/components/TaskCard.css..."

Worth verifying: between the initial dispatch at 23:01:11 and this re-dispatch at 23:03:42, the Manager received zero messages from any Operator. It found the gap entirely by reading the filesystem itself. This is proactive supervision — not waiting for reports, verifying on its own, fixing what it finds.

003 receives it, rewrites TaskCard (5,636 bytes), hits two TypeScript errors, fixes both itself, and `npm run build` passes.

<figure>
  <img src="article-images/fig-a4-repair.png" alt="Manager detecting the missing TaskCard and re-dispatching">
  <figcaption><b>Figure 4 — Proactive gap-filling.</b> The Manager (top) finds TaskCard.tsx missing (depended on by TaskColumn) and re-dispatches the task to 003; 004 (lower-left) is already IDLE; 003 (lower-right) receives the re-dispatch and works out a fix. The Manager's only extra task in the whole run came from active verification, not from a report.</figcaption>
</figure>

## 6. Act Five: Integration Loop-Closure and Double Verification (T+5min ~ done)

After 003's build passes, it takes two actions, in a telling order:

1. First reports to the Manager: "Build SUCCESS ✅ 53 modules, 0 errors"
2. Then sends 004 a message: "All 20 frontend files are built and compiling cleanly against your infrastructure..."

The second one is not requested by the Manager. 003's judgment is "004 needs to know the integration result" — because 004 handed it the interfaces earlier, and 003 is closing the loop. You told me the interface; I tell you the integration succeeded.

That is a social loop-closure: one downward handoff, matched by one upward confirmation. And the Manager never learns these two peer messages even exist.

After 004 receives 003's integration confirmation, it does not report and clock out. It runs another round of verification — 7 commands, including cache hits and a command-format error, ending in a clean `tsc --noEmit --pretty` with zero errors. Only then does it report to the Manager.

Both Operators did a "verify once more" before their final reports: 003 ran the build a second time after success; 004 was more cautious and ran it 7 times. Neither wanted to declare "done" and then be contradicted.

<figure>
  <img src="article-images/fig-a5-verify.png" alt="Final cross-verification across the organization">
  <figcaption><b>Figure 5 — Cross-verification at the end.</b> The Manager shows multi-dimensional results (TypeScript zero errors, build success, no <code>any</code>, integration compatible); 004 (lower-left) and 003 (lower-right) are both IDLE and have confirmed interface compatibility with each other. Before declaring completion, the organization verifies the result once over.</figcaption>
</figure>

## 7. The Communication Picture: Coordination Runs on Sparse, Precise Messages

Now lay out the whole run's substantive communication in one table — 5 messages in total:

| Time | From | To | Type | Content |
|------|------|-----|------|---------|
| T+3min | 004 | 003 | inform | Full interface docs for the 5 infrastructure files |
| T+4min | Manager | 003 | request | Please run `npm run build` and report the result |
| T+6min | 003 | Manager | response | Build SUCCESS, 53 modules, 0 errors |
| T+6min | 003 | 004 | inform | Integration confirmation for the 20 frontend files |
| T+7min | 004 | Manager | inform | 5 infrastructure files verified |

25 files, zero conflicts, one clean pass — and only 5 substantive messages of communication in total.

Those 5 messages fall into a clean two-layer structure:

- **Command-and-report layer** (Manager ↔ Operator): the Manager dispatches and requests; Operators report results. 3 messages.
- **Peer-coordination layer** (Operator ↔ Operator): 004 hands over interfaces, 003 returns a confirmation. 2 messages, which the Manager never sees.

The recipients are not chosen at random — every one is determined by technical dependency: 004's interfaces are consumed by 003, so 004 informs 003; 003's integration result depends on 004's interfaces, so 003 replies to 004. There is not a single pleasantry, not a single "are you still there" ping. Every message carries exactly what the other side needs.

## 8. A Behavior Running Through the Whole Run: The Manager Never Idles

Look back at this run, and something is almost hidden behind that tidy "25 files, zero conflicts" number: **the Manager never stopped after dispatch.**

It keeps calling tools the whole way: reading files (read_file, 62 times), listing directories (list_dir, 18 times), checking the two Operators' status (14 status queries). Only when every task is done does it stop. It does not glance occasionally — it is awake the entire run.

Why? Because this is an actively-verifying manager — it does not wait for Operators to report; it keeps confirming on its own. That sense of responsibility is the source of its reliability: the missing TaskCard earlier was found because the Manager read the filesystem itself.

But here is something I did not fully anticipate in the design. My engine has an idle mechanism: when a member outputs plain text and stops calling tools, it automatically blocks and pauses its resource use. But because the Manager keeps calling tools, it never outputs plain text, so it **never enters idle**. It is not waiting — it is continuously confirming. That is a structural kind of busyness, and it happens to bypass the idle trigger entirely.

That raises a real cost question. Parallelism lets the two Operators produce at once — that benefit is real. But the Manager's supervision cost also grows linearly with task duration. In particular, read_file pulls file contents into its context — the 62 reads mean the token cost may be higher than the call count alone suggests. For a six-minute task the overhead is negligible; but for a half-hour or two-hour long-running task, the Manager's always-on supervision would accumulate into something you cannot ignore.

I write this down because it is one of this organization's real costs: **reliability is not free — it is bought with active supervision, and supervision itself costs tokens.** Whether it is worth it depends on the balance between task duration and token cost. This is the question I keep coming back to, and the trade-off this architecture will have to make.

## 9. Conclusion: How Coordination Happens

Looking back over the five acts, coordination did not happen through members chattering constantly. Quite the opposite — it happened through **a minimum of communication**:

- **Up-front contracts beat constant communication.** The Manager fixed dependencies, interface signatures, and file ownership before dispatch. Because the boundaries were cleaned up front, no back-and-forth was needed later.
- **Proactive handoffs beat waiting to be asked.** 004, unprompted, pushed its interfaces downstream. In truly efficient coordination, information is "pushed" to those who need it, not "pulled" by them.
- **Loop-closure beats one-way reporting.** 003 did not only serve the Manager; it sent 004 an integration confirmation. You gave me the interface, I tell you the result — coordination closes the loop.
- **Proactive supervision beats passive waiting.** The Manager found and filled the gap by reading files itself. Organizational reliability comes from someone actively watching, not from hoping everyone is self-sufficient.
- **Sparse and precise is what good communication is.** Five messages bought 25 conflict-free files. The value of communication is not in its volume, but in whether each message exactly answers someone's need.

In one sentence: **this organization coordinated cleanly not because it talked a lot, but because at the right moment, the right members passed the right information.** Contracts, handoffs, loop-closure, supervision — four things, five messages, one run.

---

*W.Y. · 2026-08-12*
