---


---

<h1 id="mock-vs-stub">Mock vs Stub</h1>
<p>mock이랑 stub 의 차이가 뭔지 알아볼려고 하다가 <a href="https://blog.trello.com/2-billion-cards-with-trello-home">Mocks Aren’t Stubs</a>을 읽고 살짝 정리하면서 나의 언어로 옮겨보자아아아.</p>
<h4 id="test-double">Test Double</h4>
<p>mock 과 stub의 차이점을 알기 위해서는 double이라는 개념을 먼저 이해해야한다. <em>double은 테스트 목적을 위해서 진짜 object를 대신하는  어떠한 종류의 Object를 뜻</em>하는 단어이다.  예를 들면 mock이나 stub이 이에 포함되는 개념이라고 할 수 있다.</p>
<ul>
<li><strong>Dummy</strong> : Object를 어떠한 함수에 전달하지만 아에 사용하지 않는 Object</li>
<li><strong>Fake</strong> : Production에 쓰지 않는 shortcut한 implementation의 Object이다. ex) local database대신 간단하게 구현된 Memory database가 있을 수 있겠다.</li>
<li><strong>Stub</strong> : Object 자신이 어떻게  호출되는지 정보를 가지고 있는 Object이다.</li>
<li><strong>Mock</strong> : 어떠한 행동에 대한 기대를 갖고 그 행동이 그대로 수행되는지 verifiy 하는 Object 이다.</li>
</ul>
<h5 id="원문">원문</h5>
<ul>
<li><strong>Dummy</strong> objects are passed around but never actually used. Usually they are just used to fill parameter lists.</li>
<li><strong>Fake</strong> objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an <a href="https://martinfowler.com/bliki/InMemoryTestDatabase.html">in memory database</a> is a good example).</li>
<li><strong>Stubs</strong> provide canned answers to calls made during the test, usually not responding at all to anything outside what’s programmed in for the test.</li>
<li><strong>Spies</strong> are stubs that also record some information based on how they were called. One form of this might be an email service that records how many messages it was sent.</li>
<li><strong>Mocks</strong> are what we are talking about here: objects pre-programmed with expectations which form a specification of the calls they are expected to receive.</li>
</ul>
<h4 id="state-verification-vs-behavior-verfication">State Verification vs Behavior Verfication</h4>
<p>간단하게 개념을 정리해 봤지만 아직도 stub과  mock의 차이가 뭔지 제대로 알아먹을 수가 없다. 저자는 그래서 여기서 코드와 함께 차이점을 보여주게 된다.<br>
stub을 사용하는 코드는 다음과 같다.</p>
<pre><code>public interface MailService {
  public void send (Message msg);
}

public class MailServiceStub implements MailService {
  private List&lt;Message&gt; messages = new ArrayList&lt;Message&gt;();
  public void send (Message msg) {
    messages.add(msg);
  }
  public int numberSent() {
    return messages.size();
  }
}                                 
</code></pre>
<p>We can then use state verification on the stub like this.</p>
<pre><code>class OrderStateTester...

  public void testOrderSendsMailIfUnfilled() {
    Order order = new Order(TALISKER, 51);
    MailServiceStub mailer = new MailServiceStub();
    order.setMailer(mailer);
    order.fill(warehouse);
    assertEquals(1, mailer.numberSent());
  }
</code></pre>
<p>우리는 위 코드에서 mailer 변수가 우리가 구현한 stub으로서 상태를 Verifiy하고 있다는 것을 알 수 있다.  즉 , <strong>Stub</strong>은 <strong>State Verification</strong>을 사용한다.</p>
<p>이번에는 Mock의 구현을 살펴보자.</p>
<pre><code>class OrderInteractionTester...
	  public void testOrderSendsMailIfUnfilled() {
	    Order order = new Order(TALISKER, 51);
	    Mock warehouse = mock(Warehouse.class);
	    Mock mailer = mock(MailService.class);
	    order.setMailer((MailService) mailer.proxy());
    mailer.expects(once()).method("send");
    warehouse.expects(once()).method("hasInventory")
      .withAnyArguments()
      .will(returnValue(false));
    order.fill((Warehouse) warehouse.proxy());
  }
}
</code></pre>
<p>위의 코드를 살펴보면 warehouse와 mailer 변수가 mock( ) 함수를 이용하여 mocking 한 Object가 되고 어떻게 <strong>행동</strong>하는지 expects()를 이용하여 기대값을 설정하는 것을 볼 수 있다. 그  후 verify()함수를 통하여 예상한 행동을 제대로 수행하는 확인하게 된다. 즉 , <strong>Mock</strong>은 <strong>Behavior Verification</strong>을 수행하는 Test Object라는 것을 알 수 있다.</p>
<h3 id="summary">Summary</h3>
<p>mock 과 stub은 둘 다 Test Double에 속하는 Test를 위한 가짜 Object라고 분류할 수 있지만 Verification하는 대상이 다르다고 생각할 수 있다.  <strong>Stub</strong> 은 <strong>state</strong>를 verification하면서 테스트를 수행하게 되고  <strong>Mock</strong>은 <strong>Behavior</strong>을 verification하면서 테스트를 수행하게 된다.</p>
<ul>
<li>Stub Object is State Verification</li>
<li>Mock Object is Behavior Verification</li>
</ul>

