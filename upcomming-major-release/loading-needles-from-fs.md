# Loading image needles from filesystem for image recognition

Suggestion extracted from https://github.com/nut-tree/nut.js/pull/175.

Currently needles for image recognition are always red from the file system as @zephraph pointed out.  
If one would assume that a certain needle, identified by its path is invariant during execution, one could cache the images which would save some time for sure.

There is still a possibility that needles change over time if you e.g. generate the needles from a certain area of the screen to use them later on.  
Another issue could come up, if you have a machine with few RAM. Keeping a lot of images in memory could potentially exhaust the RAM capacity.  
To exceed RAM limits today, a lot of high resolution needles would be required. Nevertheless, this is a scenario to consider IMHO.  
From my perspective, those cases are rare anyways. Therefore I'd suggest to change the needle-management for the next major release as follows:

1. Assume a needle is identified by its path and invariant over time (default)
2. Cache those needles and reuse them in case they are accessed again
3. Add a property that makes it possible to reload the images over and over again as an opt-in.
