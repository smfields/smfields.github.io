## Shuffle
Shuffle is a web-based music party game where players can either compete or work together to try and identify a song that is playing. Players are on the clock, and must try to name the song and the artist as quickly as possible. Shuffle integrates with popular music streaming platforms, such as Spotify, to allow users to play with their own playlists, or they can use one of the pre-built genre playlists.

*Source code available upon request*

### Architecture

### Challenges
In this section I explore some of the aspects of designing the game that I found challenging or particularly interesting.

<details>
<summary>Fetching random songs from a playlist</summary>

#### Problem
The whole game relies on being able to select some random songs off of whatever playlist the user selects, and using those songs to play the game. The challenging part is that the songs on the playlist are paged, and so at no point do we ever have all of the songs in memory.

When designing the selection algorithm we have a few requirements:
- Pick songs as randomly as possible. Picking from a subset of all songs is not an option.
- Minimize the number of page requests we have to make. Most of the third-party APIs are rate limited, so we really need to make as few requests as possible. That's on top of the fact that minimizing requests is also going to improve performance.
- We can't assume that all the songs on the playlist will fit in memory at the same time. Therefore we need to keep the songs in pages, and only store the pages we need.

#### Solution
```csharp
    Math.random();
```

</details>

\
[Back](README.md)