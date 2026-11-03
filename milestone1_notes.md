# Milestone 1: Community Notes

Community: Hacker News comment threads (news.ycombinator.com), public, no login.

I pulled 220 recent top-level comments from the front page through the public Hacker News API and read through them. Anything under 60 characters or over 600 was dropped. Reddit was my first choice, but it blocked unauthenticated requests, so I switched. The unit of classification is the comment, not the story it sits under.

## Candidate Distinctions

- **Checkable vs. take-my-word:** Some comments hand you something to verify: a link, a CVE number, a quoted passage, a shell command, "46% to 33%". Others make the same kind of point with nothing to check.
- **First-person account vs. general claim:** A lot of comments are "I did X" or "my T420 still works", which is different from both a fact and a judgment. They are about the writer.
- **Asking vs. telling:** A visible group of comments only ask something ("What is keeping people on Ubuntu instead of Debian?"). Many others use a question mark to make a point ("How on earth do they see this as an asset?").
- **Short praise vs. substance:** "Very neat project!" and "This is great!" are common, and they offer a verdict but nothing behind it.

These came from the comments I read.

## AI Stress Test

Claude wrote eight comments that sit on the boundaries of the four definitions, and then classified them itself, so this checks that the rules read clearly but is not an independent test. I classified them with the draft rules:

1. `I ran this on my 2019 laptop and it took 4 minutes.` - `experience`
2. `Per the changelog, this shipped in 2.4, so your complaint is out of date.` - `evidence`
3. `Why would anyone use this when SQLite exists?` - `opinion` (rhetorical, not a request for information)
4. `Has anyone benchmarked this against SQLite?` - `question`
5. `I've been using it for years and it's the best option.` - `experience`
6. `According to a post I read once, it's 3x faster.` - `opinion` (no link, quote, or number I can check)
7. `Does anyone know why the build is failing? It's been broken since 2.4, per the issue tracker.` - `question`
8. `Every company I've worked at has dropped it eventually.` - `experience`

These cases are the reason the definitions already say a number in a first-person account is not `evidence` unless a stranger could check it, and that rhetorical questions are `opinion`. The test did not change the definitions.
