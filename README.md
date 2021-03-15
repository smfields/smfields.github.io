## About Me
I'm a 24-year-old software developer living and working in Madison, Wisconsin. I graduated from North Carolina State University in 2020 with a Bachelor of Science degree in Computer Science. Currently I work for Epic Systems, one of the largest healthcare software companies in the United States. My job involves full stack web development on an electronic medical record system, with a focus on tools used by physicians for inpatient encounters.

### Interests
I have a particular interest in software security. While working toward my degree I completed an elective track in security which included classes such as software and network security, common attacks and defenses, and cryptography. At Epic, I'm working with the security team on a project to integrate smart card-based authentication into a new web application. 

### Projects
Most of my personal projects are born from problems that I face in my life where I can't find good existing solutions, so I make them myself. These types of situations have arisen multiple times in my life ranging from the time when I created tools to detect plagiarism as a teaching assistant, to more recently when I decided to make a website that would allow friends to come together to play a game even when they're forced to be apart. When choosing which technologies to use to build these projects my approach is always to find the best tool for the job and learn how to use it rather than trying to force the technologies I already know into a role they weren't made for. Thanks to this approach I have been exposed to a wider range of technologies and tools than I might have otherwise used.


<div class="indent">

#### [Shuffle: Music Party Game](https://playshuffle.tv)

Shuffle is a web-based music party game where players compete against one another to identify a song that is playing. It's a race against the clock as players try to name the song and artist as quickly as possible. Shuffle integrates with popular music streaming platforms, such as Spotify, to allow users to play with their own playlists, or they can use a pre-built playlist based on a popular genre.

[Click here to learn more about how I made Shuffle.](Shuffle.md)

</div>


### Resume
<canvas id="resume-canvas"></canvas>

<script src="//mozilla.github.io/pdf.js/build/pdf.js"></script>
<script>
    // PDF Loading Script - https://mozilla.github.io/pdf.js/examples/index.html#interactive-examples
    var url = '/assets/img/2021Resume.pdf';

    var pdfjsLib = window['pdfjs-dist/build/pdf'];

    pdfjsLib.GlobalWorkerOptions.workerSrc = '//mozilla.github.io/pdf.js/build/pdf.worker.js';

    var loadingTask = pdfjsLib.getDocument(url);
    loadingTask.promise.then(function(pdf) {

        // Fetch the first page
        var pageNumber = 1;
        pdf.getPage(pageNumber).then(function(page) {

            var scale = 1.5;
            var viewport = page.getViewport({scale: scale});

            // Prepare canvas using PDF page dimensions
            var canvas = document.getElementById('resume-canvas');
            var context = canvas.getContext('2d');
            canvas.height = viewport.height;
            canvas.width = viewport.width;

            // Render PDF page into canvas context
            var renderContext = {
                canvasContext: context,
                viewport: viewport
            };

            page.render(renderContext);
        });
    }, function(error) {
        console.error(error);
    });  
</script>