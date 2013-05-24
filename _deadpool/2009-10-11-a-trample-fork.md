<p>I'm a fan of James Golick's load simulation gem Trample. I ran into some issues using it on a Rails project recently though, so I forked it and made some changes.</p>&#13;
<h2>:3000 is not a variable to be interpolated</h2>&#13;
<p>Trample allows you to interpolate variables into the URLs you specify in your test script. This is a great feature, but as implemented has one problem: port numbers. These get interpolated as well and if you haven't supplied a value for the variable, then the port number gets removed.</p>&#13;
<p>Granted, load testing on my local machine at port 3000 is not really a good environment for load testing. But I believe most people will want to tweak their load testing scripts in a local environment before cutting them loose elsewhere.</p>&#13;
<p>It's also reasonable to assume that I might be running the service I want to test on a port other than 80.</p>&#13;
<p>I tightened up the regex so that colons followed by only digits are not interpolated.</p>&#13;
<h2>My authentication scheme might require more than a single post</h2>&#13;
<p>Trample has a simple DSL for describing the series of requests that make up the load test. Part of this DSL is the <code>login</code> block. The request specified in this block is run only once at the beginning of each of the concurrent threads to handle any authentication. You can specify multiple requests in this block, however, only the last one actually gets executed.</p>&#13;
<p>Because I have forgery protection enabled, my Rails app really requires two requests in order to authenticate. The initial GET to the sign in form returns the authenticity token which must be present in the subsequent POST for authentication to succeed.</p>&#13;
<p>The first step in enabling this in Trample was to allow the <code>login</code> block to define multiple requests to be run to simulate a login.</p>&#13;
<h2>The authenticity token</h2>&#13;
<p>The final step was to make Trample aware of authenticity tokens. Any time Trample is asked to perform a POST, it will look in the previous response body for a hidden input field with an authenticity token and include it in the POST parameters.</p>&#13;
<p>The fork is available on <a href="http://gemcutter.org/gems/wgibbs-trample">Gemcutter</a>.</p> 
