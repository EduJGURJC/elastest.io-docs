<div class="range range-xs-left">
<div class="cell-xs-10 cell-lg-6 text-md-left inset-md-right-80 cell-lg-push-1 offset-top-50 offset-lg-top-0">
<h2 id="content" class="h1">Running end to end tests (Web Browsers)</h2>
<div class="offset-top-30 offset-md-top-30">
</div>
</div>
</div>

If you want to use a web browser in your test, you can “install” a web browser in your [TJob](/#elastest-core-concepts) Docker container. The problem with this approach is that you can not see what happens in the browser. You can not see the browser console. Also, it is a bit of a challenge to execute the same tests with different browser versions.

One of the main features provided by ElasTest is allowing tests to use web browsers. That browsers will be fully monitored, showing its windows in real time in ElasTest web interface and also recording it to later inspection. Also, the browser console will be shown alongside test and SuT logs.

Let's see how to launch a TJob that makes use of a web browser inside Elastest.
Here we will run our [JUnit5 Multi Browser Test](https://github.com/elastest/demo-projects/tree/master/webapp/junit5-web-multiple-browsers-test) provided by default in ElasTest, which makes use of a Spring Boot Application as a SuT that has two input fields (title and body) and a button to add them as a table row. Also has three test that are responsible for add rows to that Sut and verify that the added row has the expected content.
This test has been developed in Java using [JUnit5](https://junit.org/junit5/):

<div class="row">
<h5 class="small-subtitle">WebAppTest class</h5>
<pre>
<code class="java">
public class WebAppTest extends ElastestBaseTest {

    @Test
    public void addMsgAndClear(TestInfo info)
            throws InterruptedException, MalformedURLException {
        Thread.sleep(2000);

        String newTitle = "MessageTitle";
        String newBody = "MessageBody";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        // Added
        logger.info("Checking Message...");
        assertEquals(newTitle, title);
        assertEquals(newBody, body);

        Thread.sleep(1000);

        int titleExist = driver.findElements(By.id("title")).size();
        int bodyExist = driver.findElements(By.id("body")).size();

        try {
            assertNotEquals(0, titleExist);
            assertNotEquals(0, bodyExist);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    @Test
    public void findTitleAndBody(TestInfo info)
            throws InterruptedException, MalformedURLException {

        Thread.sleep(2000);

        String newTitle = "MessageTitle";
        String newBody = "MessageBody";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertEquals(newTitle, title);
            assertEquals(newBody, body);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    @Test
    public void checkTitleAndBodyNoEmpty(TestInfo info)
            throws InterruptedException, MalformedURLException {

        Thread.sleep(2000);

        String newTitle = "";
        String newBody = "";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertNotEquals(newTitle, title);
            assertNotEquals(newBody, body);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }

    }

    /* ********************* */
    /* *** Other methods *** */
    /* ********************* */

    public void addRow(String newTitle, String newBody)
            throws InterruptedException {
        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        Thread.sleep(2000);

        logger.info("Adding Message...");

        driver.findElement(By.id("submit")).click();
    }

    public void clearRows() throws InterruptedException {
        logger.info("Clearing Messages...");
        driver.findElement(By.id("clearSubmit")).click();
        Thread.sleep(1000);
    }
}
</code>
</pre>
</div>

>-  In the code **`Thread.sleep()`** is used for a better visualization of the video that ElasTest records from the test, but it is not necessary to use it.
>-  The **`checkTitleAndBodyNoEmpty`** test simulates a test that fails.   

<p>In addition, as can be seen in the example, this test class extends a class called ElasTestBase which is responsible for printing logs and start/stop browsers at the beginning and end of each test. These two logs have a specific structure and are used by ElasTest to filter the logs corresponding to each test. We explain this in more detail <a href="/docs/testing/unit#xmlAndtestResultsPath">here</a>.</p>
<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">
public class ElastestBaseTest {
    protected static final Logger logger = LoggerFactory
            .getLogger(ElastestBaseTest.class);

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;
    protected static String eusURL;
    protected static String sutUrl;

    protected WebDriver driver;

    @BeforeAll
    public static void setupClass() {
        String sutHost = System.getenv("ET_SUT_HOST");
        String sutPort = System.getenv("ET_SUT_PORT");
        String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

        if (sutHost == null) {
            sutUrl = "http://localhost:8080/";
        } else {
            sutPort = sutPort != null ? sutPort : "8080";
            sutProtocol = sutProtocol != null ? sutProtocol : "http";

            sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
        }
        logger.info("Webapp URL: " + sutUrl);

        browserType = System.getProperty("browser");
        logger.info("Browser Type: {}", browserType);
        eusURL = System.getenv("ET_EUS_API");
        
        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                WebDriverManager.chromedriver().setup();
            } else {
                WebDriverManager.firefoxdriver().setup();
            }
        }
    }

    @BeforeEach
    public void setupTest(TestInfo info) throws MalformedURLException {
        String testName = info.getTestMethod().get().getName();
        logger.info("##### Start test: {}", testName);

        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                driver = new ChromeDriver();
            } else {
                driver = new FirefoxDriver();
            }
        } else {
            DesiredCapabilities caps;
            if (browserType == null || browserType.equals(CHROME)) {
                caps = DesiredCapabilities.chrome();
            } else {
                caps = DesiredCapabilities.firefox();
            }

            browserVersion = System.getProperty("browserVersion");
            if (browserVersion != null) {
                logger.info("Browser Version: {}", browserVersion);
                caps.setVersion(browserVersion);
            }

            caps.setCapability("testName", testName);
            driver = new RemoteWebDriver(new URL(eusURL), caps);
        }

        driver.get(sutUrl);
    }

    @AfterEach
    public void teardown(TestInfo info) {
        if (driver != null) {
            driver.quit();
        }

        String testName = info.getTestMethod().get().getName();
        logger.info("##### Finish test: {}", testName);
    }

}
</code>
</pre>
</div>

>-  **`ET_SUT_HOST`**, **`ET_SUT_PORT`** and **`ET_SUT_PROTOCOL`**  variables will be the IP, port and protocol of our SuT respectively. ElasTest will automatically inject the right value (Know more about <a href="/docs/testing/environment-variables/">Environment Variables</a>)

>-  **`ET_EUS_API`** variable tells us where to connect to use Elastest browsers (standard Selenium Hub). If the variable has no value, we can consider that this service is no available and then local browsers have to be used (here we are using <a href="https://github.com/bonigarcia/webdrivermanager" target="_blank">WebDriver Manager</a> Java library. This library is responsible to download and configure any additional software needed to use installed browsers from tests)

>-  The values of the variables **browserType** and **browserVersion** are taken from the **properties** browser and browserVersion respectively, which you can pass in the test run command with **`-Dbrowser=chrome`**.


If you prefer, ElasTest also provides the same example but making use of a single browser for all tests, whose name in this case is [JUnit5 Single Browser Test](https://github.com/elastest/demo-projects/tree/master/webapp/junit5-web-single-browser-test).
The WebAppTest class is exactly the same, the change is in the ElastestBaseTest class:

<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">

public class ElastestBaseTest {
    protected static final Logger logger = LoggerFactory
            .getLogger(ElastestBaseTest.class);

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;
    protected static String eusURL;
    protected static String sutUrl;

    protected static WebDriver driver;

    @BeforeAll
    public static void setupClass() throws MalformedURLException {
        // If first time, init
        if (driver == null) {
            String sutHost = System.getenv("ET_SUT_HOST");
            String sutPort = System.getenv("ET_SUT_PORT");
            String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

            if (sutHost == null) {
                sutUrl = "http://localhost:8080/";
            } else {
                sutPort = sutPort != null ? sutPort : "8080";
                sutProtocol = sutProtocol != null ? sutProtocol : "http";

                sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
            }
            logger.info("Webapp URL: " + sutUrl);

            browserType = System.getProperty("browser");
            logger.info("Browser Type: {}", browserType);
            eusURL = System.getenv("ET_EUS_API");

            if (eusURL == null) {
                if (browserType == null || browserType.equals(CHROME)) {
                    WebDriverManager.chromedriver().setup();
                    driver = new ChromeDriver();
                } else {
                    WebDriverManager.firefoxdriver().setup();
                    driver = new FirefoxDriver();
                }
            } else {
                DesiredCapabilities caps;
                if (browserType == null || browserType.equals(CHROME)) {
                    caps = DesiredCapabilities.chrome();
                } else {
                    caps = DesiredCapabilities.firefox();
                }

                browserVersion = System.getProperty("browserVersion");
                if (browserVersion != null) {
                    logger.info("Browser Version: {}", browserVersion);
                    caps.setVersion(browserVersion);
                }
                driver = new RemoteWebDriver(new URL(eusURL), caps);
            }

            // driver quit when all tests end
            Runtime.getRuntime().addShutdownHook(new Thread() {
                public void run() {
                    logger.info("Shutting down browser...");
                    driver.quit();
                }
            });
        }
    }

    @BeforeEach
    public void setupTest(TestInfo info) throws MalformedURLException {
        String testName = info.getTestMethod().get().getName();
        ((JavascriptExecutor) driver).executeScript(
            "'{\"elastestCommand\": \"startTest\", \"args\": {\"testName\": \"" + testName + "\"} }'");

        logger.info("##### Start test: {}", testName);

        driver.get(sutUrl);
    }

    @AfterEach
    public void teardown(TestInfo info) {
        String testName = info.getTestMethod().get().getName();
        logger.info("##### Finish test: {}", testName);
    }
}
</code>
</pre>
</div>

>   Java shutdown hook (**`Runtime.getRuntime().addShutdownHook`**) is used to make sure to close the browser once all the tests have been executed.

The start/end log traces are still printed in the **@BeforeEach** but the browser starts now inside the **@BeforeAll**. Also, for Elastest to know when each test starts, it is possible to send a **`command by invoking a script`** in **@BeforeEach**.

            ((JavascriptExecutor) driver).executeScript(
                "'{\"elastestCommand\": \"startTest\", \"args\": {\"testName\": \"" + testName + "\"} }'");

The content of the script consists of a JSON object inside single quotes *(**'**)* which must contain the following keys:

-   **`elastestCommand`**: the command that ElasTest must execute, in this case the value would be **`startTest`**.
-   **`args`**: a JSON object with the arguments needed to execute the command. In this case the object will only contain one key-value pair: **`testName`**

The keys must be sorted as stated above, as ElasTest will intercept the script when reading *elastestCommand*.

<!-- ******************* -->
<!-- ******* RUN ******* -->
<!-- ******************* -->

<h4 class="holder-subtitle link-top">Running the "JUnit5 Multi Browser Test" TJob</h4>

To Run "JUnit5 Multi Browser Test" TJob you only need follow these steps:

<h5 class="small-subtitle">1. Access your ElasTest dashboard</h5>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/images/your-first-test/dashboard.png"><img class="img-responsive img-wellcome" src="/docs/images/your-first-test/dashboard.png"/></a>
</div>

<h5 class="small-subtitle">2. Get into "WebApp" project</h5>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/project_selection.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/project_selection.png"/></a>
</div>

<h5 class="small-subtitle">3. Run the 'JUnit5 Multi Browser Test' TJob</h5>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/run_tjob.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/run_tjob.png"/></a>
</div>

<h5 class="small-subtitle">4. Execution screen is open automatically</h5>

<p>Our TJob will start running: you will see the test information and log.</p>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/execution_running.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/execution_running.png"/></a>
</div>

<p>Once the test is finished you will see the test results and log. The execution must end with two satisfactory tests and one failed test.</p>

<p>
    <div class="range range-xs range-xs-center warning-range">
        <div class="cell-xs-2 cell-lg-1" style="text-align: center;"><span class="icon mdi mdi-information-outline warning-span"></span></div>
        <div class="cell-xs-10 cell-lg-11 warning-text"><p><i>IMPORTANT: ElasTest make use of  <a href="/docs/testing/unit#xmlAndtestResultsPath" title="View XML Report explanation">xml results file</a> to get all the test results information.</i></p></div>
    </div>
</p>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/execution_finished_1.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/execution_finished_1.png"/></a>
</div>

And at the bottom you can see the recordings:

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/execution_finished_2.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/execution_finished_2.png"/></a>
        <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/execution_finished_recording.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/execution_finished_recording.png"/></a>
</div>

You can click on the Test Suite to display it and see each of the Test Cases. You can also click on one of them to navigate to the information section of this Test Case, where you can see their logs filtered thanks to the two traces of logs that the class ElasTestBase prints at the beginning and end of each test and that we have commented above. We are going to click on checkTitleAndBodyNoEmpty:

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/test_case_result.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/test_case_result.png"/></a>
</div>

You can click too on the **`View In LogAnalyzer`** button for navigate to <a href="/docs/log-analyzer/">Log Analyzer</a> Tool and view the execution logs:

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/view_in_loganalyzer.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/view_in_loganalyzer.png"/></a>
</div>

Or you can click on the **`View Case In LogAnalyzer`** button view the specific test execution logs:

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/view_case_in_loganalyzer.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/view_case_in_loganalyzer.png"/></a>
</div>

<!-- ****************************** -->
<!-- *** CREATING TJOB YOURSELF *** -->
<!-- ****************************** -->

<h4 class="holder-subtitle link-top">Creating a "JUnit5 Multi Browser Test" TJob yourself</h4>
If you want to create the TJob yourself, you only need follow these steps:

<h5 class="small-subtitle">1. Access your ElasTest dashboard</h5>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/images/your-first-test/dashboard.png"><img class="img-responsive img-wellcome" src="/docs/images/your-first-test/dashboard.png"/></a>
</div>

<h5 class="small-subtitle">2. Create a New Project</h5>
The Projects serve to organize the TJobs related to each other. You can create a new Project by clicking on the button with the same name. You only need to indicate the name of the project and then click on SAVE button:
<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/images/your-first-test/new_project.png"><img class="img-responsive img-wellcome" src="/docs/images/your-first-test/new_project.png"/></a>
</div>

Immediately you will be redirected to the project page:

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/images/your-first-test/project_page.png"><img class="img-responsive img-wellcome" src="/docs/images/your-first-test/project_page.png"/></a>
</div>

<h5 class="small-subtitle">3. Create a SuT</h5>
There are several ways to deploy a Sut in ElasTest, but they can be grouped in two ways:

-   **`Deployed by Elastest`**: your software is packaged as Docker container/s. It can be a single Docker image or a docker-compose.
    -   **With Commands Container**: Your SuT is packaged as a Docker image. You must write the _Commands Container Image_ and the commands that will run like the docker image CMD.
    -   **With Docker Image**: Your SuT is packaged as a Docker image. ElasTest will pull it from DockerHub and run it as the `Dockerfile` states.
    -   **With Docker Compose**: Your SuT is declared as a docker-compose. ElasTest will pull all the necessary images from DockerHub and run them as the field `Docker Compose`.
-   **`Deployed outside ElasTest`**:  your software is already deployed somewhere.
    -   **No instrumentation**: No monitoring traces sent to ElasTest.
    -   **Instrumented by ElasTest**:  Elastest will be responsible for accessing your Sut to send monitoring traces.
    -   **Manual Instrumentation**: If you want to manually send its logs and metrics to ElasTest.
    -   **Use External Elasticsearch**: if you use your own [Elasticsearch](https://www.elastic.co/guide/index.html) to save the monitoring traces and you want ElasTest to access it to retrieve them.


You can read [Software under Test](/testing/sut) for more detailed information about Sut.

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/testing/images/rest/new_sut_btn.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/rest/new_sut_btn.png"/></a>
    <a data-fancybox="gallery-1" href="/docs/testing/images/rest/new_sut_page.png"><img class="img-responsive img-wellcome" src="/docs/testing/images/rest/new_sut_page.png"/></a>
</div>

In our case, we will need to insert the following data for create the SuT of "JUnit5 Rest Test" TJob:

-   **SuT Name**: can be called as you want, but we will call it **`WebApp`**
-   **With Commands Container** / **With Docker Image** / **With Docker Compose**: **`With Docker Image`**
-   **Docker Image**: image of the SuT (**`elastest/demo-web-java-test-sut`**)
-   **Wait for http port**: which port of the SuT should ElasTest wait to be available before starting the TJob (**`8080`**)

<br>
<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/sut_creation.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/sut_creation.png"/></a>
</div>

<h5 class="small-subtitle">4. Create a new TJob</h5>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/new_tjob_btn.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/new_tjob_btn.png"/></a>
</div>
When a TJob is created, the minimum information that you have to provide is the following:

-   **TJob Name**: name of the TJob
-   **Select a SuT**: If your TJob make use of a Software under Test. In this case, none.
-   **Environment docker image**: the docker image that will host your test. This docker images should contain a client to download your code from your repository hosting service. For example, if your tests are hosted in GitHub and implemented in a Maven project with Java, you need to include a git client, Maven and the Java Development Kit (JDK) in the image.
    <!-- Modify when all images are available for testing with different hostsing services and technologies: Java, Maven, Pyhton, Ruby, Node... -->
-   **Commands**: these are the bash commands to be executed to download the code from a repository and to execute the tests. The specific commands depends on the source code repository type and the technology.

In our case, we will need to insert the following data for the TJob "JUnit5 Multi Browser Test":

-   **TJob Name**: can be called as you want, but we will call it **`JUnit5 Multi Browser Test`**
-   **Test Results Path**: **`/demo-projects/webapp/junit5-web-multiple-browsers-test/target/surefire-reports`**. This is the complete path where the xml reports of the execution in the container will be saved. We explain this in more detail <a href="/docs/testing/unit#xmlAndtestResultsPath">here</a>.

*   **Select a SuT**: already created SuT to be tested through to the TJob (**`WebApp`**)

-   **Environment docker image**:  [**`elastest/test-etm-alpinegitjava`**](https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitjava/Dockerfile) (image that contains Git, Maven and Java).
-   **Commands**:

        git clone https://github.com/elastest/demo-projects;
        cd /demo-projects/webapp/junit5-web-multiple-browsers-test;
        mvn -B -Dbrowser=chrome test;

-   **Test Support Services**: Check **`EUS`**

<div class="docs-gallery inline-block" style="margin-top: 20px">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/new_tjob_creation.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/new_tjob_creation.png"/></a>
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/new_tjob_creation_2.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/new_tjob_creation_2.png"/></a>
</div>

By clicking on SAVE the TJob will be saved and you will be redirected to the project page again, where you will be able to execute the TJob.

<h5 class="small-subtitle">5. Run the TJob from the Project's page</h5>

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/web-browsers/images/e2e/project_page_with_tjob.png"><img class="img-responsive img-wellcome" src="/docs/web-browsers/images/e2e/project_page_with_tjob.png"/></a>
</div>

<!-- ********************************** -->
<!-- ********* Other Examples ********* -->
<!-- ********************************** -->

<h4 class="holder-subtitle link-top" id="moreExamples">More examples</h4>
The following examples, also offered by default in ElasTest, are implemented with different technologies:

<div class="badges-menu main-badge">
    <span id="junit4-btn" class="badge badge-default my-badge selected">Java JUnit4</span>
    <span id="python-btn" class="badge badge-default my-badge my-badge-disabled">Python</span>
    <span id="cucumber-btn" class="badge badge-default my-badge my-badge-disabled">Java Cucumber</span>
    <span id="gauge-btn" class="badge badge-default my-badge my-badge-disabled">Java Gauge</span>
    <span id="protractor-btn" class="badge badge-default my-badge my-badge-disabled">JS Protractor</span>
</div>

<!-- JUNIT4 -->
<div id="junit4" class="testExample badge-tutorial">


<div class="badges-menu sub-badge">
    <span id="junit4-multi-btn" class="badge badge-default my-badge selected">A Browser for Each</span>
    <span id="junit4-single-btn" class="badge badge-default my-badge my-badge-disabled">A Browser for All</span>
</div>

<div id="junit4-multi" class="subTestExample">
<p>
You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/junit4-web-multiple-browsers-test">Source Code in GitHub</a>.
This test has been developed in Java using <a target="_blank" href="https://junit.org/junit4">JUnit4</a>.
</p>

<div class="row">
<h5 class="small-subtitle">WebAppTest class</h5>
<pre>
<code class="java">
// Uses a browser for each test
public class WebAppTest extends ElastestBaseTest {

    @Test
    public void addMsgAndClear()
            throws InterruptedException, MalformedURLException {
        Thread.sleep(2000);

        String newTitle = "MessageTitle";
        String newBody = "MessageBody";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        // Added
        logger.info("Checking Message...");
        assertEquals(newTitle, title);
        assertEquals(newBody, body);

        Thread.sleep(1000);

        int titleExist = driver.findElements(By.id("title")).size();
        int bodyExist = driver.findElements(By.id("body")).size();

        try {
            assertNotEquals(0, titleExist);
            assertNotEquals(0, bodyExist);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    @Test
    public void findTitleAndBody()
            throws InterruptedException, MalformedURLException {
        Thread.sleep(2000);

        String newTitle = "MessageTitle";
        String newBody = "MessageBody";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertEquals(newTitle, title);
            assertEquals(newBody, body);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    @Test
    public void checkTitleAndBodyNoEmpty()
            throws InterruptedException, MalformedURLException {
        Thread.sleep(2000);

        String newTitle = "";
        String newBody = "";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertNotEquals(newTitle, title);
            assertNotEquals(newBody, body);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* ********************* */
    /* *** Other methods *** */
    /* ********************* */

    public void addRow(String newTitle, String newBody)
            throws InterruptedException {
        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        Thread.sleep(2000);

        logger.info("Adding Message...");

        driver.findElement(By.id("submit")).click();
    }

    public void clearRows() throws InterruptedException {
        logger.info("Clearing Messages...");
        driver.findElement(By.id("clearSubmit")).click();
        Thread.sleep(1000);
    }

}
</code>

</pre>
</div>

<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">
public class ElastestBaseTest {
    @Rule
    public TestName name = new TestName();

    protected static final Logger logger = LoggerFactory
            .getLogger(ElastestBaseTest.class);

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;
    protected static String eusURL;
    protected static String sutUrl;

    protected WebDriver driver;

    @BeforeClass
    public static void setupClass() {
        String sutHost = System.getenv("ET_SUT_HOST");
        String sutPort = System.getenv("ET_SUT_PORT");
        String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

        if (sutHost == null) {
            sutUrl = "http://localhost:8080/";
        } else {
            sutPort = sutPort != null ? sutPort : "8080";
            sutProtocol = sutProtocol != null ? sutProtocol : "http";

            sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
        }
        logger.info("Webapp URL: " + sutUrl);

        browserType = System.getProperty("browser");
        logger.info("Browser Type: {}", browserType);
        eusURL = System.getenv("ET_EUS_API");

        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                WebDriverManager.chromedriver().setup();
            } else {
                WebDriverManager.firefoxdriver().setup();
            }
        }
    }

    @Before
    public void beforeEach() throws MalformedURLException {
        String testName = name.getMethodName();
        logger.info("##### Start test: {}", testName);

        browserVersion = System.getProperty("browserVersion");

        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                driver = new ChromeDriver();
            } else {
                driver = new FirefoxDriver();
            }
        } else {
            DesiredCapabilities caps;
            if (browserType == null || browserType.equals(CHROME)) {
                caps = DesiredCapabilities.chrome();
            } else {
                caps = DesiredCapabilities.firefox();
            }

            if (browserVersion != null) {
                logger.info("Browser Version: {}", browserVersion);
                caps.setVersion(browserVersion);
            }

            caps.setCapability("testName", testName);

            driver = new RemoteWebDriver(new URL(eusURL), caps);
        }

        driver.get(sutUrl);
    }

    @After
    public void afterEach() {
        if (driver != null) {
            driver.quit();
        }

        String testName = name.getMethodName();
        logger.info("##### Finish test: {}", testName);
    }

}
</code>
</pre>
</div>

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Junit4 Multi Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/junit4-web-multiple-browsers-test/target/surefire-reports</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitjava/Dockerfile"><code>elastest/test-etm-alpinegitjava</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/junit4-web-multiple-browsers-test;
    mvn -B -Dbrowser=chrome test;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>
</div> <!-- end of junit4 multi -->


<div id="junit4-single" class="subTestExample">
<p>
You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/junit4-web-single-browser-test">Source Code in GitHub</a>.
This test has been developed in Java using <a target="_blank" href="https://junit.org/junit4">JUnit4</a>.
</p>

<div class="row">
<h5 class="small-subtitle">WebAppTest class</h5>
<pre>
<code class="java">
// Uses a browser for all test
public class WebAppTest extends ElastestBaseTest {

    @Test
    public void addMsgAndClear()
            throws InterruptedException, MalformedURLException {
        Thread.sleep(2000);

        String newTitle = "MessageTitle";
        String newBody = "MessageBody";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        // Added
        logger.info("Checking Message...");
        assertEquals(newTitle, title);
        assertEquals(newBody, body);

        Thread.sleep(1000);

        int titleExist = driver.findElements(By.id("title")).size();
        int bodyExist = driver.findElements(By.id("body")).size();

        try {
            assertNotEquals(0, titleExist);
            assertNotEquals(0, bodyExist);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    @Test
    public void findTitleAndBody()
            throws InterruptedException, MalformedURLException {
        Thread.sleep(2000);

        String newTitle = "MessageTitle";
        String newBody = "MessageBody";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertEquals(newTitle, title);
            assertEquals(newBody, body);

        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    @Test
    public void checkTitleAndBodyNoEmpty()
            throws InterruptedException, MalformedURLException {
        Thread.sleep(2000);

        String newTitle = "";
        String newBody = "";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertNotEquals(newTitle, title);
            assertNotEquals(newBody, body);
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* ********************* */
    /* *** Other methods *** */
    /* ********************* */

    public void addRow(String newTitle, String newBody)
            throws InterruptedException {
        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        Thread.sleep(2000);

        logger.info("Adding Message...");

        driver.findElement(By.id("submit")).click();
    }

    public void clearRows() throws InterruptedException {
        logger.info("Clearing Messages...");
        driver.findElement(By.id("clearSubmit")).click();
        Thread.sleep(1000);
    }
}
</code>

</pre>
</div>

<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">
public class ElastestBaseTest {
    @Rule
    public TestName name = new TestName();

    protected static final Logger logger = LoggerFactory
            .getLogger(ElastestBaseTest.class);

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;
    protected static String eusURL;
    protected static String sutUrl;

    protected static WebDriver driver;

    @BeforeClass
    public static void setupClass() throws MalformedURLException {
        // If first time, init
        if (driver == null) {
            String sutHost = System.getenv("ET_SUT_HOST");
            String sutPort = System.getenv("ET_SUT_PORT");
            String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

            if (sutHost == null) {
                sutUrl = "http://localhost:8080/";
            } else {
                sutPort = sutPort != null ? sutPort : "8080";
                sutProtocol = sutProtocol != null ? sutProtocol : "http";

                sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
            }
            logger.info("Webapp URL: " + sutUrl);

            browserType = System.getProperty("browser");
            logger.info("Browser Type: {}", browserType);
            eusURL = System.getenv("ET_EUS_API");

            if (eusURL == null) {
                if (browserType == null || browserType.equals(CHROME)) {
                    WebDriverManager.chromedriver().setup();
                    driver = new ChromeDriver();
                } else {
                    WebDriverManager.firefoxdriver().setup();
                    driver = new FirefoxDriver();
                }
            } else {
                DesiredCapabilities caps;
                if (browserType == null || browserType.equals(CHROME)) {
                    caps = DesiredCapabilities.chrome();
                } else {
                    caps = DesiredCapabilities.firefox();
                }

                browserVersion = System.getProperty("browserVersion");
                if (browserVersion != null) {
                    logger.info("Browser Version: {}", browserVersion);
                    caps.setVersion(browserVersion);
                }
                driver = new RemoteWebDriver(new URL(eusURL), caps);
            }
        }

        // driver quit when all tests end
        Runtime.getRuntime().addShutdownHook(new Thread() {
            public void run() {
                logger.info("Shutting down browser...");
                driver.quit();
            }
        });
    }

    @Before
    public void beforeEach() throws MalformedURLException {
        String testName = name.getMethodName();

        ((JavascriptExecutor) driver).executeScript(
                "'{\"elastestCommand\": \"startTest\", \"args\": {\"testName\": \""
                        + testName + "\"} }'");

        logger.info("##### Start test: {}", testName);

        driver.get(sutUrl);
    }

    @After
    public void afterEach() {
        String testName = name.getMethodName();
        logger.info("##### Finish test: {}", testName);
    }
}
</code>
</pre>
</div>

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Junit4 Single Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/junit4-web-single-browser-test/target/surefire-reports</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitjava/Dockerfile"><code>elastest/test-etm-alpinegitjava</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/junit4-web-single-browser-test;
    mvn -B -Dbrowser=chrome test;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>
</div> <!-- end of junit4 single -->

</div>

<!-- PYTHON -->
<div id="python" class="testExample badge-tutorial">

<div class="badges-menu sub-badge">
    <span id="python-multi-btn" class="badge badge-default my-badge selected">A Browser for Each</span>
    <span id="python-single-btn" class="badge badge-default my-badge my-badge-disabled">A Browser for All</span>
</div>

<div id="python-multi" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/python-web-multiple-browsers-test">Source Code in GitHub</a>.
This test has been developed in <a target="_blank" href="https://www.python.org/">Python</a>.
</p>

<div class="row">
<h5 class="small-subtitle">TestWebApp class</h5>
<pre>
<code class="python">
import unittest
import os
import sys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time
import xmlrunner
import ElasTestBase


class TestWebApp(ElasTestBase.ElasTestBase):
    def test_check_title_and_body_not_empty(self):
        driver = ElasTestBase.driver
        try:
            time.sleep(2)
            addRow(driver, '', '')
            time.sleep(2)

            title = getElementById(driver, 'title').text
            body = getElementById(driver, 'body').text
            print 'Checking Message...'

            self.assertNotEqual('', title)
            self.assertNotEqual('', body)
        except Exception as e:
            clearData(driver)
            sys.exit(1)
        clearData(driver)

    def test_find_title_and_body(self):
        driver = ElasTestBase.driver
        try:
            time.sleep(2)
            addRow(driver, 'MessageTitle', 'MessageBody')
            time.sleep(2)

            title = getElementById(driver, 'title').text
            body = getElementById(driver, 'body').text
            print 'Checking Message...'

            self.assertEqual('MessageTitle', title)
            self.assertEqual('MessageBody', body)
        except Exception as e:
            clearData(driver)
            sys.exit(1)
        clearData(driver)


def getElementById(driver, id, timeout=10):
    wait = WebDriverWait(driver, timeout)
    return wait.until(EC.presence_of_element_located((By.ID, id)))


def addRow(driver, title, body):
    getElementById(driver, 'title-input').send_keys(title)
    getElementById(driver, 'body-input').send_keys(body)
    print 'Adding Message...'
    getElementById(driver, 'submit').click()


def clearData(driver):
    print 'Clearing Messages...'
    getElementById(driver, 'clearSubmit').click()


if __name__ == '__main__':
    file_path = './testresults'
    if not os.path.exists(file_path):
        os.makedirs(file_path)
    file_name = file_path + '/results.xml'
    with open(file_name, 'wb') as output:
        unittest.main(
            testRunner=xmlrunner.XMLTestRunner(output=output),
            failfast=False, buffer=False, catchbreak=False)
</code>
</pre>
</div>

<div class="row">
<h5 class="small-subtitle">ElasTestBase class</h5>
<pre>
<code class="python">
import unittest
import os
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities


driver = None
sutUrl = None


class ElasTestBase(unittest.TestCase):
    def setUp(self):
        global driver
        global sutUrl
        testName = self._testMethodName
        print '##### Start test: ' + testName

        # os.environ['ET_EUS_API'] = 'http://172.18.0.1:8091/eus/v1/'
        if('ET_EUS_API' in os.environ):
            capabilities = DesiredCapabilities.CHROME
            if('BROWSER' in os.environ):
                if(os.environ['BROWSER'] == 'firefox'):
                    capabilities = DesiredCapabilities.FIREFOX

            capabilities['testName'] = testName

            driver = webdriver.Remote(
                command_executor=os.environ['ET_EUS_API'],
                desired_capabilities=capabilities)
        else:
            driver = webdriver.Chrome('/usr/lib/chromium-browser/chromedriver')

        sutIp = '172.18.0.8'
        if('ET_SUT_HOST' in os.environ):
            sutIp = os.environ['ET_SUT_HOST']

        sutUrl = 'http://' + sutIp + ':8080'
        driver.get(sutUrl)

    def tearDown(self):
        global driver
        testName = self._testMethodName
        print '##### Finish test: ' + testName
        driver.close()
</code>

</pre>
</div>

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Python Multi Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/python-web-multiple-browsers-test/testresults</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitpython/Dockerfile"><code>elastest/test-etm-alpinegitpython</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/python-web-multiple-browsers-test;
    python WebappTest.py;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>
</div><!-- end of python multi -->

<div id="python-single" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/python-web-single-browser-test">Source Code in GitHub</a>.
This test has been developed in <a target="_blank" href="https://www.python.org/">Python</a>.
</p>

<div class="row">
<h5 class="small-subtitle">TestWebApp class</h5>
<pre>
<code class="java">
import unittest
import os
import sys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time
import xmlrunner
import ElasTestBase


class TestWebApp(ElasTestBase.ElasTestBase):
    def test_check_title_and_body_not_empty(self):
        driver = ElasTestBase.driver
        try:
            time.sleep(2)
            addRow(driver, '', '')
            time.sleep(2)

            title = getElementById(driver, 'title').text
            body = getElementById(driver, 'body').text
            print 'Checking Message...'

            self.assertNotEqual('', title)
            self.assertNotEqual('', body)
        except Exception as e:
            clearData(driver)
            sys.exit(1)
        clearData(driver)

    def test_find_title_and_body(self):
        driver = ElasTestBase.driver
        try:
            time.sleep(2)
            addRow(driver, 'MessageTitle', 'MessageBody')
            time.sleep(2)

            title = getElementById(driver, 'title').text
            body = getElementById(driver, 'body').text
            print 'Checking Message...'

            self.assertEqual('MessageTitle', title)
            self.assertEqual('MessageBody', body)
        except Exception as e:
            clearData(driver)
            sys.exit(1)
        clearData(driver)


def getElementById(driver, id, timeout=10):
    wait = WebDriverWait(driver, timeout)
    return wait.until(EC.presence_of_element_located((By.ID, id)))


def addRow(driver, title, body):
    getElementById(driver, 'title-input').send_keys(title)
    getElementById(driver, 'body-input').send_keys(body)
    print 'Adding Message...'
    getElementById(driver, 'submit').click()


def clearData(driver):
    print 'Clearing Messages...'
    getElementById(driver, 'clearSubmit').click()


if __name__ == '__main__':
    file_path = './testresults'
    if not os.path.exists(file_path):
        os.makedirs(file_path)
    file_name = file_path + '/results.xml'
    with open(file_name, 'wb') as output:
        unittest.main(
            testRunner=xmlrunner.XMLTestRunner(output=output),
            failfast=False, buffer=False, catchbreak=False)
</code>
</pre>
</div>

<div class="row">
<h5 class="small-subtitle">ElasTestBase class</h5>
<pre>
<code class="java">
import unittest
import os
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities


driver = None
sutUrl = None


class ElasTestBase(unittest.TestCase):

    @classmethod
    def setUpClass(cls):
        global driver
        global sutUrl

        if('ET_EUS_API' in os.environ):
            capabilities = DesiredCapabilities.CHROME
            if('BROWSER' in os.environ):
                if(os.environ['BROWSER'] == 'firefox'):
                    capabilities = DesiredCapabilities.FIREFOX

            driver = webdriver.Remote(
                command_executor=os.environ['ET_EUS_API'],
                desired_capabilities=capabilities)
        else:
            driver = webdriver.Chrome('/usr/lib/chromium-browser/chromedriver')

        sutIp = '172.18.0.8'
        if('ET_SUT_HOST' in os.environ):
            sutIp = os.environ['ET_SUT_HOST']

        sutUrl = 'http://' + sutIp + ':8080'

    @classmethod
    def tearDownClass(cls):
        driver.close()

    def setUp(self):
        global driver
        global sutUrl
        testName = self._testMethodName
        etScript = '\'{"elastestCommand": "startTest", "args": {"testName": "' + testName + '"}}\''
        driver.execute_script(etScript)
        print '##### Start test: ' + testName

        driver.get(sutUrl)

    def tearDown(self):
        global driver
        testName = self._testMethodName
        print '##### Finish test: ' + testName
</code>
</pre>
</div>

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Python Single Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/python-web-single-browser-test/testresults</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitpython/Dockerfile"><code>elastest/test-etm-alpinegitpython</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/python-web-single-browser-test;
    python WebappTestBrowserForAll.py;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>

</div><!-- End of single python  -->

</div>

<!-- CUCUMBER -->
<div id="cucumber" class="testExample badge-tutorial">
<div class="badges-menu sub-badge">
    <span id="cucumber-multi-btn" class="badge badge-default my-badge selected">A Browser for Each</span>
    <span id="cucumber-single-btn" class="badge badge-default my-badge my-badge-disabled">A Browser for All</span>
</div>

<div id="cucumber-multi" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/cucumber-web-multiple-browsers-test">Source Code in GitHub</a>.
This test has been developed in Java using <a target="_blank" href="https://docs.cucumber.io/">Cucumber</a>.
</p>
<div class="row">
<h5 class="small-subtitle">webapp-test.feature</h5>
<pre>
<code class="java">
Feature: Test WebApp Application
Scenario: Check that the title and body are not empty
    Given app url
    When i add an empty title and body
    Then row with empty title and body added
    
    
Scenario: Find title and body
    Given app url
    When i add a row with title and body
    Then row with the same title and body added
</code>

</pre>
</div>

<div class="row">
<h5 class="small-subtitle">WebAppTestDefinition class</h5>
<pre>
<code class="java">
// With a browser for each test
public class WebAppTestDefinition extends ElastestBaseTest {
    String newTitle;
    String newBody;

    @Before
    public void beforeScenario(Scenario scenario) {
        super.beforeScenario(scenario);
    }

    @After
    public void afterScenario(Scenario scenario) {
        super.afterScenario(scenario);
    }
    
    /* ************************ */
    /* ******** Common ******** */
    /* ************************ */

    @Given("^app url$")
    public void app_url() throws Throwable {
        browserVersion = System.getProperty("browserVersion");

        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                driver = new ChromeDriver();
            } else {
                driver = new FirefoxDriver();
            }
        } else {
            DesiredCapabilities caps;
            if (browserType == null || browserType.equals(CHROME)) {
                caps = DesiredCapabilities.chrome();
            } else {
                caps = DesiredCapabilities.firefox();
            }

            if (browserVersion != null) {
                logger.info("Browser Version: {}", browserVersion);
                caps.setVersion(browserVersion);
            }

            caps.setCapability("testName", currentTestScenarioName);

            driver = new RemoteWebDriver(new URL(eusURL), caps);
        }

        driver.get(sutUrl);
    }

    /* *************************************** */
    /* **** Check title and body no empty **** */
    /* *************************************** */

    @When("^i add an empty title and body$")
    public void i_add_an_empty_title_and_body() throws Throwable {
        Thread.sleep(2000);

        newTitle = "";
        newBody = "";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);
    }

    @Then("^row with empty title and body added$")
    public void row_with_empty_title_and_body_added() throws Throwable {
        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertThat(title, not(equalTo(newTitle)));
            assertThat(body, not(equalTo(newBody)));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* *************************************** */
    /* ********* Find title and body ********* */
    /* *************************************** */

    @When("^i add a row with title and body$")
    public void i_add_a_row_with_title_and_body() throws Throwable {
        Thread.sleep(2000);

        newTitle = "MessageTitle";
        newBody = "MessageBody";

        this.addRow(newTitle, newBody);
        Thread.sleep(2000);
    }

    @Then("^row with the same title and body added$")
    public void row_with_the_same_title_and_body_added() throws Throwable {
        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        // Added
        logger.info("Checking Message...");

        try {
            assertThat(title, equalTo(newTitle));
            assertThat(body, equalTo(newBody));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* ********************** */
    /* *** Common methods *** */
    /* ********************** */

    public void addRow(String newTitle, String newBody)
            throws InterruptedException {
        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        Thread.sleep(2000);

        logger.info("Adding Message...");

        driver.findElement(By.id("submit")).click();
    }

    public void clearRows() throws InterruptedException {
        logger.info("Clearing Messages...");
        driver.findElement(By.id("clearSubmit")).click();
        Thread.sleep(1000);
    }
}
</code>

</pre>
</div>

<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">
public class ElastestBaseTest {
    protected static final Logger logger = LoggerFactory
            .getLogger(WebAppTestDefinition.class);

    protected String currentTestScenarioName;
    protected static String eusURL;
    protected static String sutUrl;

    protected WebDriver driver;

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;


    public void beforeScenario(Scenario scenario) {
        currentTestScenarioName = scenario.getName();

        String sutHost = System.getenv("ET_SUT_HOST");
        String sutPort = System.getenv("ET_SUT_PORT");
        String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

        if (sutHost == null) {
            sutUrl = "http://localhost:8080/";
        } else {
            sutPort = sutPort != null ? sutPort : "8080";
            sutProtocol = sutProtocol != null ? sutProtocol : "http";

            sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
        }
        logger.info("Webapp URL: {}", sutUrl);

        browserType = System.getProperty("browser");
        logger.info("Browser Type: {}", browserType);
        eusURL = System.getenv("ET_EUS_API");

        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                WebDriverManager.chromedriver().setup();
            } else {
                WebDriverManager.firefoxdriver().setup();
            }
        }
        logger.info("##### Start test: {}", currentTestScenarioName);
    }

    public void afterScenario(Scenario scenario) {
        if (driver != null) {
            driver.quit();
        }

        currentTestScenarioName = scenario.getName();
        logger.info("##### Finish test: {}", currentTestScenarioName);
    }

}
</code>
</pre>
</div>

>   As you can see in the code, the @Before and @After hooks are declared in the test, but make use of the implementation of ElastestBaseTest. This is because Cucumber does not allow to extend classes that define hooks or step definition.

<div class="row">
<h5 class="small-subtitle">WebAppTestsRunner class</h5>
<pre>
<code class="java">
@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources/webapp-test.feature", plugin = {
        "html:target/surefire-reports/cucumber-html-report",
        "json:target/surefire-reports/cucumber.json",
        "pretty" }, glue = { "io.elastest.demo.web" })

public class WebAppTestRunner {

}
</code>
</pre>
</div>
<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Cucumber Multi Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/cucumber-web-multiple-browsers-test/target/surefire-reports</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitjava/Dockerfile"><code>elastest/test-etm-alpinegitjava</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/cucumber-web-multiple-browsers-test;
    mvn -B -Dtest=WebAppTestRunner -Dbrowser=chrome test;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>
</div><!-- end of cucumber multi -->

<div id="cucumber-single" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/cucumber-web-single-browser-test">Source Code in GitHub</a>.
This test has been developed in Java using <a target="_blank" href="https://docs.cucumber.io/">Cucumber</a>.
</p>
<div class="row">
<h5 class="small-subtitle">webapp-test.feature</h5>
<pre>
<code class="java">
Feature: Test WebApp Application 
Scenario: Check that the title and body are not empty 
    Given app url
    When i add an empty title and body
    Then row with empty title and body added
    
Scenario: Find title and body 
    Given app url
    When i add a row with title and body
    Then row with the same title and body added
</code>

</pre>
</div>

<div class="row">
<h5 class="small-subtitle">WebAppTestDefinition class</h5>
<pre>
<code class="java">
// With a browser for all test
public class WebAppTestDefinition extends ElastestBaseTest {
    String newTitle;
    String newBody;

    @Before()
    public void beforeFeature() throws MalformedURLException {
        super.beforeFeature();
    }

    @Before
    public void beforeScenario(Scenario scenario) {
        super.beforeScenario(scenario);
    }

    @After
    public void afterScenario(Scenario scenario) {
        super.afterScenario(scenario);
    }

    /* ************************ */
    /* ******** Common ******** */
    /* ************************ */

    @Given("^app url$")
    public void app_url() throws Throwable {
        driver.get(sutUrl);
    }

    /* *************************************** */
    /* **** Check title and body no empty **** */
    /* *************************************** */

    @When("^i add an empty title and body$")
    public void i_add_an_empty_title_and_body() throws Throwable {
        Thread.sleep(2000);

        newTitle = "";
        newBody = "";

        this.addRow(currentTestScenarioName, newTitle, newBody);

        Thread.sleep(2000);
    }

    @Then("^row with empty title and body added$")
    public void row_with_empty_title_and_body_added() throws Throwable {

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertThat(title, not(equalTo(newTitle)));
            assertThat(body, not(equalTo(newBody)));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* *************************************** */
    /* ********* Find title and body ********* */
    /* *************************************** */

    @When("^i add a row with title and body$")
    public void i_add_a_row_with_title_and_body() throws Throwable {
        Thread.sleep(2000);

        newTitle = "MessageTitle";
        newBody = "MessageBody";

        this.addRow(currentTestScenarioName, newTitle, newBody);
        Thread.sleep(2000);
    }

    @Then("^row with the same title and body added$")
    public void row_with_the_same_title_and_body_added() throws Throwable {

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        // Added
        logger.info("Checking Message...");

        try {
            assertThat(title, equalTo(newTitle));
            assertThat(body, equalTo(newBody));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* ********************* */
    /* *** Other methods *** */
    /* ********************* */

    public void addRow(String testName, String newTitle, String newBody)
            throws InterruptedException {
        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        Thread.sleep(2000);

        logger.info("Adding Message...");

        driver.findElement(By.id("submit")).click();
    }

    public void clearRows() throws InterruptedException {
        logger.info("Clearing Messages...");
        driver.findElement(By.id("clearSubmit")).click();
        Thread.sleep(1000);
    }

}
</code>
</pre>
</div>


<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">
public class ElastestBaseTest {
    protected static final Logger logger = LoggerFactory
            .getLogger(WebAppTestDefinition.class);
    protected String currentTestScenarioName;

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;
    protected static String eusURL;
    protected static String sutUrl;

    protected static WebDriver driver;

    public void beforeFeature() throws MalformedURLException {
        // If first time, init
        if (driver == null) {
            String sutHost = System.getenv("ET_SUT_HOST");
            String sutPort = System.getenv("ET_SUT_PORT");
            String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

            if (sutHost == null) {
                sutUrl = "http://localhost:8080/";
            } else {
                sutPort = sutPort != null ? sutPort : "8080";
                sutProtocol = sutProtocol != null ? sutProtocol : "http";

                sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
            }
            logger.info("Webapp URL: {}", sutUrl);

            browserType = System.getProperty("browser");
            logger.info("Browser Type: {}", browserType);
            eusURL = System.getenv("ET_EUS_API");

            if (eusURL == null) {
                if (browserType == null || browserType.equals(CHROME)) {
                    WebDriverManager.chromedriver().setup();
                    driver = new ChromeDriver();
                } else {
                    WebDriverManager.firefoxdriver().setup();
                    driver = new FirefoxDriver();
                }
            } else {
                DesiredCapabilities caps;
                if (browserType == null || browserType.equals(CHROME)) {
                    caps = DesiredCapabilities.chrome();
                } else {
                    caps = DesiredCapabilities.firefox();
                }

                browserVersion = System.getProperty("browserVersion");
                if (browserVersion != null) {
                    logger.info("Browser Version: {}", browserVersion);
                    caps.setVersion(browserVersion);
                }

                caps.setCapability("testName", currentTestScenarioName);

                driver = new RemoteWebDriver(new URL(eusURL), caps);
            }
            
            // driver quit when all tests end
            Runtime.getRuntime().addShutdownHook(new Thread() {
                public void run() {
                    logger.info("Shutting down browser...");
                    driver.quit();
                }
            });
        }
    }

    public void beforeScenario(Scenario scenario) {
        currentTestScenarioName = scenario.getName();
        ((JavascriptExecutor) driver).executeScript(
                "'{\"elastestCommand\": \"startTest\", \"args\": {\"testName\": \""
                        + currentTestScenarioName + "\"} }'");

        logger.info("##### Start test: {}", currentTestScenarioName);
    }

    public void afterScenario(Scenario scenario) {
        currentTestScenarioName = scenario.getName();
        logger.info("##### Finish test: {}", currentTestScenarioName);
    }
}
</code>
</pre>
</div>

>   As you can see in the code, the @Before and @After hooks are declared in the test, but make use of the implementation of ElastestBaseTest. This is because Cucumber does not allow to extend classes that define hooks or step definition.

<div class="row">
<h5 class="small-subtitle">WebAppTestRunner class</h5>
<pre>
<code class="java">
@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources/webapp-test.feature", plugin = {
        "html:target/surefire-reports/cucumber-html-report",
        "json:target/surefire-reports/cucumber.json",
        "pretty" }, glue = { "io.elastest.demo.web" })

public class WebAppTestRunner {
}
</code>
</pre>
</div>

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Cucumber Single Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/cucumber-web-single-browser-test/target/surefire-reports</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitjava/Dockerfile"><code>elastest/test-etm-alpinegitjava</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/cucumber-web-single-browser-test;
    mvn -B -Dtest=WebAppTestRunner -Dbrowser=chrome test;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>

</div><!-- End of single cucumber  -->

</div>

<!-- GAUGE -->
<div id="gauge" class="testExample badge-tutorial">
<div class="badges-menu sub-badge">
    <span id="gauge-multi-btn" class="badge badge-default my-badge selected">A Browser for Each</span>
    <span id="gauge-single-btn" class="badge badge-default my-badge my-badge-disabled">A Browser for All</span>
</div>

<div id="gauge-multi" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/gauge-web-multiple-browsers-test">Source Code in GitHub</a>.
This test has been developed in Java using <a target="_blank" href="https://www.gauge.org/">Gauge</a>.
</p>
<div class="row">
<h5 class="small-subtitle">webapp-test.spec</h5>
<pre>
<code class="java">
Test WebApp Application
======================= 
Test an application
  
  
Check that the title and body are not empty
------------------------------------------- 
    * Navigate to app url
    * Add an empty title and body
    * Check that row with empty title and body has been added

Find title and body
------------------- 
    * Navigate to app url
    * Add a row with title and body
    * Check that row with the same title and body has been added
</code>

</pre>
</div>

<div class="row">
<h5 class="small-subtitle">MultipleWebAppTests class</h5>
<pre>
<code class="java">
public class WebAppTest extends ElastestBaseTest {
    String newTitle;
    String newBody;

    @BeforeScenario()
    public void beforeScenario(ExecutionContext context) {
        super.beforeScenario(context);
    }

    @AfterScenario()
    public void afterScenario(ExecutionContext context) {
        super.afterScenario(context);
    }

    /* ************************ */
    /* ******** Common ******** */
    /* ************************ */
    @Step("Navigate to app url")
    public void navigateToAppUrl() throws MalformedURLException {
        browserVersion = System.getProperty("browserVersion");

        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                driver = new ChromeDriver();
            } else {
                driver = new FirefoxDriver();
            }
        } else {
            DesiredCapabilities caps;
            if (browserType == null || browserType.equals(CHROME)) {
                caps = DesiredCapabilities.chrome();
            } else {
                caps = DesiredCapabilities.firefox();
            }

            if (browserVersion != null) {
                logger.info("Browser Version: {}", browserVersion);
                caps.setVersion(browserVersion);
            }

            caps.setCapability("testName", currentTestScenarioName);

            driver = new RemoteWebDriver(new URL(eusURL), caps);
        }

        driver.get(sutUrl);
    }

    /* *************************************** */
    /* **** Check title and body no empty **** */
    /* *************************************** */

    @Step("Add an empty title and body")
    public void addAnEmptyTitleAndBody() throws InterruptedException {
        Thread.sleep(2000);

        newTitle = "";
        newBody = "";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);
    }

    @Step("Check that row with empty title and body has been added")
    public void checkThatRowWithEmptyTitleAndBodyHasBeenAdded()
            throws InterruptedException {

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertThat(title, not(equalTo(newTitle)));
            assertThat(body, not(equalTo(newBody)));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* *************************************** */
    /* ********* Find title and body ********* */
    /* *************************************** */

    @Step("Add a row with title and body")
    public void addARowWithTitleAndBody() throws InterruptedException {
        Thread.sleep(2000);

        newTitle = "MessageTitle";
        newBody = "MessageBody";

        this.addRow(newTitle, newBody);
        Thread.sleep(2000);
    }

    @Step("Check that row with the same title and body has been added")
    public void CheckThatRowWithTheSameTitleAndBodyHasBeenAdded()
            throws InterruptedException {

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        // Added
        logger.info("Checking Message...");

        try {
            assertThat(title, equalTo(newTitle));
            assertThat(body, equalTo(newBody));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* ********************** */
    /* *** Common methods *** */
    /* ********************** */

    public void addRow(String newTitle, String newBody)
            throws InterruptedException {
        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        Thread.sleep(2000);

        logger.info("Adding Message...");

        driver.findElement(By.id("submit")).click();
    }

    public void clearRows() throws InterruptedException {
        logger.info("Clearing Messages...");
        driver.findElement(By.id("clearSubmit")).click();
        Thread.sleep(1000);
    }
}
</code>
</pre>
</div>

<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">
public class ElastestBaseTest {
    protected static final Logger logger = LoggerFactory
            .getLogger(WebAppTest.class);

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;
    protected static String eusURL;
    protected static String sutUrl;

    protected WebDriver driver;
    protected String currentTestScenarioName;
    
    
    public void beforeScenario(ExecutionContext context) {
        currentTestScenarioName = context.getCurrentScenario().getName();

        String sutHost = System.getenv("ET_SUT_HOST");
        String sutPort = System.getenv("ET_SUT_PORT");
        String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

        if (sutHost == null) {
            sutUrl = "http://localhost:8080/";
        } else {
            sutPort = sutPort != null ? sutPort : "8080";
            sutProtocol = sutProtocol != null ? sutProtocol : "http";

            sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
        }
        logger.info("Webapp URL: {}", sutUrl);

        browserType = System.getProperty("browser");
        logger.info("Browser Type: {}", browserType);
        eusURL = System.getenv("ET_EUS_API");

        if (eusURL == null) {
            if (browserType == null || browserType.equals(CHROME)) {
                WebDriverManager.chromedriver().setup();
            } else {
                WebDriverManager.firefoxdriver().setup();
            }
        }

        logger.info("##### Start test: {}", currentTestScenarioName);
    }

    public void afterScenario(ExecutionContext context) {
        if (driver != null) {
            driver.quit();
        }

        currentTestScenarioName = context.getCurrentScenario().getName();
        logger.info("##### Finish test: {}", currentTestScenarioName);
    }
}
</code>
</pre>
</div>

>   As you can see in the code, the @Before and @After hooks are declared in the test, but make use of the implementation of ElastestBaseTest. This is because Cucumber does not allow to extend classes that define hooks or step definition.

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Gauge Multi Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/gauge-web-multiple-browsers-test/target/surefire-reports</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitjavagauge/Dockerfile"><code>elastest/test-etm-alpinegitjavagauge</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/gauge-web-multiple-browsers-test;
    mvn clean test;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>
</div><!-- end of gauge multi -->

<div id="gauge-single" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/gauge-web-single-browser-test">Source Code in GitHub</a>.
This test has been developed in Java using <a target="_blank" href="https://www.gauge.org/">Gauge</a>.
</p>

<div class="row">
<h5 class="small-subtitle">webapp-test.spec</h5>
<pre>
<code class="java">
Test WebApp Application
======================= 
Test an application
  
  
Check that the title and body are not empty
------------------------------------------- 
    * Navigate to app url
    * Add an empty title and body
    * Check that row with empty title and body has been added

Find title and body
------------------- 
    * Navigate to app url
    * Add a row with title and body
    * Check that row with the same title and body has been added
</code>
</pre>
</div>

<div class="row">
<h5 class="small-subtitle">WebAppTest class</h5>
<pre>
<code class="java">
public class WebAppTest extends ElastestBaseTest {
    String newTitle;
    String newBody;

    @BeforeSpec()
    public void beforeFeature() throws MalformedURLException {
        super.beforeFeature();
    }

    @BeforeScenario()
    public void beforeScenario(ExecutionContext context) {
        super.beforeScenario(context);
    }

    @AfterScenario()
    public void afterScenario(ExecutionContext context) {
        super.afterScenario(context);
    }

    /* ************************ */
    /* ******** Common ******** */
    /* ************************ */

    @Step("Navigate to app url")
    public void navigateToAppUrl() throws MalformedURLException {
        driver.get(sutUrl);
    }

    /* *************************************** */
    /* **** Check title and body no empty **** */
    /* *************************************** */

    @Step("Add an empty title and body")
    public void addAnEmptyTitleAndBody() throws InterruptedException {
        Thread.sleep(2000);

        newTitle = "";
        newBody = "";

        this.addRow(newTitle, newBody);

        Thread.sleep(2000);
    }

    @Step("Check that row with empty title and body has been added")
    public void checkThatRowWithEmptyTitleAndBodyHasBeenAdded()
            throws InterruptedException {

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        logger.info("Checking Message...");

        try {
            assertThat(title, not(equalTo(newTitle)));
            assertThat(body, not(equalTo(newBody)));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* *************************************** */
    /* ********* Find title and body ********* */
    /* *************************************** */

    @Step("Add a row with title and body")
    public void addARowWithTitleAndBody() throws InterruptedException {
        Thread.sleep(2000);

        newTitle = "MessageTitle";
        newBody = "MessageBody";

        this.addRow(newTitle, newBody);
        Thread.sleep(2000);
    }

    @Step("Check that row with the same title and body has been added")
    public void CheckThatRowWithTheSameTitleAndBodyHasBeenAdded()
            throws InterruptedException {

        String title = driver.findElement(By.id("title")).getText();
        String body = driver.findElement(By.id("body")).getText();

        // Added
        logger.info("Checking Message...");

        try {
            assertThat(title, equalTo(newTitle));
            assertThat(body, equalTo(newBody));
        } finally {
            Thread.sleep(2000);
            clearRows();
        }
    }

    /* ********************* */
    /* *** Other methods *** */
    /* ********************* */

    public void addRow(String newTitle, String newBody)
            throws InterruptedException {
        driver.findElement(By.id("title-input")).sendKeys(newTitle);
        driver.findElement(By.id("body-input")).sendKeys(newBody);

        Thread.sleep(2000);

        logger.info("Adding Message...");

        driver.findElement(By.id("submit")).click();
    }

    public void clearRows() throws InterruptedException {
        logger.info("Clearing Messages...");
        driver.findElement(By.id("clearSubmit")).click();
        Thread.sleep(1000);
    }
}
</code>
</pre>
</div>

>   As you can see in the code, the @Before and @After hooks are declared in the test, but make use of the implementation of ElastestBaseTest. This is because Cucumber does not allow to extend classes that define hooks or step definition.

<div class="row">
<h5 class="small-subtitle">ElastestBaseTest class</h5>
<pre>
<code class="java">
public class ElastestBaseTest {
    protected static final Logger logger = LoggerFactory
            .getLogger(WebAppTest.class);

    protected static final String CHROME = "chrome";
    protected static final String FIREFOX = "firefox";

    protected static String browserType;
    protected static String browserVersion;
    protected static String eusURL;
    protected static String sutUrl;

    protected static WebDriver driver;
    String currentTestScenarioName;

    public void beforeFeature() throws MalformedURLException {
        // If first time, init
        if (driver == null) {
            String sutHost = System.getenv("ET_SUT_HOST");
            String sutPort = System.getenv("ET_SUT_PORT");
            String sutProtocol = System.getenv("ET_SUT_PROTOCOL");

            if (sutHost == null) {
                sutUrl = "http://localhost:8080/";
            } else {
                sutPort = sutPort != null ? sutPort : "8080";
                sutProtocol = sutProtocol != null ? sutProtocol : "http";

                sutUrl = sutProtocol + "://" + sutHost + ":" + sutPort;
            }
            logger.info("Webapp URL: {}", sutUrl);

            browserType = System.getProperty("browser");
            logger.info("Browser Type: {}", browserType);
            eusURL = System.getenv("ET_EUS_API");

            if (eusURL == null) {
                if (browserType == null || browserType.equals(CHROME)) {
                    WebDriverManager.chromedriver().setup();
                    driver = new ChromeDriver();
                } else {
                    WebDriverManager.firefoxdriver().setup();
                    driver = new FirefoxDriver();
                }
            } else {
                DesiredCapabilities caps;
                if (browserType == null || browserType.equals(CHROME)) {
                    caps = DesiredCapabilities.chrome();
                } else {
                    caps = DesiredCapabilities.firefox();
                }

                browserVersion = System.getProperty("browserVersion");
                if (browserVersion != null) {
                    logger.info("Browser Version: {}", browserVersion);
                    caps.setVersion(browserVersion);
                }

                caps.setCapability("testName", currentTestScenarioName);

                driver = new RemoteWebDriver(new URL(eusURL), caps);
            }

            // driver quit when all tests end
            Runtime.getRuntime().addShutdownHook(new Thread() {
                public void run() {
                    logger.info("Shutting down browser...");
                    driver.quit();
                }
            });
        }
    }

    public void beforeScenario(ExecutionContext context) {
        currentTestScenarioName = context.getCurrentScenario().getName();
        ((JavascriptExecutor) driver).executeScript(
                "'{\"elastestCommand\": \"startTest\", \"args\": {\"testName\": \""
                        + currentTestScenarioName + "\"} }'");

        logger.info("##### Start test: {}", currentTestScenarioName);
    }

    public void afterScenario(ExecutionContext context) {
        currentTestScenarioName = context.getCurrentScenario().getName();
        logger.info("##### Finish test: {}", currentTestScenarioName);
    }
}
</code>
</pre>
</div>


<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Gauge Single Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/gauge-web-single-browser-test/target/surefire-reports</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>:  <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitjavagauge/Dockerfile"><code>elastest/test-etm-alpinegitjavagauge</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/gauge-web-single-browser-test;
    mvn clean test;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>

</div><!-- End of single gauge  -->

</div>

<!-- PROTRACTOR -->
<div id="protractor" class="testExample badge-tutorial">
<div class="badges-menu sub-badge">
    <span id="protractor-multi-btn" class="badge badge-default my-badge selected">A Browser for Each</span>
    <span id="protractor-single-btn" class="badge badge-default my-badge my-badge-disabled">A Browser for All</span>
</div>

<div id="protractor-multi" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/protractor-web-multiple-browsers-test">Source Code in GitHub</a>.
This test has been developed in JavaScript using <a target="_blank" href="http://protractortest.org/">Protractor</a>.
</p>

<div class="row">
<h5 class="small-subtitle">test-webapp-spec.js</h5>
<pre>
<code class="javascript">
describe('Test WebApp Application', function() {
    it('Check that the title and body are not empty', async () => {
        // Add row
        await addRow('', '');
        await sleep(2000);

        // Check row
        var title = element(by.id('title')).getText();
        var body = element(by.id('body')).getText();

        console.log('Checking Message...');
        expect(title).not.toEqual('');
        expect(body).not.toEqual('');

        clearMessages();
    });

    it('Find title and body', async () => {
        // Add row
        await addRow('MessageTitle', 'MessageBody');
        await sleep(2000);

        // Check row
        var title = element(by.id('title')).getText();
        var body = element(by.id('body')).getText();

        console.log('Checking Message...');

        expect(title).toEqual('MessageTitle');
        expect(body).toEqual('MessageBody');

        clearMessages();
    });
});

async function addRow(title, body) {
    console.log('Inserting data...');
    element(by.id('title-input')).sendKeys(title);
    element(by.id('body-input')).sendKeys(body);
    await sleep(2000);
    console.log('Adding Message...');
    element(by.id('submit')).click();
}

function clearMessages() {
    console.log('Clearing Messages...');
    element(by.id('clearSubmit')).click();
}

function sleep(millis) {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(true);
        }, millis);
    });
}
</code>
</pre>
</div>

<div class="row">
<h5 class="small-subtitle">conf.js</h5>
<pre>
<code class="javascript">
var envs = {
    // The address of a running selenium server.
    seleniumAddress: process.env.ET_EUS_API || 'http://localhost:4444/wd/hub',
    sutUrl: process.env.ET_SUT_HOST ? 'http://' + process.env.ET_SUT_HOST + ':8080' : 'http://172.18.0.8:8080',

    // Capabilities to be passed to the webdriver instance.
    capabilities: {
        browserName: process.env.BROWSER || 'chrome',
        version: process.env.ET_EUS_API ? (process.env.BROWSER_VERSION ? process.env.BROWSER_VERSION : '') : 'ANY',
    },
};


class ElasTestBrowserManager {
    
    constructor() {
        this.firstTime = true;
        /* `_asyncFlow` is a promise.
        * It is a "flow" that we create in `specStarted`.
        * function will wait for the flow to finish before running the next spec.
        * This is not needed since Jasmine 3.0.
         * See https://github.com/jasmine/jasmine/issues/842#issuecomment-336077418
         */
        this._asyncFlow = null;
    }

    jasmineStarted() {
        /* Wait for async tasks triggered by `specStarted`. */
        beforeEach(async () => {
            await this._asyncFlow;
            this._asyncFlow = null;
        });
    }

    specStarted(result) {
        /* Convert async method to promise */
        this._asyncFlow = this.asyncSpecStarted(result);
    }

    async asyncSpecStarted(result) {
        if (!this.firstTime) {
            await browser.restart();
        }
        this.firstTime = false;
        browser.waitForAngularEnabled(false);
        await browser.executeScript('\'{"elastestCommand": "startTest", "args": {"testName": "' + result.description + '"} }\'');
        console.log('##### Start test: ' + result.description);
        browser.get(envs.sutUrl);
    }

    specDone(result) {
        console.log('##### Finish test: ' + result.description);
    }

    jasmineDone(result) {
        browser.close();
    }
}

exports.config = {
    seleniumAddress: envs.seleniumAddress,
    specs: ['test-webapp-spec.js'],
    sutUrl: envs.sutUrl,
    capabilities: envs.capabilities,
    onPrepare: function() {
        var jasmineReporters = require('jasmine-reporters');
        jasmine.getEnv().addReporter(
            new jasmineReporters.JUnitXmlReporter({
                consolidateAll: true,
                savePath: 'testresults',
                // this will produce distinct xml files for each capability
                filePrefix: 'xml-report',
            }),
        );
        jasmine.getEnv().addReporter(new ElasTestBrowserManager());

        browser.waitForAngularEnabled(false);
    },
};
</code>
</pre>
</div>

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Protractor Multi Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/protractor-web-multiple-browsers-test/testresults</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>: <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitnode/Dockerfile"><code>elastest/test-etm-alpinegitnode</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/protractor-web-multiple-browsers-test;
    protractor conf.js;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>

</div><!-- end of multi protractor -->

<div id="protractor-single" class="subTestExample">
<p>You can view the <a target="_blank" href="https://github.com/elastest/demo-projects/tree/master/webapp/protractor-web-single-browser-test">Source Code in GitHub</a>.
This test has been developed in JavaScript using <a target="_blank" href="http://protractortest.org/">Protractor</a>.
</p>

<div class="row">
<h5 class="small-subtitle">test-webapp-spec.js</h5>
<pre>
<code class="javascript">
describe('Test WebApp Application', function() {
    it('Check that the title and body are not empty', async () => {
        // Add row
        await addRow('', '');
        await sleep(2000);

        // Check row
        var title = element(by.id('title')).getText();
        var body = element(by.id('body')).getText();

        console.log('Checking Message...');
        expect(title).not.toEqual('');
        expect(body).not.toEqual('');

        clearMessages();
    });

    it('Find title and body', async () => {
        // Add row
        await addRow('MessageTitle', 'MessageBody');
        await sleep(2000);

        // Check row
        var title = element(by.id('title')).getText();
        var body = element(by.id('body')).getText();

        console.log('Checking Message...');

        expect(title).toEqual('MessageTitle');
        expect(body).toEqual('MessageBody');

        clearMessages();
    });
});

async function addRow(title, body) {
    console.log('Inserting data...');
    element(by.id('title-input')).sendKeys(title);
    element(by.id('body-input')).sendKeys(body);
    await sleep(2000);
    console.log('Adding Message...');
    element(by.id('submit')).click();
}

function clearMessages() {
    console.log('Clearing Messages...');
    element(by.id('clearSubmit')).click();
}

function sleep(millis) {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(true);
        }, millis);
    });
}
</code>
</pre>
</div>

