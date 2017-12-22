<div class="range range-xs-left">
<div class="cell-xs-10 cell-lg-6 text-md-left inset-md-right-80 cell-lg-push-1 offset-top-50 offset-lg-top-0">
<h2 id="content" class="h1">Monitoring outside ElasTest</h2>
<div class="offset-top-30 offset-md-top-50">
</div>
</div>
</div>

When ElasTest is executing tests against an already deployed [SuT](../../docs#elastest-core-concepts), it is necessary to explicitly gather monitoring information from it if we want to benefit from ElasTest monitoring and analysis features. The required instrumentation can be done in two different ways:

 - **Automated instrumentation**: If the SuT is a linux box with ssh access, ElasTest will be able to instrument it automatically. ***(Feature coming soon)***
 - **Manual instrumentation**: The SuT admin may install itself the instrumentation agents or configure the platform to send monitoring information to ElasTest.



<h4 class="holder-subtitle link-top">Manual instrumentation</h4>

Manual instrumentation can be done by:

- **Using Beats technology: [Beats](https://www.elastic.co/products/beats)** is a platform for single-purpose data shippers created to work with Logstash and ElasticSearch. There are a lot of beats agents to collect monitoring information from all kinds of sources. For example: [Filebeat](https://www.elastic.co/products/beats/filebeat), [Metricbeat](https://www.elastic.co/products/beats/metricbeat) or [Packetbeat](https://www.elastic.co/products/beats/packetbeat). If you can't find a certain beat agent among the official ones, you can always search [beats bult by the community](https://www.elastic.co/guide/en/beats/libbeat/current/community-beats.html) or [create your own beat](https://www.elastic.co/guide/en/beats/devguide/current/new-beat.html).

- **Sending metrics with http POST requests**: If beats agents doesn’t suit your needs, you can always send monitoring information with http requests. Check out section [Custom monitoring](/monitoring/custom) to learn how.

To create a SuT ready to be manually instrumented, fill the form as shown in the image below on "New SuT" page:

<div class="docs-gallery inline-block">
    <a data-fancybox="gallery-1" href="/docs/monitoring/images/new_SuT_manual_instrumentation_1.png"><img class="img-responsive img-wellcome" src="/docs/monitoring/images/new_SuT_manual_instrumentation_1.png"/></a>
</div>

Once you click on _Save and get monitoring details_  button, the following configuration parameters will be shown:

- **Logstash IP**: The public host (IP or FQDN) in which Logstash is located.
- **Logstash Beats Port**: The port in which Logstash can receive beats monitoring information.
- **HTTP port**: The port in which Logstash listens for http requests with monitoring information.
- **Exec ID**: The execution identification value that has to be included in events sent to Logstash. This value is used to identify what events belongs to this SuT.

We will be using them in the following section.

<div class="docs-gallery more-margin-top inline-block">
    <a data-fancybox="gallery-2" href="/docs/monitoring/images/new_SuT_manual_instrumentation_2.png"><img class="img-responsive img-wellcome" src="/docs/monitoring/images/new_SuT_manual_instrumentation_2.png"/></a>
</div>

<h4 id="send-metrics-with-beats" class="holder-subtitle link-top">Send metrics with Beats</h4>

As ElasTest already includes the required packages for using Beats ([Logstash](https://www.elastic.co/products/logstash) and [ElasticSearch](https://www.elastic.co/products/elasticsearch)), you will only have to follow the [official documentation](https://www.elastic.co/guide/en/beats/libbeat/current/installing-beats.html) for installing and configuring beat agents inside your deployed SuT.

To illustrate this process, let's see a pretty common use case: you have an app deployed somewhere in the cloud, and you want to run a TJob in ElasTest to test some feature of it. And you also want ElasTest to **monitor certain log** that your app produces on some custom path and the **CPU usage on your server**.




<div id="badges-beats" class="badges-menu badges-menu-beats noselectionable link-top">
    <span id="monitor-custom-log-btn" class="badge badge-default my-badge my-big-badge selected">Monitor my<br>custom log</span>
    <span id="monitor-custom-metric-btn" class="badge badge-default my-badge my-big-badge">Monitor my<br>CPU usage</span>
</div>





<div id="monitor-custom-log" class="beats-tutorial">

  <p>Monitor custom log in my server thanks to <a href="https://www.elastic.co/products/beats/filebeat">Filebeat</a></p>

  <h6 style="color: #666666">1. Create a new SuT</h6>

  <p>First of all, create a new SuT inside any Project in your ElasTest dashboard. Do it with <strong>Deployed SuT</strong> and <strong>Instrumented by SuT Admin</strong> options, just as shown in the images above.</p>

  <h6 style="color: #666666">2. Install Filebeat on your SuT machine</h6>

  <p>Let's suppose we have an Ubuntu machine hosting our app. We connect to it through <i>ssh</i> and run the command that <a href="https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html">Step 1 of Filebeat documentation</a> tells us. In this case, by using Debian packages:</p>

  <pre><code class="bash hljs">curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.1.0-amd64.deb
sudo dpkg -i filebeat-6.1.0-amd64.deb</code></pre>

  <h6 style="color: #666666">3. Configure Filebeat agent to suit your needs</h6>

  <p>Take a look to <a href="https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-configuration.html">Step 2 of Filebeat documentation</a>. In our machine the Filebeat configuration file is <code>/etc/filebeat/filebeat.yml</code>. The predefined default value for Filebeat is to gather all logs inside "/var/log/" folder:</p>

  <pre><code class="yml hljs">filebeat.prospectors:
- type: log
  enabled: true
  paths:
    - /var/log/*.log</code></pre>

  <h6 style="color: #666666">4. Configure Filebeat agent to connect to ElasTest</h6>

  <p>Again, just need to take a quick look to <a href="https://www.elastic.co/guide/en/beats/filebeat/current/config-filebeat-logstash.html">Step 3 of Filebeat documentation</a>.</p>

  <p>Our brand new agent has to be configured to send information to the Logstash instance of ElasTest. Logstash Host and beats port have to be used in Filebeat configuration file (again <code>/etc/filebeat/filebeat.yml</code>):</p>

  <pre><code class="yml hljs">#----------------------------- Logstash output --------------------------------
output.logstash:
  hosts: ["127.0.0.1:5044"]</code></pre>

  <p>Where <strong>127.0.0.1</strong> has to be changed to the value of <strong>Logstash IP</strong> and <strong>5044</strong> to the value of <strong>Logstash Beats Port</strong>. To include the <strong>Exec ID</strong> value, it is necessary to add custom fields in config file. Those fields are:</p>

  <pre><code class="yml hljs">fields_under_root: true
fields:
  exec: XX
  component: YY
  stream: ZZ</code></pre>

  <p>The value of the fields are:</p>

  <ul>
    <li><strong>exec</strong>: The value of <strong>Exec ID</strong> shown in ElasTest dashboard when SuT is created.</li>
    <li><strong>component</strong>: <code>sut</code></li>
    <li><strong>stream</strong>: The name of the event stream. For example: <code>my_custom_log</code></li>
  </ul>

  <p>So the final look of <code>filebeat.yml</code> in its simplest form is:
    <pre><code class="yml hljs">#====== "filebeat.yml" to send to ElasTest the logs generated on "/path/to/MY_CUSTOM_LOG.log" ======

filebeat.prospectors:
- type: log
  enabled: true
  paths:
    - /path/to/MY_CUSTOM_LOG.log

output.logstash:
  hosts: ["YOUR_SUT_IP:5044"]

fields_under_root: true
fields:
  exec: XX
  component: sut
  stream: YY
</code></pre>
  </p>

  <p>Whenever you are happy with your Filebeat configuration, run the service:
  <pre><code class="bash hljs">service filebeat start</code></pre>
  </p>

  <p>And that's all. You can now see on ElasTest dashboard these remote logs when running your TJob. To do so click on Monitoring Configuration button:
  </p>

  <div class="docs-gallery more-margin-top inline-block">
    <a data-fancybox="gallery-3" href="/docs/monitoring/images/monitoring_conf.png"><img class="img-responsive img-wellcome" src="/docs/monitoring/images/monitoring_conf.png"/></a>
  </div>

  <p>And select your new log stream. Here it is called <i>mystream</i>, because that's the name we gave it in "filebeat.yml" file on our server.</p>

  <div class="docs-gallery more-margin-top inline-block">
    <a data-fancybox="gallery-3" href="/docs/monitoring/images/my_stream.png"><img class="img-responsive img-wellcome" src="/docs/monitoring/images/my_stream.png"/></a>
  </div>
</div>



<div id="monitor-custom-metric" class="beats-tutorial" hidden>

  <p>Monitor a custom metric in my server thanks to <a href="https://www.elastic.co/products/beats/metricbeat">Metricbeat</a></p>

  <h6 style="color: #666666">1. Create a new SuT</h6>

  <p>First of all, create a new SuT inside any Project in your ElasTest dashboard. Do it with <strong>Deployed SuT</strong> and <strong>Instrumented by SuT Admin</strong> options, just as shown in the images above.</p>

  <h6 style="color: #666666">2. Install Metricbeat on your SuT machine</h6>

  <p>Let's suppose we have an Ubuntu machine hosting our app. We connect to it through <i>ssh</i> and run the command that <a href="https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation.html">Step 1 of Metricbeat documentation</a> tells us. In this case, by using Debian packages:</p>

  <pre><code class="bash hljs">curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-6.1.0-amd64.deb
sudo dpkg -i metricbeat-6.1.0-amd64.deb</code></pre>

  <h6 style="color: #666666">3. Configure Metricbeat agent to suit your needs</h6>

  <p>Take a look to <a href="https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-configuration.html">Step 2 of Metricbeat documentation</a>. In our machine the Metricbeat configuration file is <code>/etc/metricbeat/metricbeat.yml</code>. According to Metricbeat docs, we could configure it like this:</p>

  <pre><code class="yml hljs">metricbeat.modules:
- module: system
  metricsets:
    - cpu
  enabled: true
  period: 1s
  processes: ['.*']
  cpu_ticks: false</code></pre>

  <p>Take a look to the <a href="https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-modules.html">full list of metrics</a> you can send with Metricbeat. Just add all the "metricsets" fields you want inside the proper "module" filed (for example, in <i>module: system</i> you could send <i>- memory</i> or <i>- network</i> appart from <i>- cpu</i></p>

  <h6 style="color: #666666">4. Configure Metricbeat agent to connect to ElasTest</h6>

  <p>Again, just need to take a quick look to <a href="https://www.elastic.co/guide/en/beats/metricbeat/current/config-metricbeat-logstash.html">Step 3 of Metricbeat documentation</a>.</p>

  <p>Our brand new agent has to be configured to send information to the Logstash instance of ElasTest. Logstash Host and beats port have to be used in Metricbeat configuration file (again <code>/etc/metricbeat/metricbeat.yml</code>):</p>

  <pre><code class="yml hljs">#----------------------------- Logstash output --------------------------------
output.logstash:
  hosts: ["127.0.0.1:5044"]</code></pre>

  <p>Where <strong>127.0.0.1</strong> has to be changed to the value of <strong>Logstash IP</strong> and <strong>5044</strong> to the value of <strong>Logstash Beats Port</strong>. To include the <strong>Exec ID</strong> value, it is necessary to add custom fields in config file. Those fields are:</p>

  <pre><code class="yml hljs">fields_under_root: true
fields:
  exec: XX
  component: YY
  stream: ZZ
  stream_type: WW</code></pre>

  <p>The value of the fields are:</p>

  <ul>
    <li><strong>exec</strong>: The value of <strong>Exec ID</strong> shown in ElasTest dashboard when SuT is created.</li>
    <li><strong>component</strong>: <code>sut</code></li>
    <li><strong>stream</strong>: The name of the event stream. For example: <code>my_custom_metrics</code></li>
    <li><strong>stream_type</strong>: <code>composed_metrics</code></li>
  </ul>

  <p>So the final look of <code>metricbeat.yml</code> in its simplest form is:
    <pre><code class="yml hljs">#====== "metricbeat.yml" to send to ElasTest the cpu usage of our server ======

metricbeat.modules:
- module: system
  metricsets:
    - cpu
  enabled: true
  period: 1s
  processes: ['.*']
  cpu_ticks: false

output.logstash:
  hosts: ["YOUR_SUT_IP:5044"]

fields_under_root: true
fields:
  exec: XX
  component: sut
  stream: YY
  stream_type: composed_metrics
</code></pre>
  </p>

  <p>Whenever you are happy with your Filebeat configuration, run the service:
  <pre><code class="bash hljs">service metricbeat start</code></pre>
  </p>

  <p>And that's all. You can now see on ElasTest dashboard these remote metrics when running your TJob. To do so click on Monitoring Configuration button:
  </p>

  <div class="docs-gallery more-margin-top inline-block">
    <a data-fancybox="gallery-4" href="/docs/monitoring/images/monitoring_conf.png"><img class="img-responsive img-wellcome" src="/docs/monitoring/images/monitoring_conf.png"/></a>
  </div>

  <p>And select your new metric stream. Here it is called <i>my_custom_metrics</i>, because that's the name we gave it in "metricbeat.yml" file on our server. And it has as only child a node called <i>system_cpu</i>, since we only configured one "module" filed (<i>system</i>) with only one "metricsets" field (<i>cpu</i>). You can select the sub-metrics you want to be graphed, since for each metric Metricbeat sends varied information.</p>

  <div class="docs-gallery more-margin-top inline-block">
    <a data-fancybox="gallery-4" href="/docs/monitoring/images/my_custom_metrics.png"><img class="img-responsive img-wellcome" src="/docs/monitoring/images/my_custom_metrics.png"/></a>
  </div>

</div>

<script>
$('#monitor-custom-log-btn').click(function(event) {
  activateBadge('monitor-custom-log');
});
$('#monitor-custom-metric-btn').click(function(event) {
  activateBadge('monitor-custom-metric');
});

function activateBadge(sectionName) {
  $('.beats-tutorial').hide();
  $('#' + sectionName).show();
  $('.selected').removeClass('selected');
  $('#' + sectionName + '-btn').addClass('selected');
  window.location.hash = sectionName + '-beats';
}

var badgesSections = [
  "monitor-custom-log-beats",
  "monitor-custom-metric-beats"
];

window.onload = function() {
  var hash = window.location.hash.replace('#', '');
  if (badgesSections.indexOf(hash) > -1) {
    activateBadge(hash.substring(0, hash.indexOf('-beats')));

    // Go to section of beats tutorials if location has proper hash
    $('html, body').animate({
      scrollTop: $("#badges-beats").offset().top
    }, 1);
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
    loop: true,
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
