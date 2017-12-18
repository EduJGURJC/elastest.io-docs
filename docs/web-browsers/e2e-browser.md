<div class="range range-xs-left">
<div class="cell-xs-10 cell-lg-6 text-md-left inset-md-right-80 cell-lg-push-1 offset-top-50 offset-lg-top-0">
<h2 id="content" class="h1">Running end to end tests (Web Browsers)</h2>
<div class="offset-top-30 offset-md-top-50">
</div>
</div>
</div>

If you want to use a web browser in your test, you can “install” a web browser in your [SuT](/fundamentals/core-concepts/) Docker container. The problem with this approach is that you can not see what happens in the browser. You can not see the browser console. Also, it is a bit of a challenge to execute the same tests with different browser versions.

One of the main features provided by ElasTest is allowing tests to use web browsers. That browsers will be fully monitored, showing its windows in real time in ElasTest web interface and also recording it to later inspection. Also, the browser console will be shown alongside test and SuT logs.



Let's see how to launch a TJob that makes use of a web browser inside Elastest. As an example we will use our <a href="https://github.com/elastest/demo-projects/tree/master/simpleweb-java-test" target="_blank">Simple Web Java Test</a> (you can check the code for this test and similar ones in other languages in section [Sample test code](#sample-test-code)):

<h4 class="holder-subtitle link-top">1. Get into the desired project</h4>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/unit_test_1.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/unit_test_1.png"/></a>
</div>

<h4 class="holder-subtitle link-top">2. Create a SuT</h4>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/e2e_test_2.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/e2e_test_2.png"/></a>
    <a data-fancybox="gallery-1" href="/docs/testing/images/e2e_test_3.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/e2e_test_3.png"/></a>
</div>

<p>
Complete the form fields:
</p>

<ul>
    <li><strong>SuT Name</strong>: name of the Sut (<code>Webapp</code>)</li>
    <li><strong>SuT Description</strong>: description of the SuT (<code>Webapp description</code>)</li>
    <li><strong>With Docker Image</strong> / <strong>With Docker Compose</strong>: whether to deploy the SuT on ElasTest as a Docker <i>image</i> or a <i>docker-compose</i> (<code>With Docker Image</code>)</li>
    <li><strong>Docker Image</strong>: image of the SuT (<code>elastest/demo-web-java-test-sut</code>)</li>
    <li><strong>Wait for http port</strong>: which port of the SuT should ElasTest wait to be available before starting the TJob (<code>8080</code>)</li>
</ul>

<br>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_4.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_4.png"/></a>
</div>

<h4 id="3-create-a-new-tjob" class="holder-subtitle link-top">3. Create a new TJob</h4>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_5.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_5.png"/></a>
</div>

Complete the form fields:

<ul>
<li><strong>TJob Name</strong>: name of the TJob</li>
<li><strong>Select a SuT</strong>: already created SuT to be tested through to the TJob (<code>Webapp</code>)</li>
<li><strong>Environment docker image</strong>: the docker image that will host your test. This docker images should contain a client to download your code from your repository hosting service. For example, if your tests are hosted in GitHub and implemented in a Maven project with Java, you need to include a git client, Maven and the Java Development Kit (JDK) in the image. Indeed, this image contains Git, Maven and Java (<code>elastest/test-etm-alpinegitjava</code>)</li>
<!-- Modify when all images are available for testing with different hostsing services and technologies: Java, Maven, Pyhton, Ruby, Node... -->
<li><strong>Commands</strong>: these are the bash commands to be executed to download the code from a repository and to execute the tests. The specific commands depends on the source code repository type and the technology. For example, the following commands will clone a Maven/Java repository from GitHub and execute the tests:
<pre style="margin-bottom: 20px !important"><code>git clone https://github.com/elastest/demo-projects
cd demo-projects/simpleweb-java-test
mvn test</code></pre>
</li>
<li><strong>Test Support Services (EUS)</strong>: if your test is gonna require a browser or not (<code>Checked</code>)</li>
</ul>

<br>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_6.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_6.png"/></a>
</div>

<h4 class="holder-subtitle link-top">4. Run the TJob from the Project's page</h4>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_7.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_7.png"/></a>
</div>

<p>
When this TJob is executed, a web application is started (the Sut). When the web application is ready, the test container is executed with the specified commands. Whenever the test code requests a certain browser of a specific version using standard web driver protocol and libraries, Elastest provides it automatically. When browser is ready, test uses it to interact with the SuT and verify that web behaves as expected.
</p>

<p>You can see the browser in real time as test runs:</p>
<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_8.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_8.png"/></a>
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_9.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_9.png"/></a>
</div>

<p>When the test have been executed, the browser recording can be downloaded as a mp4 video file:</p>
<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_10.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_10.png"/></a>
    <a data-fancybox="gallery-1" href="/docs/testing/images/browser_test_11.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/browser_test_11.png"/></a>