<div class="row">
<h5 class="small-subtitle">conf.js</h5>
<pre>
<code class="javascript">
var envs = {
    // The address of a running selenium server.
    seleniumAddress: process.env.ET_EUS_API || 'http://localhost:4444/wd/hub',
    sutUrl: process.env.ET_SUT_HOST ? 'http://' + process.env.ET_SUT_HOST + ':8080' : 'http://172.17.0.3:8080',

    // Capabilities to be passed to the webdriver instance.
    capabilities: {
        browserName: process.env.BROWSER || 'chrome',
        version: process.env.ET_EUS_API ? (process.env.BROWSER_VERSION ? process.env.BROWSER_VERSION : '') : 'ANY',
    },
};


class ElasTestBrowserManager {
    
    constructor() {
        /* `_asyncFlow` is a promise.
         * It is a "flow" that we create in `specStarted`.
         * function will wait for the flow to finish before running the next spec. 
         * This is not needed since Jasmine 3.0.
         * See https://github.com/jasmine/jasmine/issues/842#issuecomment-336077418
        */
        this._asyncFlow = null;
    }

    jasmineStarted() {
        /* Wait for async tasks triggered by `specStarted`. */
        beforeEach(async () => {
            await this._asyncFlow;
            this._asyncFlow = null;
        });
    }

