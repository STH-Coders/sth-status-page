# **STH STATUS PAGE**

v1.3.0

- Monitors important domains, servers, and components within the STH ecosystem of services, data, and other software products.
- Boosts SEO, showcases 90-day historical reliability, and provides real-time Slack notification for all status changes.
- The code is open-source under the MIT License and free for you to reuse and redistribute if you so desire.
- Built with **Cloudflare Workers**, **CRON Triggers,** and **KV storage**; originally developed by [Adam Janiš](https://github.com/eidam).

- ### _*[VIEW LIVE](https://status.safethishome.workers.dev)*_

---

   <p align=center
  
[![made-with-react](https://img.shields.io/badge/Made%20with-React-1f425f.svg?style=for-the-badge)](https://reactjs.org/)

   <p align=center

[![StackShare](http://img.shields.io/badge/tech-stack-0690fa.svg?style=for-the-badge)](https://stackshare.io/sthcoders/main-site-stack)
[![MIT license](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](https://lbesson.mit-license.org/)

   <p align=center

[![CodeQL](https://github.com/STH-Coders/sth-status-page/actions/workflows/codeql-analysis.yml/badge.svg?style=for-the-badge)](https://github.com/STH-Coders/sth-status-page/actions/workflows/codeql-analysis.yml)

## Pre-requisites

You'll need a [Cloudflare Workers account](https://dash.cloudflare.com/sign-up/workers) with

- A workers domain set up
- The Workers Bundled subscription \($5/mo\)
  - [It works with Workers Free!](https://blog.cloudflare.com/workers-kv-free-tier/) Check [more info](#workers-kv-free-tier) on how to run on Workers Free.
- Some websites/APIs to watch 🙂

Also, prepare the following secrets

- Cloudflare API token with `Edit Cloudflare Workers` permissions
- Slack incoming webhook \(optional\)
- Discord incoming webhook \(optional\)

## Getting started

You can either deploy with GitHub Actions or deploy on your own.

### Deploy with GitHub Actions

1. Clone this repository
2. Navigate to your new **GitHub repository &gt; Settings &gt; Secrets** and add the following secrets:

   ```yaml
   - Name: CF_API_TOKEN

   - Name: CF_ACCOUNT_ID

   - Name: SECRET_SLACK_WEBHOOK_URL (optional)
   - Value: your-slack-webhook-url

   - Name: SECRET_DISCORD_WEBHOOK_URL (optional)
   - Value: your-discord-webhook-url
   ```

3. Navigate to the **Actions** settings in your repository and enable them
4. Edit [config.yaml](./config.yaml) to adjust configuration and list all of your websites/APIs you want to monitor

   ```yaml
   settings:
     title: 'Status Page'
     url: 'https://status.safethishome.workers.dev/' # used for Slack & Discord messages
     logo: logo-192x192.png # image in ./public/ folder
     daysInHistogram: 90 # number of days you want to display in histogram
     collectResponseTimes: false # experimental feature, enable only for <5 monitors or on paid plans

     # configurable texts across the status page
     allmonitorsOperational: 'All Systems Operational'
     notAllmonitorsOperational: 'Not All Systems Operational'
     monitorLabelOperational: 'Operational'
     monitorLabelNotOperational: 'Not Operational'
     monitorLabelNoData: 'No data'
     dayInHistogramNoData: 'No data'
     dayInHistogramOperational: 'All good'
     dayInHistogramNotOperational: 'Some checks failed'

   # list of monitors
   monitors:
     - id: workers-cloudflare-com # unique identifier
       name: workers.cloudflare.com
       description: 'You write code. They handle the rest.' # default=empty
       url: 'https://workers.cloudflare.com/' # URL to fetch
       method: GET # default=GET
       expectStatus: 200 # operational status, default=200
       followRedirect: false # should fetch follow redirects, default=false
       linkable: false # should the titles be links to the service, default=true
   ```

5. Push to `main` branch to trigger the deployment
6. 🎉
7. _\(optional\)_ Go to [Cloudflare Workers settings](https://dash.cloudflare.com/?to=/workers) and assign custom domain/route
   - e.g. `status-page.sthome.dev/*` _\(make sure you include `/*` as the Worker also serve static files\)_
8. _\(optional\)_ Edit [wrangler.toml](./wrangler.toml) to adjust Worker settings or CRON Trigger schedule, especially if you are on [Workers Free plan](#workers-kv-free-tier)

### Telegram notifications

To enable telegram notifications, you'll need to take a few additional steps.

1. [Create a new Bot](https://core.telegram.org/bots#creating-a-new-bot)
2. Set the api token you received when creating the bot as content of the `SECRET_TELEGRAM_API_TOKEN` secret in your github repository.
3. Send a message to the bot from the telegram account which should receive the alerts (Something more than `/start`)
4. Get the chat id with `curl https://api.telegram.org/bot<YOUR TELEGRAM API TOKEN>/getUpdates | jq '.result[0] .message .chat .id'`
5. Set the retrieved chat id in the `SECRET_TELEGRAM_CHAT_ID` secret variable
6. Redeploy the status page using the github action

### Deploy on your own

You can clone the repository yourself and use Wrangler CLI to develop/deploy, extra list of things you need to take care of:

- create KV namespace and add the `KV_STATUS_PAGE` binding to [wrangler.toml](./wrangler.toml)
- create Worker secrets _\(optional\)_
  - `SECRET_SLACK_WEBHOOK_URL`
  - `SECRET_DISCORD_WEBHOOK_URL`

## Workers KV free tier

The Workers Free plan includes limited KV usage, but the quota is sufficient for 2-minute checks only

- Change the CRON trigger to 2 minutes interval (`crons = ["*/2 * * * *"]`) in [wrangler.toml](./wrangler.toml)

## Known issues

- **Max 25 monitors to watch in case you are using Slack notifications**, due to the limit of subrequests Cloudflare Worker can make \(50\).

  The plan is to support up to 49 by sending only one Slack notification per scheduled run.

- **KV replication lag** - You might get Slack notification instantly, however it may take couple of more seconds to see the change on your status page as [Cron Triggers are usually running on underutilized quiet hours machines](https://blog.cloudflare.com/introducing-cron-triggers-for-cloudflare-workers/#how-are-you-able-to-offer-this-feature-at-no-additional-cost).

- **Initial delay (no data)** - It takes couple of minutes to schedule and run CRON Triggers for the first time

## Future plans

WIP - Support for Durable Objects - Cloudflare's product for low-latency coordination and consistent storage for the Workers platform. There is a working prototype, however, we are waiting for at least open beta.

There is also a managed version of this project, currently in beta. Feel free to check it out <https://statusflare.com> (<https://twitter.com/statusflare_com>).

## Running project locally

**Requirements**

- Linux or WSL
- Yarn (`npm i -g yarn`)
- Node 14+

### Steps to get server up and running

**Install wrangler**

```
npm i -g wrangler
```

**Login With Wrangler to Cloudflare**

```
wrangler login
```

**Create your KV namespace in cloudflare**

```
On the workers page navigate to KV, and create a namespace
```

**Update your wrangler.toml with**

```
kv-namespaces = [{binding="KV_STATUS_PAGE", id="<KV_ID>", preview_id="<KV_ID>"}]
```

_Note: you may need to change `kv-namespaces` to `kv_namespaces`_

**Install packages**

```
yarn install
```

**Create CSS**

```
yarn run css
```

**Run**

```
yarn run dev
```

_Note: If the styles do not come through try using `localhost:8787` instead of `localhost:8080`_

---

Copyright (c)2020 Adam Janiš
Copyright (c)2021 STHCoders/Michael Rehnert
