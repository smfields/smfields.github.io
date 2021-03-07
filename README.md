## About Me
I'm a 24 year old software developer living and working in Madison, Wisconsin. I graduated from North Carolina State University in 2020 with a Bachelor of Science degree in Computer Science. Right now I'm working for Epic Systems, one of the largest healthcare software companies in the United States. My job involves full stack web development on an electronic medical record system, with a focus on tools used by physicians for inpatient encounters.

### Interests
I have a particular interest in software security. While I was in school I completed a track in security as a part of receiving my degree, which included classes in software and network security, cryptography, common attacks and defenses, among other things. At Epic, I'm working with the security team on a project to integrate smart card login and authentication into a soon to be released web application. 

### Projects
Most of my personal projects are things that I think should exist, but don't. When choosing which technologies to use, my approach is to find the best tool for the job and learn how to use it rather than trying to force the technologies I already know into a role they weren't made for. This approach has exposed me to a large number of technologies and tools that I otherwise may never have tried.

*Click on a project to learn more about it.*

#### [Shuffle: Music Party Game](Shuffle.md)
Play here: [https://shuffle.smfields.net](https://shuffle.smfields.net)

Shuffle is a web-based music party game where players can either compete or work together to try and identify a song that is playing. Players are on the clock, and must try to name the song and the artist as quickly as possible. Shuffle integrates with popular music streaming platforms, such as Spotify, to allow users to play with their own playlists, or they can use one of the pre-built genre playlists.

### Resume
<canvas id="resume-canvas"></canvas>

<script src="//mozilla.github.io/pdf.js/build/pdf.js"></script>
<script>
    // PDF Loading Script - https://mozilla.github.io/pdf.js/examples/index.html#interactive-examples
    var url = '/assets/img/2020Resume.pdf';

    var pdfjsLib = window['pdfjs-dist/build/pdf'];

    pdfjsLib.GlobalWorkerOptions.workerSrc = '//mozilla.github.io/pdf.js/build/pdf.worker.js';

    var loadingTask = pdfjsLib.getDocument(url);
    loadingTask.promise.then(function(pdf) {
        console.log('PDF Loaded');

        // Fetch the first page
        var pageNumber = 1;
        pdf.getPage(pageNumber).then(function(page) {
            console.log('Page loaded');

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

            var renderTask = page.render(renderContext);
            renderTask.promise.then(function () {
                console.log('Page rendered');
            });
        });
    }, function(error) {
        console.error(error);
    });  
</script>