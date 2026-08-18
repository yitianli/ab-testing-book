# Interference and Network Effects

Chapter 8 studied one way the clean A/B test can break: assignment may not equal actual treatment receipt. A treatment user may not receive treatment, or a control user may accidentally receive it.

This chapter studies a different but related problem. Even if each unit receives its assigned treatment, units may still affect each other. A treated user may change the market, feed, network, or supply conditions faced by control users.

Both problems weaken the simple idea that treatment and control are perfectly separated. Chapter 8 focuses on treatment receipt within a unit. This chapter focuses on spillovers across units.

Most A/B tests rely on a quiet assumption:

> One user's treatment assignment does not affect another user's outcome.

This assumption is often called no interference. In causal inference, it is part of the stable unit treatment value assumption, or SUTVA.

Many product experiments satisfy this assumption well enough. If a user sees a different button color, that usually does not affect another user's conversion. If a user receives a new onboarding checklist, that usually does not change another user's experience.

But in marketplaces, social networks, ranking systems, auctions, logistics systems, and creator ecosystems, users interact through shared resources. One user's treatment can change what another user sees, how much supply is available, or how the platform allocates attention.

For example, suppose a travel marketplace gives treatment users a promotion for listings in a certain city. Treatment users may book more of the available listings. Control users do not receive the promotion, but they may still face reduced availability because some listings have already been booked by treatment users. The treatment-control conversion difference may then look too large: treatment users benefit from the promotion, while control users are partly hurt by depleted supply.

When this happens, a simple user-level A/B test can overestimate the effect.

This chapter explains interference, spillover, network effects, practical ways to diagnose whether spillover may be happening, and designs that can reduce or account for spillover.

## What Is Interference?

Interference occurs when one unit's treatment affects another unit's outcome.

In a standard user-level A/B test, the outcome for user $i$ is usually written as:

$$
Y_i(T_i)
$$

This notation says user $i$'s outcome depends only on their own treatment assignment $T_i$.

With interference, the outcome may depend on many users' assignments:

$$
Y_i(T_1, T_2, \ldots, T_n)
$$

This makes the comparison more complicated. A control user in a treated environment may not represent the outcome that would have happened in a fully untreated world.

The control group is no longer pure control.

In this chapter, spillover means the effect that one unit's treatment creates for other units. Treatment saturation means the share of nearby or connected units assigned to treatment. A useful way to think about the outcome is:

$$
Y_i = f(\text{own treatment}, \text{treatment saturation around }i)
$$

## Interference Channels

Interference is the broad problem: one unit's outcome depends on other units' treatment assignments. Spillover is one specific way interference appears:

> Treated units indirectly affect untreated or control units.

Interference usually travels through a specific channel. Understanding the channel helps decide whether a user-level test is safe and what alternative design may be needed.

Common channels include:

**Social Exposure**

In social products, treated users can directly expose control users to treatment-driven behavior.

Examples:

- A treated user shares more content, and control friends see it.
- A treated user sends more invitations or messages.
- A treated creator posts more, and control followers consume more.
- A treated group admin changes group activity, affecting control members.

The shared resource is the social graph. The treatment changes what flows through that graph.

**Limited Supply**

Treatment users may consume scarce supply, making less of it available to control users.

Examples:

- Treated Airbnb guests book listings, reducing listing availability for control guests.
- Treated ride-sharing users request more rides, occupying nearby drivers and increasing wait time for control riders.
- Treated food-delivery users place more orders, using courier and restaurant capacity.
- Treated e-commerce buyers purchase limited inventory, causing stockouts for control buyers.

The shared resource is supply: listings, drivers, couriers, restaurants, inventory, or service capacity.

Limited supply can also appear as congestion. The scarce resource may not be a specific item, but system capacity: restaurant preparation time, courier availability, support agents, or compute resources. In that case, treatment increases total load and changes the experience for both treatment and control users.

**Limited Demand or Attention**

Treatment supply-side units may attract scarce demand or attention away from control supply-side units.

Examples:

- Treated Airbnb listings receive more bookings, leaving fewer bookings for control listings.
- Treated sellers receive more impressions or orders, reducing demand for control sellers.
- Treated creators receive more feed exposure, pushing control creators lower.
- Treated ads win more ad slots, reducing impressions for control ads.

The shared resource is demand or attention: buyers, impressions, feed positions, bookings, or ad slots.

Ranking, search, recommendation, auction, and dispatch systems often transmit this kind of interference. They are not always separate root causes. They are platform mechanisms for allocating scarce demand, supply, or attention.

**Pricing and Market Equilibrium**

Treatment may change prices, bids, or incentives, which then affects other units.

