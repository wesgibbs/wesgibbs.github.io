<p>I am referring to transactional fixtures, and I had a mistaken understanding of how they worked. For my own notes, here's what I learned.</p>&#13;
<p>By default, TestCase and RSpec use transactional fixtures. This is set in the respective helper file (<code>test/test_helper.rb</code> or <code>spec/spec_helper.rb</code>). Transactional fixtures is a terrible name for this feature, because the fixtures are not transactional. What transactional fixtures actually means is that each test method is executed in a database transaction which is rolled back when the test is finished.</p>&#13;
<p>Here's what happens when you have a test class with several test methods and one fixture (let's say <code>:users</code>) specified</p>&#13;
<ul><li> all rows are deleted from the users table </li>
<li> an insert is executed for each user listed in the <code>:users</code> fixture </li>
<li> for each test method in the test case   &#13;
<ul><li> a transaction is started </li>
<li> the test is run </li>
<li> the transaction is rolled back </li>
</ul></li>
</ul><p>This means that when the class is finished executing, all the fixture data that was loaded is left in the database.</p>&#13;
<p>There's a gotcha lurking here, which is what bit me. Let's say you have two models as follows</p>&#13;
<pre><code class="ruby">class User &lt; ActiveRecord::Base&#13;
  has_many :zebras&#13;
end&#13;
&#13;
class Zebra &lt; ActiveRecord::Base&#13;
  belongs_to :user&#13;
end&#13;
&#13;
class UserTest &lt; Test::Unit::TestCase&#13;
  fixtures :users&#13;
  # test methods here&#13;
end&#13;
&#13;
class ZebraTest &lt; Test::Unit::TestCase&#13;
  fixtures :zebras&#13;
  # test methods here&#13;
end&#13;
</code></pre>&#13;
<p>When you run <code>rake test:models</code>, all your zebra tests pass. However, if you <code>rake db:test:prepare</code> and run <code>ruby test/unit/zebra_test.rb</code>, suddenly tests are failing. This is because when you run the entire suite, the User test case is running first and its fixture, <code>:users</code>, is getting loaded into the database. When the suite gets to the Zebra test case, the test methods that rely on rows in the users table find the data that was left over from the User test case and so they work. However, when you run the Zebra test case in isolation on a fresh test database, that user data is not loaded, so the tests fail.</p>&#13;
<p>This has led to a new best practice for me when writing tests. I always make sure I'm running the TestCase against a clean database while I'm writing it. I do this by frequently running <code>rake db:test:prepare</code>. That way I know that the TestCase has the fixture data it needs to run on its own and is not dependent on data that might be left over from the fixtuers of other tests.</p> 
