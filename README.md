ios-safari-nested-iframe-scroll
===============================

An example of nested iframes and scrolling specifically to fix scrolling issues in iOS safari.

This may be wrong, I'm still working on this investigation. I am trying to be right, and if you
want to help me be right, make a pull request or prove to me I should remove the repo.

## Why on earth?
Scrolling an iframe in iOS Safari has a
[commonly known workaround](https://davidwalsh.name/scroll-iframes-ios) which works fine.
Unfortunately this stops working when you find yourself nested in multiple iframe layers.

This is clearly not good application design, but that is not the point of this example. At the
time this application was created, it was probably a great idea (originally used framesets btw).

### An assumption
This example currently has no example of dynamic content, since the backend for the app I am trying
to fix creates the html on the server.

## What is in this repo?
A series of phtml files! Why?
Because the ipad seems to aggressively cache iframe content and I got bored trying to clear that
cache. Even with it plugged into a mac and specifically setting empty cache/ignore cache etc, it
loaded the cached iframe content. I could change the index page content and the iframe content
and only the index page updated.

I have wrapped the iframes in divs because I didn't want to assume the iframe would be the only
thing in the body.

I have nested content 4 iframes down.

## How does it work?
### tl;dr
You must wrap an iframe in a div or similar and set `overflow: auto` and
`-webkit-overflow-scrolling: touch` on that container. This needs to be set on every container
down through your nested iframes, all the way to the container that contains the iframe that loads
the content you wish to scroll.

Weirdly, you don't need to set it on the top level container! Boom. Have **that** in the face.

### You want to read more, OK here is some more

#### iframe auto sizing
In a standards based (real) browser, an iframe body scrolls. In the joke that is iOS safari an
iframe will automatically calculate it's height based on the content loaded inside it.
This will make your body/container height change and possibly (definitely) fuck up the layout of
your application.

Because of the wide use of iPads now, and the fact that Apple have forced all apps to use Safari
under the hood, assholes, we now have to cater for their shit poor browser which is, relatively,
worse than developing for IE6 when Chrome and Firefox were steaming ahead. I remember. I was there.

I will leave you to speculate why Apple would want to contribute to making web application
development suck. No I won't. It's because they are greedy and want you to make an iPad app instead.

Anyway, this poor behaviour from Apple means that the parent element must take over the scrolling,
since your iframe will now be behaving like a div.

Not only this, but any nested iframes will require that pattern to be followed all the way through
the nesting.

```
top
|_div <- note, you don't seem to need it here
  |_iframe
    |_window
      |_div `overflow: auto; -webkit-overflow-scrolling: touch;`
        |_iframe
          |_window
            |_div `overflow: auto; -webkit-overflow-scrolling: touch;`
              |_iframe
                |_window <- you want scrolling here
```

#### nested iframe scrollbars
If you set an iframe to be 100% height of the body, you will see scroll bars.

An iframe is by default an inline element. I cannot think of a single reason why except because of
the [HTML4 spec](https://www.w3.org/TR/CSS21/visudet.html#inline-width).

It is an inline replaced element though, so like an img tag and unlike a span, it can have a
[width](https://www.w3.org/TR/CSS21/visudet.html#inline-replaced-width) and
[height](https://www.w3.org/TR/CSS21/visudet.html#inline-replaced-height) set on it.
This has caught me out so often.

Unfortunately this means it has a line, 4px in Chrome. This means we get a whole stack of
scrollbars, all trying to allow the 4px. If you set `display: block` on the iframe the scroll bars
will disappear. You could set `overflow: hidden` on the top body, however this doesn't seem to work
in the nested iframes.

If an iframe contains (an iframe contains an iframe) that needs to scroll, setting block on any of
the nested 'container' iframes will stop the the scroll working. Just try setting it on
either nested-iframe{1,2}.phtml layouts. Again, just like overflow rules, this doesn't seem to apply
to the index layout (the top).

So, we can't use `display: block`, `overflow: hidden` or `margin-bottom: -4px`.

`line-height: 0` on the container doesn't work because of a minimum line height so that's no use.

`font-size: 0` on the container works for the scroll bars but breaks the scrolling.

The fix we found in the end was to set `font-size: 1px` on the iframe containers.