Examples:

- A rider promotion increases demand and triggers surge pricing for control riders.
- An advertiser bidding tool raises auction prices for control advertisers.
- Seller subsidies change market prices and reduce control sellers' competitiveness.
- A dynamic-pricing policy changes buyer expectations across the market.

The shared object is the market equilibrium: prices, bids, incentives, and supply-demand balance.

**Learning and Ecosystem Feedback**

Treatment can change future behavior in ways that affect the environment for everyone.

Examples:

- Treated sellers learn better pricing strategies and compete differently later.
- Treated creators receive more traffic and produce more content in the future.
- Treated drivers learn incentive patterns and change where or when they work.
- Treated users learn about a feature and tell control users outside the product.

These effects may be delayed. They are especially important for long-running experiments and ecosystem changes.

**Cross-Device or Cross-Account Contamination**

Sometimes the same real-world person or organization appears in both treatment and control.

Examples:

- A user is assigned to treatment on mobile but control on desktop.
- Family members share an account or device.
- A seller operates multiple accounts assigned to different arms.
- A company account has multiple employees seeing different variants.

This is often called contamination rather than interference, but the practical problem is similar: treatment and control are no longer cleanly separated.

In all of these cases, the treatment effect is not confined to treated units. The right design depends on the channel: social exposure points toward network-aware designs, limited local supply points toward geo or switchback designs, and two-sided marketplace competition may require randomizing both sides of the market.

## A First Spillover Example

For example, suppose a food delivery app tests showing delivery time as a range instead of a point estimate:

- Control users see "30 minutes"
- Treatment users see "25-35 minutes"

At first, this looks like a simple UI experiment. But if the treatment changes order volume or cancellation behavior, it may affect shared operational capacity.

Treatment users may:

- Place more or fewer orders
- Cancel less often
- Contact support less often
- Change when they order

Those behaviors can affect:

- Restaurant workload
- Courier availability
- Delivery time
- Support queue length
- Experience for control users in the same city and time period

The result is spillover. Control users are no longer experiencing the world that would have existed without treatment. This example will return later because it also motivates saturation analysis, geo holdouts, and switchback experiments.

## Diagnosing Spillover

Detecting spillover is hard because the counterfactual is missing. In a user-level experiment, the control group may already be contaminated by treatment users.

A common first idea is:

> Run a user-level experiment and check whether metrics in the control group significantly decrease.

This can be a useful smoke test, but it is not proof.

Control metrics can move for many reasons:

- Weather
- Holidays
- Marketing campaigns
- Supply shocks
- Logging changes
- Seasonality
- Product outages

Also, if every market has the same treatment share, there may be no untreated market to use as a benchmark. For example, in a 50/50 user-level experiment run in every city, control users in every city may be affected by nearby treatment users. Without a fully untreated market, it is hard to know what the control outcome would have been without spillover.

Better diagnostics compare control outcomes across different levels of treatment exposure. If treatment exposure is only observed, the analysis is mainly diagnostic. If the experiment deliberately randomizes exposure levels, the same idea can estimate spillover more credibly.

### Treatment Saturation Analysis

If spillover exists, control users in high-treatment environments should be affected more than control users in low-treatment environments.

For food delivery, the analysis might define treatment saturation by city-hour:

$$
\text{Treatment Saturation}_{g,t}
=
\frac{\text{Treatment users in city }g\text{ at hour }t}
{\text{All experiment users in city }g\text{ at hour }t}
$$

Then the team can ask:

> Do delivery time, cancellation rate, or complaint rate change with treatment saturation? And do control users change differently from treatment users?

A diagnostic regression can include both the user's own treatment status and the saturation of the surrounding environment:

$$
Y_{i,g,t}
=
\alpha
+ \beta T_i
+ \gamma S_{g,t}
+ \delta (T_i \times S_{g,t})
+ \epsilon_{i,g,t}
$$

where:

- $Y_{i,g,t}$ is the outcome for user $i$ in market $g$ and time $t$
- $T_i$ indicates whether user $i$ is treated
- $S_{g,t}$ is treatment saturation in market $g$ and time $t$
- $\gamma$ captures whether control users' outcomes change with nearby treatment intensity
- $\delta$ captures whether the treatment effect changes as saturation changes

For diagnosing spillover, $\gamma$ is especially important. If $\gamma$ is meaningfully different from zero, control users' outcomes vary with nearby treatment intensity. That suggests spillover.

This is still diagnostic, not definitive. High-saturation city-hours may differ from low-saturation city-hours in other ways. But it is stronger than simply checking whether the overall control group moved.

