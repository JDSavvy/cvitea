<div align="center">

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/roadmap.en.dark.svg">
  <img src="assets/roadmap.en.light.svg" width="100%" alt="Roadmap from v1.0 in use now to an application tracker and preparation tools next, then job search and map, more connections, a drafting assistant, passkeys and phone later, and open ideas.">
</picture>

**English** · [Deutsch](ROADMAP.de.md) · [Back to the overview](README.md)

</div>

# Roadmap

As of release 46. The order is a plan, not a promise; the later a step, the more open it is.

## The order and why

**Own use first.** Each step uses the data of the one before: the tracker and the building blocks feed the job search, the job search feeds the assistant, and the assistant writes from building blocks that already exist.

## Now · v1.0 in use

Shipped in October 2026. A public business card, a tailored profile per application behind a personal link with an optional PIN, a hero video or image in the head of a profile with a position per device and an editor that shows the guest's screen, one download for all documents, a web app for the owner in German and English, AI agents that propose over MCP, and GitHub sync and LinkedIn import. Browser tests can run in parallel, each worker with its own test server and database. See [Where it stands](README.md#where-it-stands).

## Next

In progress or planned in version 1.x:

- Agents propose a whole new application, for example from a job ad, which counts as untrusted input. Changes to an existing application can already be proposed.
- Agents withdraw their own proposals and see which of them are still open.
- Connecting AI assistants such as Claude and ChatGPT by entering the server address and signing in on a consent page that lists the rights. Claude Desktop and claude.ai already connect with an OAuth sign-in; personal tokens remain as a fallback.

### Application tracker and preparation

- **Application tracker**: every application from first idea to answer, with status, deadlines, contacts, notes and reminders, and a calendar feed (ICS).
- **Preparation tools** per application, with matching examples from the building blocks.

## Later

### Job search and map

- **Interactive search with a map.** Commute times show as reachable areas for public transport, bike and car.
- **Sources**: official interfaces and feeds, employers' own career pages where their terms allow it, and ads pasted in by hand. No scraping of job portals.

### More connections

- **LinkedIn and GitHub in one click**, with one clear direction per entry and a three-way comparison when both sides changed. Writing to LinkedIn profiles is reserved for LinkedIn partners, so suggestions for LinkedIn stay ready-to-copy texts.
- **Further sources where they add value**: certificates and badges, publications (ORCID), calendars (Google, Microsoft) and design portfolios.

### Drafting assistant

- **From a job ad to a first draft**: a proposed selection of building blocks, CV, cover letter and documents, in German and English, which the person reviews, edits and approves. Version 1.0 lets agents propose single blocks and texts.
- **Skill gaps**: what a target role asks for that the profile does not show yet, with concrete ways to close the gap or to show evidence.
- **The human approval stays.** Nothing leaves without the person's yes. Job ads and uploaded documents count as untrusted input.

### Passkeys and phone

- **Passkeys** for sign-in and an **installable web app** (PWA) for phones.

## Open ideas

- More languages: French and Italian.
- A privacy dashboard with one switch to revoke every link.
- A second pair of eyes: a trusted person reviews an application before it goes out.
- An accessibility statement per profile.
