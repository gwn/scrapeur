# scrapeur

Simple high-level declarative scraper.


## usage

    npm install -g scrapeur
    scrapeur config.js

config.js:

```javascript
module.exports = {
    url: 'https://loremipsum.com',

    parsers: {
        main: document =>
            document._('.menu-bar-list > li')
                .map(categoryEl => ({
                    title: categoryEl._('span')[0].textContent.trim(),
                    link: categoryEl._('a')[0].href,
                })),

        subCategories: document =>
            document._('.category-sidebar li a')
                .map(el => ({
                    title: el.textContent.trim().match(/^(.*) \(\d+\)$/)[1],
                    link: el.href,
                })),
    },

    links: {
        main: {link: 'subCategories'},
        subCategories: {link: 'subCategories'},
    },
}
```

**Note**: `document._(x)` is short for
`Array.from(document.querySelectorAll(x))`. Same with
`Node.prototype._`.

That was a slightly modified version of a real example. Here is
how `scrapeur` executes this:

1. Start with fetching the page pointed by the given `url`.
2. Parse the fetched page with `parsers.main`.
3. Recursively look for links called `link` (as declared by
   `links.main`) in the object returned by the parser.
4. Fetch the pages pointed by the found links, parse them using the
   parser declared by `links.main`, and inject the results into the
   objects with the related `link` keys.
5. Goto 3.

The output will look like:

```
[
    {
        "title": "lorem",
        "link": "http://loremipsum.com/lorem",
        "children": [
            {
                "title": "ipsum",
                "link": "http://loremipsum.com/ipsum",
                "children": [
                    {
                        "title": "dolor",
                        "link": "http://loremipsum.com/dolor",
                        "children": [],
                    },
                    ...
                ],
            },
            {
                ...
            },
        ],
    },
    {
        "title": "sit amet",
        "link": "http://loremipsum.com/sit-amet",
        "children": [
            ...
        ],
    },
    ...
]
```


## philosophy

In progress. Basically something about saving you from writing
your own request and following logic thanks to `scrapeur`s
declarative mini-DSL.


## api

Config object:

```javascript
{
    url: 'http://loremipsum.com',
    parsers: {
        main: document => ...,
        aux: document => ...,
    },
    links: {
        main: {
            link: {
                parser: 'aux',
                propName: 'children',
            },
            link2: 'aux', // shorthand
        },
    },
    limit: {
        fetch: 1000000,
        level: 1000000,
    },
}
```

- `url`: URL to start scraping.

- `parsers`: Map of parser functions that accept the document as
  their single argument and expected to return an array or an
  object. The parser that parses the `url` has to be named `main`.

- `links`: Links to look for and follow in each parser's payload.
  Links will be followed and parsed by the declared parser and the
  payload will be injected into the object containing the link.

- `limit`: Limiting options for development. Limiting by a maximum
  number of fetches or depth are supported.


## where is cheerio?

JSDOM is being used at the moment.

Cheerio is faster and leaner than JSDOM so support for it will be
added sooner or later.

The good thing about JSDOM though is that it's the standard DOM
API so if you already know how to work with it you don't have to
learn nor remember new stuff. Also, since it's a complete DOM
implementation, you can execute scripts and do everything you can
do in a real browser. This can be useful for example to toggle
initially hidden content, or some content generated according to
user actions.
