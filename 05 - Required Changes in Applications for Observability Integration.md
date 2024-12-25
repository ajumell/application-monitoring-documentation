**Required Changes in Applications for Observability Integration**

In the previous article, we discussed the infrastructure needed for deploying our observability stack. Now, let’s explore the changes required in your applications to fully leverage the power of monitoring, logging, tracing, and alerting. These changes ensure that your applications provide meaningful telemetry data to the observability stack, enabling you to monitor performance, trace requests, and diagnose issues effectively.

---

### **Why Application Changes Are Necessary**

Observability tools need your applications to expose key data such as metrics, logs, and traces. Without this, tools like Prometheus, Loki, and Jaeger can’t provide the insights you need. Here’s a breakdown of the required changes:

1. **Expose Metrics**: Applications must provide measurable data such as response times, error rates, and resource usage.
2. **Enable Structured Logging**: Logs should be emitted in a structured format (e.g., JSON) for easier parsing and querying.
3. **Implement Tracing**: Applications need to track and tag requests as they move through different services.
4. **Integrate Telemetry Collectors**: Use OpenTelemetry to unify the collection of logs, metrics, and traces.

---

### **1. Frontend Changes**

#### **Technologies**: React, Vue.js, HTML/CSS/JS

#### **Key Requirements**:
- **Real User Monitoring (RUM)**: Collect metrics like page load times, user interactions, and errors.
- **Tracing**: Track user actions across the frontend, such as button clicks or navigation events.

#### **Steps to Implement**:
1. **Install OpenTelemetry SDK**:
   - For JavaScript frameworks like React or Vue.js:
     ```bash
     npm install @opentelemetry/api @opentelemetry/sdk-trace-web @opentelemetry/exporter-collector
     ```

2. **Instrument the Application**:
   - Initialize the SDK and configure it to capture telemetry data:
     ```javascript
     import { WebTracerProvider } from '@opentelemetry/sdk-trace-web';
     import { CollectorTraceExporter } from '@opentelemetry/exporter-collector';

     const provider = new WebTracerProvider();
     const exporter = new CollectorTraceExporter({
       url: 'http://<otel-collector>:4318/v1/traces',
     });

     provider.addSpanProcessor(new SimpleSpanProcessor(exporter));
     provider.register();
     ```

3. **Capture User Events**:
   - Track user actions like button clicks:
     ```javascript
     const span = provider.getTracer('default').startSpan('button_click');
     span.end();
     ```

4. **Forward Data**:
   - Send traces to the OpenTelemetry Collector for further processing.

#### **Outcome**:
- Monitor frontend performance and user behavior to optimize user experience.

---

### **2. Backend Changes**

#### **Technologies**: Python-Django, Java-Spring Boot

#### **Key Requirements**:
- Expose metrics for system and application performance.
- Emit structured logs for easier analysis.
- Enable distributed tracing to follow requests across services.

#### **Steps to Implement**:

**For Python-Django**:
1. **Expose Metrics**:
   - Install the Prometheus exporter for Django:
     ```bash
     pip install django-prometheus
     ```
   - Add middleware in `settings.py`:
     ```python
     INSTALLED_APPS += ('django_prometheus', )
     MIDDLEWARE += (
         'django_prometheus.middleware.PrometheusBeforeMiddleware',
         'django_prometheus.middleware.PrometheusAfterMiddleware',
     )
     ```
   - Expose metrics at `/metrics`.

2. **Enable Structured Logging**:
   - Use `python-json-logger` to format logs as JSON:
     ```bash
     pip install python-json-logger
     ```
   - Configure logging in `settings.py`:
     ```python
     LOGGING = {
         'version': 1,
         'handlers': {
             'console': {
                 'class': 'logging.StreamHandler',
                 'formatter': 'json',
             },
         },
         'formatters': {
             'json': {
                 'format': '{"message": "%(message)s", "level": "%(levelname)s", "time": "%(asctime)s"}',
             },
         },
     }
     ```

3. **Add Tracing**:
   - Install the OpenTelemetry instrumentation for Django:
     ```bash
     pip install opentelemetry-instrumentation-django
     ```
   - Start the server with instrumentation:
     ```bash
     opentelemetry-instrument --trace_exporter otlp --service_name django_app python manage.py runserver
     ```

**For Java-Spring Boot**:
1. **Expose Metrics**:
   - Add Micrometer and Prometheus dependencies to `pom.xml`:
     ```xml
     <dependency>
       <groupId>io.micrometer</groupId>
       <artifactId>micrometer-registry-prometheus</artifactId>
     </dependency>
     ```
   - Enable metrics export at `/actuator/prometheus`.

2. **Enable Structured Logging**:
   - Use a JSON logging library like Logback:
     ```xml
     <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
       <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder" />
     </appender>
     ```

3. **Add Tracing**:
   - Attach the OpenTelemetry Java agent:
     ```bash
     java -javaagent:/path/to/opentelemetry-javaagent.jar \
          -Dotel.service.name=spring_app \
          -Dotel.exporter.otlp.endpoint=http://<otel-collector>:4317 \
          -jar app.jar
     ```

---

### **3. Middleware and Web Server Changes**

#### **Technologies**: Apache Tomcat, JBoss, WebLogic, Nginx

#### **Key Requirements**:
- Collect middleware and server metrics.
- Forward logs to Loki.

#### **Steps to Implement**:

**For Apache Tomcat**:
1. **Expose Metrics**:
   - Install the JMX Exporter for JVM metrics:
     ```bash
     java -javaagent:/path/to/jmx_prometheus_javaagent.jar=1234:/path/to/config.yaml -jar app.jar
     ```

2. **Forward Logs**:
   - Use Fluentd to collect and forward logs:
     ```yaml
     <source>
       @type tail
       path /path/to/tomcat/logs/*.log
       pos_file /var/log/fluentd/tomcat.pos
       tag tomcat.logs
     </source>
     
     <match tomcat.logs>
       @type loki
       url http://<loki-server>:3100/loki/api/v1/push
     </match>
     ```

**For Nginx**:
1. **Expose Metrics**:
   - Use the Nginx Prometheus Exporter:
     ```bash
     nginx-prometheus-exporter -nginx.scrape-uri=http://127.0.0.1:8080/nginx_status
     ```

2. **Forward Logs**:
   - Configure Nginx to log in JSON format:
     ```nginx
     log_format json_combined '{ "remote_addr": "$remote_addr", "status": "$status", "request": "$request" }';
     access_log /var/log/nginx/access.log json_combined;
     ```
   - Forward logs to Loki using Promtail or Fluentd.