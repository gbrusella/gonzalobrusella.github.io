---
title: "Titan: one theme, two static site generators"
date: 2026-11-26
draft: false
description: "A gold-on-black theme with an animated tessellation background, published for both Hugo and Jekyll. What it does, how to install it, and how to make it yours."
tags: ["hugo", "jekyll", "theme", "titan", "static-sites"]
---

This blog runs on Titan, a theme I built: gold on black, with an animated tessellation drifting behind the content. The look comes from the Deus Ex Titan shield. I published it twice — [one for Hugo](https://github.com/gbrusella/hugo-theme-titan), [one for Jekyll](https://github.com/gbrusella/jekyll-theme-titan) — so you can use it whichever generator you prefer.

The background is a `<canvas>`, and visitors can turn it off. The choice is saved in `localStorage`. Animated backgrounds are fun for about ten seconds, then somebody wants to read. That toggle isn't optional.

## What you get

Ten tessellations, picked with one config value:

- **2-D:** `overlay` (Delaunay triangles with Voronoi edges), `voronoi`, `triangles`, `hexagons`, `squares`, `rhombille`
- **3-D lattices:** `hcp`, `fcc`, `bcc`, `cubic` — these always spin

This site uses `rhombille` with `motion = "sheet"`. The 2-D shapes also take `relax` (0 to 3, from raw random to very even cells) and `cell`, the cell size in pixels. Smaller means more cells and more work for the canvas.

There's a live demo for each: [Hugo](https://gbrusella.github.io/hugo-theme-titan/) and [Jekyll](https://gbrusella.github.io/jekyll-theme-titan/).

## Using it in Hugo

Add it as a submodule:

```bash
git submodule add https://github.com/gbrusella/hugo-theme-titan themes/titan
```

Set `theme = "titan"` and configure in `hugo.toml`:

```toml
[params.titan]
shape  = "rhombille"
motion = "sheet"
relax  = 2
cell   = 66

[params.hero]
kicker  = "Welcome"
title   = "Your Name's Blog"
tagline = "Thoughts, projects, and ideas"
```

It needs Hugo extended, v0.110 or later.

## Using it in Jekyll

Add the gem to your `Gemfile`:

```ruby
gem "jekyll-theme-titan", git: "https://github.com/gbrusella/jekyll-theme-titan"
```

Then `theme: jekyll-theme-titan` in `_config.yml`, with the same options under `titan:` and `hero:`. Same names, same values, YAML instead of TOML.

## Making it yours

Every colour is a CSS custom property on `:root`, so retheming is a stylesheet override, not a fork:

```css
:root {
  --gold-bright: #e8eaf6;
  --gold:        #9fa8da;
  --gold-deep:   #3949ab;
  --ink:         #050a1a;
}
```

That's silver on navy. Same theme, different mood.

The background also has a small JavaScript API if you want to drive it from your own page:

```js
Titan.toggle()
Titan.setConfig({ shape: "hexagons", cell: 50 })
```

## Try it

Both repos are on GitHub, issues welcome. If you build something with it, I'd like to see it.
