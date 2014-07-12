    ImgsOptimizer 
Set of scripts to optimize size of images (jpeg, png) in directory.
It search all png and jpeg files in directory (must be define in init script).
If found file is not marked yet (has no attribute optimized), then script trying
to optimize it and mark it with attribute. Also if any new files appear in the 
directory (listen via inotifywait) it try to optimize it.

    Requirements:
- jpegoptim;
- optipng;
- advpng (advancecomp);
- pngcrush;
- inotify-tools,
- support of extended file system attributes (mount with user_xattr option).

