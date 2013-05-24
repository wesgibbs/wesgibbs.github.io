<p>I built MadPolitic to be an account-per-subdomain site. So when a user signs up for an account, they can choose a site name that gets prepended to the domain, e.g. <code>mysite.madpolitic.com</code> Frequently, MadPolitic's users will have registered their own domain name and want to map it to their MadPolitic site. MadPolitic is a publishing system for people or groups with a cause, so they might have domains like <code>savethefarmersmarket.com</code> or <code>stopeatingthewhales.com</code> How do you map a domain name to a specific user site in your Rails app? Here's how I did it.</p>&#13;
<h2>Part I: Some Assembly Required</h2>&#13;
<p>First of all, it's worth noting that there is no way to fully automate this process. The user has to purchase their domain outside of your application through the registrar of their choice. They also have to use that registrar's administration console to make some DNS configuration changes. Since I can't do any of those things for them, I instead provide my users with detailed instructions on making the DNS changes for each of the more popular registrars. This is a bit of a hassle since these sites are beyond my control and can and do frequently tweak their menus or UI enough to render my instructions incorrect.</p>&#13;
<h3>Buy a Domain</h3>&#13;
<p>Let's say we have a photosensitive user named Linda who is outraged at the city for installing street lights in her neighborhood. She created her community activism site at <code>lindasmad.madpolitic.com</code> but would rather people could find her at a more user-friendly URL like <code>www.darkenmystreet.com</code></p>&#13;
<p>Linda is also a big Danica Patrick fan, so she heads to GoDaddy and registers her new domain.</p>&#13;
<h3>Configure DNS: the A record</h3>&#13;
<p>To map <code>darkenmystreet.com</code> and <code>www.darkenmystreet.com</code> to her MadPolitic site, Linda has to make changes to some DNS records on her registrar's name servers. A newly registered domain is typically set to use a pair of name servers under the control of the registrar. Most of the more popular registrars offer some version of a DNS control panel that lets users modify the DNS settings for their domain on those name servers. In the GoDaddy administration console, for instance, that feature is called _Total DNS Control and MX Records_.</p>&#13;
<p>First, she has to edit what is called an A record, also called an address record. The sole purpose of an address record is to map a domain name to an IP address. This is the function that most of us think of when we think of DNS servers. They translate domain names to IP addresses. When you register a new domain, an A record is typically created that points your domain to one of the registrars IP addresses.</p>&#13;
<p>Linda edits the A record and sets 'Host' to 'darkenmystreet.com' and 'Points to' to '72.249.74.216', which is the MadPolitic IP address. In most DNS servers there is a shorthand for referring to the current domain: the 'at' sign (<code>@</code>). So for 'Host', Linda could also enter <code>@</code>.</p>&#13;
<h3>Configure DNS: the CNAME record</h3>&#13;
<p>Linda has one more change to make to the DNS records. She also wants to be sure that someone entering <code>www.darkenmystreet.com</code> winds up at her MadPolitic site. To do this, she needs to create a CNAME record, also called an alias. This record has two primary pieces of information: the alias name and the host it points to. Linda enters <code>www</code> for the alias name and <code>@</code> for the 'points to' field. Sometimes this record is automatically created by the registrar and so nothing needs to be done.</p>&#13;
<p>Once all the DNS information for her new domain propagates out through the Internet, both <code>darkenmystreet.com</code> and <code>www.darkenmystreet.com</code> will resolve to the MadPolitic IP address.</p>&#13;
<h3>Tell MadPolitic about the new domain</h3>&#13;
<p>The final step for Linda is to log into her MadPolitic account and register her new domain name with my Rails app. I'll use this to map requests coming from the <code>darkenmystreet.com</code> host to Linda's MadPolitic site.</p>&#13;
<h2>Part II: About That Rails App</h2>&#13;
<p>That takes care of the first half of the equation: requests for the mapped domain are getting routed to my server. At this point, everything else is under my control. I have an apache virtual host set up to catch requests for port 80 from any host and proxy them to my Rails app. In the app, I have a before_filter in <code>ApplicationController</code> called <code>set_site</code> with this logic</p>&#13;
<pre><code class="ruby">&#13;
...&#13;
if request.host.ends_with? 'madpolitic.com'&#13;
  # the request is for a site on the madpolitic domain with no domain mapping&#13;
  sitename = request.subdomains(tld_length = 1)[0]&#13;
  session[:site] = Site.find_by_sitename(sitename).id&#13;
else&#13;
  # the host is not *.madpolitic.com so see if the host maps to a madpolitic site&#13;
  # I actually cache a mapping of mapped_domain =&gt; sitename so I don' t hit the db each time; removed for this example&#13;
  session[:site] = Site.find_by_mapped_domain(request.host)&#13;
end&#13;
if !session[:site]&#13;
  logger.error "[#{Time.now.utc.strftime('%m-%d-%Y %H%:%M:%S')}]: Failed to locate site based on host of #{request.host}."&#13;
  redirect_to 'http://www.madpolitic.com/500.html', :status =&gt; 500&#13;
  return false&#13;
end&#13;
...&#13;
</code></pre>&#13;
<p>This method checks <code>request.host</code> to see if it ends in <code>.madpolitic.com</code> and handles it like a typical subdomain-based request and serves up the correct site accordingly. If the host does not end in <code>.madpolitic.com</code> then it might be a mapped domain and I try pulling a site from the database using the mapped domain.</p>&#13;
<p>Since Linda registered her mapped domain with my Rails app, the find is successful. If the find fails, then it's possible someone followed all the directions, but failed to go to the MadPolitic administration console and enter the mapped domain. When I don't find a site, I can serve an oops page and offer some helpful troubleshooting suggestions.</p>&#13;
<h2>Epilogue: Follow That Request</h2>&#13;
<p>Now let's meet Robert. He lives on Linda's street and has been infatuated with her for months. When he heard about her campaign to have the street lights removed, he became an impassioned supporter of her cause. He gets the URL from a flyer in the local coffee shop and logs on to sign the petition.</p>&#13;
<p>When Robert enters http://www.darkenmystreet.com into his browser and hits enter, here's what happens (assuming he's never hit that domain before).</p>&#13;
<ol><li> His browser sends a request out to one of the DNS servers of Robert's ISP. </li>
<li> That server (called the Resolver in this scheme) has never had a request for <code>www.darkenmystreet.com</code> before, so it doesn't know the IP. It passes the request on to one of the 13 root name servers. </li>
<li> That server replies to the Resolver saying "I don't know about that domain, but this root name server should." </li>
<li> The Resolver then contacts that root name server with the request. That root server has a record mapping the domain to a name server and returns the address of that name server to the Resolver. </li>
<li> Now the Resolver contacts that name server. This is the name server run by GoDaddy that Linda configured when she set the A record and CNAME record. </li>
<li> Those record entries translate <code>www.darkenmystreet.com</code> to the MadPolitic IP address and return it to the Resolver. </li>
<li> The Resolver caches the information for future requests and returns the IP to the browser. </li>
<li> The browser sends its request directly to the MadPolitic IP. </li>
<li> My Apache server is listening on that IP on port 80, and intercepts the request. The host in the request header matches up with a virtual server I have configured and Apache proxies the request on to Mongrel and my Rails app. </li>
<li> Rails serves up the page Robert was looking for. </li>
<li> Robert comments on one of Linda's blog postings with an awkwardly-worded poem wherein he likens his love for Linda to a streetlight. </li>
</ol>
