<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/banner.svg">
    <img src="assets/banner-light.svg" width="100%"
         alt="Sitraka Rasatarivony — Full Stack Developer, System Design. 4s to 300ms, 10+ modules in production, DevSecOps. Core stack: TypeScript, NestJS, React, PostgreSQL.">
  </picture>
</div>

Three years shipping backend and frontend systems to production, from architecture
to continuous delivery. Antananarivo, Madagascar.

Three projects follow. Each had one thing it had to get right, and that thing is what
its section shows.

---

### cine-taste — a trending ranking that decays on its own, with no scheduled job anywhere in the codebase

A movie's score is the attention it has received, each interaction discounted by
how long ago it happened:

$$S(t) = \sum_i w_i\, e^{-\lambda (t - t_i)}, \qquad \lambda = \frac{\ln 2}{t_{1/2}}$$

Factor the constant out and what is left is an accumulator that only ever needs an
O(1) update — no rescan of history, ever. And since $e^{-\lambda t}$ is positive and
shared by every movie, it cannot change their order: sorting by the accumulator is
enough. That is why nothing has to run on a schedule. The ranking decays correctly
by construction.

The accumulator overflows a float64 quickly, so it is stored as $\log A$ and updated
with log-sum-exp, which keeps it finite for decades.

Three half-lives run in parallel, giving three views of the same journal:

<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/decay.svg">
    <img src="assets/decay-light.svg" width="100%"
         alt="Three exponential decay curves over fourteen days, with half-lives of one hour, one day and one week. Each crosses half its original score at its own half-life.">
  </picture>
</div>

Next.js 16 · Postgres · Drizzle &nbsp;·&nbsp;
[repo](https://github.com/SitrakaRasata/cine-taste) &nbsp;·&nbsp;
[live](https://cine-taste-sr.vercel.app)

### immo-grant — every access rule lives in the Postgres schema, and a delegated mandate expires without any row changing

The textbook case, "I can see what I own", proves nothing. The one worth building is
delegation: an agent holds a mandate on a listing they do not own, and it lapses on
its own. Every cell below is one named test.

|                      | SELECT | INSERT | UPDATE | DELETE |
|---|:--:|:--:|:--:|:--:|
| anonymous, published | yes | — | — | — |
| anonymous, draft     | —   | — | — | — |
| client               | yes | — | — | — |
| owning agent         | yes | yes | yes | yes |
| mandated agent       | yes | — | yes | — |
| unrelated agent      | —   | — | — | — |

The suite applies the production schema verbatim inside an in-process PostgreSQL.
No container, no external service, no server to start.

Three traps it walks into on purpose: policy recursion between two tables, which
needs a `SECURITY DEFINER` function to break; the discipline that function then owes,
since it is an RLS bypass; and the fact that `GRANT` is not RLS — a mandated agent may
edit a listing but must not reassign it, which `WITH CHECK` cannot express because it
only ever sees the row as it will be, never as it was.

SvelteKit · Supabase · PGlite &nbsp;·&nbsp;
[repo](https://github.com/SitrakaRasata/immo-grant) &nbsp;·&nbsp;
[live](https://immo-grant.vercel.app)

### tournament-engine — a format is a pure function returning a graph, so no `if (format === …)` exists anywhere else

A format takes its entrants and returns the matches to play plus the edges saying
where each winner *and each loser* goes next. Nothing else.

```mermaid
graph LR
  M1[Match 1] --> SF[Semi final]
  M2[Match 2] --> SF
  M1 -.-> LB[Losers bracket]
  M2 -.-> LB
  SF --> F[Final]
  LB --> F
  classDef signal fill:#FBBF24,stroke:#B45309,stroke-width:2px,color:#0D1117;
  class F signal;
```

Solid edges carry the winner, dashed ones the loser. Everything downstream reads that
graph and knows nothing about brackets: propagating a result, cascading a correction
through every match it invalidates, computing live title odds, deciding the tournament
is over. Adding a format is one file and one registry entry.

The odds are exact on edge formats — one dynamic-programming pass over the DAG, each
pairing weighted by the Elo expected score, no sampling error. Round robin has no
edges, so outcomes are enumerated exactly while few enough remain and sampled beyond
that. Both are covered by property tests asserting the probabilities sum to one and
stay monotone in rating.

NestJS · React · SQLite &nbsp;·&nbsp; 249 tests &nbsp;·&nbsp;
[repo](https://github.com/SitrakaRasata/tournament-engine) &nbsp;·&nbsp;
no public demo — it runs from one Docker command

---

### What I have shipped for other people

- **Cut a VMware monitoring app's data load from ~4s to 300–500ms** by rewriting its
  backend in FastAPI — vSphere REST for routine operations, pyVmomi for the advanced
  SOAP SDK — with caching and ClickHouse query optimization. *Stellar-IX (Axian Group)*
- **Shipped 10+ full-stack modules to production** with Docker CI/CD and systematic
  code reviews; brought TypeScript to new modules on a legacy ejs/Node codebase.
  Team of 10, Agile.
- **Built the full backoffice of an e-learning platform** for a European client, in a
  team of 8. *Internship — Ingenosya*
- **Co-founding [NEXTRI](https://nextrimg.github.io)**, a three-engineer collective,
  as its technical lead — an internal application end to end: NestJS/TypeScript API,
  PostgreSQL schema, CI/CD, licensing, DevSecOps from the design phase.

DevSecOps Professional — Practical DevSecOps, January 2024.

### The rest of the stack, beyond the four on the banner

Java and Spring Boot, Python with FastAPI and Flask, Express. Angular before React.
MySQL, MongoDB and ClickHouse alongside PostgreSQL. Docker, Nginx, Linux, and CI/CD
pipelines I maintain rather than inherit.

AI-assisted development every day (Claude Code).

---

<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/SitrakaRasata/SitrakaRasata/metrics/latency.svg">
    <img src="https://raw.githubusercontent.com/SitrakaRasata/SitrakaRasata/metrics/latency-light.svg" width="100%"
         alt="Median response time of the two live demos, measured every six hours from GitHub's runners.">
  </picture>
</div>

Both demos are probed every six hours from GitHub's runners, five requests each, and
the figure above is the median — a measurement, not a claim. A failed probe breaks the
line rather than being drawn as a fast response.
[The workflow](.github/workflows/metrics.yml) rewrites a single-commit `metrics`
branch each time, so no machine noise ever reaches this history.

---

[LinkedIn](https://linkedin.com/in/sitraka-rasata) &nbsp;·&nbsp; rasatasitraka2@gmail.com

*Malagasy (native) · French (fluent) · English (professional, written)*
