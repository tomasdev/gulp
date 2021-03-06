# Combining streams to handle errors #

By default, emitting an error on a stream will cause it to be thrown
unless it already has a listener attached to the `error` event. This
gets a bit tricky when you're working with longer pipelines of streams.

By using [lazypipe](https://github.com/OverZealous/lazypipe) you can
turn a series of streams into a single stream, meaning you
only need to listen to the `error` event in one place in your code.

Here's an example of using it in a gulpfile:

``` javascript
var lazypipe = require('lazypipe'),
    uglify = require('gulp-uglify'),
    gulp = require('gulp');

gulp.task('test', function() {
    // Once the partial pipeline is ready to use,
    // call the last result from .pipe() directly as a function
    var combined = lazypipe()
        .pipe(uglify)
        .pipe(gulp.dest, 'public/bootstrap')();

    gulp.src('bootstrap/js/*.js').pipe(combined);

    // any errors in the above streams
    // will get caught by this listener,
    // instead of being thrown:
    combined.on('error', function(err) {
        console.warn(err.message);
    });

    return combined;
});
```

You can use this technique in your gulp plugins and node modules too using
the [multipipe](http://npmjs.org/package/multipipe) module, and is a generally useful means of ensuring that errors aren't thrown from inaccessible streams
somewhere deep inside `./node_modules`.
