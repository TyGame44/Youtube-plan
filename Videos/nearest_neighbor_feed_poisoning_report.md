# Nearest-Neighbor Feed Poisoning and Algorithmic Resistance

## 1. Summary

**Nearest-neighbor feed poisoning** is a recommender-system attack where a bot network mirrors the viewing behavior of a target person or group, then consumes or engages with payload content the attacker wants delivered. The goal is to make the platform infer that “people who watched what the target watched also liked this,” causing the target’s feed to surface attacker-selected content.

The danger is that the attack can be simple. It does not require hacking the platform, knowing the model internals, or persuading the target directly. It only requires creating artificial behavioral neighbors around the target.

## 2. Attack Model

The simplified attack flow is:

1. Observe the target’s visible content environment: posts they watch, reply to, like, quote, bookmark, or repeatedly encounter.
2. Use a bot or sockpuppet network to watch the same posts.
3. Have that same account network watch or engage with payload posts.
4. The recommender detects a behavioral association between the target’s content cluster and the payload content.
5. The payload becomes more likely to appear in the target’s feed.

This can be described as **co-view poisoning**, **taste-neighborhood hijacking**, or **nearest-neighbor feed poisoning**.

The core vulnerability is that recommender systems often treat behavioral similarity as evidence of shared interest. But in this attack, behavioral similarity is manufactured.

## 3. Why This Is Dangerous

This attack is more dangerous than ordinary bot amplification because it can be **targeted without being obvious**.

A botnet boosting a post globally is visible. A botnet shaping the content neighborhood around one artist, journalist, teenager, activist, or niche community may be much harder to notice.

Likely high-risk targets include:

- artists and creators whose emotional state is affected by feedback;
- people in ideological, health, fandom, or identity-based communities;
- small groups whose feed environment can be shifted with fewer accounts;
- public figures who monitor their own niche constantly;
- users vulnerable to harassment, flattery, obsession, paranoia, or demoralization.

This overlaps with coordinated inauthentic behavior, where networks of accounts act together deceptively to manipulate perception or platform systems.

## 4. The Larger Solution: Algorithmic Resistance

The larger solution is to make feeds **resistant to private behavioral capture**.

The principle:

> Personalization should filter from a trustworthy distribution layer, not create a private reality from raw behavioral correlations.

A platform should not allow a narrow, suspicious cluster of accounts to make content eligible for a target simply because those accounts mirrored the target’s behavior.

## 5. Global Daily Ranking / Eligibility Gate

A **global daily ranking system** is a strong structural defense.

The idea is:

> Before a post can be recommended to a targeted person or group, it must pass a broader ranking or eligibility layer.

This changes the attacker’s job. Instead of only poisoning the target’s nearest-neighbor graph, the attacker must make the payload content pass a broader legitimacy threshold.

A better name than “source of truth” might be:

**Global recommendation eligibility layer**

This avoids implying that the platform decides truth. The layer does not need to say, “This post is true.” It only needs to say:

> This post has enough organic, non-coordinated, reputation-backed legitimacy to be algorithmically distributed beyond its direct follower graph.

## 6. Recommended Architecture

A robust ranking system could use several tiers:

### Follower Delivery

Followers can see posts directly, especially in chronological feeds.

### Local / Community Eligibility

A post can be recommended within a relevant niche if it receives organic approval from trusted accounts in that niche.

### Global Daily Eligibility

A post can be broadly recommended only after passing a slower, more auditable ranking process.

### Personalization

The user’s feed can then reorder eligible posts based on interests, language, format preference, and explicit settings.

This would reduce the power of bot-created taste neighborhoods. The botnet could still watch the same content as the target, but the payload would not automatically become recommendable unless it passed the eligibility gate.

## 7. Why the Daily Cadence Matters

A daily ranking slows the attack loop.

Many manipulation strategies depend on rapid iteration:

> observe target → mirror behavior → inject payload → measure response → adjust payload.

A daily eligibility layer adds friction. It gives the platform time to detect coordination before payload content spreads. It also makes manipulation more visible because attackers must influence a broader ranking layer rather than a hidden personalized recommendation path.

## 8. Risks of the Global Ranking Approach

The main risk is over-centralization.

If the global ranking is too dominant, it may suppress:

- new creators;
- niche art;
- minority viewpoints;
- experimental content;
- small communities;
- posts that are valuable locally but not globally popular.

The solution is not one universal ranking for all content. It should be a **tiered eligibility system** with local, niche, follower-based, and global pathways.

The goal is not to make everyone see the same feed. The goal is to prevent suspicious micro-clusters from privately steering a person’s recommendations.

## 9. Defenses Platforms Can Implement Now

Current recommender systems can reduce this risk without a full redesign.

### A. Downweight Passive Co-Views

Passive views should carry less recommendation influence than strong, trusted actions. A brief view from a new account should not shape another user’s feed much.

Higher-value signals should include long-term account history, meaningful follows, saves, replies, and repeated organic behavior.

### B. Detect Synchronized Co-View Trails

Platforms should look for account clusters that repeatedly follow the same pattern:

> target-adjacent posts → payload posts

Suspicious signs include tight timing, repeated overlap, new accounts, low diversity, shared infrastructure, or coordinated engagement.

### C. Limit Influence From New or Low-Trust Accounts

New accounts should not be able to rapidly become influential recommendation neighbors. Their behavior can still count for their own feed, but should have low weight in shaping other people’s feeds.

### D. Separate Similarity From Trust

Behavioral similarity should not automatically imply recommendation trust.

A platform should ask:

> Are these accounts similar because they are genuinely independent users, or because they are coordinated?

High similarity appearing quickly should be treated as suspicious, not automatically useful.

### E. Exposure Caps by Cluster

A user’s feed should not become dominated by one behavioral cluster, topic cluster, or newly emerging content bridge.

This is especially important for vulnerable targets and creators.

### F. User-Side Controls

Users should have controls such as:

- reduce recommendations from similar viewers;
- reset my recommendation neighborhood;
- show only followed accounts;
- why am I seeing this?;
- reduce emotionally intense recommendations;
- do not learn from this session.

These controls would make personalization less brittle.

### G. Creator Protection Mode

Creators should have a mode that detects artificial waves of praise, harassment, imitation, obsession, or demoralizing content. The system should be especially careful when the same suspicious clusters repeatedly interact with a creator’s niche and then push new content into that creator’s feed.

## 10. Conclusion

Nearest-neighbor feed poisoning is dangerous because it is simple. The attacker does not need to understand the full recommender system. They only need to manufacture behavioral neighbors around a target.

A global daily ranking proposal addresses the core flaw by adding an eligibility gate before personalization. The strongest version is:

> Posts should not become algorithmically recommendable to a targeted person or group solely because suspicious accounts mirrored that person’s viewing behavior.

The practical path forward is a combination of:

1. global or community-level recommendation eligibility;
2. slower daily ranking for broader distribution;
3. detection of synchronized co-view behavior;
4. lower influence from new or suspicious accounts;
5. exposure caps and user controls;
6. special protections for creators and vulnerable groups.

This would not eliminate manipulation, but it would make hidden, personalized feed poisoning much harder and more visible.
