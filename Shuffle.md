## Shuffle
Shuffle is a web-based music party game where players can either compete or work together to try and identify a song that is playing. Players are on the clock, and must try to name the song and the artist as quickly as possible. Shuffle integrates with popular music streaming platforms, such as Spotify, to allow users to play with their own playlists, or they can use one of the pre-built genre playlists.

*Source code available upon request*

### Architecture

### Challenges
In this section I explore some of the aspects of designing the game that I found challenging or particularly interesting.

<details>
<summary>Fetching random songs from a playlist</summary>
<p>

#### Problem
The whole game relies on being able to choose some random songs off of whatever playlist the user selects, and using those songs to play the game. The challenging part is that the songs are returned from the music platform APIs in pages.

When designing the selection algorithm we have a few requirements:
- Pick songs as randomly as possible. Picking from a subset of all songs is not an option.
- Minimize the number of page requests we have to make. Most of the third-party APIs are rate limited, so we really need to make as few requests as possible. That's on top of the fact that minimizing requests is also going to improve performance.
- We can't assume that all the songs on the playlist will fit in memory at the same time. Therefore we need to keep the songs in pages, and only store the pages we need.

#### Solution
Below are the steps of the solution that I designed to meet the above requirements, along with some code samples:
1. We allow the caller to specify the playlist off of which they want to select the songs, and the number of songs they want to retrieve.
2. From the playlist provided by the caller we already have some basic information, such as the total number of songs on the playlist.
3. Lets generate some random numbers to represent the indices of the songs we will eventually fetch.
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

</p>
</details>

\
[Back](README.md)