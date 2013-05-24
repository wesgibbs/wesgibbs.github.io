<p>Like a lot of web applications, MadPolitic has different levels of user accounts. One of the account differentiators is bandwidth allowance; higher-level accounts have an increased bandwidth limit. To implement this, I needed a way to track bandwidth usage per account. Here's how I did it using Apache.</p>&#13;
<h2>Use CustomLog and cronolog to create and rotate a custom bandwidth log</h2>&#13;
<p>The standard Apache access.log was missing several things I needed in order to track bandwidth.</p>&#13;
<ul><li> the format did not include the host from the request header </li>
<li> it also did not include the number of bytes Apache sent back across the network to the client in response to each request </li>
<li> the log rotation didn't give me a clean history of log files by day </li>
</ul><p>To remedy this, I created a CustomLog and used "cronolog":http://cronolog.org/ to get some more advanced log rotation. Apache provides log rotation capability out of the box, but the Apache docs themselves "recommend":http://httpd.apache.org/docs/2.0/logs.html#piped cronolog for more flexible logging.</p>&#13;
<p>Here's the CustomLog I created, from my <code>httpd.conf</code> file</p>&#13;
<pre><code>CustomLog "|/usr/local/sbin/cronolog -l /etc/httpd/logs/bandwidth_today -P /etc/httpd/logs/bandwidth_yesterday /etc/httpd/logs/%Y/%m/%d/bandwidth.log" bandwidth env=!dontlog&#13;
</code></pre>&#13;
<p>Apache passes most of the work off to cronolog, which does a few things</p>&#13;
<ul><li> creates a symbolic link called <code>bandwidth_today</code> to the current bandwidth log file </li>
<li> creates a symbolic link called <code>bandwidth_yesterday</code> to the previous day's bandwidth log file </li>
<li> defines a spec that indicates how I want the logfiles rotated and stored, in this case, in directories based on the month, day and year </li>
</ul><p>After the piped invocation of cronolog, the CustomLog directive has two more parameters. The first, <code>bandwidth</code>, references a LogFormat I created. More on that in a moment. The last parameter enables the ability to _not_ log certain requests. For example, if you have Apache serving up other sites or content that you do not want logged, you can add lines like this to your <code>httpd.conf</code> file</p>&#13;
<pre><code>SetEnvIf Host "myblog\.mydomain\.com" dontlog&#13;
</code></pre>&#13;
<p>and that final parameter to CustomLog makes sure requests to myblog.mydomain.com don't wind up in the log.</p>&#13;
<h2>Create a custom LogFormat</h2>&#13;
<p>Recall that <code>bandwidth</code> parameter I passed to the CustomLog directive. It should reference a LogFormat directive that tells Apache how I want each line of the log file to look. By default, the Apache access log is missing the two key pieces of information I needed: the host from the request header, and the number of bytes that Apache sent back across the network to the client.</p>&#13;
<p>Logging the host was one of the keys to making all this work for my application. That's because I use a subdomain-per-account approach, so each account gets a unique subdomain like bob.madpolitic.com. Capturing that host information in the log files along with the bytes transferred would give me all the raw data I needed to map bandwidth usage to accounts.</p>&#13;
<p>I started by adding those bits into the format and came up with the following</p>&#13;
<pre><code>LogFormat "%t %{Host}i %O %h \"%r\" %&gt;s" bandwidth&#13;
</code></pre>&#13;
<p>You can reference the Apache docs for the "LogFormat directive":http://httpd.apache.org/docs/2.0/mod/mod_log_config.html#logformat for more detail, but in brief, this line tells Apache to log the time (<code>%t</code>), the contents of the Host: header in the request (<code>%{Host}i</code>), the bytes sent over the network to the client (<code>%O</code>), the remote host (<code>%h</code>), the first line of the HTTP request, quoted (<code>\"%r\"</code>), and the HTTP status of the response (<code>%&gt;s</code>). Finally, it gives this format a name, <code>bandwidth</code>.</p>&#13;
<p>I now had a log file being written out in a format that I specified that included, most importantly, the host and the number of bytes sent to the client for each request. That log file was being rotated daily and previous logs were stored in a directory structure based on the date. I also had some handy symbolic links to today's and yesterday's bandwidth log files.</p>&#13;
<h2>Make Apache write your Ruby code</h2>&#13;
<p>This worked great. The plan was to fill a log with this information, and have one log file per day. Then I could use a log analyzer and some Ruby to parse through all those log files, compile the bandwidth statistics, and write it all into my application's database so that my Rails app could get to it to do things like show the user how much bandwidth they've used. After trying it out for a little while, however, I came up with another idea.</p>&#13;
<p>Why not have Apache just write the Ruby code for me? I could drop the log analyzer and basically just execute the log file like a Ruby script and have it give me the data in the structure I wanted: a Ruby hash mapping hosts to bytes.</p>&#13;
<p>I went back to work on my log format and came up with this</p>&#13;
<pre><code>LogFormat "bw[\"%{Host}i\"] = (bw[\"%{Host}i\"]) ? bw[\"%{Host}i\"] = bw[\"%{Host}i\"] + %O : bw[\"%{Host}i\"] = %O" bandwidth&#13;
</code></pre>&#13;
<p>This gave me logs with lines like this</p>&#13;
<pre><code>bw["bob.madpolitic.com"] = (bw["bob.madpolitic.com"]) ? bw["bob.madpolitic.com"] = bw["bob.madpolitic.com"] + 123 : bw["bob.madpolitic.com"] = 123&#13;
bw["frank.madpolitic.com"] = (bw["frank.madpolitic.com"]) ? bw["frank.madpolitic.com"] = bw["frank.madpolitic.com"] + 3442 : bw["frank.madpolitic.com"] = 3442&#13;
bw["bob.madpolitic.com"] = (bw["bob.madpolitic.com"]) ? bw["bob.madpolitic.com"] = bw["bob.madpolitic.com"] + 33 : bw["bob.madpolitic.com"] = 33&#13;
</code></pre>&#13;
<p>Now I had a file full of Ruby code that kept a hash called <code>bw</code> which mapped the host to the number of bytes transferred. Each line would check to see if that host already existed as a key in the hash, and if it did, add the bytes to the total already in the hash. Otherwise it would create a new entry in the hash, host =&gt; bytes.</p>&#13;
<h2>Get the data from the hash into the application database</h2>&#13;
<p>Next I wrote a Ruby script that gets invoked by cron once a day just after midnight. The script does the following</p>&#13;
<ol><li> creates a new empty hash called <code>bw</code> </li>
<li> uses the <code>bandwidth_yesterday</code> symbolic link to read in yesterday's log file all at once to a string. I process bandwidth totals once a day, and remember that this script is being run just after midnight, so I want to process the previous day's log </li>
<li> sends the <code>eval</code> message to <code>Kernel</code> and passes the string that contains the log file along </li>
<li> when this is finished, my <code>bw</code> hash is now populated with <code>host =&gt; bytes</code> entries </li>
<li> the script iterates through each entry in the hash, and for each </li>
<li> uses the key (the host) to find an account in my application. Remember that each account gets a unique subdomain, so the host <code>bob.madpolitic.com</code> would map to bob's account </li>
<li> looks to see if there is already a row for that account, for the current month, with a bandwidth total and creates one if there is not. This would happen the first time that account is ever used, or at the start of each month </li>
<li> adds the value from the current hash entry, which is the bytes transferred for that host for the previous day, to the existing value </li>
</ol><p>When it's all finished I have a table that is populated with the number of bytes transferred for each account, for each month. The table looks like this</p>&#13;
<pre><code>create_table "web_stats", :force =&gt; true do |t|&#13;
  t.column "account_id",   :integer&#13;
  t.column "month",        :string # stored in a %m%Y format, like 042008&#13;
  t.column "bytes",        :bigint&#13;
end&#13;
</code></pre>&#13;
<p>I've also got all of the historical log files still sitting on my file system (getting backed up once a day) in the event I need to recreate the data or mash it up some other way.</p> 