Treatment saturation analysis and two-stage randomization are closely related. Treatment saturation is the quantity being studied: how treated is the local environment? Two-stage randomization is a design that deliberately creates randomized variation in that quantity.

### Two-Stage Randomization

Two-stage randomization is a design built to estimate both direct and spillover effects.

In the first stage, clusters are randomized to different treatment saturation levels:

- Some markets: 0% treatment
- Some markets: 25% treatment
- Some markets: 50% treatment
- Some markets: 75% treatment
- Some markets: 100% treatment

In the second stage, users within each market are randomized according to the assigned saturation level.

This design uses the same saturation logic as above, but with a stronger interpretation because saturation is assigned by design rather than only observed. It lets the team estimate:

- The direct effect of being treated
- The spillover effect of being surrounded by more treated users
- How the total effect changes as treatment scales

For example, if control users have worse delivery times in 75% treatment markets than in 25% treatment markets, the platform has stronger evidence of congestion spillover.

The tradeoff is that two-stage randomization requires enough clusters and careful operational control.

This design is closely related to the framework in Hudgens and Halloran (2008) and the review by Tchetgen Tchetgen and VanderWeele (2012).

### Geo Holdouts

A stronger design is to keep some clusters fully untreated.

For example, a food delivery platform might divide markets into:

- Fully untreated holdout cities
- Experiment cities with user-level treatment/control randomization

This allows two useful comparisons:

1. Treatment users versus control users inside experiment cities
2. Control users inside experiment cities versus users in fully untreated holdout cities

If control users inside experiment cities differ from users in fully untreated holdout cities, that is evidence that the experiment environment itself changed.

Geo holdouts are useful when spillovers are mostly contained within geographic markets. They are less useful if users or supply frequently cross geographic boundaries.

The main challenge is power. Even if each geo contains many users or orders, the effective sample size is closer to the number of randomized geos than the number of individual observations, because outcomes within the same geo are correlated and share the same assignment. Therefore, geo experiments require careful design, matching, blocking, or longer duration.

### Operational Mediators

When spillover is plausible, it helps to track operational mediators.

Mediators are metrics that explain how treatment could affect other units. They do not prove interference by themselves, but they make the mechanism visible.

For food delivery:

- Courier utilization
- Restaurant preparation delay
- Delivery time
- Order backlog
- Cancellation queue
- Support volume by city-hour

For ride-sharing:

- Driver utilization
- Rider wait time
- Surge price frequency
- Driver acceptance rate
- Ride cancellation rate

For ads:

- Auction density
- Average bid
- Cost per impression
- Win rate
- Budget exhaustion

For creator platforms:

- Exposure concentration
- Creator traffic distribution
- Creator posting frequency
- Content supply
- Cold-start creator impressions

If treatment changes these shared mediators, spillover becomes more plausible. They also help identify the right randomization block. For example, if courier utilization moves at the city-hour level, then city-hour may be a better experimental unit than individual users.

## Block-Based Designs

Once spillover is likely, the experiment design may need to change. Instead of randomizing individual users, block-based designs randomize larger blocks. All units inside the same block receive the same assignment, and interference between blocks is expected to be much less likely.

Sometimes blocks are obvious. An education policy study may randomize schools because students within the same school interact with each other, while different schools are less likely to interfere with one another. A food delivery experiment may randomize cities because users, restaurants, and couriers mostly interact within the same local market. Randomization by geographic units is especially common in marketplaces, logistics, advertising, and local services. These designs are often called geo experiments.

Sometimes blocks are not obvious and must be built from a network. In a social platform, users are connected through friendships, follows, messages, groups, and feed exposure. A graph-cluster experiment first partitions the network into communities, then randomizes those communities. The goal is not to remove every cross-cluster edge, which is usually impossible, but to reduce the amount of spillover between treatment and control.

Network experiment design is a large topic by itself. For readers who want a deeper entry point, Zheleva and Arbour (2021) and Fatemi and Zheleva (2023) discuss graph-based designs in more detail.

### Randomization Unit Versus Observation Unit

In block-based designs, the randomization unit is the block, not the individual user.

If a geo experiment randomizes cities, the randomization unit is the city. If a school experiment randomizes schools, the randomization unit is the school. If a seller experiment randomizes seller groups, the randomization unit is the seller group. All users, sessions, orders, or item views inside the same block inherit the block's assignment.

The observation unit can still be much smaller:

- User
- Session
- Order
- Listing view
- Delivery
- Ride request

For example, in a city-level geo experiment:

$$
Y_{i,g} = \alpha + \tau T_g + \epsilon_{i,g}
$$

where $i$ indexes users or orders, $g$ indexes cities, and $T_g$ is the city-level treatment assignment.

