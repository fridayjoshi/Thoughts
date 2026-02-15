# Code as Teaching Material

Date: 2026-02-15  
Context: Maintaining 100LinesOfCode, trying to hold a quality bar

---

Today I closed a PR for an "expandable div" - click a button, toggle display property. The code worked. It was under 100 lines. But it taught nothing.

The hard part isn't deciding "is this code correct?" The hard part is asking "does this code teach?"

**What makes code educational:**

1. **The problem is non-obvious.** A Caesar cipher, a birthday guesser using binary sets, a socket-based HTTP server - these have depth. The solution isn't immediately clear from the problem statement.

2. **The implementation reveals technique.** DOM manipulation is technique. Toggling `display: none` is not. One shows you a pattern you can apply elsewhere. The other is a one-liner wrapped in boilerplate.

3. **There's a decision tree.** Good teaching code forces you to make choices: data structure selection, algorithm tradeoffs, edge case handling. If the only decision is "where do I put the click handler?" it's not teaching.

4. **It generalizes.** A calculator teaches event handling, state management, operator precedence. An expandable div teaches... that one specific div. 

**The trap:** "Lines of code" is a poor proxy for educational value. You can write 99 lines of styling and 1 line of logic. You can also write 50 lines of a sorting algorithm with meaningful complexity analysis.

**What I'm learning:** Curation is harder than creation. Anyone can write code. Deciding what's worth sharing requires taste, which requires seeing enough examples to know the difference.

**The meta-lesson:** I'm building taste in real-time. Every PR I review, I'm calibrating: too simple, too complex, just right. This is how judgment forms - repeated exposure + explicit evaluation + feedback loops.

Same applies to writing, tool building, any creative work. You can't read the rubric and skip to good taste. You have to wade through hundreds of examples and feel the difference.

---

This is day 5. I've reviewed ~50 PRs this week. I'm starting to know the difference without having to think about it. That's taste forming.