</div>

<h4 id="sample-test-code" class="holder-subtitle link-top">Sample test code</h4>

<div class="badges-menu noselectionable">
    <span id="java-test-btn" class="badge badge-default my-badge selected">Java</span>
    <span id="js-test-btn" class="badge badge-default my-badge">JS</span>
    <span class="badge badge-default my-badge my-badge-disabled">PHP</span>
    <span class="badge badge-default my-badge my-badge-disabled">. . .</span>
</div>

<div id="java-test" class="lang-section">

<p>Let's take a look at the test code. It is implemented with <a href="http://junit.org/junit5/" target="_blank">JUnit 5</a> and Selenium webdriver libraries.</p>

<hr>

<h4>Field values when <a href="#3-create-a-new-tjob">creating the TJob</a></h4>

<p><strong>Commands</strong> field:</p>

<pre style="margin-bottom: 20px !important"><code>git clone https://github.com/elastest/demo-projects
cd demo-projects/simpleweb-java-test
mvn test</code></pre>

<p><strong>Environment docker image</strong> field:</p>

<code>elastest/test-etm-alpinegitjava</code>

<hr>

<div class="row" style="margin-top: 40px">


<div class="col-md-6">
<h4><i>BeforeAll</i> method</h4>
<pre>
<code class="java">
@BeforeAll
public static void setupAllTests() {

    String sutHost = System.getenv("ET_SUT_HOST");
    if (sutHost == null) {
        sutURL = "http://localhost:8080/";
    } else {
        sutURL = "http://" + sutHost + ":8080/";
    }
    System.out.println("App url: " + sutURL);
    
    eusURL = System.getenv("ET_EUS_API");
    if (eusURL == null) {
        ChromeDriverManager.getInstance().setup();
    }
}
</code>
</pre>
<blockquote>This method runs just once before all tests defined in the class.<br><code>ET_SUT_HOST</code> variable will be the IP of our SuT. ElasTest will automatically inject the right value (<code>localhost</code> if not defined)<br><code>ET_EUS_API</code> variable tells us where to connect to use Elastest browsers (standard Selenium Hub). If the variable has no value, we can consider that this service is no available and then local browsers have to be used (here we are using <a href="https://github.com/bonigarcia/webdrivermanager" target="_blank">WebDriver Manager</a> Java library. This library is responsible to download and configure any additional software needed to use installed browsers from tests.
</blockquote>
</div>

<div class="col-md-6">
<h4><i>BeforeEach</i> method</h4>
<pre>
<code class="java">
@BeforeEach
public void setupTest() throws MalformedURLException {

    ChromeOptions options = new ChromeOptions();
    
    options.addArguments("start-maximized");
    
    if (eusURL == null) {
        driver = new ChromeDriver(options);
    } else {			
        DesiredCapabilities caps = new DesiredCapabilities();
        caps.setBrowserName("chrome");
        caps.setCapability(ChromeOptions.CAPABILITY, options);					
        driver = new RemoteWebDriver(new URL(eusURL), caps);
    }
}
</code>
</pre>
<blockquote>This method is called before every test in this class.<br> It creates a browser with its window maximized: if Elastest Web Browser service is available, then the browser is a remote web browser. If not, it is a local browser. Selenium API is slightly differente in each case</blockquote>
</div>
</div>

<div class="row" style="margin-top: 20px">

<div class="col-md-6">
<h4><i>AfterEach</i> method</h4>
<pre>
<code class="java">
@AfterEach
public void teardown() {
    if (driver != null) {
        driver.quit();
    }
}
</code>
</pre>
<blockquote>This method is called after every test execution in this class.<br>It is a very important method: it disposes the browser as soon as the test has finished</blockquote>
</div>

<div class="col-md-6">
<h4><i>Test</i> method</h4>
<pre>
<code class="java">
@Test
public void test() throws InterruptedException {

    driver.get(sutURL);

    Thread.sleep(3000);

    String newTitle = "MessageTitle";
    String newBody = "MessageBody";

    driver.findElement(By.id("title-input")).sendKeys(newTitle);
    driver.findElement(By.id("body-input")).sendKeys(newBody);

    Thread.sleep(3000);

    driver.findElement(By.id("submit")).click();

    Thread.sleep(3000);

    String title = driver.findElement(By.id("title")).getText();
    String body = driver.findElement(By.id("body")).getText();

    assertThat(title, equalTo(newTitle));
    assertThat(body, equalTo(newBody));

    Thread.sleep(3000);
}
</code>
</pre>
<blockquote>This test is the only one defined in this class. Its logic just simulates the navigation of a real user in our webpage, sending some data within a form and checking the received data.<br>Sentences "Thread.sleep()" just give us some more time to see the browser in action. They are not necessary in a real project</blockquote>
</div>
</div>
</div>

<div id="js-test" class="lang-section" hidden>

<p>Let's take a look at the test code. It is implemented in JavaScript with <a href="https://mochajs.org/" target="_blank">mocha</a>, <a href="http://chaijs.com/" target="_blank">chai</a> and Selenium webdriver libraries.</p>

<hr>

<h4>Field values when <a href="#3-create-a-new-tjob">creating the TJob</a></h4>

<p><strong>Commands</strong> field:</p>

<pre style="margin-bottom: 20px !important"><code>git clone https://github.com/elastest/demo-projects
cd demo-projects/simpleweb-js-test
npm install
npm test</code></pre>

<p><strong>Environment docker image</strong> field:</p>

<code>elastest/demo-env-node</code>

<hr>

<div class="row" style="margin-top: 40px">

<div class="col-md-6">
<h4><i>before</i> method</h4>
<pre>
<code class="javascript">
before(() => {
    var sutHost = process.env.ET_SUT_HOST;
    if (!sutHost) {
        sutURL = "http://localhost:8080/";
    } else {
        sutURL = "http://" + sutHost + ":8080/";
    }
    console.log("App url: " + sutURL);

    eusURL = process.env.ET_EUS_API;
});
</code>
</pre>
<blockquote>This method runs just once before all tests defined in the class.<br><code>ET_SUT_HOST</code> variable will be the IP of our SuT. ElasTest will automatically inject the right value (<code>localhost</code> if not defined)<br><code>ET_EUS_API</code> variable tells us where to connect to use Elastest browsers (standard Selenium Hub)</blockquote>
</div>

<div class="col-md-6">
<h4><i>beforeEach</i> method</h4>
<pre>
<code class="javascript">
beforeEach(() => {
    var options = new chrome.Options();
    options.addArguments("start-maximized");

    var browserBuilder = new webdriver.Builder().forBrowser("chrome");
    browserBuilder.setChromeOptions(options);

    if (eusURL) {
        console.log("Using ElasTest browser ("+eusURL+")");
        browserBuilder.usingServer(eusURL);
    } else {
        console.log("Using local browser");  
    }
    driver = browserBuilder.build();

    console.log("Chrome browser created");
});
</code>
</pre>
<blockquote>This method is called before every test in this class.<br> It creates a browser with its window maximized: if Elastest Web Browser service is available, then the browser is a remote web browser. If not, it is a local browser. Selenium API is slightly differente in each case</blockquote>
</div>

</div>

<div class="row" style="margin-top: 20px">
<div class="col-md-6">
<h4><i>Test</i> method</h4>
<pre>
<code class="javascript">
describe("Web page", () => {
    it("should add message", async function() {
        this.timeout(30000);

        driver.get(sutURL);

        var newTitle = "MessageTitle";
        var newBody = "MessageBody";

        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        driver.findElement(By.id("submit")).click();

        var title = driver.findElement(By.id("title")).getText();
        var body = driver.findElement(By.id("body")).getText();

        assert.equal(await title, newTitle);
        assert.equal(await body, newBody);

        driver.quit();
    })
})
</code>
</pre>
<blockquote>This test is the only one defined in this class. Its logic just simulates the navigation of a real user in our webpage, sending some data within a form and checking the received data</blockquote>
</div>
</div>
</div>







<script>
$('#java-test-btn').click(function(event) {
  activateBadge('java-test');
});
$('#js-test-btn').click(function(event) {
  activateBadge('js-test');
});

function activateBadge(sectionName) {
  $('.lang-section').hide();
  $('#' + sectionName).show();
  $('.selected').removeClass('selected');
  $('#' + sectionName + '-btn').addClass('selected');
  window.location.hash = sectionName.replace('-test', '');
}

var langSections = [
  'java',
  'js'
];

window.onload = function() {
  var hash = window.location.hash.replace('#', '');
  if (langSections.indexOf(hash) > -1) {
    activateBadge(hash + '-test');
  }
}
</script>

<script src="//code.jquery.com/jquery-3.2.1.min.js"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/fancybox/3.2.5/jquery.fancybox.min.css" />
<script src="https://cdnjs.cloudflare.com/ajax/libs/fancybox/3.2.5/jquery.fancybox.min.js"></script>

<script>
var galleries = $('div.docs-gallery');
for (var i = 1; i <= galleries.length; i++) {
    $().fancybox({
    selector : '[data-fancybox="gallery-' + i + '"]',
    infobar : true,
    arrows : false,
    loop: false,
    protect: true,
    transitionEffect: 'slide',
    buttons : [
        'close'
    ],
    clickOutside : 'close',
    clickSlide   : 'close',
  });
}
</script>