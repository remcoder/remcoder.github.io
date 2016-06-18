---
published: true
title: Incremental Lua uploads for nodemcu
layout: post
tags: [nodemcu, esp8266, gulp, lua]
categories: [IoT]
---
## The problem

When writing Lua scripts for the nodemcu, I frequently need to upload new files. And since it's so convenient that the nodemcu has an actual filesytem the number of files tends to increase rather fast. So if you're like me, you'll have dozen or so scripts and everytime you make changes to a few of them you'll need to upload them again.

This is where it'll start to hurt. As you'll probably know, uploading those files is slooooww. It takes a few seconds to transfer for a script thats just a few lines long. And when you have multiple files that need to be updated in one go it can easily take 20 or 30 seconds. At which point you'll start to cringe.. I know I did.

So I went on to explore some options how to optimize the process. Note that everything here was done using [Andi Dittrich](https://github.com/AndiDittrich)'s handy dandy [nodemcu-tool](https://www.npmjs.com/package/nodemcu-tool).

## Faster, *faster*!
One of the most obvious things to do is to increase the bitrate at which data is transferred over the line.

I changed it to a whopping 115200 instead of the default 9600 by adding this line at the top of my `init.lua` file:

`uart.setup(0, 115200, 8, 0 , 1);`

*note:* Also make sure to change the baudrate at which nodemcu-tool operates.

Result: although this was a trivial method causing an overall speedup, the improvement is marginal. It shaves of only 10% or so, causing me to think that there's some time lost during the setup of the transfer or something.

## Less is more
Another no-brainer is to limit the number of bytes that are being sent across the wire. Less bytes means less time. And nodemcu-tool has a nice option to 'optimize' files before they get sent. It does this by removing comments and reducing whitespace, a simple version of a process know to web developers as minifying.

Result: again, this technique too manages to shave off roughly 10%. Which is nice but not... great. Maybe you'd say I need to write more comments in my code, and you'd be right! ..but.. that won't actually *speed up* the transfer now would it? :-P

A side-effect of this optimization is that when you get a run-time error, the line numbers are off. For this reason and because the speed-up was marginal I decided not to use this feature anymore.

## Gulp to the rescue

Actually, at this point I was thinking of Makefiles but since they've somewhat fallen out of fashion I went for Gulpfiles instead. Gulpfiles are the kind of script files you feed to a tool called [Gulp](http://gulpjs.com/), but they're written in Javascript really. And just like the Makefiles of yore, you can use 'm to write your source transformations.

So Gulp lets you write tasks which get executed whenever you enter the following on the command-line:

`gulp <my-task>`

Now instead of compiling the source files, we could just run some arbitrary shell command to, let's say, upload files to your nodemcu!

In code, it looks like this:

``` javascript
var SRC = 'src/**/*.lua';
var DEST = 'dist';

gulp.task('upload', function() {
  return gulp.src(SRC, { base: 'src' })
    .pipe(shell("echo uploading: <%= file.relative %>"))
    .pipe(shell("nodemcu-tool --silent upload --remotename <%= file.relative %> src/<%= file.relative %>"))
    .pipe(gulp.dest(DEST));
});
```

Some things to note:

* `gulp.src` is a readable stream of files, meaning that files can be read from it
* `pipe` is used to define a transformation on those files
* `shell` lets you execute a shell command
* `gulp.dest` is a writable stream that takes in files and puts them in a directoy

Have you guessed what happens when you enter `gulp upload`? Exactly, all your Lua scripts will get transferred to your nodemcu.

Unlike make, gulp isn't lazy unfortunately. That means it'll upload each and every file whenever you run `gulp upload`. The reason is, I think, because gulp is mostly used in conjunction with a technique called *file watching*, whereby files are individually processed whenever they are changed on disk. That of course would totally obviate lazy execution but it doesn't seem to be the right approach in our case.

## A difference solution

Actually, there's only a minor change we need to make to optimize the hell out of this. See if you can spot it:

``` javascript
var SRC = 'src/**/*.lua';
var DEST = 'dist';

gulp.task('upload', function() {
  return gulp.src(SRC, { base: 'src' })
    .pipe(changed(DEST))
    .pipe(shell("echo uploading: <%= file.relative %>"))
    .pipe(shell("nodemcu-tool --silent upload --remotename <%= file.relative %> src/<%= file.relative %>"))
    .pipe(gulp.dest(DEST));
});
```

The key is the line that says `.pipe(changed(DEST))`. It compares the input, which you recall is a stream of files, to the output of the previous run, the generated files in the 'dist' directory. Only files that are *newer* then their counterparts of the previous run will get processed this time. So, effectively only updated files are copied to the nodemcu!

## Closing words

For me, this solution is a life-saver. I can just run the same simple command every time instead of hand-picking the scripts that have been changed. And the upload is normally done in a few seconds compared to 4x, 5x or 10x that when I just copy all scripts every time.

Even if you dont fully understand the part about Gulp and streams, don't worry, I'll bet you can just use the script as is and modify it to your situation.

Here's the link to the [full gulpfile](https://gist.github.com/remcoder/408c1979055810d29e3fbd622c51500a). Note that you'll need [Node.js and npm](https://nodejs.org) installed. Also, although I haven't mentioned it yet, this gulpfile uses several gulp plugins. These can be installed by running `npm install` in the same directory.

This technique should also be usable with different upload tools, such as [luatool](https://github.com/4refr0nt/luatool) and [nodemcu-uploader](https://github.com/kmpm/nodemcu-uploader). In fact, it would be interesting to benchmark these tools and go with the fastest :-) Let me know if you have some findings to share.

Go ahead, give it a try!

**UPDATE** I've since switched to [nodemcu-uploader](https://github.com/kmpm/nodemcu-uploader) for uploading files because [it's about 7x faster](https://remcoder.github.io/2016/06/18/nodemcu-upload-speed-benchmark.html). I've [updated the build script](https://gist.github.com/remcoder/408c1979055810d29e3fbd622c51500a) accordingly.