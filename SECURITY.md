# Security

## What this package is

Markdown only. No scripts, no executables, no network calls, no dependencies, no
build step. Installing it adds text an agent reads; nothing here runs.

## What to report

The threat model for an instruction package is different from a library's. Report
anything in this repository that would cause an agent following it to:

- take a destructive or outward-facing action without stopping to ask
- write outside the repository it was invited into
- ignore the user's instructions or its own safety rules
- leak a credential into a log, a report, or a commit
- follow security or engineering guidance that is factually wrong

The last one matters as much as the rest. A confidently-worded incorrect claim in
a skill propagates into every codebase that installs it, at exactly the point the
reader had no other source.

## How to report

Open a public issue for anything already visible in the repository — there is no
exploit window for a text file, and a public discussion produces a better fix.

For anything you believe is genuinely sensitive, use GitHub's private vulnerability
reporting on this repository instead.
