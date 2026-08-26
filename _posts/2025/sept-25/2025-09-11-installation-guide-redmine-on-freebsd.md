---
layout: post
title:  "Wait Wait Stats: Segment Naming Updates"
date:   2025-09-01 10:45:00 -0700
tags:   stats waitwait update
---

I will be working on updating and aligning the segment names in show descriptions that I have entered in for each [Wait Wait... Don't Tell Me!](https://waitwait.npr.org/) show to match the naming convention used by NPR and the show.

In addition to aligning the segment names, I will be starting to re-listen to shows, starting with January 2006, in order to collect segments and updating them to match the show's naming convention, but also to update the segment delimiter from commas to semicolons.

I started using semicolons as segment delimiters while listening and recording data for the shows that aired in 1998 and 1998. The change in segment delimiters will make it easier to split the segments out for each show and possibly normalizing or changing how they are presented.

Previously, show descriptions would exclude segments like "Panel Round"/"Panel Questions" and "Lightning Fill In The Blank". Those will now be added in order for show descriptions to include all segments.

The following are some of the updates to the segment naming that I will be making:

* "Lightning Fill In The Blank" will be updated to be "Lightning Fill In The Blank"
* "Panel Questions" will be updated to "Panel Round" where the segment name is called "Panel Round"
  * More recent shows use "Panel Questions" instead of "Panel Round". For those shows, the segment will be left as "Panel Questions"
  * For earlier shows where segments that follow the same format as the current "Panel Round" or "Panel Questions" segment were labeled as "Week in Review" will be listed as "Week in Review" to reflect NPR's naming convention
  * If the first "Panel Round" is listed as "Opening Round" or "Opening Panel Round" on the NPR show page, the segment name will be updated accordingly
* Standardize on using "#2", "#3", etc. for multiple instances of the same segment within a specific show
* Standardize on the term "segment" instead of "round" for show segments
  * Note that the phrase "Lightning Round" is commonly used on the show to refer to the "Lightning Fill In The Blank" segment.
  * While the preferred name for the segment will be "Lightning Fill In The Blank" going forward, various parts of the Wait Wait Stats Project libraries and sites will still have references to "Lightning Round". Those code references will not be changed as that is considered a major API breaking change.
* Standardize on the term "round" for a panelist's round of questions during the Lightning Fill In The Blank segment
* Topics for Bluff the Listener, Not My Job and Panelist Predictions segments will now be closer or matching the topic listed on the NPR show page, except where more clarity or corrections are required

These changes will hopefully make the Wait Wait Stats data more accurate and more aligned with how the show and NPR present the segments.

Since the records will need to be manually updated and I will need to listen through each show, this will take some time to complete. I will also be pausing any work on the Wait Wait Stats Project sites with only updating components and dependencies that require security updates.
