<p><a href="http://github.com/tpope">Tim Pope</a> set <a href="http://github.com/paulelliott">Paul Elliott</a> and me straight recently on the right place to put your sass and css files to get them to play nicely with rails.vim.</p>&#13;
<p>The short answer is to do it the way the sass <a href="http://sass-lang.com/docs/yardoc/SASS_REFERENCE.md.html">docs</a> tell you to do it. Put your sass in <code>public/stylesheets/sass</code> and your css will automatically be compiled into <code>public/stylesheets</code>.</p>&#13;
<p>If you follow that convention, then <code>:Rstyle site</code> will open <code>public/stylesheets/sass/site.sass</code>.</p>&#13;
<p>Note that this will only work once the sass files have been compiled to css.</p>&#13;
<p>We put any third-party css files that are added to the project into <code>public/ stylesheets/vendor</code>. Then we .gitignore <code>public/stylesheets/*.css</code>.</p> 
