Dns resolution is a process that the browser has to undertake to convert a
domain/hostname to an ip address required to access a resource (this process is 
what converts a user friendly url like: http:
//[www.medium.com][1] to <http://80.72.139.101> ); this requires a certain time
and adds to the page loading process. An average dns loookup takes an average of
‘60-120ms’ followed by a full roundtrip to perform the TCP handshake which can 
add up to ‘100-200ms’ just to send a request for a file. Slow mobile experiences
are in part due to much longer full round trips ‘200-1000ms
’.

DNS-prefetch is a safe, backward-compatible HTML5 feature that can help with
perceived performance. This is particularly true on pages that load dynamic 
content from several domains.

DNS preresolution/prefetching is the process of figuring out the IP address of
every resource on the page before the browser makes it’s request, with the goal 
to save the DNS resolution time when the resource is requested.

Inspecting the source of amazon.com you’ll find the following code right at
the top of their homepage:

    **<meta http-equiv='x-dns-prefetch-control' content='on'>**  
    <link rel='**dns-prefetch'** href='http://g-ecx.images-amazon.com'>  
    <link rel='**dns-prefetch'** href='http://z-ecx.images-amazon.com'>  
    <link rel='**dns-prefetch'** href='http://ecx.images-amazon.com'>  
    <link rel='**dns-prefetch'** href='http://completion.amazon.com'>  
    <link rel='**dns-prefetch'** href='http://fls-na.amazon.com'>

Amazon.com uses DNS-prefetch to resolve **multiple domain names, **from which
different resources such as images, javascript, and css are accessed. When the 
browser encounters these URLs, it first checks it’s cache, and then, lacking a 
cached copy, resolves the domain to the associated IP address through a request 
from the DNS server. These requests happen in the background in a way that doesn
’t block the rendering of the page.

As we’ve seen above, chrome in some cases makes this automatically, but even
in those cases we can help him do it’s job by telling him what to look for, by 
giving him insights that are specific to our website.

DNS lookups are very low cost — they only send a few hundred bytes over
the network, so there’s not a lot of risk.

By having a look under the hood through some websites i’ve found out that 
[net-a-porter][2], [airbnb][3] and [csswizardry][4] also use this.

 [1]: http://www.medium.com
 [2]: http://www.net-a-porter.com/
 [3]: https://www.airbnb.com/
 [4]: http://csswizardry.com/