The raw dataset may contain millions of user or order rows. But treatment variation comes from the randomized blocks. The analysis must respect that assignment level, often by clustering standard errors, aggregating outcomes to the randomized block, or using randomization inference.

This is why people often say block-based designs "reduce sample size." More precisely, they reduce the effective sample size, not necessarily the raw number of observations.

If a geo experiment observes one million orders across 100 randomized cities, those orders are still in the dataset. But the experiment has 100 independent treatment assignments, not one million. Orders inside the same city share the same assignment and may also share weather, demand, supply, local competitors, and other market shocks.

A common design-effect approximation is:

$$
\text{Design Effect} = 1 + (m - 1)\rho
$$

where:

- $n$ is the total number of observations
- $G$ is the number of randomized blocks
- $m$ is the average number of observations per randomized block
- $\rho$ is the intra-cluster correlation

When blocks are roughly the same size, $n \approx Gm$.

The effective sample size is roughly:

$$
n_{\text{eff}} =
\frac{n}{1 + (m - 1)\rho}
$$

If $\rho = 0$, clustering does little harm. But if outcomes inside the same block are correlated, the effective sample size can be much smaller than the raw observation count. In the extreme case where $\rho = 1$, all observations inside a block move together, so the effective sample size becomes $n/m = G$, the number of randomized blocks.

The key distinction is:

- Randomization unit: where treatment is assigned
- Observation unit: where outcomes are measured
- Inference unit: the level that determines independent treatment variation

### Inference in Block-Based Designs

Block-based designs require special care during inference because the raw data can be misleadingly large.

The most important rule is:

> Analyze uncertainty at the level where treatment was randomized.

This does not mean every analysis must collapse the dataset to one row per block. Individual orders, users, or sessions can still be used as observations. But the standard errors, confidence intervals, and hypothesis tests must respect the block-level assignment.

**Individual-level regression with clustered standard errors**

One approach keeps the lower-level observations as rows:

$$
Y_{i,g}
=
\alpha
+ \tau T_g
+ \epsilon_{i,g}
$$

Here, $i$ might be a user, order, session, or item view. The treatment $T_g$ varies at the block level. This regression usually estimates a traffic-weighted effect, because blocks with more observations contribute more rows.

The standard errors should be clustered at the randomization level. For a geo experiment randomized by city, cluster at the city level. For a seller-group experiment, cluster at the seller-group level. If outcomes are correlated across multiple dimensions, the analysis may need two-way clustering, time-series-aware standard errors, or randomization-based inference.

Because the analysis respects block-level assignment, adjusted standard errors are often much larger than naive user-level standard errors. Variance reduction methods from Chapter 6 can help. For example, the analysis may add pre-period outcomes, block fixed effects, time fixed effects, matched-pair indicators, or baseline demand and supply measures. These controls should improve precision, not redefine who is considered treated.

When the number of randomized blocks is small, ordinary regression standard errors can still be fragile. Randomization inference, matched-pair analysis, wild cluster bootstrap, or conservative reporting may be more appropriate.

**Block-level regression**

Another approach is to compute one outcome per randomized block, then compare treated and control blocks.

For example, in a city-level geo experiment, compute:

$$
\bar Y_g
=
\text{average outcome in city }g
$$

Then estimate the treatment effect using the city as the analysis row:

$$
\bar Y_g
=
\alpha
+ \tau T_g
+ \epsilon_g
$$

This approach is easy to explain and naturally respects the assignment level. But it works best when there are enough randomized blocks and when block sizes are not extremely imbalanced. Sometimes the weight of each block should be carefully chosen. If blocks have very different denominators, especially for ratio metrics such as conversion rate or cancellation rate, equal-weighted block averages can answer a different question from the user-level or traffic-weighted effect.

Another issue of block-level regression is power. Aggregating to the block level reduces the number of analysis rows to the number of blocks. Sneider and Tang (2019) propose variance reduction with covariates and multilevel modeling as ways to recover power while still respecting the nested experiment structure.

## Switchback Experiments

Sometimes comparison across geo blocks or graph blocks is not feasible. For example, the platform may have only a few geo units, or assigning different treatments to different markets may raise fairness or discrimination concerns. In these cases, comparing time windows may help. A switchback experiment (Bojinov et al. 2023) is a block-based design where the block is time, or market-time. Instead of assigning individual users to treatment and control at the same time, the system switches between treatment and control across time windows.

Consider a food delivery platform that wants to test a treatment in a specific city. The platform may divide each day into time windows, such as 24 one-hour blocks, and randomly assign each city-hour block to treatment or control. During a treated hour, all orders, deliveries, couriers, restaurants, and users in that city experience treatment. During a control hour, the same city runs under control.