    specStarted(result) {
        /* Convert async method to promise */
        this._asyncFlow = this.asyncSpecStarted(result);
    }

    async asyncSpecStarted(result) {
        browser.waitForAngularEnabled(false);
        await browser.executeScript('\'{"elastestCommand": "startTest", "args": {"testName": "' + result.description + '"} }\'');
        console.log('##### Start test: ' + result.description);
        browser.get(envs.sutUrl);
    }

    specDone(result) {
        console.log('##### Finish test: ' + result.description);
    }

    jasmineDone(result) {
        browser.close();
    }
}

exports.config = {
    seleniumAddress: envs.seleniumAddress,
    specs: ['test-webapp-spec.js'],
    sutUrl: envs.sutUrl,
    capabilities: envs.capabilities,
    onPrepare: function() {
        var jasmineReporters = require('jasmine-reporters');
        jasmine.getEnv().addReporter(
            new jasmineReporters.JUnitXmlReporter({
                consolidateAll: true,
                savePath: 'testresults',
                // this will produce distinct xml files for each capability
                filePrefix: 'xml-report',
            }),
        );
        jasmine.getEnv().addReporter(new ElasTestBrowserManager());

        browser.waitForAngularEnabled(false);
    },
};
</code>
</pre>
</div>

<h5 class="small-subtitle">TJob Configuration</h5>

