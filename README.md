# LLM Scraper with Nuxt & Cloudflare AI/Browser

This is a demo of a LLM Scraper with Nuxt & Cloudflare AI/Browser based on [llm-scraper-worker](https://github.com/threepointone/llm-scraper-worker) and [workers-ai-provider](https://github.com/threepointone/workers-ai-provider).

## Features

- Leverage Cloudflare AI with `@cf/meta/llama-3.1-70b-instruct`
- Use the Cloudflare Browser to scrape the web
- Cache the result for 10 minutes to avoid abuse
- One click deploy on 275+ locations for free on your Cloudflare account with [NuxtHub](https://hub.nuxt.com)

## Top 5 stories on Hacker News

This is a simple API endpoint that returns the top 5 stories on Hacker News as JSON on `/api/top-5-hn` & cached for 10 minutes using Cloudflare KV:

```ts
import { z } from 'zod'
import LLMScraper from 'llm-scraper-worker'
import { createWorkersAI } from 'workers-ai-provider'

export default defineCachedEventHandler(async () => {
  const { page } = await hubBrowser()
  const workersAI = createWorkersAI({ binding: hubAI() })
  const llm = workersAI('@cf/meta/llama-3.1-70b-instruct')
  const scraper = new LLMScraper(llm)

  await page.goto('https://news.ycombinator.com')

  // Define schema to extract contents into
  const schema = z.object({
    top: z
      .array(
        z.object({
          title: z.string(),
          points: z.number(),
          by: z.string(),
          commentsURL: z.string(),
        }),
      )
      .length(5)
      .describe('Top 5 stories on Hacker News'),
  })

  const { data } = await scraper.run(page, schema, {
    format: 'html',
  })
  return data
}, {
  maxAge: 60 * 10,
})
```

Visiting http://localhost:3000/api/top-5-hn will return the top 5 stories on Hacker News as JSON:

```json
{
  "top": [
    {
      "title": "Visualizing All ISBNs",
      "points": 107,
      "by": "RyanShook",
      "commentsURL": "https://news.ycombinator.com/item?id=42652577"
    },
    {
      "title": "Samsung Display to Begin Mass Production of Rollable OLED for Laptops",
      "points": 21,
      "by": "gnabgib",
      "commentsURL": "https://news.ycombinator.com/item?id=42653352"
    },
    {
      "title": "Predictions Scorecard, 2025 January 01",
      "points": 158,
      "by": "timr",
      "commentsURL": "https://news.ycombinator.com/item?id=42651275"
    },
    {
      "title": "Show HN: Tetris in a PDF",
      "points": 947,
      "by": "ThomasRinsma",
      "commentsURL": "https://news.ycombinator.com/item?id=42645218"
    },
    {
      "title": "Bird-inspired drone uses legs to walk and jump into the air",
      "points": 40,
      "by": "bookofjoe",
      "commentsURL": "https://news.ycombinator.com/item?id=42617825"
    }
  ]
}
```

## Setup

Make sure to install the dependencies with [pnpm](https://pnpm.io/installation#using-corepack):

```bash
pnpm install
```

Make sure to link your project with your Cloudflare account & NuxtHub account to use Workers AI locally:

```bash
npx nuxthub link
```

## Development Server

Start the development server on `http://localhost:3000`:

```bash
pnpm dev
```

## Production

Build the application for production:

```bash
pnpm build
```

## Deploy


Deploy the application on the Edge with [NuxtHub](https://hub.nuxt.com) on your Cloudflare account:

```bash
npx nuxthub deploy
```

Then checkout your server logs, analaytics and more in the [NuxtHub Admin](https://admin.hub.nuxt.com).
