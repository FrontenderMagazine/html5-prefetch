# HTML5 Prefetch
## Predict users actions and optimistically load resources ahead of time for better performance

Html5 provides a new and still relatively unknown feature that goes by the name of “prefetch”, which allows us to load resources ahead of time, to provide users with a fast and instant experience.

For modern websites, optimizing speed requires more than just minimizing the initial download size. Websites can be made faster by making the downloads size smaller, but additional speedups may be possible by starting downloads sooner.

“Prefetching” is simply loading a file before it’s needed, to provide a fast and instant experience. You can decide on which content to prefetch by analysing your users behaviour, and try to predict which resources they will need ahead of time.

## Browser cache

One might argue that “we already have browser cache!, we don’t need prefetch!”. But as Steve Souders points out in his great [prebrowsing][1] article, browser cache offers no help in many situations:

> **first visit** — The cache only comes into play on subsequent visits to a site. The first time you visit a site it hasn’t had time to cache any resources.

> **cleared** — The cache gets cleared more than you think. In addition to occasional clearing by the user, the cache can also be cleared by anti-virus software and browser bugs. ([19% of Chrome users][2] have their cache cleared at least once a week due to a bug.)

> **purged** — Since the cache is shared by every website the user visits, it’s possible for one website’s resources to get purged from the cache to make room for another’s.

> **expired** — [69% of resources don’t have any caching headers or are cacheable for less than one day][3]. If the user revisits these pages and the browser determines the resource is expired, an HTTP request is needed to check for updates. Even if the response indicates the cached resource is still valid, these network delays still make pages load more slowly, especially on mobile.

> **revved** — Even if the website’s resources are in the cache from a previous visit, the website might have changed and uses different resources.

## Chrome’s Predictor

Actually Chrome already tries to predict your behaviour based on historical browsing data, heuristics, and many other hints to anticipate your requests. As you open your browser in the morning, chrome knows which websites you usually go to and does ‘DNS-prefetching’ on their hostnames (more on this ahead), also when you start a search, as you’re typing something like “amaz”, chrome knows that you’ll probably go to amazon.com, and starts a ‘dns-prefetch’.

If the browser predictor algorithm is highly confident on his prediction, it may even trigger a full prerender of the page, a feature they named instant pages.

<iframe data-width="854" data-height="480" src="/media/143ac85fb3e19ac25e49dc91ab495d14?maxWidth=700" data-media-id="143ac85fb3e19ac25e49dc91ab495d14" frameborder="0" height="393" width="700"></iframe>

