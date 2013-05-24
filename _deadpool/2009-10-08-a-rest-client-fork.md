<p>I had cause recently to fork Adam Wiggin's popular rest-client gem. The current HEAD of that gem failed me when it came to pulling cookies from the response.</p>&#13;
<h2>Writing the specs is only half the battle</h2>&#13;
<p>You have to actually run them too.</p>&#13;
<p>The first issue I ran into was that only specs in the <code>/spec</code> directory were being run by the default rake tasks. Specs in subdirectories of <code>/spec</code> were not being included. There is only one spec in a subdirectory of <code>/spec</code> and it happens to be the one I needed to test the cookie behavior of the response object.</p>&#13;
<h2>There can be more than one cookie in the response</h2>&#13;
<p>rest-client does some nice things to basically beautify the response data and make it accessible via a hash with a uniform convention for keys (:content_type instead of "Content-type"). This applies to cookies returned in the response via the <code>Set-cookie</code> header.</p>&#13;
<p>The problem is that rest-client only pulls the first cookie from the <code>Set-cookie</code> field and leaves the rest behind.</p>&#13;
<p>My fork makes sure all the cookies wind up in the <code>@cookies</code> hash, and also makes sure the specs covering that logic are actually run.</p>&#13;
<p>It's available on <a href="http://gemcutter.org/gems/wgibbs-rest-client">Gemcutter</a>.</p> 