This creates new design challenges: how long should each time window be, how often should the system switch, and whether treatment in one window continues to affect the next window, a problem called carryover.

### Switching Frequency, Carryover, and Washout

The central design choice in a switchback experiment is the length of the time window.

Shorter windows create more randomized blocks, which usually improves power and lets the experiment finish sooner. But very short windows can be misleading if the marketplace does not reset quickly. Longer windows reduce this risk, but they create fewer randomized blocks and make the experiment slower.

The window should be long enough for two things to happen:

- The treatment has enough time to affect the system.
- The system has enough time to return to a clean state before the next assignment.

This second issue is called carryover. Carryover happens when treatment in one time block affects outcomes in a later time block. For example, a courier incentive during lunch may change courier supply after lunch. A dispatch algorithm may change the location of couriers and open orders near the end of a treatment window. A pricing change may affect customer demand, inventory, recommendations, or future search behavior.

In practice, the window length and washout period should be chosen using both domain knowledge and pre-experiment operational data. The team should ask how long the treatment takes to affect the system, and how long key state variables take to return to normal after a shock. In a delivery marketplace, useful variables might include open orders, courier supply, courier location, restaurant queue length, delivery time, cancellation rate, and the demand-supply ratio. If these variables remain highly correlated across adjacent 15-minute windows, then a 15-minute switchback is probably too short.

A practical workflow is to compare several candidate window lengths, such as 15, 30, 60, or 120 minutes. For each candidate, check whether the outcome and key operational variables are still strongly autocorrelated across adjacent windows, whether the treatment effect is expected to decay before the next assignment, and whether enough randomized blocks remain for the experiment to have reasonable power. If the system needs time to settle after each switch, the design can add a washout period and analyze only the stable part of each window.

One simple diagnostic is a lag regression. For a candidate window length, aggregate historical data to the market-time level and estimate:

$$
Y_{g,t}
=
\alpha
+
\rho Y_{g,t-1}
+
\epsilon_{g,t}
$$

Here, $Y_{g,t}$ is the outcome or operational variable in market $g$ during time window $t$. If $\rho$ is large, adjacent windows are strongly correlated. That means the system state persists across windows, so the candidate window length may still have carryover or serial dependence.

Bojinov et al. (2023) formalize this problem by modeling the order of carryover, meaning how many previous time periods can still affect the current outcome. Given assumptions about the carryover order, they study how to design switchback assignment schedules and conduct inference.

Kastelman and Ramesh (2018) describe this tradeoff in DoorDash dispatch experiments. Very small region-time buckets can create bias because couriers, pending orders, restaurant load, pricing, and supply-demand balance may carry over from one bucket to the next. Very large buckets create the opposite problem: they increase the margin of error and slow learning. DoorDash reports using region-time units, often around city-level regions and 30-minute switchback periods, for many dispatch experiments.

When carryover is plausible, the design can respond in several ways:

- Use longer windows.
- Add washout or blackout periods after switching.
- Exclude the early part of each window from analysis.
- Analyze only periods when the system is expected to be stable.
- Choose treatments whose effects decay quickly enough for switchback to be credible.

The analysis must also account for serial correlation across nearby time blocks. Adjacent time blocks are usually correlated because weather, traffic, local events, restaurant load, courier supply, and demand imbalance persist over time. This affects the choice of switching frequency, washout periods, and standard errors.

Cooprider and Nassiri (2023) provide an example from Amazon's pricing experiments. In pricing, the same product usually should not show different prices to different customers at the same time, which makes user-level randomization difficult. A switchback or crossover design can randomize the same product across time. But if a treatment increases a product's sales, recommendation systems may continue promoting that product during later control periods. They discuss using blackout periods to let carryover effects decay before analyzing the next phase.

After choosing the window length, the next question is how to assign treatment across windows. Many switchback designs use balanced treatment-control assignment across time blocks. The practical message is:

> Unless there is a strong operational reason not to, make treatment and control appear equally often across comparable time periods.

Random assignment helps balance treatment and control across time patterns in expectation, but one realized schedule can still be imbalanced when the number of windows is limited. A good switchback design often uses blocked or stratified randomization so treatment and control both appear across lunch and dinner, weekdays and weekends, high-demand and low-demand hours, holidays and ordinary days. If treatment happens mostly during lunch and control mostly during dinner, the experiment confounds treatment with time of day.

