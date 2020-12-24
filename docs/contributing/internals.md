# Internals

Furo's internals contain an interesting mix of stupid-things-that-work and all kinds of bodges. Most of this project was built slowly over the span of a few weeks, as a part of an attempt to take a break from working on Python packaging stuff.

## Repository Layout

The repository layout is pretty standard for a Python project, with a few quirks due to the need for compiling Sass and JS files.

- `CODE_OF_CONDUCT.md`
- `LICENSE`
- `README.md`
- `.nox/` -- Generated by [nox](https://nox.readthedocs.io/).
- `dist/` -- Generated as part of the release process.
- `docs/` -- Sources for the documentation.
- `src/`
  - `furo/` -- actual source code for the package
    - `__init__.py` -- Handles interaction with Sphinx and some configuration.
    - `navigation.py` -- Generates the sidebar navigation HTML.
    - `sphinxext.py` -- Defines the internal-only `furo-demo` directive.
    - `assets/` -- contains Sass and JS source code.
    - `theme/` -- the actual folder that Sphinx looks inside.
      - `static/` -- contains compiles CSS and JS code.
      - everything else here -- the underlying HTML templates.
- `gulpfile.js` -- for [Gulp](https://gulpjs.com/).
- `noxfile.py` -- for [nox](https://nox.readthedocs.io/).
- `package.json` -- for [NPM](https://npmjs.com/).
- `pyproject.toml` -- for Python Packaging.

## Theme build process

Furo's build process uses Gulp. Running `gulp build` in the repository root will compile the theme's CSS and JS assets (`src/furo/assets/`) into the correct final files (inside `src/furo/theme/static`).

When building the distributions for upload, `gulp build` is run once and the `src/furo/assets/` directory is excluded for the final distribution. Thus, _both_ the source distribution and wheel distribution do not contain the original source code for Furo and only contain the compiled SCSS and JS files.

```{note}
It is not ideal that the version-controlled source tree is not installable using pip directly. There is a need for a `gulp build` command to be run between the clone and installation.

Things are set up this way due to the lack of a "build" step support in Flit. There is [an open issue for enhancement with a proposal awaiting feedback](https://github.com/takluyver/flit/issues/119#issuecomment-687779285).
```

## How stuff works

### Sidebar Navigation Tree

This is was really tricky to get right. It is essentially a pure-CSS port of the Just the Docs theme's sidebar navigation.

Each list item has a fixed height and a different color on hover. Items with a subtree underneath have a hidden checkbox, that is triggered when clicking the item. The checkbox holds the state of "expanded vs collapsed", which is used in the CSS to show the subtree appropriately. The checkbox is automatically checked if it is for the current page, or a parent of the current page.

There was a painful amount of tweaking the CSS to get the subtrees to look just right even when there's long text and lots of content/subtrees etc. It works now, and that's all I care about.

### Contents sidebar

This uses the `toc` variable that's provided by Sphinx. There's some CSS to hide the top-level bullet referring to this page's title. The rest of the tree is stylised without much complexity.

The fancy bit is that Gumshoe.js is used to highlight the currently active heading (which heading is the content at the top pixel under). This is combined with some custom JS to scroll to the currently active heading in the toctree, which results in the contents sidebar scrolling with the user as they go.

### CSS variables for customisation

This is pretty much the "USP" of this theme. In `src/furo/theme/partials/_head_css_variables.html`, the user provided CSS variables are translated and written into each page's HTML. This is set up, such that these declarations overrides any other declarations made in the CSS of the theme.

This essentially allows the user to control the values of the CSS variables, and hence control how the theme looks.

### `furo-demo` directive

This directive was written to make it easier to write the examples used in the [Reference](../reference/index) section. The way it works is pretty straightforward really:

- It only works in MyST documents, since it performs an in-place substitution.
- It takes the contents of the block, splits it at "+++" into Markdown and reStructuredText snippets.
- For both of these, it renders a tab that has a code block containing the snippet followed by the actual code itself.
  - This is carefully crafted to ensure that things are evaluated correctly.

This approach has significant limitations however, since the A/B comparision format means that it is not directly usable to showcase functionality that is different between the two.

For that, we keep one of the snippets empty, which thanks to a conditional, results in the tab for that language (MyST or reST) not being rendered.