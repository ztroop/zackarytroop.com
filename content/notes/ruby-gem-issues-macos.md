+++
title = "Ruby Gem Installation Issues in MacOS"
slug = "ruby-gem-issues-macos"
date = 2021-03-09
+++

My day job requires me to use a MacOS laptop for work. I encountered a puzzling issue when I was updating the [yajl-ruby](https://github.com/brianmario/yajl-ruby) gem.

```sh
current directory: /Library/Ruby/Gems/2.6.0/gems/yajl-ruby-1.4.1/ext/yajl
/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/bin/ruby -I /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0 -r ./siteconf20210310-23807-1oknzj5.rb extconf.rb

creating Makefile
current directory: /Library/Ruby/Gems/2.6.0/gems/yajl-ruby-1.4.1/ext/yajl
make "DESTDIR=" clean
current directory: /Library/Ruby/Gems/2.6.0/gems/yajl-ruby-1.4.1/ext/yajl
make "DESTDIR="
make: *** No rule to make target `/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/include/ruby-2.6.0/universal-darwin19/ruby/config.h', needed by `yajl.o'.  Stop.
make failed, exit code 2
```

If you're also using MacOS and Ruby from Xcode Command-Line Tools, you might be interested in the solution. It looks like `universal-darwin19` doesn't exist, but `universal-darwin20` does.

Instead of spending too much time in the Apple melodrama, I simply created a symlink to point to the `universal-darwin20` directory. Re-run the `gem update` and we're good to go.

```sh
cd /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/include/ruby-2.6.0/
ln -sf universal-darwin20 universal-darwin19
```

It would seem I'm not the only one running into this [problem](https://stackoverflow.com/questions/63729369/commonmarker-gem-cannot-be-installed-needed-for-jekyll-macos).
