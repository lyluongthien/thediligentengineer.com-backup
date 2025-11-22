---
title: "The Cloudf|are Outage [mDraft]"
datePublished: Sat Nov 22 2025 11:13:09 GMT+0000 (Coordinated Universal Time)
cuid: cmia6xe4e000402jv6jxqexfc
slug: the-cloudfare-outage-mdraft
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1763809347069/7b9bc9c9-06d6-4c68-ba84-a66a44a9d27c.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1763809969461/6a7aaa30-a6ac-469d-9168-06d23f8922fe.png
tags: memes

---

If you tried to use the internet, or accessing [thediligentengineer.com](https://thediligentengineer.com/) on November 18, you probably noticed it was broken. For a solid couple of hours, half the web was throwing 502 errors, loading spinners were spinning into eternity, and 𝕏 (the E's bird) was full of people asking if the apocalypse had started.

Cloudflare released their [post-mortem](https://blog.cloudflare.com/18-november-2025-outage/) recently. I read it so you don't have to. It wasn't a super-advanced AI cyber-attack or a guy tripping over a power cord.

*It was a hardcoded limit and a SQL query.*

Here is the breakdown of how the internet broke, presented without judgment (mostly).

### The "Happy Path" is a ('white')Lie

According to the report, the issue started with a "feature file" used by their Bot Management system. This file tells the network how to spot bad bots.

The software reading this file had a memory limit. It was hardcoded to handle a certain amount of data. Then, due to a database issue, the file size doubled.

Instead of saying, "Hey, this file is too big, I'm just going to ignore it and keep the website running," the software decided to do the honorable thing and **crash the entire process**.

It's like burning down your kitchen because your toast got stuck. Sure, the toast is gone, but so is the house.

### The SQL Query

The root cause was a change in database permissions. Someone decided to make permissions "better" (always a dangerous word).

They had a query running that looked something like this:



```sql
SELECT name, type FROM system.columns WHERE table = 'http_requests_features' order by name;

```

Notice anything missing? Like, maybe a `WHERE database = 'default'` clause?

Because they didn't specify *which* database to look in, and the new permissions allowed the user to see *more* databases, the query returned duplicate columns. It returned the columns for the default view *and* the underlying storage.

SQL does exactly what you tell it to do, which is usually the problem. The system assumed, "I will only ever see what I expect to see." The database replied, "Here is everything you asked for," and the application choked on it.

### The "Is it a DDoS?" Phase

My favorite part of the report is the "Fog of War."

When the system started failing, the engineers looked at the graphs. The error rates were fluctuating wildly---up, then down, then up again.

Why? because the bad config file was being generated every 5 minutes. Depending on which database shard the query hit, it would either generate a good file (recovery!) or a bad file (crash!).

Because of this weird pattern, the team naturally assumed: **"We are under attack."**

They thought it was a massive DDoS. At the same time, the Cloudflare Status Page went down. This turned out to be a total coincidence, hosted on a completely different infrastructure. But imagine being in that war room: The network is crashing, the graphs look crazy, and even your Status Page is dead. You'd probably start looking out the window for alien spaceships.

### The Fix

Once they realized it wasn't hackers, but their own code checking out early, the fix was simple:

1.  Stop generating the bad file.

2.  Put the old, good file back.

3.  Restart the thing.

It's the enterprise version of "Have you tried turning it off and on again?"

 
### The Internet Blames Rust (Naturally)

Cloudflare is famous for loving the Rust programming language because it is mem safe.

Naturally, as soon as the outage happened, everyone on Hacker News and Reddit glanced at the headline, saw "limit exceeded," and immediately decided the entire Cloudflare codebase must look exactly like this single line of code:


```rust
fn load_global_config() {
    // We expect the file to be small.
    // We are seemingly 100% sure it will be small.
    // So we use the "Trust Me Bro" function:

    let safe_config = download_features()
        .fit_into_memory_limit()
        .unwrap(); // <--- The exact moment the internet died
}

```

It's cutesy. We love to blame `.unwrap()` because it's the developer saying, "I promise this value will never be an error." And on November 18, the database looked at that promise and said, "Bet."

Technically, the code *was* safe. It safely exited the building... it just took all our traffic with it. 😉
### The Verdict

Cloudflare is great, and their transparency is gold standard. But this is a gentle reminder to all of us who write code:

-   **Validate your inputs**, even if they come from inside the house.

-   **Be explicit in SQL**, because databases love to be technically correct but practically destructive.

-   **Don't hardcode limits** that crash the main thread if exceeded. Just log an error. We promise we'll read the logs eventually. Maybe.

The internet is a fragile ecosystem held together by duct tape and `SELECT *` statements. Stay safe out there.