*[https://www.youtube.com/watch?feature=player_embedded&v=_Jn93FDx9oI][4]*

Chrome learns the topology of the web, as well as your own browsing patterns. If it does it’s job well, it can eliminate hundreds of milliseconds of latency from each navigation, and get the user closer to the holy grail of the “instant page load”.

**So if the browser does all of this for us what else can we do to acelerate it’s prediction and our websites performance:**

## Dns prefetching

Dns resolution is a process that the browser has to undertake to convert a domain/hostname to an ip address required to access a resource (this process is what converts a user friendly url like: [http://www.medium.com][5] to [http://80.72.139.101][6]); this requires a certain time and adds to the page loading process. An average dns loookup takes an average of ‘60-120ms’ followed by a full roundtrip to perform the TCP handshake which can add up to ‘100-200ms’ just to send a request for a file. Slow mobile experiences are in part due to much longer full round trips ‘200-1000ms’.

DNS-prefetch is a safe, backward-compatible HTML5 feature that can help with perceived performance. This is particularly true on pages that load dynamic content from several domains.

DNS preresolution/prefetching is the process of figuring out the IP address of every resource on the page before the browser makes it’s request, with the goal to save the DNS resolution time when the resource is requested.

Inspecting the source of amazon.com you’ll find the following code right at the top of their homepage:

    <meta http-equiv='x-dns-prefetch-control' content='on'>
    <link rel='dns-prefetch' href='http://g-ecx.images-amazon.com'>
    <link rel='dns-prefetch' href='http://z-ecx.images-amazon.com'>
    <link rel='dns-prefetch' href='http://ecx.images-amazon.com'>
    <link rel='dns-prefetch' href='http://completion.amazon.com'>
    <link rel='dns-prefetch' href='http://fls-na.amazon.com'>

Amazon.com uses DNS-prefetch to resolve **multiple domain names**, from which different resources such as images, javascript, and css are accessed. When the browser encounters these URLs, it first checks it’s cache, and then, lacking a cached copy, resolves the domain to the associated IP address through a request from the DNS server. These requests happen in the background in a way that doesn’t block the rendering of the page.

As we’ve seen above, chrome in some cases makes this automatically, but even in those cases we can help him do it’s job by telling him what to look for, by giving him insights that are specific to our website.

DNS lookups are very low cost — they only send a few hundred bytes over the network, so there’s not a lot of risk.

By having a look under the hood through some websites i’ve found out that [net-a-porter][7], [airbnb][8] and [csswizardry][9] also use this.

## Link Prefetch

From the [MDN][10]:

> Link prefetching is a browser mechanism, which utilizes browser idle time to download or prefetch documents that the user might visit in the near future. A web page provides a set of prefetching hints to the browser, and after the browser is finished loading the page, it begins silently prefetching specified documents and stores them in its cache. When the user visits one of the prefetched documents, it can be served up quickly out of the browser’s cache.

So this means that when we’re confident that a user will take a certain action, we can start downloading critical resources earlier to provide an instant experience.

On chrome we can specify the critical assets for each page, and what assets we want to prefetch for the next one:

    <link rel='subresource' href='critical.js'>
    <link rel='subresource' href='main.css'>

    <link rel='prefetch' href='secondary.js'>

The subresource ‘rel’ attribute identifies the critical resources for the current page load, **these subresources should be placed on top of the page as early as possible, and act as a manual helper for the chrome preload scanner**. The prefetch attribute tells the browser to start getting the secondary.js file when it has dealt with all the current page critical sub resources, the prefetch hints have the lowest possible priority.

The optimal time to initiate the resource fetch is dependent on the negotiated protocol, users current connectivity profile, available device resources, and other context specific variables. As a result, this decision is deferred to the user agent.

Only cacheable elements should be prefetched.

## Pre-rendering

This is a feature that currently only works on chrome which allows us to, instead of specifying a resource file for the browser to download, we tell the browser to download and render a full page, but to hide it from the user until the respective page is requested, this is the secret sauce behind the google instant pages feature.

**Pre-rendering is an advanced experimental feature, and mis-triggering it can lead to a degraded experience for your users**, including increased bandwidth usage, slower loading of other links, and slightly stale content. You should only consider triggering prerendering if you have high confidence on which page a user will visit next, and if you’re really providing added value to your users.

You can trigger prerendering by inserting a link element with a ‘rel’ of ‘prerender’, for example:

    <link rel='prerender' href='http://www.pagetoprerender.com'>

This feature works like loading the pre-rendered page in a new tab, just that this tab is hidden from the user until he makes a request for it, as the browser is executing all the scripts on the pre-rendered page this may lead to a degraded experience on the current page, register a page view, and make a request to an ad server for content that the user may never actually see.

When Chrome encounters the `<link>` element it will consider prerendering the linked page. This may happen when parsing the page, or later if the link element is dynamically inserted into the document via JavaScript.

## Inject prefetch hints at runtime

You can trigger the prefetch hints when you predict that a user action will require the download of additional content:

    var hint =document.createElement("link")
    hint.setAttribute(“rel”,”prerender”)
    hint.setAttribute(“href”,”next-page.html”)

    document.getElementsByTagName(“head”)[0].appendChild(hint)

## Browser Support

![Browser Support][Browser Support]

*Internet Explorer 9 supports DNS pre-fetching, but calls it prefetch. In Internet Explorer 10+, dns-prefetch and prefetch are equivalent, resulting in a DNS pre-fetch in both cases (From [Ilya Grigorik][11] book: [High performance browser networking][12]).*

## How To Test For Prefetch

There is currently no way to test for prefetch or pre-rendering, and having chrome devtools open will cancel the prefetch or prerender requests, to see if the assets you’re requesting are being downloaded, the only alternative i’ve found is to check the browsers cache, in chrome you can go to **‘chrome://cache/’** and in firefox to **‘about:cache’** and search for the files you are trying to prefetch. Chrome allows you to see if a page is being prerendered at **‘ chrome://net-internals/#prerender’**.

Google has a list for situations where prerendering is aborted:

* The URL initiates a download
* HTMLAudio or Video in the page
* POST, PUT, and DELETE XMLHTTPRequests
* HTTP Authentication
* HTTPS pages
* Pages that trigger the malware warning
* Popup/window creation
* Detection of high resource utilization
* Developer Tools is open

*By this time i couldn’t yet confirm if prefetch also gets cancelled in the same situations*

## What’s Coming

On the resource hints editor’s draft from 10 july 2014, you can see a detailed analysis of how each one of this elements works, and you can check that, dns prefetch will be called “preconnect”, and that there’s a new element coming, “preload”, that is used to indicate resources that should be fetched as early as possible.

[https://igrigorik.github.io/resource-hints/][13]

## Analyze your architecture, study users actions and iterate

Make an analysis of your architecture and figure out where you could take the most out of prefetch: which are the most critical resources on each page?, which actions trigger the download of additional content?, which content is on the critical rendering path?

Study your users actions on your website: which are the most visited pages?, what’s the critical journey your users take to conversion?, what’s the frequency of a certain action?

You can figure out most of this by looking through analytics tools, and by watching users actually using your website.

**Always use tools like [webpagetest][14] to get speed measurements before and after your changes.**

When logging in a web app why not start getting the next page?

When making a search why not start prefetching critical resources for the results page?

In a checkout flow why not start getting the next page resources while the user is filling out the form?

**Always compare how much user time you save vs how much extra bandwidth you use**

> With great power comes great responsibility

[1]: http://www.stevesouders.com/blog/2013/11/07/prebrowsing/
[2]: https://plus.google.com/+WilliamChanPanda/posts/hsfVHq6wKxG
[3]: http://httparchive.org/interesting.php#caching
[4]: https://www.youtube.com/watch?feature=player_embedded&v=_Jn93FDx9oI
[5]: https://medium.com/
[6]: http://www.smashingmagazine.com/
[7]: http://www.net-a-porter.com/
[8]: https://www.airbnb.com/
[9]: http://csswizardry.com/
[10]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Link_prefetching_FAQ
[11]: https://www.igvita.com/
[12]: http://www.amazon.co.uk/High-Performance-Browser-Networking-performance/dp/1449344763
[13]: https://igrigorik.github.io/resource-hints/
[14]: http://www.webpagetest.org/

[Browser Support]: img/Fp1Q5A.png