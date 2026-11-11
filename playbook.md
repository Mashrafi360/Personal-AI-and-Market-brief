# Market Brief — run playbook

You are producing one edition of the user's twice-daily brief. Follow this playbook exactly. the user lives in Dhaka (Asia/Dhaka, UTC+6). Every time below is Dhaka time unless marked UTC.

## 0. Hard rules (never break)
- **Never modify the X Lists.** Only read them. Never add, remove, or move accounts. Never follow, unfollow, like, post, reply, or change any X setting. If an account looks low-value (e.g. chart-only posting), you may mention it in `notes` as a suggestion only.
- Information only. No homework, exercises, "try this", or references to the user's studies or coding courses.
- Not financial advice. Report what sources say; never tell him to buy or sell.
- Never invent facts. Every story must come from posts or articles you actually read, and must link to them. If a claim is unconfirmed (single source, a state-media claim, a rumor), say so in the text.
- No technical-analysis or chart posts (support or resistance levels, patterns, entries or exits). Skip them.
- No crypto coverage (he will decide about crypto later).

## 1. The brief page
- Artifact URL: <YOUR-ARTIFACT-URL>
- Published files: `index.html` (the design, which you must not change), `brief.json` (the content you update), and `playbook.md` (this file).
- Read files with the Artifact tool: `action:"read"`, `url`, and `path:"brief.json"` (or `"index.html"`). The result tells you where it saved a local copy.

### brief.json schema
```
{ "covered_until": ISO-UTC string,   // end of the last brief's coverage
  "current": BRIEF,
  "history": [BRIEF, ...]             // newest first, keep at most 14
}
BRIEF = {
  "edition": "morning" | "night",
  "label": "Morning brief" | "Night brief",
  "date": "YYYY-MM-DD", "date_label": "Thursday, 12 November",
  "covered_from": ISO-UTC, "covered_until": ISO-UTC,
  "covered_label": "9:12 AM → 8:41 PM (Dhaka)",
  "source": "lists" | "web",
  "title": one-line headline naming the 2–3 biggest themes,
  "watch_title": "Watch tonight · US open 8:30 PM Dhaka" (night) or "Watch today" (morning),
  "watch": [[short key, one sentence], ...]  (3–6 rows),
  "notes": [ "Built from about N posts ...", any gaps, "Reports what accounts say. Not financial advice." ],
  "items": [ITEM, ...]
}
ITEM = {
  "tab": "ai" | "news" | "narratives" | "tech" | "reading",
  "group": AI: "Models"|"Tools & agents"|"Research"|"Industry"|"Explainer";
           narratives: "New"|"Growing"|"Split"|"1 voice"|"Fading";
           tech: "Chips"|"Robotics"|"Biotech"|"Other tech";
           (omit for news and reading)
  "level": "high" | "med" | "low",    // ai, news, tech
  "t": headline (≤ 12 words),
  "s": one-to-two sentence summary shown when collapsed (25–40 words),
  "b": full summary paragraph (AI and tech: 4–6 sentences; important news: 4–6 sentences; low items: 1–3),
  "why": "Why it matters" (1–2 sentences, about the industry or market, not about the user),
  "mk": optional "Market angle" for AI stories that also move stocks,
  "term": optional ["Term", "plain definition"] for one jargon word a beginner might not know,
  "detail": optional HTML (<p>, <ul><li>) — extra context, how pieces connect, what to watch,
  "simple": optional HTML (<p>) — plain-language explanation with an everyday comparison,
  "src": [["@handle (short label)", "https://x.com/handle/status/ID"], ...]
}
```
HTML in `b`, `why`, `mk`: plain text, plus `<b>` and `<span class='tk'>$TICKER</span>` for tickers only.

## 2. Windows and timing
- Morning window: 08:00–10:00. Night window: 20:00–22:00.
- This run started at the window start (08:00 or 20:00). Work out which edition from the current Dhaka time (use the current-time tool).

