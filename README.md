[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fbootc-dev%2Fbootc-dev.github.io.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fbootc-dev%2Fbootc-dev.github.io?ref=badge_shield)

# This is the main site!

This site is built using [Zola](https://www.getzola.org/), a static site generator built with Rust. The theme is [Juice](https://www.getzola.org/themes/juice/).
- Zola [Installation instructions](https://www.getzola.org/documentation/getting-started/installation/)
- Juice [Installation info](https://juice.huhu.io/#installation) - Note that I've explicitly dropped the git history and VCS info from the theme directory.

Under the hood, Zola uses [Tera](https://keats.github.io/tera/) for templating.

## Structure

The site lives in the `site` directory. Content like pages or blog posts live in, appropriately, `content`. Any kind of image like screenshots would live in `static`.

The `templates` directory extends elements from the theme, which is stored in, appropriately, `themes/juice`.

## Building

Once you've installed Zola, proceed as follows:

```bash
cd site
zola serve
```

Go to the URL for the webserver (likely `localhost:1111`) to explore the site.

You can also use `zola build` to build it once to the `public` directory if you so choose.

## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fbootc-dev%2Fbootc-dev.github.io.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fbootc-dev%2Fbootc-dev.github.io?ref=badge_large)
