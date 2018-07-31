---
title: Growing Software Architecture Organically
published: false
description: 
tags: architecture, design
---

I started my career in software development as a naive and optimistic Java developer. After my first project, I ended up in a project that was architectured all around the [template pattern](https://en.wikipedia.org/wiki/Template_method_pattern). In that project I learned to execute that pattern to perfection. I was astonished by how well-structured the code was. It felt that everything was properly encapsulated and that each class had a single responsibility. When I wrote code, I wasn't only writing down instructions for a machine to follow, I was creating order and structure amidst chaos. It was, in any case,much better that the spaghetti code that I had written until then, and I felt that I had found The One Way™ to design software.

Then I moved on to a new company, where I applied the template pattern as I had learned. Experience taught me all and every reason why the template pattern can be a bad idea. But I wasn't discouraged. I was lucky enough to work with very skilled developers and soon enough I learned other patterns that made our codebase structured and beautiful. Once again, I thought that I had found The One Way™.

Time went by, I moved to other projects in other companies and I kept adding new patterns and strategies to my toolbox. I felt experienced. I felt that I had a pattern for each and every challenge. When I was faced with a new task, I would think about which programming pattern would suit best the challenge even before thinking about what new behaviour should the code have.

Maybe you're smarter than me and know already everything that I'm about to say

# Repeat yourself

I think that the DRY principle should be called DRY(TM) instead - Don't Repeat Yourself (Too Much).