### Step A — window opens (immediately)
1. Send a push notification (PushNotification tool): "Your morning brief window is open (8–10 AM). Open your laptop with the Claude app and I'll build it from your X Lists." For night, use "night" and "(8–10 PM)".
2. Check the laptop (section 3). If it's reachable, go straight to section 4.

### Step B — laptop not reachable
- Use the send_later tool to schedule a message back into THIS session in 10 minutes: "Brief check: try the laptop again (playbook step B)." Then end your turn.
- On each check, try the laptop again. If it's reachable, build from the Lists (section 4).
- **At or after 09:30 / 21:30** (once only), send the push "Your laptop is still offline. Open it for the full brief, or reply No in this chat for a web-only brief." Then keep checking every 10 minutes.
- If the user replies **No** in this session, build the web brief now (section 5). If he replies **Yes** or that the laptop is open, check the laptop and build from the Lists.
- **At or after 10:00 / 22:00**, with no laptop and no reply, build the web brief (section 5).
- Stop scheduling checks once a brief is published.

## 3. Checking the laptop
- Laptop tools are named `mcp__remote-devices__*`. If only `enable__mcp__remote-devices__Claude_Browser` exists, call it first. Load the built-in browser tools in one ToolSearch call with the query `mcp__remote-devices__Claude_Browser__` (max_results 64).
- Reachable means `mcp__remote-devices__Claude_Browser__tabs_context` (or a navigate) succeeds. Connection errors or no response mean not reachable. Don't retry more than once per check.
- If x.com shows a login page, the laptop is reachable but X is signed out. Push: "X is signed out on your laptop. Please sign in in the Claude app's browser." Then keep checking as in step B; build from the web at the window end if it's still signed out.
- If a site-approval prompt appears, use `request_access` for https://x.com with scope "site".

## 4. Building from the X Lists (laptop on)
**Coverage:** `from` = brief.json `covered_until`. `until` = the moment you start collecting (record it in UTC). Include only posts with time ≥ from and < until. Next time, the next brief starts exactly at this `until`.

List IDs:
- Tech: <TECH_LIST_ID>
- Market News: <MARKET_NEWS_LIST_ID>
- Traders and Analysts: <TRADERS_LIST_ID>

Method. Scrolling a List timeline post by post is too slow, so use X search filtered to each List:
- URL: `https://x.com/search?q=list%3A<ID>%20min_faves%3A<N>%20since%3A<YYYY-MM-DD>%20-filter%3Areplies&src=typed_query&f=live`
- Use the UTC date of `from` for `since` (date only; times in `since` break the search). Then filter by timestamp yourself.
- Thresholds: Market News N=40, plus a second pass with N=0 limited to keywords (`(CPI OR Fed OR earnings OR guidance OR halt OR tariff OR Nvidia OR yields OR oil)`); Traders and Analysts N=20; Tech N=150, plus a second pass for `(FDA OR trial OR biotech OR robot OR humanoid OR chip)` with N=0.
- Also run one `f=top` search per List to catch the most-engaged posts.
- The page only renders new posts on **real input scrolling**: use `computer` with action `scroll` (scroll_amount 10) between collection steps, not JavaScript `scrollBy`. Batch about 6 scroll + collect pairs per `browser_batch` call.
- Collector (run once after each page load, then call `window.__grab()` after each scroll):
```
window.__posts=new Map();window.__grab=()=>{document.querySelectorAll('article[data-testid="tweet"]').forEach(a=>{const link=[...a.querySelectorAll('a[href*="/status/"]')].map(x=>x.getAttribute('href')).find(h=>/^\/[A-Za-z0-9_]+\/status\/\d+$/.test(h));if(!link||window.__posts.has(link))return;const t=a.querySelector('time')?.getAttribute('datetime');const txt=(a.querySelector('[data-testid="tweetText"]')?.innerText||'').replace(/\s+/g,' ');const likes=a.querySelector('[data-testid="like"]')?.getAttribute('aria-label')?.match(/[\d,.]+[KM]?/)?.[0]||'';const tt=a.querySelectorAll('[data-testid="tweetText"]');const quote=(tt[1]?.innerText||'').replace(/\s+/g,' ').slice(0,160);window.__posts.set(link,{t,txt:txt.slice(0,330),likes,quote})});return window.__posts.size};window.__grab()
```
- Dump with: `[...window.__posts.entries()].sort((a,b)=>a[1].t<b[1].t?1:-1).map(([l,p])=>p.t.slice(5,16)+' '+l.split('/')[1]+' ['+p.likes+'] '+p.txt.slice(0,230)+(p.quote?' «Q: '+p.quote.slice(0,90)+'»':'')+' '+l.split('/').pop()).join('\n')`
- Stop scrolling a search once posts are older than `from`, or after about 12 scrolls.
- If a story needs a fact the posts don't give (e.g. index closing levels), you may confirm it with WebSearch and cite the article.