<ul>
<li><strong>TJob Name</strong>: can be called as you want, but we will call it <strong><code>Protractor Single Browser Test</code></strong></li>
<li><strong>Test Results Path</strong>: <strong><code>/demo-projects/webapp/protractor-web-single-browser-test/testresults</code></strong></li>
<li><strong>Select a SuT</strong>: <strong><code>WebApp</code></strong></li>
<li><strong>Environment docker image</strong>: <a href="https://github.com/elastest/elastest-torm/blob/master/docker/services/examples/test-etm-alpinegitnode/Dockerfile"><code>elastest/test-etm-alpinegitnode</code></a></li>
<li><strong>Commands</strong>: 
<pre>
    <code>
    git clone https://github.com/elastest/demo-projects;
    cd /demo-projects/webapp/protractor-web-single-browser-test;
    protractor conf.js;
    </code>
</pre>
</li>
<li><strong>Test Support Services</strong>: <strong><code>EUS</code></strong></li>
</ul>

</div> <!-- end of single protractor -->

</div>

<!-- ********************* -->
<!-- ****** Scripts ****** -->
<!-- ********************* -->
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

<script>
$('#junit4-btn').click(function(event) {
  activateBadge('junit4');
  activateSubBadge('junit4-multi');
});
$('#python-btn').click(function(event) {
  activateBadge('python');
  activateSubBadge('python-multi');
});
$('#cucumber-btn').click(function(event) {
  activateBadge('cucumber');
  activateSubBadge('cucumber-multi');
});
$('#gauge-btn').click(function(event) {
  activateBadge('gauge');
  activateSubBadge('gauge-multi');
});
$('#protractor-btn').click(function(event) {
  activateBadge('protractor');
  activateSubBadge('protractor-multi');
});

