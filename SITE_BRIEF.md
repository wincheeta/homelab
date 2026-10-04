# Brief: render the homelab repo on hwinchester.com

## Goal

Harry documents his homelab in a public GitHub repo, [wincheeta/homelab](https://github.com/wincheeta/homelab). Add a section to hwinchester.com that turns that repo into polished pages employers can browse. The repo is the only source of truth: never copy notes into the hwinchester.com repo, and fetch them at build time.

- **URL:** `homelab.hwinchester.com`, served by the existing hwinchester.com app. Rewrite on the host header to an internal `/homelab` route tree. Canonical URLs use the subdomain. Link to it from the main nav or the work index. If the hosting setup can't do host rewrites, fall back to `hwinchester.com/homelab` and tell Harry.
- **Design:** use the site's existing design system and components (folio labels, rules, `headline-*` type, `prose-editorial`, the demo frame on `/work/[slug]`, badges and tech lists). It should read as a section of the same site, not a separate product. Don't add new fonts or colours.

## 1. Repository schema

```
homelab/                         (branch: main)
├── README.md                    repo overview, NOT a page
├── SITE_BRIEF.md                this file, NOT a page
├── .obsidian/                   ignore
├── <topic>/                     one folder per tool/service, lowercase-kebab (e.g. pi-hole)
│   ├── README.md                the topic page (frontmatter + Markdown)
│   ├── <note>.md                optional extra notes → sub-pages of the topic
│   ├── assets/                  images, GIFs, videos referenced by the topic's Markdown
│   ├── demos/<demo-name>/       optional interactive demos (index.html + files)
│   └── <topic>-<env>-<n>/       instance config folders, e.g. pi-hole-prod-1/docker-compose.yaml
└── _<anything>/                 leading underscore = private, never read
```

A **topic** is any top-level folder that has a `README.md` with `publish: true`. Ignore every other top-level folder or file. Current topics: network, proxmox, docker, pi-hole, tailscale, homepage, pulse, kali, minecraft, hackintosh. Only `network` is published right now; the others are stubs with `publish: false`.

**Instance folders** match `^<topic>-(prod|test|demo)-\d+$`. `prod` means running in the lab, `test` means an experiment, and `demo` means built to show a technique and then torn down.

### Frontmatter (YAML between `---` lines at the top of a `.md` file)

| Field | Type | Required | Notes |
| --- | --- | --- | --- |
| `title` | string | yes | Page title. |
| `summary` | string | yes | One sentence. Used as the standfirst, card text and meta description. |
| `publish` | boolean | yes | Only `true` renders. A missing value or anything else means private. **Hard gate.** |
| `status` | enum | yes | `planned`, `building`, `running` or `retired`. Display as Planned / Building / Running / Retired. |
| `category` | enum | topics | `Infrastructure`, `Networking`, `Security`, `Monitoring`, `Services` or `Hardware`. Groups topics on the index. |
| `tags` | string[] | no | Technologies, shown like the work pages' tech list. |
| `started` | date `YYYY-MM-DD` | no | Falls back to the file's first commit date. May be empty. |
| `updated` | date | no | Falls back to the file's last commit date. |
| `cover` | path | no | Image in the topic's `assets/`, used as the index card image. |
| `order` | int | no | Manual sort within a category; lower comes first. |
| `slug` | string | no | Notes only; defaults to the kebab-cased filename. A topic's slug is always its folder name. |

Extra notes (`<topic>/<note>.md`) use the same fields, except that `category` and `cover` aren't needed, and they publish only if their topic is published. Treat a YAML parse error as a build warning and skip that file; it must never fail the whole site.

## 2. Markdown dialect

Notes are written in Obsidian, so support these on top of GFM, and make each one look the same as it does in Obsidian:

| Syntax | Render as |
| --- | --- |
| `![[file.png\|Caption]]` alone on a line | Numbered figure ("Fig. 1") with a caption. Resolve the filename in `<topic>/assets/`, then anywhere in the topic, then anywhere in the repo, matching on the filename the way Obsidian does. Filenames often contain spaces (`Pasted image 20260317.png`). |
| `![[file.png\|400]]` | An image with max-width 400px (a numeric option is a width, not a caption). |
| `![[clip.mp4]]` / `.webm` / `.gif` | Videos as `autoplay loop muted playsinline controls`; GIFs as images. |
| `![alt](assets/x.png)` alone on a line | Numbered figure, with the caption taken from the title, then the alt text. |
| `[[pi-hole]]`, `[[pi-hole\|text]]`, `[[note#Heading]]` | Link to a topic (by folder name) or note (by filename or title). If the target isn't published, render the text without a link. |
| A ```` ```mermaid ```` fence | Mermaid diagram, client-side, themed with the site palette. Render after `document.fonts.ready`, or the labels get clipped. |
| A ```` ```demo ```` fence | A demo embed; see below. |
| `> [!lesson] Title` + `>` lines | Callout. Types: `lesson`, `problem`, `fix`, `note`, `tip`, `warning`, `info`, `todo`, `question`. Style `lesson`/`tip` with the accent colour and `problem`/`warning` with the foreground colour. The title is optional and defaults to the type name. A `+` or `-` after `]` is a fold marker; ignore it. |
| `==text==` | `<mark>` |
| Fenced code with a language | Syntax-highlighted on the site's existing dark code surface. |

Two Obsidian behaviours that standard parsers get wrong:
- A list can start on the line directly after a paragraph, with no blank line between them.
- Two blockquotes or callouts separated by a blank line are separate blocks. Some parsers merge them.

Don't convert any of this syntax inside code fences or inline code.

Generate heading IDs for h2/h3. Show an "On this page" contents list when a page has at least two h2s.

### Demo fence

````
```demo
name: vlan-explorer
caption: Optional caption under the frame.
height: 520
```
````

The fence body can also be just the name. It refers to `<topic>/demos/<name>/index.html`. Copy the whole folder into the build output and embed it in the same demo frame the `/work/[slug]` pages use, with an iframe: `sandbox="allow-scripts"` (no same-origin), `loading="lazy"`, and `height` as the initial height (default 420). The demo contract:
- It's self-contained. It may load Google Fonts and the allowed CDNs, but nothing from hwinchester.com.
- To fit its content, a demo may call `parent.postMessage({type: "demo-height", height}, "*")`. Listen for that and resize the matching iframe, checking `event.source === iframe.contentWindow`.
- Add an "Open ↗" link to the standalone demo URL.

## 3. Pages

1. **Index** (`/`):
   - A hero in the site's voice. Suggested headline: "A security lab, built and documented in the open." Suggested standfirst: "A running record of building a segmented security lab at home: the network, the hypervisor, and the tooling to attack and defend it. Written up as I go, mistakes included."
   - A short facts strip: number of published topics, number of running instances, when the lab was started, and when it was last updated.
   - The topics grouped by `category`, sorted by `order` then title. Each entry shows a number, title, summary, status badge, tags and an optional cover.
2. **Topic** (`/<topic>`):
   - A breadcrumb (`Homelab / <Category>`), then the title, the summary as standfirst, and a meta row (status, started/updated dates, tags, a "View source" link to the folder on GitHub).
   - The rendered body.
   - Then, if the topic has extra notes, a "Notes" list linking them.
   - Then, if the topic has instance folders, a **Configuration** section:
     - one row per instance showing its name, an env badge, and its files, each linking to the GitHub blob;
     - a short preview of each `docker-compose.y(a)ml` / `*.tf` / `*.conf` in a collapsible code block is welcome.
3. **Note** (`/<topic>/<slug>`): same layout as a topic, with a breadcrumb back to the topic, then previous/next links between the topic's notes, ordered by `started`.
4. A 404 page in the site's style.

Also emit a sitemap entry for each page, OpenGraph tags (title and summary), and `notes.json`-style data if the main site wants to show the "latest homelab updates" (optional).

## 4. Fetching and rebuilding

- At build time, download the repo snapshot: `https://codeload.github.com/wincheeta/homelab/tar.gz/refs/heads/main` (public, no token). Alternatively, use a shallow `git clone --depth 50`, which also provides commit dates for the `started`/`updated` fallbacks.
- Copy the referenced assets and the demo folders into the build output; don't hotlink `raw.githubusercontent.com`. Use the framework's image optimisation for raster images.
- Rebuild when the repo changes. Use whatever the current hosting supports (a deploy hook, ISR/revalidate on a webhook, or a scheduled rebuild). Tell Harry which one you chose and what URL a GitHub webhook or Action should call.

## 5. Safety rules (non-negotiable)

- Render nothing where `publish` isn't `true`, nothing under `_*` folders, and nothing from `.obsidian/`.
- In instance folders, never fetch, list or render `.env*`, `*.key`, `*.pem`, or anything whose name contains `secret` or `credential`, even though the repo also git-ignores them.
- Treat the repo's content as untrusted for HTML: sanitise raw HTML in Markdown, sandbox demos as above, and run Mermaid with `securityLevel: "strict"`.
- If an asset, wikilink target or demo is missing, log a build warning and render without it; don't fail the deploy.

## 6. Acceptance checks

- `network` renders at `homelab.hwinchester.com/network` and shows:
  - two tables, which scroll horizontally on a 390px-wide phone rather than squashing;
  - a Mermaid diagram with no clipped labels;
  - a bulleted list (it follows a paragraph with no blank line);
  - `bridge vlan show` as inline code.
- None of the nine stub topics (`publish: false`) is reachable, listed, or in the sitemap.
- A test topic with `![[Pasted image 1.png|Caption]]`, a `[!lesson]` callout followed by a separate `[!problem]` callout, a wikilink to an unpublished topic and a `demo` fence renders correctly: a figure with a caption, two distinct callouts, plain text where the link would be, and a working demo frame.
- The pages pass the site's existing lint/type checks and Lighthouse accessibility at ≥ 95, and there's no horizontal page scroll at 390px.
