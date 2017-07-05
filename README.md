[Somisspo](https://twitter.com/somisspo)
============================================

<a href="url"><img src="preview.png" height="500"></a>

This bot generates a random San Francisco neighborhood name, and then maps out what that neighborhood could potentially look like. It also generates a house listing, with a random `adjective` `room` and the house costing some `amount of money`, and adds a little house icon somewhere in the neighborhood.

The primary motivation was to take the concept of creating new neighborhoods to their absurd extreme. See [99% Invisible](99percentinvisible.org/episode/the-soho-effect/) for more on this.

the process
===========

I had originally just planned on generating the names of a random neighborhood, but I soon realized that a map would convey the point so much better and would be well worth the effort. The final part of the process was to generate a good caption that mimicked housing listings.

I have the bot hosted on Glitch, and it posts to Twitter every day. If you want the actual code, it's not hosted on Github, but talk to me and I'll send you a link to Glitch. 

You can see it in action at [https://twitter.com/somisspo](https://twitter.com/somisspo).

part a (the NAME)
=================

The entire process kicks off with a generated neighborhood name, which is a combination of two existing neighborhoods. This neighborhood has to be plausible, so its two parents have to be close to each other. To get this relationship, I created a list of all SF neighborhoods, grabbed their centroid latitude/longitudes using the Google Maps API, and then created a dictionary where each neighborhood had associated with it other neighborhoods that were less than a distance of `n` miles away. 

I used the cartesian distance, simply because SF is small enough and this project rough enough that it wouldn't matter -- but it's always important to know that the [haversine distance](https://en.wikipedia.org/wiki/Haversine_formula) is what I really wanted.

The algorithm uses that dictionary to pick a neighborhood at random, and then one of its neighbors. Once I have these two neighborhoods, I split the names into syllables, and then take a combination of those syllables, capitalizing as necessary, to get the final output name. To ensure quality though I use some hard-coded rules: 
1) I take out some words, like `Valley` (see: `Hayes Valley`)
2) I manually split some words, such as `Marina` -> `Mar` + `ina` instead of `Ma` + `ri` + `na`
3) I make sure to remember if the syllable I take is at the beginning of the name, for capitalization reasons: that way it's `Noeres` and not `NoeRes` for `Dolores` + `Noe`, but it's still `DoloNoe` instead of `Dolonoe` (though, I guess it works in this case... we want `SoHo` not `Soho`)

part b (the MAP)
================

The [Google Maps API](https://developers.google.com/maps/documentation/) was invaluable in generating the map of the neighborhood. At this stage I already had the name of the neighborhood, and I also had its two parents. For each parent I grabbed its coordinates (using the API), then calculated the western-most, eastern-most, etc. points, then grabbed the midpoint of each of these, and ended up with a rectangle of eight points. I input these points into the `snaptoroads` endpoint, which returned the roads that were closest to the points. This resulted in a more believable neighborhood bound by streets, instead of by straight lines.

I also plotted a tiny house marker a random, small, distance away from the midpoint of the parent neighborhoods (effectively the centroid of the generated neighborhood).

The Maps API also provides a static map endpoint, where you pass in all your map parameters (a URL with the neighborhood boundaries, house coordinates, and styles) and it will return an image. It's then easy to upload and post that image to Twitter.


part c (the CAPTION)
====================

The caption is my least favorite part of the project, and I would want to re-do this part of the process the most. It simply takes a sentence template and plugs in a random relevant word from a corpus (I used [corpora](https://github.com/dariusk/corpora)). This results in a mediocre random-ish house listing. I also calculate a random asking price, somewhere between 600-1600k dollars. For some of these, I also include a randomized date for the (fake) open house. 

> We have a lifechanging #Financon home with a beautiful computer lab for sale at $1,530k.

> There's a new home on the market for $1,040k with a powerful clean room in the up-and-coming #TeleNob neighborhood!

The corpus obviously wasn't meant for house listings, so if more work was to be done I'd try harder to make this caption more realistic, with more specific words. Maybe there's also an opportunity to bring in Craigslist posts?