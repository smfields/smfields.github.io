## Shuffle
Shuffle is a web-based music party game where players can either compete or work together to try and identify a song that is playing. Players are on the clock, and must try to name the song and the artist as quickly as possible. Shuffle integrates with popular music streaming platforms, such as Spotify, to allow users to play with their own playlists, or they can use one of the pre-built genre playlists.

*Source code available upon request*

### Architecture
In this section I outline the architecture for the web application. Click on a section to read more.

<details>
<summary>Client</summary>


</details>

<details>
<summary>Web Server</summary>


</details>

<details>
<summary>Game Server</summary>


</details>

<details>
<summary>Database</summary>


</details>

<details>
<summary>Deployment</summary>


</details>


### Challenges
In this section I explore some of the aspects of designing the game that I found challenging or particularly interesting. Click on a challenge to read more.

<details>
<summary>Fetching random songs from a playlist</summary>

#### Problem
The whole game relies on being able to choose some random songs off of whatever playlist the user selects, and using those songs to play the game. The challenging part is that the songs are returned from the music platform APIs in pages.

When designing the selection algorithm we have a few requirements:
- Pick songs as randomly as possible. Picking from a subset of all songs is not an option.
- Minimize the number of page requests we have to make. Most of the third-party APIs are rate limited, so we really need to make as few requests as possible. That's on top of the fact that minimizing requests is also going to improve performance on our end.
- We can't assume that all the songs on the playlist will fit in memory at the same time. Therefore we need to keep the songs in pages, and only store the pages we need.

#### Solution
Below is an outline of the algorithm I designed to meet the above requirements, along with some code samples:
1. To start off, we allow the caller to specify the playlist from which they want to select the songs, and the number of songs they want to retrieve.
2. From the playlist provided by the caller we already have some basic information, such as the total number of songs on the playlist.
3. Lets generate some random numbers to represent the indices of the songs we will eventually fetch. These random numbers must be unique, and they must be within the range of possible song indices for the playlist.
   <details>
   <summary>Code Sample</summary>

    ```csharp
    /// <summary>
    /// Generates <paramref name="count"/> number of random integers between <paramref name="min"/> and <paramref name="max"/>.
    /// </summary>
    /// <param name="min">The minimum integer value that can be generated (inclusive).</param>
    /// <param name="max">The maximum integer value that can be generated (exclusive).</param>
    /// <param name="count">The number of integers to generate.</param>
    /// <returns></returns>
    private List<int> generateUniqueRandomIntegers(int min, int max, int count)
    {
        #region Parameter validation
        Require.True(max > min, "Invalid integer range. Max must be greater than min.");
        Require.True(max - min >= count, "Possible range of integer values is less than the number of integers being generated");
        Require.True(count > 0, "Count requested must be at least 1.");
        #endregion

        int numberRange = max - min; // Range of the possible numbers we can generate
        if (numberRange == count)
        {
            // User has requested every possible number in the range
            return Enumerable.Range(min, count).ToList();
        }

        Random rand = new Random();
        HashSet<int> randomIntegers = new HashSet<int>(); // Collection of the random numbers selected

        if (count > (numberRange / 2))
        {
            // User has requested more than half of the possible range.
            // We can invert the operation, and instead generate the numbers to exclude.
            // Start by adding every number in the range to the generated set
            randomIntegers.UnionWith(Enumerable.Range(min, numberRange));

            // Then remove random numbers until only count numbers remain
            int countToRemove = numberRange - count;
            List<int> intsToRemove = generateUniqueRandomIntegers(min, max, countToRemove);

            foreach (int num in intsToRemove)
            {
                randomIntegers.Remove(num);
            }
        }
        else
        {
            // User has requested less than half of the possible range.
            // Generate random numbers until we reach the correct count.
            while (randomIntegers.Count < count)
            {
                int randomInt = rand.Next(min, max);

                // Ignore duplicates
                if (!randomIntegers.Contains(randomInt))
                {
                    randomIntegers.Add(randomInt);
                }
            }
        }

        return randomIntegers.ToList();
    }
    ```

   </details>
