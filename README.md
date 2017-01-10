# Website Performance Optimization
Optimized an online portfolio for speed--in particular--optimized the critical rendering path and made this page render as quickly as possible by applying the techniques from the [Critical Rendering Path course](https://www.udacity.com/course/ud884).

####Installing packages

To install packages required for this project use the following:

  ```bash
  npm install grunt
  npm install
  ```

Install [ImageMagick](https://www.imagemagick.org/script/binary-releases.php) with the corresponding download

Once these items have been installed, you can run the following to build the project:

  ```bash
  grunt default
  ```

Your files will be located in the newly generated dist folder.

####Objective 1: Optimize PageSpeed Insights score for index.html

This score can be determined after first creating a simple Python server:

  ```bash
  cd /path/to/your-project-folder
  python -m SimpleHTTPServer 8080
  ```

Followed by hosting using [ngrok](https://ngrok.com/) to make the server accesible remotely.

  ``` bash
  cd /path/to/your-project-folder
  ngrok http 8080
  ```

Finally, the score can be determined using [Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/).

####Objective 2: Optimize Frames per Second in pizza.html

To optimize views/pizza.html, you will need to modify views/js/main.js until your frames per second rate is 60 fps or higher. You will find instructive comments in main.js.

##Modifications

####Modification 1: Minify JavaScript and load asynchronously

Minification can be easily performed using Grunt or other task runners.  Using [Grunt](http://gruntjs.com/) you are able to to minify the JS. I added "async" to the script tag in the HTML, which prevents the JS from render blocking, reducing the critical path length.

####Modification 2: Compress images

Loading large images can be a strain on the browser. Using Grunt with ImageMagick allows you to optimize the image for responsiveness and also replaced the filename in code with the optimzed images generated. 

####Modification 3: Selectively load CSS

The browser does not need to load CSS for every situation, rather, HTML can dictate when it is necessary for it to load. Using the media setting in the link tag allowed me to block a "print-only" CSS file from constantly loading.

  ```html
  <link "css/print.css" rel="stylesheet" media="print">
  ```

####Modification 4: Use a web font loader for custom fonts

Custom fonts, such as those found at [Google Fonts](https://www.google.com/fonts), can be optimized using a web font loading script.  I placed the code at the end of the browser, to increase the [PageSpeed Score](https://developers.google.com/speed/pagespeed/insights/):

  ```javascript
  <script>
    WebFontConfig = {
      google: {
        families: ['Open Sans:400,700']
      }
    };

    (function(w,g){w['GoogleAnalyticsObject']=g;w[g]=w[g]||function(){(w[g].q=w[g].q||[]).push(arguments)};w[g].l=1*new Date();})(window,'ga');

    // Optional TODO: replace with your Google Analytics profile ID.
    ga('create', 'UA-XXXX-Y');
    ga('send', 'pageview');

     (function(d) {
        var wf = d.createElement('script'), s = d.scripts[0];
        wf.src = 'https://ajax.googleapis.com/ajax/libs/webfont/1.5.18/webfont.js';
        s.parentNode.insertBefore(wf, s);
     })(document);
  </script>
  ```

More information can be found at the [Web Font Loader Github Repo](https://github.com/typekit/webfontloader).

####Modification 5: Inline CSS in production code

While inlining CSS is bad practice for source code, it can be safely inlined in production code to improve performance. I used the Grunt to optimize code after implementing the inlined CSS code.

####Modification 6: Altering the pizza.html page

The main.js code provides the page with moving pizzas which has trouble rendering at 60 fps. This is because of how the loops are constructed, they require the DOM to be fetched at each iteration.

Additionally, the "querySelectorAll" does not provide as narrow of a selector as "getElementsByClassName", which increases time the browser requires to find the DOM element. For example, the unaltered code followed by the altered code from the for-loop.

######Before:

  ```javascript
  var items = document.querySelectorAll('.mover');
  for (var i = 0; i < items.length; i++) {
    var phase = Math.sin((document.body.scrollTop / 1250) + (i % 5));
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }
  ```

######After:

  ```javascript
  var items = document.getElementsByClassName("mover");

  // Caching variables to improve performance
  var scrollTopLookup = document.body.scrollTop;
  var constArray = [];

  // Optimized code that takes less effort to go through the phases
  // and calculate the scrollTopLookup variable
  for (i = 0; i < 5; i++) {
    constArray.push(Math.sin((scrollTopLookup / 1250) + i));
  }

  // Utilizes the previous for-loop's ability to generate values for
  // the phases to reduce the need to lookup values
  for (i = 0; i < items.length; i++) {
    var phase = constArray[i % 5];
    items[i].style.left = items[i].basicLeft + 100 * phase + 'px';
  }
  ```

The [utilized code](https://discussions.udacity.com/t/project-4-how-do-i-optimize-the-background-pizzas-for-loop/36302) was provided by mcs in the discussion forum as a hint to help optimize the page. Thanks!

###About

This project was part of Udacity's [Front-End Web Developer Nanodegree](https://www.udacity.com/course/front-end-web-developer-nanodegree--nd001), with the goal to optimize a website above a PageSpeed of 90 and to acheive 60 fps.