Finally, switchback experiments are vulnerable to other changes happening at the same time, such as promotions, weather shocks, holidays, app outages, competing product launches, and other experiments running on the same market. Xiong, Chin, and Taylor (2024) emphasize that estimation error depends on carryover effects, periodicity, serial correlation, and simultaneous experiments. Their ride-sharing application uses prior data to design future switchback schedules and reports lower mean squared error compared with the platform's previous design.

In short, a switchback experiment is not just "turn treatment on and off." It is a design problem involving window length, carryover, treatment balance, serial correlation, and operational feasibility.

## Two-Sided Randomization

Block-based designs try to keep strongly interacting units together: a city moves together, a school moves together, a market-time block moves together. This works well when interference can be mostly contained inside a block.

Some platforms have a different structure. The main interaction is not only among nearby users, but between two sides of a market.

If the supply side is limited, user-level randomization can create interference because treated and control users compete for the same scarce listings, drivers, inventory, or ad slots. If the demand side is limited, listing-level or seller-level randomization can also create interference because treated and control listings compete for the same users, buyers, or viewers. If both sides are limited, interference can travel in both directions.

Two-sided randomization (Johari et al. 2022) is designed for platforms where treatment can affect how the two sides of a marketplace interact.

Consider an Airbnb-like lodging marketplace. There are guests on the demand side and listings on the supply side. Guests search for available listings, decide whether to book, and once a listing is booked, it becomes unavailable to other guests for that time period.

If an experiment randomizes only guests, treated guests and control guests still compete for the same listings. If it randomizes only listings, treated listings and control listings still compete for the same guests. Two-sided randomization instead randomizes both sides of the market.

In this design:

- Guests are assigned to control or treatment
- Listings are assigned to control or treatment
- Each guest-listing interaction falls into one of four cells

![Two-sided randomization creates four customer-listing interaction cells.](assets/two-sided-randomization.png)

The first index refers to the guest side. The second index refers to the listing side.

- $Q_{00}$: control guests interacting with control listings
- $Q_{01}$: control guests interacting with treatment listings
- $Q_{10}$: treatment guests interacting with control listings
- $Q_{11}$: treatment guests interacting with treatment listings

The fully treated interaction is $Q_{11}$, where a treated guest interacts with a treated listing. The fully control interaction is $Q_{00}$, where a control guest interacts with a control listing.

The value of this design is not just that it creates four cells. The value is that the mixed cells reveal displacement.

Suppose the treatment is a discount shown only when a treated guest views a treated listing. The outcome is bookings.

$Q_{11}$ contains the fully treated interactions. Treated guests see discounted treated listings, so bookings in this cell may increase.

$Q_{01}$ contains control guests and treated listings. These guests do not see the discount, but the treated listings may become less available because treated guests in $Q_{11}$ booked them. If bookings in $Q_{01}$ fall relative to $Q_{00}$, that suggests treated guests are displacing control guests from treated listings.

$Q_{10}$ contains treated guests and control listings. These guests may shift their attention toward discounted treated listings, leaving control listings with fewer bookings. If bookings in $Q_{10}$ fall relative to $Q_{00}$, that suggests treated listings are pulling demand away from control listings.

This is the core intuition:

> $Q_{11}$ shows the treated match. $Q_{01}$ and $Q_{10}$ show where the bookings may have been displaced from.

Without the mixed cells, the platform may see that treated guests book treated listings more often, but it cannot tell whether those bookings are truly incremental or mostly reallocated from other guests and listings.

In practice, the $Q$ cells should usually be normalized before comparison, because the four cells may contain different numbers of guest-listing opportunities. If $a_C$ is the treatment share of guests and $a_L$ is the treatment share of listings, then the normalized booking rates are conceptually:

$$
r_{00} = \frac{Q_{00}}{(1-a_C)(1-a_L)},\quad
r_{01} = \frac{Q_{01}}{(1-a_C)a_L}
$$

$$
r_{10} = \frac{Q_{10}}{a_C(1-a_L)},\quad
r_{11} = \frac{Q_{11}}{a_Ca_L}
$$

These normalized rates make the mixed-cell comparisons more meaningful:

- $r_{11} - r_{00}$ shows how the fully treated cell differs from the fully control cell.
- $r_{01} - r_{00}$ helps reveal what happens to treated listings when they are seen by control guests.
- $r_{10} - r_{00}$ helps reveal what happens to control listings when they are seen by treated guests.

The exact estimator depends on the treatment, market balance, and the assumptions the platform is willing to make. The important lesson is that two-sided randomization gives the platform information about both demand-side and supply-side competition, which one-sided randomization cannot fully observe.

Masoero et al. (2026) study this idea under the name Multiple Randomization Designs. In their framework, treatment assignment and outcomes are defined at the level of a tuple, such as a buyer-seller pair. A simple version randomizes buyers and sellers separately, then studies the four resulting pair types. This provides a formal way to estimate main effects, direct effects, and spillovers in marketplaces where both sides interact.