4. Now that we know which indices we're going to want to fetch, we can group them together to form the actual page requests. The purpose of the grouping is to minimize the number of page requests needed to fetch every index.
    <details>
    <summary>Code Sample</summary>

    ```csharp
    /// <summary>
    /// Organizes a list of indices into the smallest number of groups possible while ensuring that the maximum
    /// range of each group does not exceed <paramref name="maxGroupRange"/>. The total number of groups created
    /// can span from 1 all the way up to the count of indices provided.
    /// </summary>
    /// <param name="indices">The list of indices to group.</param>
    /// <param name="maxGroupRange">
    /// The maximum range that the indices of any group may span. 
    /// The range of a group is the value of the highest index minus the value of the smallest index.
    /// </param>
    /// <returns>A list of index groupings</returns>
    private List<List<int>> groupIndices(List<int> indices, int maxGroupRange = 100)
    {
        #region Parameter Validation
        Require.CollectionNotEmpty(indices);
        #endregion

        // Stores the groups of indices
        List<List<int>> groups = new List<List<int>>();

        if (indices.Count == 1)
        {
            // By default a single element is already grouped
            groups.Add(indices);
            return groups;
        }

        // Put the indices in sorted order before grouping
        indices.Sort();

        // Create the first group and place the first index element into that group
        List<int> currentGroup = new List<int>()
        {
            indices[0]
        };
        groups.Add(currentGroup);

        // Iterate through the remaining indices, grouping them into groups where the range of the indices
        // does not exceed maxGroupRange.
        for (int i = 1; i < indices.Count; i++)
        {
            int currentIndex = indices[i];
            int currentGroupMinValue = currentGroup[0];
            if (currentIndex - currentGroupMinValue < maxGroupRange)
            {
                // Element can fit in the current group
                currentGroup.Add(currentIndex);
            }
            else
            {
                // Element can't fit in the current group. We need to start a new group.
                currentGroup = new List<int>()
                {
                    currentIndex
                };
                groups.Add(currentGroup);
            }
        }

        // Return all the index groupings
        return groups;
    }
    ```

    </details>
5. From there we can make the page requests. We request each page, and then we pick out the correct songs from the pages that are returned. 
    <details>
    <summary>Code Sample</summary>

    ```csharp
    /// <summary>
    /// Fetches songs from the playlist based on their indices. It is assumed
    /// that the group contains songs that can all be accessed in a single request.
    /// </summary>
    /// <param name="group">Indices representing the songs to fetch.</param>
    /// <returns>A List of <see cref="SpotifySong"/>s based on the indices in the group.</returns>
    private async Task<List<SpotifySong>> fetchSongGroup(List<int> group)
    {
        // We need the first page of songs to be able to request additional pages
        var firstSongPage = await GetSongs();

        // Calculate the correct offset and limit to use for this group
        int offset = calculateIndexGroupOffset(group);
        int limit = calculateIndexGroupLimit(group);

        // Create the request for a page containing all the songs in the group
        var groupSongPage = await firstSongPage.GetPage(offset, limit);

        // Pull the correct songs out of the page
        List<SpotifySong> groupSongs = new List<SpotifySong>(group.Count);
        foreach (int index in group)
        {
            var song = groupSongPage.Items[index - offset];
            groupSongs.Add(song);
        }

        // Return all the songs from the group
        return groupSongs;
    }
    ```

    </details>
6. Finally we randomize the order of the songs in our list since we previously sorted them all when we were grouping. The overall random song generation function is below:
    <details>
    <summary>Code Sample</summary>

    ```csharp
    /// <summary>
    /// Retrieves a number of random songs from the playlist. Note, it is assumed
    /// that the number of songs requested can be stored in memory rather than
    /// being returned as a paging object.
    /// </summary>
    /// <param name="count">The number of random songs to retrieve</param>
    /// <returns>A collection of random songs off the playlist</returns>
    public async ITask<IReadOnlyCollection<SpotifySong>> GetRandomSongs(int count)
    {
        #region Parameter Validation
        Require.True(count > 0, "Must request at least one random song.");
        #endregion

        // Retrieve the songs off the playlist
        var playlistSongs = await GetSongs();

        #region Playlist Songs Validation
        Require.True(playlistSongs.Total.HasValue, "Number of the songs in the playlist is unknown."); // Can't fetch random songs if we don't know how many songs the playlist has
        Require.True(count <= playlistSongs.Total.Value, "Number of random songs requested exceeds the number of songs on the playlist."); // Can't request more songs than there are on the playlist
        #endregion

        int numSongs = playlistSongs.Total.Value;

        // Pick some random songs based off random indices into the playlist
        List<int> randomSongIndices = generateUniqueRandomIntegers(0, numSongs, count);

        // Group the indices together to minimize the number of requests needed
        List<List<int>> indexGroups = groupIndices(randomSongIndices);

        // Fetch all the songs from each group
        List<SpotifySong> songs = new List<SpotifySong>();
        foreach (List<int> group in indexGroups)
        {
            var groupSongs = await fetchSongGroup(group);
            songs.AddRange(groupSongs);
        }

        // Return all the random songs requested in a random order
        Random r = new Random();
        return songs.OrderBy(song => r.Next()).ToList();
    }
    ```

    </details>

</details>

<details>
<summary>Checking the player's guesses</summary>

</details>


[Back](README.md)