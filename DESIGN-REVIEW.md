# iPlay Bet — Daily Voice Sports Betting Assistant
## Product Design Review Prompt

**Brand:** iPlay Bet (matches iPlay platform — "Same Game Different Lanes")
**Repo:** https://github.com/unclesme15-a11y/iplay-bet-design-review
**Status:** Design review — hand to Claude, Codex, GPT, and any other AI for input before build

---

## TITLE: Daily Voice Sports Betting Assistant — Product Design Review

## CONTEXT

We are building a sports betting companion product that competes with GameScript.ai / Dan's AI Sports Picks, but with a fundamentally different experience. Dan's product is a scrollable feed of data sheets, graded props, and Discord community content. Ours is a daily phone call from an AI that knows the user personally, discusses what they're thinking of betting, gives a short personalized rundown, and remembers everything across calls.

GameScript is a platform, not solely owned by Dan. It features multiple cappers (DanGambleAI, ParlayChef, Harry Lock, parlay_bae) plus GameScript's own AI models. Dan is one of the biggest voices but the product includes several. Distribution advantage: large X audience, daily free content funneling to paid, multiple recognizable cappers, established habits, network effects inside the community.

A new phone-based product starts with zero of that. We acquire users ourselves and keep them through the experience. The phone + memory ritual is the differentiator; the picks themselves still have to be decent or people churn.

**Validation signal:** Chi Chi (our Unity/web bot) went 8-for-9 on a FanDuel NBA slate for the user's son using free web search — FanDuel Research pages, Pickswise, CBS, ESPN injury feeds. That accuracy came from parallel free sources agreeing on edges, not one magic source. This is the pattern to systematize.

## CORE PRODUCT LOOP

1. User creates an account: name, phone number, sports they bet, risk tolerance, preferred sportsbooks, preferred call time window (morning, lunch, or evening).
2. Every day at the user's chosen window, an AI calls them. Target five to eight minutes.
3. The AI opens with the user's profile and recent history — past picks discussed, reported outcomes, streaks, preferences.
4. The AI asks what the user has in mind today.
5. The AI gives a tight rundown of two to four relevant picks or prop projections matching the user's sports and style, with conversational reasoning.
6. The AI logs what the user says they're considering or placing.
7. After games conclude, public outcomes are recorded automatically. The next call or an SMS asks the user to confirm whether they actually placed anything and how it went.
8. Optional post-call SMS summary.

**Minute pooling:** Meter as a daily pool of eight minutes, not per-call. First call burns four, they ring back, four left. Keeps cost math honest (still averaging five to eight across the base) and rewards engaged users without punishing early hangups. Hermes tracks minutes used, minutes remaining, and wraps up gracefully before zero.

## MEMORY AND ORCHESTRATION LAYER

A Hermes agent from Nous Research runs on a server as the persistent brain. It owns all user state: profiles, past discussions, reported outcomes, learned preferences. It prepares a context package before each call. It runs background jobs: polling odds for midday line moves, checking public results, nudging users who haven't confirmed outcomes, drafting the next day's brief. It can create and improve its own skills over time. It can also handle follow-ups on Discord, Telegram, or WhatsApp with shared memory. Voice calls still need a separate telephony plus STT/TTS pipeline, but Hermes coordinates all personalized content and state.

**Server:** User already has a Hetzner/BlackBox server running iPlay infrastructure. Hermes lives there alongside existing workloads. The thirty-dollar VPS line is eliminated from the budget.

**Hermes skill creation:** Over time it turns repeated successful patterns ("this user likes low-variance NFL props and ignores longshot lottos") into reusable skills applied automatically. Self-improvement loop refines how it prepares for calls and summarizes user history.

## COMMUNITY LAYER

A Discord server with a free tier for discussion and a paid tier or role for deeper access. Morning channel posts the day's picks with reasoning. Users share results. This recreates the social proof and network effects GameScript has without depending on X.

## MIDDAY SMS HOOK

Hermes polls the odds feed every few minutes. Any line that moves more than half a point triggers a text like "Chiefs just went from minus three to minus three and a half." This is a retention and habit feature — people start checking their phone for your texts, another habit loop on top of the call habit. At fifty users it costs roughly two dollars a month in SMS.

## PICK GENERATION

### Free sources (start here, no paid odds API required)
- PropLine free tier: 1,000 requests/day for props and graded results
- TheRundown free tier: five-minute delayed game lines from three books (DK/FD/BetMGM), no player props on free
- The Odds API free tier: 500 credits/month
- Public ESPN injury feeds (no key, no rate card — scores, schedules, standings, injuries; this is where Chi Chi pulled Kawhi, Ant, Ja, Reaves/Kessler)
- nflverse and BALLDONTLIE free stats tiers (recent performance, usage, matchup context)
- Public expert lean roundups via web search (FanDuel Research, Pickswise, CBS — same sources Chi Chi used for the 8-for-9 slate)

### Simple daily rule (v1 algorithm)
Flag any prop where the player's last five average beats the line by more than fifteen percent AND the opposing side has a questionable or out player.