function activateBadge(sectionName) {
  var disabledClass = 'my-badge-disabled';
  var selectedClass = 'selected';
  
  $('.testExample').hide();
  $('#' + sectionName).show();
  
  // Disable current selected
  $('.main-badge .selected').addClass(disabledClass);
  $('.main-badge .selected').removeClass(selectedClass);
  
  // Select clicked
  $('#' + sectionName + '-btn').addClass(selectedClass);
  $('#' + sectionName + '-btn').removeClass(disabledClass);
}




<!-- SubExamples -->

$('#junit4-multi-btn').click(function(event) {
  activateSubBadge('junit4-multi');
});

$('#junit4-single-btn').click(function(event) {
  activateSubBadge('junit4-single');
});



$('#cucumber-multi-btn').click(function(event) {
  activateSubBadge('cucumber-multi');
});

$('#cucumber-single-btn').click(function(event) {
  activateSubBadge('cucumber-single');
});


$('#gauge-multi-btn').click(function(event) {
  activateSubBadge('gauge-multi');
});

$('#gauge-single-btn').click(function(event) {
  activateSubBadge('gauge-single');
});

$('#python-multi-btn').click(function(event) {
  activateSubBadge('python-multi');
});

$('#python-single-btn').click(function(event) {
  activateSubBadge('python-single');
});

$('#protractor-multi-btn').click(function(event) {
  activateSubBadge('protractor-multi');
});

$('#protractor-single-btn').click(function(event) {
  activateSubBadge('protractor-single');
});

function activateSubBadge(sectionName) {
  var disabledClass = 'my-badge-disabled';
  var selectedClass = 'selected';
  
  $('.subTestExample').hide();
  $('#' + sectionName).show();
  
  // Disable current selected
  $('.sub-badge .selected').addClass(disabledClass);
  $('.sub-badge .selected').removeClass(selectedClass);
  
  // Select clicked
  $('#' + sectionName + '-btn').addClass(selectedClass);
  $('#' + sectionName + '-btn').removeClass(disabledClass);
}

window.onload = function() {
    activateBadge('junit4');
    activateSubBadge('junit4-multi');
}
</script>
