# Week 2 — Distributed Tracing

## Task status

## Week 2 assignment proof

## Issues faces


## Configuring a Honeycomb Trace!

1. Create a new _Workspace_ and name it _Cruddur_ in your [Honeycomb account](https://ui.honeycomb.io/aggarwal.tanushree-gettingstarted/environments)

2. HoneyComb will automatically assign an **API key** to this workpace. _HoneyComb API keys are workspace spevific. The key we using in our application, determines which HoneyComb workspace it will be logged against.

3. Copy the Cruddur workspace API key from your HoneyComb. We need to set it as a GitPod environment variable. Use the below commands to set the env vars. 

```sh
export HONEYCOMB_API_KEY="mCNpsrEHKPRoNIu0JFn3MC"	
gp env HONEYCOMB_API_KEY="mCNpsrEHKPRoNIu0JFn3MC"	
```

4. Next, we will be adding a few env vars to our **backend-flask docker-compose.yml** file. These will be used to configure OpenTelemetry to send events to Honeycomb
The header `x-honeycomb-team` is your API key. Your service name will be used as the Service Dataset in Honeycomb, which is where data is stored. The service name is specified by `OTEL_SERVICE_NAME` .

```yml
OTEL_SERVICE_NAME: 'backend-flask'
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

5. Add the below packages to your _backend-flask/requirements.txt_ (these packages will be used to intrument our Flask application with OpenTelemetry)

_Note : we can check the requirements for any programming language from the [Honeycomb docs](https://docs.honeycomb.io/getting-data-in/opentelemetry/)
```txt
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

6. Let's install the above packages via the terminal using the below commands

``` sh
cd backend-flask/
pip install -r requirements.txt
```


7. Add the below line to _app.py_ . These updates will create and initialize a tracer and Flask instruction, that will enable sending data to our _Honeycomb workspace_

Add these just below the existing "import" statements
```py
# HoneyComb ---------
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# HoneyComb ---------
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
```
=== Optional block begins ===

Additonally, we can also log the OpenTelemetry logs to our backend app STDOUT. This can be helpful in troubleshooting Honeycomb issues, in case we run into them.

In case you proceed with this configuration, you will need to import the corresponding packages as well

` from opentelemetry.sdk.trace.export import ConsoleSpanExporter, SimpleSpanProcessor `

```py
# Show this in the logs within the backend-flask app (STDOUT)
simple_processor = SimpleSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(simple_processor)

trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```
=== Option block ends ===

Add these just below the existing "app = Flask(__name__)"

```py

# HoneyComb ---------
# Initialize automatic instrumentation with Flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```



8. Run and Test

In the terminal, execute the below commands:

```sh
cd ../frontend-react-js
npm i
cd ..
```

Then,
Right click _docker-compose.yml_ -> _Compose up_ (alternately run _docker compose_ command on the terminal)


9. Add the following in your _gitpod.yml_ , to automatically unlock the frontend and backend ports (so you don not need to unlock them manually from GitPod 'ports' tab every time you compose docker). These will take effect the next time our GitPod workspace is loaded.

```yml
ports:
  - name: frontend
    port: 3000
    onOpen: open-browser
    visibility: public
  - name: backend
    port: 4567
    visibility: public
  - name: xray-daemon
    port: 2000
    visibility: public
```


**Fun side note** : honeycomb-whoami.glitch.me is an app developed by Jessica, it can be used to check whom an API key belongs to 
Glich is a development environment, entirely online. It is used to host small _Node_ projects.
You can "fork" others Glich apps and do a _remix_ , this will spin up a copy of that app, and you can modify it as per your usecase.
Basically, a fun place to start coding for beginners.







backend broken - not able to find backed image
week2_backend_broken

issue identified 
week2_identified_issue


issue fixed 
week2_issued_fixed.png





compose up docker-compose.yml
week2_docker-compose-dockerup-honeycomb.png



Ports are now unlocked automatically
week2_ports_unlocked_automatically.png



Let's verify our trace in the Honeycomb UI

![honeycomb_dataset](week2_honeycomb_dataset_automatically_created.png)

![honeycomb_trace_frontend](assets/week2_honeycomb_trace.png)

We see data in our honeycomb workspace! We can see a few traces available from our Backend API endpoint "/api/activities/home"



Remember we had enabled STDOUT logging for Telemetry? Let's check if any logs were created!
From the Docker extension, click on the backend container, and select "view logs"
Scroll through the logs to check.

We see Telemetry logs! Good job!

![stdout_telemetry_logs](assets/week2_stdout_backend_telemetry_logs.png)


#### Next up,  let's create a for our Trace!