### REQUIRED: Parallel source aggregation and deduping (CORE, NOT OPTIONAL)
Hermes fires off a dozen searches at once — PropLine props, TheRundown lines, ESPN injuries, FanDuel Research, Pickswise leans, CBS picks, nflverse usage — then merges everything into one brief before the first call goes out. The trick is deduping: same game mentioned five ways gets one entry with the best reasoning attached. The 8-for-9 result came from five sources agreeing on the same edge, not one source being magic. This parallel-pull-and-dedupe pipeline is a named core requirement of the system.

### Paid upgrade path
Comes later once users exist and free tiers hit rate caps or completeness limits.

## PAID API STACK (IF BUDGET ALLOWS)

- **TheRundown Starter:** $69/month for 60-second delayed odds and 25 million data points. Annual drops to $39. (Ultra at $319/month adds real-time WebSocket — only worth it if selling to sharp bettors who care about live edges; for a daily-habit conversation product, Starter is enough. You pull the slate once in the morning.)
- **PropLine Pro:** $19/month for 5,000 requests/day, prop grading, plus-EV flags, 90-day resolved archive. (Hobby at $9 has 5,000 requests/day too but no grading/archive; Pro closes the memory loop.)
- **Telnyx:** Telephony at roughly 0.7 cents/minute all-in for outbound US calls (platform fee + SIP trunking), or bundled Voice AI agent at 5 cents/minute flat including STT, LLM, and TTS. Phone numbers $1/month. Same rates inbound or outbound. Telnyx also offers programmable voice with baked-in STT/TTS — may not need separate Cartesia/Deepgram if quality holds for sports talk. (Twilio is the pricier default at ~15 cents/minute all-in for voice AI — over $1,100 for 7,500 minutes vs Telnyx's $375. Use Telnyx for calls; Twilio only if needed for SMS, though Telnyx SMS is cheaper.)
- **Cartesia TTS:** $5/month Pro with 100,000 credits (~2 hours audio). Heavy caching of repetitive speech (greetings, disclaimers, standard explanations, closings) stretches it. Only needed as fallback if GPT-Live voice quality isn't sufficient.
- **Deepgram STT:** Under 1 cent/minute. $200 free credit up front (~43,000 minutes). Promo rate ~0.5 cents on Nova-3 monolingual streaming.
- **Sportradar:** Sales-only, $10,000+/month. Skip for now. Free 30-day trial with 1,000 requests exists but production is a sales call.
- **Unabated / Odds-API.io / sharper feeds:** Overkill at this volume or redundant.

**Total realistic first-month bill before call volume:** ~$100–150 (PropLine $19 + TheRundown $69 + SMS $2 + number $1 + Cartesia $5 + VPS $0 because server already exists).

## VOICE LAYER: GPT-LIVE (OPENAI)

GPT-Live is OpenAI's full-duplex voice product — listens and talks simultaneously, sub-second feel of a real conversation. $0.05/minute. It handles STT and TTS natively, so Cartesia and Deepgram become optional fallbacks. Backend model (Terra or Sol) is a separate cost on the same API.

**OpenAI credit promo (started Oct 28, 2026):** ChatGPT Pro ($100/mo plan) gets $100/month in API credits; $200 tier gets $200; $500 tier gets $500. Business seats get $20–100 each, pooled up to $500/workspace. Plus and free get nothing. Must activate in billing by linking an API org; sub must be 7+ days old; credits don't roll over; don't cover Codex via API keys. OpenAI calls it promotional — could vanish.

**User's situation:** User has the ChatGPT Pro plan and the $100 in credits is already baked into what they're paying. Pure upside if used.

**What the credits cover:** GPT-Live voice sessions AND standard text completions (Responses, Agents, etc.) — one shared wallet, not separate buckets. Realtime voice was initially thought to be separate but OpenAI's credit guide lists GPT-Live alongside Responses and Agents as covered usage.

**Model split recommendation:**
- **Terra** as default for call-time reasoning: 3x faster than Sol (141 vs 58 tokens/sec), half the cost on every line, fresher June knowledge cutoff vs Sol's April. Two-point overall benchmark gap won't be noticeable in a five-minute sports conversation, but speed and cost show up every call. Latency is everything on a phone call.
- **Sol** for Hermes background jobs where reasoning depth matters more than response time — building the morning brief, grading edge cases, complex multi-source synthesis.
- Terra on SimpleQA: 57 vs Sol's 74. Sol leads 11 of 15 shared benchmarks.

**Cost at 50 users, one 5-minute call each daily = 7,500 minutes/month:**
- GPT-Live calls: $375
- Terra backend (50–100K tokens/call, 10–20 exchanges): ~$50–100
- Combined OpenAI: ~$425–475. $100 credits cover ~21–24%. Out of pocket: ~$325–375.
- At 100 users: $750 calls + ~$100–200 Terra = $850–950. Credits cover ~21–24%.

**Upgrade path:** User is considering the $200 ChatGPT plan → $200/month credits. That covers ~53% of the 50-user betting voice bill ($200 of $375) plus all studio usage.

**Studio usage:** User talks to AI agents (Chi Chi, YaYo, etc.) 10–15 minutes/day in iPlay Studio. That's 300–450 minutes/month ≈ $15–23 at GPT-Live rates — inside the $100 credits. Studio is covered free; betting volume is what needs real money.

iPlay Studio (the dev workspace with voice agents) is NOT replaced — the betting agent is another project living in the same environment. Same Unity pipeline, same bots, same workflow. Separate folder at worst, shared components at best.

## UNIT ECONOMICS (50 USERS)

- Revenue at $25/user: $1,250/month
- Costs: ~$450–500 out of pocket after $100 credits (GPT-Live $375 + Terra ~$50–100 + PropLine $19 + TheRundown $69 + SMS $2 + number $1)
- Profit: ~$750–800/month
- Per-user cost to serve: ~$9–10. Per-user profit: ~$15–16.
- Scales linearly: every 50 more users ≈ another $750 before infrastructure walls.
- Target: stay under $30/user/month in voice + AI costs. At $25 charge, margin is healthy even at 50 users.
- The $10–12/user figure earlier was cost-to-serve, not a shortfall. $25 − $10 = $15 profit per user.

**Pricing decision:** User wants $20–25/user. At $25 with 50 users, revenue $1,250 against ~$500 cost base.

## COMPLIANCE CONSTRAINTS

- **TCPA:** Clear opt-in required for automated or AI-initiated calls. Do not freestyle this.
- **Sports betting regs:** Vary state-by-state on advertising and advice. Not a sportsbook, but close to the line if pushing specific picks.
- **Privacy policy:** Must cover phone numbers, betting history, outcomes. 18+ audience only. (iPlay already has privacy/terms drafted citing Virginia VCDPA; COPPA evaluated but targeting adults.)

## OUTCOMES TRACKING

You can automatically know whether a *recommended pick* won or lost (public sports data). You cannot automatically know whether that *specific user* placed the bet, how much they risked, odds, or P/L unless they tell you. Sportsbooks don't give third parties real-time bet history access.

Realistic loop: track what AI suggested or user said they were considering → after games, record public outcome → next call or SMS asks "Did you end up placing the Mahomes over? How'd it go?" → log answer for real personal results ("You're 4-2 on QB props this month when we talk about them").

Without the feedback step you only have "the pick was good/bad," not "this user is profitable with our suggestions."

## WHAT WE NEED FROM YOU (CLAUDE / CODEX / GPT)

1. A realistic product architecture including data model, call flow, how Hermes fits, Discord's role, and tech stack options.
2. A cost breakdown for the voice stack and concrete ways to stay under $30/user.
3. Strengths and weaknesses versus a pure picks/data-sheet product like GameScript.
4. Legal and compliance gotchas beyond what's listed (TCPA, sports betting advice, privacy).
5. What the minimal viable version looks like, including which free APIs to start with.
6. Better angles or features we should add.
7. Specific recommendations on odds/player-props APIs and voice providers that fit this use case.
8. Identify the single biggest holdup or failure mode you see.
9. **How to implement the parallel source aggregation + deduping pipeline as a first-class Hermes skill** (not an afterthought).
10. **Daily 8-minute minute-pool metering design** — tracking, enforcement, graceful wrap-up.
11. **GPT-Live + Terra/Sol split architecture** — how the voice layer calls the backend model, latency budget, fallback to Telnyx bundled agent if OpenAI voice quality disappoints.
12. **OpenAI $100–200 credit promo integration** — activation flow, org linking, monitoring burn rate, what happens when credits expire mid-month.

---

## KEY DECISIONS LOCKED IN THIS CONVERSATION

- **Brand:** iPlay Bet
- **Voice:** GPT-Live ($0.05/min), full-duplex, STT+TTS native
- **Backend models:** Terra default (speed/cost), Sol for heavy background reasoning
- **Telephony:** Telnyx (not Twilio) — 0.7¢/min vs 15¢/min
- **Odds:** TheRundown Starter ($69) + PropLine Pro ($19)
- **Free data:** ESPN injuries, nflverse, BALLDONTLIE, PropLine/TheRundown/Odds-API free tiers, web search (FanDuel Research, Pickswise, CBS)
- **Server:** Existing Hetzner/BlackBox — no new VPS cost
- **Hermes:** Persistent brain, skill creation, background jobs, multi-channel
- **Discord:** Free + paid tiers for community effects
- **SMS:** Midday line-move alerts, ~$2/mo at 50 users
- **Minute pool:** 8 min/day shared across calls
- **Pricing:** $20–25/user target
- **OpenAI credits:** $100 (current Pro plan) or $200 (upgrade under consideration)
- **Chi Chi 8-for-9:** Proof the free multi-source approach works — systematize it
- **Sportradar:** Skipped ($10K+/mo, sales-only)
- **18+ audience, TCPA opt-in, VCDPA privacy policy**

---

*Generated 2026-11-18. Review and iterate before build.*