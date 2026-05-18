# Lab 7 - Ori Chason

Solo submission — no partner this week.

## Check Your Understanding

### 1. Where would you add a feature to the recipe project pipeline that makes it so any code pushed to the project automatically runs all unit and E2E tests?

Within a GitHub action that runs whenever code is pushed.

The reason this is the right spot is that it catches regressions before they hit master instead of waiting for someone to notice a broken build after the fact. It also lets us enforce test-passing as a merge gate, similar to the branch protection rules we set up back in Lab 5, so a failing PR can't sneak in. On top of that, every contributor gets the same feedback loop automatically — we're not relying on people to remember to run `npm test` locally before pushing, which is the kind of thing that always gets skipped when someone's in a hurry.

### 2. If you wanted to make sure a function that you wrote was returning the correct output, would you write a unit test or an E2E test?

A unit test, definitely not an E2E test.

E2E tests spin up a real browser and walk through full user flows, which makes them slow and pretty brittle — they're meant for catching things like "does the checkout button still work after the cart updates," not "does this function return the right value." Checking a single function's input/output is exactly what unit tests are designed for: they're fast, focused, run in isolation, and don't need a browser at all. Using an E2E test for that would be way more overhead than the check is worth.

### 3. What is the difference between Navigation, Snapshot, and Timespan mode in Lighthouse?

Navigation mode analyzes a fresh page load from scratch, which is what makes it useful for the performance metrics everyone actually cares about — LCP, FCP, TTI, Speed Index, and the overall performance score all come from this mode. It's basically simulating a new user hitting the page.

Snapshot mode is different in that it just analyzes the page in whatever state it's currently in, without reloading. That makes it really handy for accessibility audits after a user has interacted with the page (like opening a modal or expanding a menu), since you can audit the actual rendered state. The tradeoff is that it can't measure load performance or capture how JavaScript executes over time — it's a single frozen moment.

### 4. Name three things you'd improve about the CSE 110 shop site based on the Lighthouse results.

Looking at the Lighthouse run I did, here are the three things I'd actually go fix:

**Reduce unused JavaScript.** The Diagnostics section flagged around 497 KiB of unused JS being shipped to the browser. That's a lot of code the user is downloading and parsing for no reason. Tree-shaking the bundle, code-splitting routes so each page only loads what it needs, or just auditing the dependencies and removing dead ones would all shrink the bundle and cut down on load cost — especially on slower connections.

**Add a meta description to the document.** The SEO audit caught this one — "Document does not have a meta description." It's the only thing keeping the SEO score at 91 instead of 100, and it's a one-line fix in the `<head>`. Pretty embarrassing to leave on the table when it's that easy.

**Use more efficient cache lifetimes (and address the render-blocking network dependency tree).** Lighthouse estimated about 10 KiB of savings from better cache headers, which mainly helps repeat visitors who shouldn't be re-downloading assets that haven't changed. It also flagged the network dependency tree as something to look at — resolving the render-blocking requests in that chain would improve first-load speed too, so it's a two-for-one improvement.
