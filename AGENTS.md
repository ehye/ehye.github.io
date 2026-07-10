# Repository Guide

## Workflow

- Use pnpm: `pnpm install`, `pnpm run server` for the local Hexo server, and `pnpm run build` to generate the site.
- There are no lint or test scripts; run `pnpm run build` to verify site and configuration changes.
- `public/` is generated and ignored. Do not edit or commit it.
- GitHub Actions builds every PR to `main` and deploys only pushes to `main` as a GitHub Pages artifact. Do not use `pnpm run deploy` for the normal Pages deployment.

## Site Content

- This is a Hexo 8 site using the NexT theme. Site settings are in `_config.yml`; theme overrides belong in `_config.next.yml` rather than dependency files.
- Publish posts under `source/_posts/`; drafts in `source/_drafts/` are excluded because `render_drafts` is false.
- `post_asset_folder` is enabled: keep a post's local images/assets in the sibling directory named after its Markdown file (for example, `source/_posts/wled-ws2812b/`).
- New post scaffolds provide only `title`, `date`, and `tags`; add `categories` and any required front matter explicitly.
- Giscus maps comments by `og:title`; avoid renaming published post titles unless splitting their comment thread is acceptable.
- The configured site URL is `https://ehye.github.io` with root `/`; preserve root-relative internal asset links such as `/images/avatar.jpg`.