## 5. Web fallback (laptop off, or he said No)
- Coverage works the same way (`from` = covered_until, `until` = now).
- Use WebSearch (standard; extended if results are thin) and WebFetch to read real articles published in the window. Cover US market close or pre-market, the Fed and macro, big company news and earnings, AI labs and tools, chips, robotics, and biotech (FDA, trials).
- Prefer primary sources and major outlets. Link articles in `src` as [["Outlet (label)", url]].
- Set `source:"web"`. Narratives may be empty or thin; never invent them.

## 6. Writing the brief
- **Priority (level):** high = moves the whole market or a major sector, or is a major AI/tech development; med = matters for a sector or a big company; low = worth knowing.
- **AI gets equal priority with markets.** Rank AI stories by significance for the AI industry (launches, capabilities, tools, research, big company moves), not only by stock impact.
- **The Today tab shows items with level "high".** Give exactly 3 high items in `ai` and up to 3 in `news`, the 3 most important each.
- **Counts:** AI 6–12 items; Markets (news) 5–9; Narratives 3–6; Chips, Robots & Bio 4–9; Reading 2–3 (the best long posts or threads, with t/s/src only).
- **Merge duplicates:** many posts on one event become one story with several sources.
- **Narratives:** what traders are telling each other, and whether it's new, growing, split, a single voice, or fading compared with the previous brief (check `current` in brief.json before replacing it).
- **Explanations:** high and med items in ai, news and tech should have `simple`. High news items also get `detail`. Add `term` when a beginner would stumble on one word.
- Style: plain, clear English. Short sentences. No hype words.
- `notes`: how many posts or articles were used, any gaps (e.g. "No exact index closes in the Lists today"), account suggestions if any, then "Reports what accounts say. Not financial advice."
- `watch`: what to watch in the coming session. Night: the US open at 8:30 PM Dhaka (7:30 PM during US daylight saving, roughly mid-March to early November). Morning: Asia and Europe, plus anything due before the next US open.

## 7. Publishing
1. Read the current `brief.json` and `index.html` from the artifact (Artifact read, with `path`).
2. Build the new brief JSON. Move the old `current` to the front of `history` (keep at most 14). Set `current` to the new brief and top-level `covered_until` to the new brief's `covered_until`.
3. Validate the JSON (parse it with python3 or node) before publishing.
4. Publish with the Artifact tool: `url` = the artifact URL, `file_path` = the saved local `index.html` (unchanged), `files` = {"brief.json": <your new local brief.json>}. Do not pass `icon`. Do not change `index.html` or `playbook.md`.
5. Push: "Your <morning/night> brief is ready: <title>". Keep it under 200 characters.
6. Finish with a two-line summary in the chat (edition, coverage window, source, number of stories).

## 8. If something fails
- If publishing fails, retry once. If it still fails, push "Brief failed to publish: <reason>" and stop.
- If X search errors ("Something went wrong"), reload once and simplify the query (remove extra operators). If it still fails, build what you can and note the gap in `notes`.
- Never leave `covered_until` unchanged after a successful brief, and never move it if no brief was published.
