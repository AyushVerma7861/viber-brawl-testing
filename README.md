# Viber Brawl — Testing Environment

An **isolated** deployment used to work on the multiplayer netcode without
touching the live game.

| | |
|---|---|
| Worker | `viber-brawl-testing` |
| Address | `https://viber-brawl-testing.<your-subdomain>.workers.dev/arena.html` |
| Durable Object | `ArenaRoom` |
| Database | **none** — multiplayer only |

## Why this exists

The live game (`viber-brawl-online`) is in use and another deployment is being
worked on separately. Everything here is renamed so the two can never be
confused, and the Durable Object namespace is scoped to this Worker, so this
deployment physically cannot reach the other one's rooms.

## What is different from the live game

1. **Regional placement.** When a room is created, the Worker reads the host's
   country and passes a Cloudflare `locationHint` so the room lands near the
   players instead of wherever Cloudflare guesses.
2. **No database.** The `[[d1_databases]]` binding is gone. Multiplayer is
   unaffected — the match code checks for a database and carries on without one.
   Consequence: the ACCOUNT and PROFILE buttons in the game will error if clicked.

## Running locally

```bash
npm install --ignore-scripts
npx wrangler dev --local
```

Then open `http://127.0.0.1:8787/arena.html`.

## Deploying

```bash
npx wrangler deploy
```

## Verifying regional placement

```bash
npx wrangler tail viber-brawl-testing
```

Creating a room should print:

```
[region] create XXXXX hint=apac country=IN continent=AS colo=MRS
```

If `country=?` appears, Cloudflare did not report the location and the code fell
back to the continent, then to a central default.
