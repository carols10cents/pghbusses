= Pittsburgh Busses

[Try it out!](http://pghbusses.github.io/p1.html)

I was excited to hear that the Port Authority of Allegheny County was doing a beta of a [real-time bus tracking service](http://www.pghcitypaper.com/Blogh/archives/2013/08/21/port-authority-launches-pilot-of-real-time-bus-tracker) but I was disappointed to see how tedious it was to use. This is my attempt at making an interface to help bus riders get the real time information they need faster.

I am not affiliated with PAT nor with the company who is creating the system, Clever Devices. I do not believe I am in violation of the [Terms of use](http://74.116.73.3/bustime/wireless/html/terms.jsp), neither in the letter of the terms (ex: I am not mirroring any content, simply linking to the real-time pages) nor the spirit of being a good net citizen (ex: I downloaded the information I needed a handful of times, slowly, as to not overwhelm their servers with anything close to a noticeable amount of load).

== How it's made

The following is a stream-of-consciousness of my thoughts, motivations, and processes while creating this prototype.

=== Initial impressions

[The City Paper article](http://www.pghcitypaper.com/Blogh/archives/2013/08/21/port-authority-launches-pilot-of-real-time-bus-tracker) explains at the end that to try out the real time service, you can go to the Port Authority's website and click on the "Real time link". When reading the online version, the words "Real time link" are a link directly to the real time service, saving you a click on PAT's website.

Your choices once you're there are presented as:

    * Text-only version (for text-readers and mobile devices)
    * Estimated Arrival Times interface
    * Bus Location Map
    * Plan trip with google maps

I mostly get bus information on my phone, so I haven't even looked at the other options-- I've only tried the text-only version.

Choosing that takes you to [http://74.116.73.3/bustime/wireless/html/home.jsp
](http://74.116.73.3/bustime/wireless/html/home.jsp) -- now, I understand this is a beta program, but they really couldn't afford/configure a domain name?? Or ideally, use portauthority.org/whatever??? Ugh, I could write a whole rant on how we should be able to trust information based on the domain name but can't because companies do stuff like this, but another time.

On this page, your choices are presented as:

    * Find by Stop # (how do I know what my stop # is?)
    * Choose your route (only choice: P1, which is in slightly larger text)
    * Sign in to Clever Devices Bus Time (oh geez i don't want another account)
    * Create a Clever Devices Bus Time account (please don't make me have an account)
    * What is Clever Devices Bus Time?
    * Questions or Comments?
    * Use Clever Devices Bus Time in the following languages: English
    * Terms of use

So if I choose the only route in the beta program, the P1, I then have to choose between 'inbound' and 'outbound'. Since the P1 makes a loop in the [downtown free fare zone](http://www.portauthority.org/paac/FaresPasses/Fares/Zones.aspx#FreeZone), I usually end up getting on at a stop that is technically "inbound" that's closer to my office, then staying on while the bus loops around to be "outbound" (but doesn't take a break or anything). Many, many people do this, and I would put money on the majority of bus riders not having a clue where the line between "inbound" and "outbound" is.

This distinction has always bothered me-- it's short for "inbound to downtown" and "outbound from downtown", but busses that don't go downtown at all still have a direction of their route that's called "inbound" and a direction that's called "outbound"!! How is someone brand new to the area supposed to figure this out on their own? This is an internal PAT implementation detail that is leaking out into the riders' world, but isn't actually useful to riders. I'd like to see if we can minimize this.

Anyway, moving on, I pick outbound for now, since I usually take the P1 after work to get to Negley or East Liberty. And I see this:

    Choose your stop (in alphabetical order):

WHYYYYYYYY ALPHABETICAL ORDER I don't know what my stop is called, exactly!! What's the closest stop to where I am? Who knows!! If they were in the order I hit them while riding the bus, I could tell!! This is not useful. It takes me a minute to read each stop in turn, think about where they are, and consider whether it's the stop I want or not. This is compounded by the fact that half the downtown stops are missing from this list because they're "inbound" (see above rant).

So finally I pick Smithfield at 6th and I can get real time bus information. Starting from the City Paper article, that was 5 clicks. Let's see if we can do better.

=== Making improvements

Luckily for us, this service provides a [hypermedia API](http://www.hypermediaapi.com/), otherwise known as the predictably-formatted content with links that we just navigated through!!

In the spirit of Lean Startup, I'm going to do the bare minimum amount of work to get to an MVP of my idea to get some feedback. So I don't actually have any code that uses this hypermedia API -- I just used `curl` to fetch what I wanted and I'm caching it on disk in files (not included in this repo so I don't get accused of mirroring content). This is a slightly more programmatic way to get this content but is essentially the same as visiting these sites in my browser and using "File > Save Page As...".

So I started by getting the home page, then I followed the links (manually, yes, I know) to the other pages, much as I did above when describing my thoughts on interacting with the site. You can replicate what I did by running:

    mkdir cache
    curl "http://74.116.73.3/bustime/wireless/html/home.jsp" -o cache/home.html
    curl "http://74.116.73.3/bustime/wireless/html/selectdirection.jsp?route=P1" -o cache/p1-select-direction.html
    curl "http://74.116.73.3/bustime/wireless/html/selectstop.jsp?route=P1&direction=INBOUND" -o cache/p1-inbound-select-stop.html
    curl "http://74.116.73.3/bustime/wireless/html/selectstop.jsp?route=P1&direction=OUTBOUND" -o cache/p1-outbound-select-stop.html

At this point, the content of the select stop pages contains the URLs of the pages I want to link to, so I don't need to actually fetch the content of those pages.

To fix the ordering issue and try to get rid of the inbound/outbound distinction, I basically copied the links to the stops from the inbound and outbound pages and then consolidated them into one list, ordered starting from Swissvale where the P1 starts and ending with the last stop in the downtown loop.

I decided to add a header for the stops that are on the busway and stops that are downtown to help scanability. I'm not sure if this is useful to most people or not.

For Swissvale and the stops in the downtown loop, there's no sense in asking people to pick inbound or outbound at all since there's only one direction the bus passes that stop. For the stops on the busway where you might want to go in either direction, AFTER choosing the stop you're asked if you want to go towards downtown/away from Swissvale or away from downtown/towards Swissvale.

At that point, we have enough information to be able to link you to the real time information page relevant to you.

So let's say there's a landing page for this real-time info with an easy to remember URL like whereismybus.portauthority.org (kidding, that's a bit long), and that you click on a link to the P1 page from there. For some of the stops, it'll be a total of 2 clicks to get to the real time info, for some it'll be 3, so let's call it 2.5 and we have just halved the number of clicks!

=== Next steps

* Handle more bus routes in a useful way
* Use browsers' geolocation feature to suggest stops nearby
* Script the caching and transformation of the data to be able to replicate this process should the URLs or IDs of the pages being linked to change
* Make it prettier and more useful on different device sizes
* When asking which direction you're going if that information is needed, I'd love to have an option to provide more detail because sometimes people might know the middles of the bus route but not the end, so have a third option that says "not sure" or something and that expands the other two options to list the next few stops in each direction, perhaps.
* Or maybe people would rather pick the stop where they're planning to get off? If they know the stops around their destination. Sometimes I get on the bus not knowing exactly where I'm going to get off though, just that the bus takes me in the general direction I'm trying to go. More thought needed.

=== Your ideas?

I'd love to hear what you think about this experiment. You can email me at carol.nichols@gmail.com or reach me on twitter @carols10cents. If you're comfortable using github, feel free to open issues or send pull requests!

=== License

MIT. See LICENSE.
