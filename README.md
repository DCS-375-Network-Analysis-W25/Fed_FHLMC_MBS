---
editor_options: 
  markdown: 
    wrap: 72
---

# Fed_FHLMC_MBS

Unpacking the Fed’s Mortgage Network When the U.S. Federal Reserve buys
mortgage-backed securities (MBS), what hidden connections tie together
different states’ housing markets? This blog post follows the Fed’s MBS
purchases through the Great Recession and the COVID-19 crisis to reveal
a geographic web of support—and surprising outliers like Puerto Rico.
We’ll map these connections using network science, guiding you from the
basics of MBS to an insightful community analysis. Strap in for a
journey that blends code, charts, and clear language as we turn a
technical R analysis into a story of policy and place.

Background: Why the Fed Buys MBS and Why Geography Matters

An MBS (mortgage-backed security) is a bundle of home loans packaged
into an asset. Homeowners’ mortgage payments flow into these securities,
which investors can buy. During the 2008 financial crisis, mortgage
markets froze and the cost of home loans spiked. In response, the Fed
began purchasing MBS in large quantities to “reduce the cost and
increase the availability of credit for the purchase of houses”
(federalreserve.gov). In other words, the Fed stepped in to lower
mortgage rates and stabilize housing finance. These large-scale asset
purchases (often called “quantitative easing”) made the Fed a major
player in mortgage markets. By mid-2024 the Fed held about 30% of all
outstanding agency MBS (some \$2.3 trillion worth)
(federalreserve.gov)(ginniemae.gov).

MBS might seem abstract, but they have a real geography. Each MBS
contains loans from different places – imagine an MBS as a bag of
mortgages from Florida, New York, Kansas, etc. When the Fed buys an MBS,
it is indirectly supporting those local housing markets. Our question:
how are these local markets connected through the Fed’s MBS holdings? If
two states often appear together in the same MBS pools, that could mean
their housing finance fortunes are intertwined. Network theory offers
tools to explore such connections. We’ll build a network where states
are linked if they share MBS in the Fed’s portfolio – a classic two-mode
(bipartite) network projected to one mode​. States that frequently
co-occur in MBS pools will cluster, revealing regional patterns. We’ll
also see hubs (states like California or New York) that connect to many
others – a hallmark of networks where a few hub nodes hold many ties​.
Using the Louvain algorithm from community detection​, we’ll identify
clusters of states that form tightly knit sub-networks, potentially
reflecting regional or economic similarities. All this matters because
the Fed’s support may not be uniform. If the Fed mostly ends up with
loans from certain states, those places effectively get more relief. By
visualizing and quantifying these patterns, we connect the high-level
policy (Fed buying MBS to stabilize credit) back to ground-level
outcomes (which states’ loans are supported).

Before diving in, let’s outline our plan: Data: Use Fed MBS holdings
data (2009–2023) broken down by state of the underlying loans.
Summaries: Chart total mortgage principal by state and time to see broad
trends (who dominates? how did it change?). Network construction: Build
a bipartite state–MBS network (states connected to the MBS pools they
are in), then project to a state–state network where edges weight how
often states share pools​

Community detection: Apply Louvain modularity clustering to find groups
of states that are tightly connected​

Holding periods: Analyze how long the Fed held each MBS (from purchase
to sale) and whether that correlates with state concentration (e.g. were
pools mostly from one state held longer?).

Puerto Rico focus: Investigate Puerto Rico’s role as an outlier – it’s
not a state, but the Fed held some PR loans. Does it sit apart in the
network, and how long did those loans stay on the Fed’s books?


