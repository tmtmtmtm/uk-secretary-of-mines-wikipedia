Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the
[Secretary for Mines](https://www.wikidata.org/wiki/Q7444267)
had very little information, and needed to be fleshed out substantially.

Step 2: Tracking page
=====================

I created a new PositionHolderHistory table. The initial version at
https://www.wikidata.org/w/index.php?title=Talk:Q7444267&oldid=1237692621
had one membership.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js) 
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 5: Scrape
==============

Comparison/source = [Secretary for Mines](https://en.wikipedia.org/wiki/Secretary_for_Mines)

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

Slight tweak for Partial dates from an odd layout, but otherwise simple.

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

11 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/635e498cd8e27

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

Nothing to add: the existing P39 already had all fields.

Step 8: Refresh the Tracking Page
=================================

New version at https://www.wikidata.org/w/index.php?title=Talk:Q7444267&oldid=1237700919
