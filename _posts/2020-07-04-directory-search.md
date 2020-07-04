---
layout: post
title: Efficient Time Interval Search and Application in Database Directory Splicing
tags: [databases, data management, efficiency]
---

## Introduction

## Storage
3.1.2.1	Slice Storage
See Figure 3 which shows the example slice layout that has now been modeled in the slice storage structure at the bottom.  Each endpoint of a slice marks a new slot or bucket in the storage array.  Each slot then holds pointers to all slices overlapping its time boundaries.  A given time t overlaps the slot at position p if t >= times[p] and t < times[p+1].  End times are exclusive so that any time overlaps one and only one time slot.  The end time of the last slot is defined as the current time.
An alternative approach would be to discretize time into small chunks and then use an array or hash map, so that lookup of a time would be in constant-time.  However, this has the downside that the amount of memory used is greatly increased, as well as the time needed to join all overlapping slices for a time range query.



Slice Lookup Algorithm
This section describes the main algorithm for determining the set of slices to be queried by a given database query at any point in time.
Time-Interval Binary Search
The main query need is to be able to specify either a time or a range of times, and receive all slices that cover those times.  This is similar to the interval stabbing problem mentioned in research literature (Schmidt, 2009), which involves finding all numerical intervals in a set that a given value “stabs” (is contained by).  Ideally the query would be constant-time, but this is difficult given that time is a continuous variable.  As discussed in 3.1.2, discretizing time into smaller chunks to make it hash-able has multiple disadvantages.  Given the relatively low number of slices in the typical case, a simple linear search algorithm could be considered, but it’s important to lower the query latency as much as possible and so this is undesirable.  Most of the accepted solutions in the literature involve balanced binary (interval) trees or skip lists (Hanson & Johnson, 1992).  A binary tree or skip list is not appropriate for this application because the tree traversal time of O(lg n) is not fast enough.  Furthermore, most additions to the structure will come at the back end as time moves forward and slice events occur, and binary trees provide poor performance when inserts are not randomly distributed.  Another alternative which is commonly used for data indexing in databases is a B-tree (Bayer & McCreight, 1970).  This balanced binary tree structure allows for packing many neighboring nodes into the same memory page, which reduces the number of page faults for queries using that index and thus is very performant in practice.  This application, though, is less concerned with reducing page faults than reducing time to find the matching slices, because the number of slices is likely to be low enough to fit within one or two pages.
Using the data structure outlined in 3.1.2, finding all slices overlapping a particular time can be accomplished with an interval-based binary search.  In this modified form of the search algorithm, query times are checked against the bounds of the candidate time range, then the candidate interval is halved to the left or right based on the comparison.  Since intervals cannot overlap and in fact are completely contiguous, the algorithm is guaranteed to return one and only one time-interval bucket.  This will lead to a time complexity of O(lg n) (Sedgewick, 1990), which is no better than the binary tree traversal.  
Optimization: Time Interpolation Search
However, this can be improved with the knowledge that time is always linearly increasing and new slices are created at regular intervals, and therefore the distribution of the bucket endpoints will very likely be roughly uniform.  Instead of choosing a candidate bucket in the exact middle of the remaining interval, a better estimate can be made by choosing the bucket that is proportionally located where the target time falls within the range.  As an analogy, this approach is akin to opening a dictionary towards the back when looking for a word starting with ‘w’ instead of in the middle, because it’s expected that words will be roughly evenly distributed throughout all letters and ‘w’ is near the end of the alphabet range.
For example, consider the set of buckets [10-30, 30-40, 40-65, 65-75, 75-90], and a target time of 70.  The traditional binary search approach would choose the midway point of the range, (40-65).  70 is greater than that range, so another iteration will be taken with the right half of the remaining range [65-75, 75-90], where it will then likely choose the correct range of (65-75).  In contrast, the time interpolation search algorithm chooses its candidates with the following function in Figure 5, which results in the 0-indexed array location to inspect.  Immediately, the interpolation search algorithm chooses the correct interval bucket that the target falls in.  It will be empirically shown below that this is not an uncommon occurrence.



Figure 5. Formula for Choosing Next Time-Interpolated Bucket
Pseudocode
In Figure 6, pseudocode for the intervalInterpolationSearch function described above is given.  End times of buckets are exclusive so that a time will overlap only one bucket.  Since the boundary conditions are checked up front in the function, and there are no gaps in time within the bucket structure, the function is guaranteed to find the one and only overlapping bucket for the given time.

int intervalInterpolationSearch(Time[] starts, Time search):
	# Check boundary conditions
if (starts.empty() || search < starts[0])
		return -1
	if (search > starts[starts.size()-1])
		return starts.size()-1
	lhs = 0, rhs = starts.size()-1, target = -1
	starts.add(Time.now())
while (lhs<rhs && search>=starts[lhs] && \
search<starts[rhs+1])
# Percent of total range where search falls
ratio=(search-starts[lhs])/(starts[rhs+1]-starts[lhs])
# Convert percentage to location within range
target=lhs+floor(ratio * (rhs-lhs))
if (search<starts[target])
	rhs=target-1 # Iterate with left group only
else if (search>=starts[target+1])
	lhs=target+1 # Iterate with right group only
else
	break # time is within the target bucket
	return target

Figure 6. Time-Interval Interpolation Search Pseudocode
Complexity Analysis
The complexity of interpolation search when the data is uniformly distributed has been shown to be O(lg lg n) (Sedgewick, 1990), which for the purpose of keeping track of database directories is near enough to constant-time.  This relationship can be seen empirically in Figure 7 where a sample directory distribution was taken from a real system and stored in the format described above.  Times in between the search boundaries were pulled from a pseudorandom uniform distribution, and the average number of loop iterations needed to get to a solution was plotted.  Number of iterations was used as an approximation for performance, because gathering reliable system timing numbers at such a small scale is difficult.  As the number of directories used increases, the scatterplot shows the number of iterations needed resembles an O(lg lg n) distribution, staying below 4 for a directory set of size 600.  With the default slice period of 12 hours, 600 directories would be created in 300 days.

Figure 7. Iterations to Solution Vs. Number of Directories
	The modified interpolation search was also compared to a typical binary search implementation by performing both algorithms on a directory set of size 100 and a given time chosen from a pseudorandom uniform distribution.  This was done for several iterations, and the average number added to a histogram, as per the Statistical Bootstrap Method of garnering statistics about a distribution via resampling (Chernick, 2007).  Figure 8 below shows that utilizing the prior knowledge to perform time-interpolation search vastly improves the number of iterations needed to reach a solution over binary search.  It not only seemingly halves the average, but also shifts the whole distribution from being heavily skewed to the right, to being more normal-shaped.


Figure 8. Number of Iterations to Solution for Time-Interpolation Vs. Binary Search
	Finally, a Monte Carlo simulation was performed by generating a pseudorandom set of slices of increasing size and searching for a pseudorandom value one million times.  The number of iterations needed to find the correct solution was averaged together across all tests.  This was done for both regular binary search and time-interpolation search.  Again, as shown in Figure 9, it can be seen that binary search increases in time at a faster rate than interpolation search, which closely tracks the reference lg(lg n)) plot.
