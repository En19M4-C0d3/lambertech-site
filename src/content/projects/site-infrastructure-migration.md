---
title: 'Rebuilding This Site: From LAMP to a Hardened, Self-Designed Stack'
description: 'Retiring an unused LAMP setup, rebuilding on Caddy and Astro, diagnosing a live outage, and designing the site and brand from scratch.'
pubDate: '2026-08-30'
tools: ['Caddy', 'Astro', 'Ubuntu Server', 'UFW', 'fail2ban', 'Git', 'GitHub', 'Azure NSG', 'CSS/design systems']
outcome: 'Replaced an unused, unnecessary PHP/MySQL stack with a static-site pipeline that eliminates a whole class of runtime vulnerabilities, diagnosed and resolved a full site outage across four layers of the stack, and designed and built a cohesive brand and homepage from a blank starter template.'
tags: ['infrastructure', 'security hardening', 'DevOps', 'design']
---

## The problem

I had a cloud-hosted Ubuntu server (provided by a mentor, on Azure) with an active domain and a LAMP stack already configured — but no real content on it. The stack itself was a liability rather than an asset: PHP and MySQL are two of the most commonly exploited components on the web, and I had no actual use case that justified carrying that risk. I also didn't remember the MySQL root password from the original setup — which, rather than being a blocker, confirmed the right call, since the plan was to remove MySQL entirely rather than recover access to it.

## Approach: the rebuild

**Stack decision.** I moved away from a dynamic, database-backed setup entirely in favor of a static site generator (Astro) served by Caddy instead of Apache. Astro compiles the whole site into plain HTML/CSS/JS ahead of time, removing SQL injection, PHP remote code execution, and session-hijacking as categories of risk entirely, rather than trying to configure around them. Caddy issues and renews HTTPS certificates automatically via Let's Encrypt, with a far smaller configuration surface than Apache or Nginx.

**Removal.** I fully purged Apache, MySQL, and PHP at the package level, including their configuration directories and MySQL's data directory — deleting the database files entirely made the forgotten password a non-issue.

**Rebuild and version control.** I installed Caddy from its official signed repository, scaffolded a new Astro project using its blog template, and set up git from the very first commit — before any manual edits — pushed to a private GitHub repository as both backup and build history.

**Content architecture.** I extended Astro's default single blog collection into two: `blog` for reflective writing, and a new `projects` collection with its own schema (tools used, outcome, tags) so project write-ups carry structured metadata a generic blog post doesn't need.

## Diagnosing a full outage

After the initial deploy, the site was completely unreachable — a good exercise in working through a stack methodically rather than guessing. I checked each layer in turn:

- **Caddy itself** — confirmed via `systemctl status` and `journalctl` that the service was healthy and had successfully obtained a valid TLS certificate
- **The firewall** — checked UFW locally, then confirmed via `ss -tlnp` that Caddy was correctly bound to ports 80 and 443
- **The network layer** — since I don't have direct access to the Azure account this VM is billed under, I used `curl` from an external machine to test connectivity from outside, isolating whether the problem was server-side or network-side, rather than assuming I needed portal access to diagnose it
- **DNS** — confirmed via `nslookup` that the domain resolved to the correct IP

Every layer checked out — the actual fault turned out to be simpler than any of that: `/var/www/site`, the directory Caddy was configured to serve from, didn't exist at all. It had been removed at some point during the LAMP cleanup and never recreated, so Caddy was correctly running and correctly reachable, but had nothing to serve. Recreating the directory and redeploying the build resolved it immediately. The lesson: check the simplest possible explanation early, rather than assuming a networking issue is automatically complex.

## Designing the brand and homepage

With the infrastructure solid, I moved on to actually designing the site rather than leaving Astro's default template in place:

- **Defined a palette and type system** — a restrained dark theme (near-black background, off-white text, a single teal accent color) paired with Inter for body/headings and JetBrains Mono for small technical accents, chosen to read as credible in security/technical circles without leaning on generic dark-mode defaults
- **Rebuilt the header and footer components** — the template's default styling was hardcoded rather than driven by CSS variables in places, including a hardcoded white header background that didn't inherit the new theme; fixing this meant editing the component-level styles directly, not just the global stylesheet
- **Designed a two-tone wordmark** — "LamberTech," a deliberate merge of my surname with "tech," referencing the lambert (a unit of luminance) as the source of the site's light/spotlight motif
- **Built a custom homepage** — a hero section introducing the pitch, and a unified timeline that merges the `blog` and `projects` collections by date into a single feed, so new work of either kind surfaces automatically without manual curation

## Outcome

The site now runs on a stack with a meaningfully smaller attack surface than what it replaced, with no PHP interpreter, no MySQL server, and no database at all — an entire category of runtime web vulnerabilities is structurally absent. Automatic HTTPS, a restricted firewall, and fail2ban cover the network layer; a full git history backed up independently on GitHub covers the source. On top of that infrastructure sits an actual designed identity — palette, typography, wordmark, and a homepage that automatically surfaces new work — rather than an unstyled template.

This write-up is itself the first entry the homepage timeline is displaying.
