<p>Let's say you have an ActiveRecord model with an attribute called <code>permissions</code> and it's required. You add the attribute to the <code>validate_presence_of</code> call and in your view, the validation error message prints out as "Permissions can't be blank."</p>&#13;
<p>Then your client wants to change the name of that field in the UI from "Permissions" to "Reuse Setting". You don't want to change the attribute name, so you just update the label in the view. However, that validation error message still says "Permissions can't be blank" and you need it to say "Reuse Setting can't be blank"</p>&#13;
<p><code>validate_presence_of</code> takes a <code>:message</code> option, but that only customizes the text appended after the humanized attribute name.</p>&#13;
<p>If you want your ActiveRecord validation messages to humanize the attribute name differently, override <code>ActiveRecord::Base.human_attribute_name</code>. For the example above, that would look like this:</p>&#13;
<pre><code class="ruby">def self.human_attribute_name(name)&#13;
  {'permissions' =&gt; 'Reuse Setting'}[name] || name.humanize&#13;
end&#13;
</code></pre>&#13;
<p>If you want to customize the humanizing of any other attribute names for the model, just add them to the hash.  Thanks to <a href="http://github.com/tpope">tpope</a> for showing me the way.</p> 