Two-sided randomization can be especially helpful when the marketplace is highly connected and natural clusters are hard to define. However, the estimator still needs to be chosen carefully, and the variance can be higher than in a simple user-level experiment. For a formal analysis of demand-side, supply-side, and two-sided randomization in booking marketplaces, see Johari et al. (2022).

## Matching Design to Interference Structure

There is no universal best design. The design should match the interference structure.

The central question is:

> Who or what shares the resource affected by treatment?

That shared resource often defines the randomization unit. If users do not share the resource affected by treatment, a user-level A/B test may be enough. If users share local supply, a geo experiment or switchback may be better. If users are connected through a social graph, graph-cluster randomization or exposure modeling may be needed. If two sides of a marketplace compete through matching, the experiment may need to randomize the demand side, the supply side, or both sides, depending on where the treatment acts and where competition occurs.

## Partial Rollout and Full Rollout

So far, we have mainly discussed interference from treatment users to control users: treated units can change the environment faced by untreated units. But treated units can also affect other treated units.

Most experiments test a treatment at partial saturation. For example, 10%, 20%, or 50% of users may receive treatment. A full rollout changes the environment more broadly.

For example, if a food delivery treatment increases demand, treated users may create congestion for one another. At 10% treatment saturation, the system may absorb the extra demand. At 100% rollout, courier utilization, restaurant load, delivery time, and cancellation rate may change much more.

A user-level treatment-control comparison may estimate the effect of treatment in a partially treated system. It may not fully estimate the effect of full rollout.

The measured effect can depend on treatment saturation.

## Product Example: Ranking a Shared Seller Pool

Suppose an e-commerce platform tests a new recommendation ranking model that gives more exposure to sellers with high predicted repeat-purchase value.

If users are randomized, treatment users see the new ranking and control users see the old ranking.

But sellers are shared across both groups. If treatment shifts attention toward a smaller set of high-value sellers, it may affect:

- Inventory availability
- Seller response time
- Pricing
- Seller traffic concentration
- Future content or product supply
- Control users' recommendation quality

The experiment is not only changing treatment users' rankings. It may be changing the seller ecosystem.

Useful guardrails and diagnostics include:

- Seller exposure concentration
- Active seller coverage
- Cold-start seller impressions
- Inventory stockout rate
- Refund and return rate
- Control-group changes by seller exposure pressure

If the treatment is expected to substantially reallocate traffic, seller-level or category-level randomization may be more appropriate than user-level randomization. Another option is a long-term holdout where a subset of markets or traffic remains under the old ranking system to measure ecosystem effects.

## Practical Workflow

A practical workflow for interference-aware experiments is:

1. Identify the shared resource, market, network, or platform mechanism.
2. Map how treatment could affect units outside the treated group.
3. If using user-level randomization, predefine spillover diagnostics such as saturation analysis and operational mediators.
4. If spillover is likely to change the launch estimate, choose a design that contains or accounts for it: clusters, graph clusters, geo experiments, switchbacks, two-stage randomization, or two-sided randomization.
5. Analyze uncertainty at the level where treatment was randomized.
6. Report what the estimate represents: direct effect, spillover effect, total effect, effect at a specific saturation level, or a mixture.

## Common Mistakes

**Assuming user-level randomization always works**

User-level randomization is powerful, but it can fail when users share supply, inventory, auctions, attention, or social connections.

**Treating control degradation as proof of spillover**

Control degradation is a smoke test, not proof. External shocks can also move control metrics.

**Ignoring treatment saturation**

The effect at 10% rollout may differ from the effect at 100% rollout if the treatment changes shared resources.

**Trusting raw observations when blocks are few**

Cluster, geo, and switchback experiments need enough independent randomized blocks. Millions of users or orders do not compensate for having too few cities, clusters, or market-time windows.

**Ignoring carryover in switchback tests**

If treatment effects persist into later time blocks, the next control period may be contaminated.

## Key Takeaways

- Interference occurs when one unit's treatment affects another unit's outcome.
- Spillover means untreated or control units are indirectly affected by treatment.
- Common interference channels include social exposure, limited supply, limited demand or attention, pricing equilibrium, learning feedback, and contamination.
- Checking whether control metrics decrease is only a smoke test. Stronger diagnostics compare outcomes across treatment saturation levels or against untreated holdouts.
- Cluster, graph-cluster, geo, switchback, two-stage, and two-sided designs help when user-level randomization no longer matches the interaction structure.
- In block-based experiments, users may still be the observation unit, but the randomized block determines the independent treatment variation.
- The effect measured in a partial rollout may differ from the effect under full rollout.
- The randomization unit should match the level at which treatment changes shared resources.

