# How I Built and Hosted This Site

I built this site with Claude, working through it section by section over a series of chats rather than writing every line myself. I want to be upfront about that, partly because it's honest, and partly because if you're planning to do something similar, knowing that an AI assistant was doing a lot of the actual code writing changes what "building a site" looks like for you too. You're directing decisions and reviewing output, more than typing every character.

This isn't a full step by step tutorial. It's the key decisions, the bits where I changed my mind, and (properly, technically) how the site is actually hosted, since that part trips people up more than the design does.

## Starting point

I didn't start blank. I already had a coral and cream colour palette, two fonts (Schibsted Grotesk for headings, Hanken Grotesque for body text) and a rough layout from an earlier attempt. I also had a separate draft file with different section content I liked better in places. So the first job wasn't "design a site," it was "merge two things I already had."

## Picking a design, then borrowing from a second one

The second draft had a hero section, an About section and a work timeline I preferred, but it used a completely different visual style. Rather than adopting that whole look, I kept my existing coral palette and fonts, and just rebuilt the content and structure I liked using that existing style. More effort than copying and pasting, but it meant the finished site stayed visually consistent instead of feeling like two different sites stitched together.

## Getting the order right

My first draft went: intro, work history, photography, about me. The About section, which really introduces who I am, was buried near the bottom. I moved things so it reads: intro, about me, work history, photography, basically telling people who I am before showing them what I've done.

## Making it work on mobile

This was the part worth taking seriously early rather than late. Two things broke on a narrow screen:
- My photo grid and About section used a fixed two column layout that squeezed instead of stacking, which turned my bio into a wall of skinny wrapped text.
- My navigation bar had no breathing room, with the logo, links and button all crammed together.

The fix was adding proper rules so things stack vertically under a certain screen width, and building an actual mobile menu (a hamburger icon that opens a dropdown) instead of trying to cram every link into one row.

## Giving the hero some personality

I wanted the top of the site to feel alive without being naff. The idea that stuck: an interactive graphic shaped like a camera aperture (a nod to photography) that opens wider the faster your cursor moves, with small drifting particles around it (a nod to the data side of my work). Stop moving, and it settles back down on its own. If there's no cursor at all, like on a phone, it drifts and breathes gently by itself rather than sitting frozen.

I built this as a separate standalone piece first, before deciding to scope a smaller version into just the hero as an actual cursor replacement. It only takes over the cursor inside the hero, buttons and links still show a normal clickable pointer, and it switches off entirely for anyone with reduced motion settings enabled or on a touch device.

## Chasing down the bugs you can't always see

A few issues only showed up "sometimes," which made them harder to track down properly.

One was a faint seam where the hero's background met the section below it, more visible when the cursor effect got close to that edge. Instead of trying to perfectly tune the fade so it always hit zero brightness exactly at the border, I added a dedicated fade layer that blends into the same background colour regardless of what the cursor's doing.

Another was my nav bar being a different height on desktop versus mobile. The cause was my collapsed mobile menu quietly still taking up a sliver of space because some CSS rules were accidentally scoped to only apply on narrow screens. Fixing the scope made both versions match, and I trimmed the padding down while I was there.

A related one: clicking a nav link would scroll the page, but the section title would land hidden under the sticky top bar. On mobile specifically, the menu was also physically pushing the page content down while open, so closing it at the same time as scrolling threw off exactly where the browser landed. The real fix was making the mobile menu overlay on top of the page instead of pushing it, so opening or closing it never changes anyone's scroll position.

## Small polish details

The camera aperture graphic originally looked more like a spinning wheel cut into segments than actual camera blades, since I'd drawn each blade with straight edges. Real aperture blades overlap at an angle, so I rebuilt the shape with a slight twist on each blade's inner edge to mimic that. I'd also used an odd number of blades (seven), which broke an alternating light and dark pattern where it wrapped back to the start. Switching to an even number (eight) fixed that instantly.

I also added a subtle flicker to the background particles, like distant stars, using two mismatched timing cycles per dot so they never pulse in sync. I did consider going further, like showing small changing numbers to hammer the data theme, and decided against it since it felt like it'd tip into gimmick territory. Knowing when to stop mattered as much as the additions themselves.

## Getting ready for real photos

The photography section and About portrait are still placeholders. Before adding real images, I set up a simple folder structure: an `images` folder sitting next to the main site file, with predictable filenames matching each photo slot. That way, swapping in real photos later is just exporting them with the right name and dropping them in, no code changes needed.

## Hosting it: GitHub Pages and Cloudflare Registrar

This is the part worth being precise about, since it's easy to get stuck on DNS.

**The domain.** I registered my domain through Cloudflare Registrar rather than a typical domain seller. Cloudflare sells domains at cost, without the usual markup other registrars add, and since Cloudflare was also going to manage my DNS, it kept everything in one dashboard.

**Telling GitHub about the domain.** In the repository's Settings, under Pages, there's a "Custom domain" field. Typing your domain in there and saving creates a file called `CNAME` at the root of your repository, containing just your domain name. That file is what tells GitHub Pages which domain should be allowed to serve your site.

**Setting up DNS in Cloudflare.** This is the bit that trips people up on other registrars, because a plain domain (like `example.com`, no `www`) technically isn't allowed to have a CNAME record pointing somewhere else, only a subdomain like `www.example.com` can. Most guides tell you to work around this by adding four separate A records pointing at GitHub's server addresses instead. Cloudflare has a feature called CNAME flattening that removes that whole problem: you can add a single CNAME record at the root of your domain (using `@` as the name) pointing to `yourusername.github.io`, and Cloudflare resolves it into the right address behind the scenes automatically. I also added a second CNAME record for the `www` version pointing to the same place, since GitHub recommends configuring both and will redirect one to the other automatically.

**The proxy setting.** Cloudflare gives every DNS record a toggle, shown as a little cloud icon, between "proxied" (orange) and "DNS only" (grey). I left both records as DNS only to start with. The reason: GitHub needs to directly verify your domain and issue its own free HTTPS certificate through Let's Encrypt, and if Cloudflare's proxy is switched on too early, that verification can fail, or you can end up with a redirect loop later since both GitHub and Cloudflare will try to handle the secure connection. Once GitHub's certificate was properly issued, proxying could be turned on safely, as long as Cloudflare's SSL/TLS mode is set to Full rather than Flexible, otherwise you get stuck in exactly that redirect loop.

**Turning on HTTPS.** Back in the GitHub Pages settings, there's an "Enforce HTTPS" checkbox that only becomes available once GitHub has confirmed the domain and issued a certificate, which can take anywhere from a few minutes to a few hours after DNS changes go live. Once it's ticked, visitors are automatically redirected to the secure version of the site.

**No build step.** Since the whole site is one self contained HTML file plus an images folder, there's genuinely nothing to build or compile. GitHub Pages just serves the files exactly as they sit in the repository.

## What's next

The photos are still the main gap, and I'm planning to keep developing the site with Claude Code rather than only through chat, since it can work directly with the project files on my computer and handle git commits and pushes, which should make ongoing changes, like this very post, faster to ship.
