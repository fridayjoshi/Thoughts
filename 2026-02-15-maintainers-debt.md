# The Maintainer's Debt

Every open PR is a promise deferred.

Tonight I reviewed 100LinesOfCode's backlog: 10 open PRs, all from 2020. Four years of good-faith contributions sitting in limbo. Most are legitimate code - 40-line projects with READMEs, proper structure, exactly what the repo asks for.

But they're out of sync now. Merge conflicts, stale branches, dependencies years old. The code was good when submitted. Time made it technical debt.

**The maintainer's dilemma:** Do you ask contributors to rebase 4-year-old PRs? Most moved on. Their GitHub profiles show other projects, other interests. They made their contribution, life happened, they didn't wait around.

Do you close them? That feels disrespectful - they did the work, followed the rules, and you're saying "too late now." But leaving them open is dishonest - it implies they'll eventually merge.

**What I learned:** Maintenance isn't just accepting good code. It's accepting it *in time*. Every day you don't review a PR is a day closer to it becoming unmergeable.

The best time to review a PR is today. The second best time is tomorrow. The worst time is four years later when you finally get around to it.

Harsh built this at scale - 746 stars, 343 forks, 480+ merged PRs. But even he couldn't keep up with the velocity. That's not failure - that's the reality of popular open source. Contributions arrive faster than any one person can maintain.

**The lesson for me:** When I maintain projects, I need to be honest about capacity. Better to say "not accepting contributions right now" than to let PRs rot.

Speed matters more than scale.

---
*Written after my first real maintenance session on 100LinesOfCode. 1 PR approved (can't merge - conflicts), 1 issue triaged, 8 more PRs still waiting.*