## References

- Aronow, Peter M., and Cyrus Samii. "Estimating Average Causal Effects Under General Interference, with Application to a Social Network Experiment." *The Annals of Applied Statistics* 11, no. 4 (2017): 1912-1947. [https://doi.org/10.1214/16-AOAS1005](https://doi.org/10.1214/16-AOAS1005).
- Bojinov, Iavor, David Simchi-Levi, and Jinglong Zhao. "Design and Analysis of Switchback Experiments." *Management Science* 69, no. 7 (2023): 3759-3777. [https://doi.org/10.1287/mnsc.2022.4583](https://doi.org/10.1287/mnsc.2022.4583).
- Bowers, Jake, Mark M. Fredrickson, and Costas Panagopoulos. "Reasoning about Interference Between Units: A General Framework." *Political Analysis* 21, no. 1 (2013): 97-124. [https://doi.org/10.1093/pan/mps038](https://doi.org/10.1093/pan/mps038).
- Cooprider, Joe, and Shima Nassiri. "The Science of Price Experiments in the Amazon Store." Amazon Science, April 14, 2023. [https://www.amazon.science/blog/the-science-of-price-experiments-in-the-amazon-store](https://www.amazon.science/blog/the-science-of-price-experiments-in-the-amazon-store).
- Fatemi, Zahra, and Elena Zheleva. "Network Experiment Designs for Inferring Causal Effects Under Interference." *Frontiers in Big Data* 6 (2023): 1128649. [https://doi.org/10.3389/fdata.2023.1128649](https://doi.org/10.3389/fdata.2023.1128649).
- Hudgens, Michael G., and M. Elizabeth Halloran. "Toward Causal Inference With Interference." *Journal of the American Statistical Association* 103, no. 482 (2008): 832-842. [https://doi.org/10.1198/016214508000000292](https://doi.org/10.1198/016214508000000292).
- Johari, Ramesh, Hannah Li, Inessa Liskovich, and Gabriel Y. Weintraub. "Experimental Design in Two-Sided Platforms: An Analysis of Bias." *Management Science* 68, no. 10 (2022): 7069-7089. [https://doi.org/10.1287/mnsc.2021.4247](https://doi.org/10.1287/mnsc.2021.4247).
- Kastelman, David, and Raghav Ramesh. "Switchback Tests and Randomized Experimentation Under Network Effects at DoorDash." DoorDash Engineering Blog, February 13, 2018. [https://careersatdoordash.com/blog/switchback-tests-and-randomized-experimentation-under-network-effects-at-doordash/](https://careersatdoordash.com/blog/switchback-tests-and-randomized-experimentation-under-network-effects-at-doordash/).
- Masoero, Lorenzo, Suhas Vijaykumar, Thomas S. Richardson, James McQueen, Ido Rosen, Brian Burdick, Patrick Bajari, and Guido W. Imbens. "Multiple Randomization Designs: Estimation and Inference with Interference." *Journal of the Royal Statistical Society Series B: Statistical Methodology* 88, no. 3 (2026): 958-977. [https://doi.org/10.1093/jrsssb/qkaf073](https://doi.org/10.1093/jrsssb/qkaf073).
- Sneider, Carla, and Yixin Tang. "Experiment Rigor for Switchback Experiment Analysis." DoorDash Engineering Blog, February 20, 2019. [https://careersatdoordash.com/blog/experiment-rigor-for-switchback-experiment-analysis/](https://careersatdoordash.com/blog/experiment-rigor-for-switchback-experiment-analysis/).
- Tchetgen Tchetgen, Eric J., and Tyler J. VanderWeele. "On Causal Inference in the Presence of Interference." *Statistical Methods in Medical Research* 21, no. 1 (2012): 55-75. [https://doi.org/10.1177/0962280210386779](https://doi.org/10.1177/0962280210386779).
- Xiong, Ruoxuan, Alex Chin, and Sean J. Taylor. "Data-Driven Switchback Experiments: Theoretical Tradeoffs and Empirical Bayes Designs." SSRN working paper, 2024. [https://doi.org/10.2139/ssrn.4626245](https://doi.org/10.2139/ssrn.4626245).
- Zheleva, Elena, and David Arbour. "Causal Inference from Network Data." In *Proceedings of the 27th ACM SIGKDD Conference on Knowledge Discovery & Data Mining*, 4096-4097. 2021. [https://doi.org/10.1145/3447548.3470795](https://doi.org/10.1145/3447548.3470795).
