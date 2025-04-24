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


Data

We use a dataset of 31,713 Fed MBS transactions (2009–2023). Each record
includes:

CUSIP (security ID)

Purchase Date and Sale Date

State-level breakdowns of mortgage principal (UPB) and percentages

We compute HoldingDays (Sale Date – Purchase Date). Mortgages still held
appear as NA holdings.

Methodology

1.  Data Cleaning & Reshaping

```{r}
library(dplyr) library(tidyr)

# Read and parse

raw \<- read_csv("org_full_time_series.csv", col_types = cols(.default =
col_character())) clean \<- raw %\>% mutate(across( where(is.character)
& ends_with("\_UPB"), parse_number ), `Purchase Date` =
as.Date(`Purchase Date`), `Sale Date` = as.Date(`Sale Date`),
HoldingDays = as.numeric(`Sale Date` - `Purchase Date`))

# Long format: one row per (CUSIP, State)

state_long \<- clean %\>% pivot_longer(cols =
ends_with("\_Percent_UPB"), names_to = "State", names_pattern =
"(.\*)\_Percent_UPB", values_to = "PercentUPB") %\>% filter(PercentUPB
\> 0)
```
2. Summary Statistics

Total Unpaid Principal Balance By State For Years, 2009-2023

(down![download-1](https://github.com/user-attachments/assets/881a9927-83b3-4bae-b2a5-047c8d7ec504)
load-1.png)

Top states by total Fed MBS principal: California, Illinois, Texas, New York, Florida.

Holding period summary:

```{r}
summary(clean$HoldingDays)
#   Min. 1st Qu. Median  Mean 3rd Qu.   Max.  NA's
#      0     616    721 1201   1372  5019  5096
```
<img width="338" alt="Screen Shot 2025-04-24 at 1 48 11 PM" src="https://github.com/user-attachments/assets/02c40299-243a-483d-bd9a-52154a295617" />

![download-2](https://github.com/user-attachments/assets/e1688528-c1fb-4036-9bba-f09f39937df2)
3. Concentrated vs Diverse Network Views

We explore how MBS that are concentrated in one state differ from more diverse pools at the network level.  A CUSIP is labeled Concentrated if one state accounts for over 50% of its principal; otherwise it is Diverse.

4. Building the State Network

Now the fun part: connecting states into a network graph. We create a bipartite incidence matrix incidence_mat of size (States × CUSIPs) where each cell is the percentage of that MBS’s UPB from that state.

For example, if Security X has 50% California loans, incidence_mat["California", "X"]=50. Using the bipartite R package, we treat this as a two-mode network. In such networks, nodes come in two types and connections only run between types. Here, edges run from states to CUSIPs when a state’s loans are in that security. To analyze state-to-state relations, we “project” the bipartite network to a one-mode state network, connecting states that share securities.

Specifically, we compute a weighted adjacency matrix state_adj where entry (i,j) is the sum of products of state i’s and state j’s percentages in each security. This captures how strongly i and j are linked via common pools. In code:
To analyze state-to-state relations, we “project” the bipartite network to a one-mode state network, connecting states that share securities. Specifically, we compute a weighted adjacency matrix state_adj where entry (i,j) is the sum of products of state i’s and state j’s percentages in each security. This captures how strongly i and j are linked via common pools. In code:

Incidence matrix: States × CUSIPs (weighted by PercentUPB).

Projection: State–state adjacency via matrix multiplication.

```{r}

state_adj <- incidence_mat %*% t(incidence_mat)  
g_state <- graph_from_adjacency_matrix(state_adj, mode="undirected", weighted=TRUE)

```
![download-4](https://github.com/user-attachments/assets/23ee66cd-851e-4ae1-b81e-d82dd17e735a)

![download-5](https://github.com/user-attachments/assets/83ce208c-d743-4eb1-a41b-435204048c29)

![download-6](https://github.com/user-attachments/assets/bdc82985-1783-4b05-ad56-05e74c788bb2)

We also consider a simplified version: treat any shared security as a link (ignoring weight) to see the unweighted network of co-occurrence.

In the concentrated-only network (second plot), many edges disappear. Only CUSIPs dominated by one state remain, so connections shrink—yet hubs like California still appear highly connected, showing it had many state‑heavy pools.

The weighted bipartite view (third) restores edge weights: state nodes (blue) connect to red CUSIP nodes with thickness proportional to percentage. We see that some states (again CA, NY, FL) link to many heavy‑weight edges, underlining their dominant role in concentrated MBS.

These visualizations justify thresholding weak ties (e.g., requiring ≥5 shared CUSIPs) before community detection and highlight structural differences between diverse and concentrated pools.


5. Community Detection

We apply the Louvain algorithm (Blondel et al., 2008) to g_state:

comm <- cluster_louvain(g_state, weights = E(g_state)$weight)
length(comm)         # number of communities
modularity(comm)     # modularity score Q

A moderate Q≈0.28 indicates meaningful but not extreme regional clustering.

Results

State Clusters
![download-8](https://github.com/user-attachments/assets/cffb687e-ab44-4d68-b1e4-6e6977ac337f)

Plotting the network with nodes colored by community reveals ~5 clusters (e.g., West, Northeast, Midwest, South, Plains) and outliers:

Puerto Rico and Guam appear as nearly isolated nodes — their mortgages seldom mix with other states’ pools.

Holding Periods vs. Concentration

We label each CUSIP as Concentrated (>50% in one state) or Diverse, then compare holding days:                              

![download-3](https://github.com/user-attachments/assets/e207d340-8827-49f7-978d-8abaec953c11)

Concentrated pools show longer median holds, consistent with less liquid, niche securities.

Puerto Rico Deep Dive

n=179 unique PR CUSIPs.

Median hold ~2142 days vs 721 days overall.

Mean hold ~2316 days vs 1201 days overall.

16.8% PR pools still unsold vs 16.1% portfolio-wide.

PR’s mortgages sat on the Fed’s books nearly three times longer, likely due to thin secondary markets and territory-specific loan programs.

Discussion

Regional clusters mirror economic/geographic bands: West, Northeast, etc. (Newman, 2006).

Hub states (CA, NY) connect widely—typical of scale-free networks (Barabási & Albert, 1999).

Outliers like PR reflect distinct market structures (Board of Governors, 2008; FRBNY Staff, 2015).

Policy insight: Nationwide asset purchases still have uneven geographic impacts, leaving some markets (territories) dependent on the Fed’s offloading strategy.

Conclusion

This network analysis of Fed MBS holdings shows how a large-scale monetary intervention creates a web of state-level exposures. By applying tools from network theory—bipartite projection, modularity clustering, hub analysis—we uncovered both expected hubs and surprising peripheries. As the Fed unwinds QE-era positions, understanding these geographic patterns can inform more targeted policy and risk management.

Puerto Rico’s isolation in the Fed’s MBS network—where the Fed purchased nearly exclusive Puerto Rican mortgage pools—provided crucial short-term liquidity and lower rates when private demand was scarce, cushioning homeowners during the crisis. However, those concentrated pools stayed on the Fed’s balance sheet roughly three times longer than diversified ones, signaling low private-sector appetite to refinance or trade PR loans. As a result, while Puerto Ricans enjoyed stability during the downturn, the lack of integration into broader U.S. mortgage markets entrenches structural vulnerabilities, leaving the island reliant on sporadic federal interventions rather than a dynamic, private-sector–driven housing finance system.

Allen, F., & Gale, D. (2000). Financial contagion. Journal of Political Economy, 108(1), 1–33.

Barabási, A.-L., & Albert, R. (1999). Emergence of scaling in random networks. Science, 286(5439), 509–512.

Blondel, V. D., Guillaume, J.-L., Lambiotte, R., & Lefebvre, E. (2008). Fast unfolding of communities in large networks. Journal of Statistical Mechanics: Theory and Experiment, 2008(10), P10008.

Borgatti, S. P., & Everett, M. G. (2000). Models of core/periphery structures. Social Networks, 21(4), 375–395.

Board of Governors of the Federal Reserve (2008). Press Release: Fed to Purchase Mortgage-Backed Securities.

Federal Reserve Bank of New York (2015). Understanding Mortgage Spreads. Staff Report.

Newman, M. E. J. (2006). Modularity and community structure in networks. Proceedings of the National Academy of Sciences, 103(23), 8577–8582.

Opsahl, T. (2010). Structure and evolution of weighted networks. Physica A: Statistical Mechanics and its Applications, 387(1), 340-345.
