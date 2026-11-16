# Nihal's website

A Hugo site published at https://nihalchoudhary.in/.

## Local development

Install Hugo **0.170.0** and the standalone Dart Sass **1.108.2** executable,
and ensure both `hugo` and `sass` are on your `PATH`.
Use the standalone Dart Sass distribution, whose executable supports Hugo's
embedded compilation protocol.

```bash
git submodule update --init --recursive
hugo server
```

Build the production site:

```bash
HUGO_ENVIRONMENT=production hugo --gc --minify --cleanDestinationDir
```

The `public/` output, `resources/` cache, and `.hugo_build.lock` are generated
locally and excluded from Git.

## Theme compatibility

The hello-friend-ng theme is pinned as a submodule. Site-owned templates in
`layouts/` override its Sass compilation and RSS locale handling for current
Hugo releases. The theme still emits Sass deprecation warnings for legacy
`@import` syntax and an invalid leading-combinator selector. Dart Sass 1.x
omits the invalid selector; migrate these upstream styles before upgrading
to Sass 2 or later, and replace imports with Sass modules before Sass 3.

## Continuous integration

Pull requests targeting `main` build the site without deploying it.
Pushes to `main`, and manual runs on `main`, build and deploy to GitHub Pages.
Dependabot checks GitHub Actions versions weekly. Hugo and Dart Sass versions
are pinned in `.github/workflows/hugo.yaml`; update them together with the
local requirements above.
