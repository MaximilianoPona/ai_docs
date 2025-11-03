### Install Google Cloud Pub/Sub Client Library (Go)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

Installs the Google Cloud Pub/Sub client library for Go using the go get command.

```go
go get cloud.google.com/go/pubsub
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (Go)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=fr

Installs the Google Cloud Pub/Sub client library for Go using the go get command. This command fetches and installs the specified package.

```go
go get cloud.google.com/go/pubsub

```

--------------------------------

### Vertex AI Agent Builder - Prerequisites

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/retrieve-examples_hl=ja

Installs and initializes the Vertex AI SDK for Python for Example Store.

```APIDOC
## Prerequisites

Before you use the Python samples on this page, install and initialize the Vertex AI SDK for Python in your local Python environment.

1. Run the following command to install the Vertex AI SDK for Python for Example Store.
```
pip install --upgrade google-cloud-aiplatform>=1.87.0
```

2. Use the following code sample to import and initialize the SDK for Example Store.
```python
import vertexai
from vertexai.preview import example_stores

vertexai.init(
  project="PROJECT_ID",
  location="LOCATION"
)
```

Replace the following:
     * PROJECT_ID: Your project ID.
     * LOCATION: Your region. Only `us-central1` is supported.
```

--------------------------------

### Install Vertex AI SDK for Python

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/create-examplestore

Installs the Vertex AI SDK for Python, specifically for Example Store. Ensure you have version 1.87.0 or later.

```bash
pip install --upgrade google-cloud-aiplatform>=1.87.0
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (Node.js)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=fr

Installs the Google Cloud Pub/Sub client library for Node.js using npm. This command downloads and installs the package and its dependencies.

```bash
npm install @google-cloud/pubsub

```

--------------------------------

### Install Google Cloud Pub/Sub client library

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=pt-br

Installs the Google Cloud Pub/Sub client library for various programming languages. This library is required to interact with the Pub/Sub service. Installation methods vary depending on the language and package manager.

```C#
Install-Package Google.Cloud.PubSub.V1 -Pre
```

```Go
go get cloud.google.com/go/pubsub
```

```Maven
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>libraries-bom</artifactId>
      <version>26.70.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-pubsub</artifactId>
  </dependency>

</dependencies>

```

```Gradle
implementation platform('com.google.cloud:libraries-bom:26.71.0')

implementation 'com.google.cloud:google-cloud-pubsub'
```

```sbt
libraryDependencies += "com.google.cloud" % "google-cloud-pubsub" % "1.143.0"
```

```Node.js
npm install @google-cloud/pubsub
```

```PHP
composer require google/cloud-pubsub
```

```Python
pip install --upgrade google-cloud-pubsub
```

```Ruby
gem install google-cloud-pubsub
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (Ruby)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

Installs the Google Cloud Pub/Sub client library for Ruby using the gem command.

```bash
gem install google-cloud-pubsub
```

--------------------------------

### Prerequisites

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/retrieve-examples_hl=ko

Install and initialize the Vertex AI SDK for Python.

```APIDOC
## Prerequisites

Before you use the Python samples on this page, install and initialize the Vertex AI SDK for Python in your local Python environment.

1. Run the following command to install the Vertex AI SDK for Python for Example Store.
```
pip install --upgrade google-cloud-aiplatform>=1.87.0
```

2. Use the following code sample to import and initialize the SDK for Example Store.
```python
import vertexai
from vertexai.preview import example_stores

vertexai.init(
  project="PROJECT_ID",
  location="LOCATION"
)
```

Replace the following:
* PROJECT_ID: Your project ID.
* LOCATION: Your region. Only `us-central1` is supported.
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (C#)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

Installs the Google Cloud Pub/Sub client library for C# using the NuGet package manager. This is a pre-release version.

```powershell
Install-Package Google.Cloud.PubSub.V1 -Pre
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (Node.js)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

Installs the Google Cloud Pub/Sub client library for Node.js using npm.

```bash
npm install @google-cloud/pubsub
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (Python)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

Installs or upgrades the Google Cloud Pub/Sub client library for Python using pip.

```bash
pip install --upgrade google-cloud-pubsub
```

--------------------------------

### Create Pub/Sub Topic and Subscription in Node.js

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=es-419

This Node.js sample demonstrates creating a Google Cloud Pub/Sub topic and a subscription to that topic. It uses the `@google-cloud/pubsub` library and requires Application Default Credentials. The `quickstart` function accepts project ID, topic name, and subscription name, then creates both resources and logs their creation.

```javascript
// Imports the Google Cloud client library
const {PubSub} = require('@google-cloud/pubsub');

async function quickstart(
  projectId = 'your-project-id', // Your Google Cloud Platform project ID
  topicNameOrId = 'my-topic', // Name for the new topic to create
  subscriptionName = 'my-sub', // Name for the new subscription to create
) {
  // Instantiates a client

  const pubsub = new PubSub({projectId});

  // Creates a new topic

  const [topic] = await pubsub.createTopic(topicNameOrId);
  console.log(`Topic ${topic.name} created.`);

  // Creates a subscription on that new topic

  const [subscription] = await topic.createSubscription(subscriptionName);

  // Receive callbacks for new messages on the subscription

  subscription.on('message', message => {
    console.log('Received message:', message.data.toString());
    process.exit(0);
  });

  // Receive callbacks for errors on the subscription

  subscription.on('error', error => {
    console.error('Received error:', error);
    process.exit(1);
  });

  // Send a message to the topic

  await topic.publishMessage({data: Buffer.from('Test message!')});
}
```

--------------------------------

### Initialize Vertex AI SDK for Python

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/create-examplestore_hl=fr

Imports necessary libraries and initializes the Vertex AI SDK for Python with your project ID and location. This setup is crucial for using Example Store functionalities.

```python
import vertexai
from vertexai.preview import example_stores

vertexai.init(
  project="PROJECT_ID",
  location="LOCATION"
)
```

--------------------------------

### Fetch Examples using Python Client Library

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/retrieve-examples

This snippet demonstrates how to fetch examples from Google Agent Builder using the Python client library. It utilizes the `example_store.fetch_examples` method to retrieve examples based on provided IDs. Ensure the `google-cloud-aiplatform` library is installed.

```python
from google.cloud import aiplatform

# TODO: Define PROJECT_ID, LOCATION, EXAMPLE_STORE_ID
# Example:
# PROJECT_ID = "your-project-id"
# LOCATION = "us-central1"
# EXAMPLE_STORE_ID = "your-example-store-id"

client_options = {"api_endpoint": f"{LOCATION}-aiplatform.googleapis.com"}
# Initialize client that will be used to create and manage examples.
aiplatform.init(project=PROJECT_ID, location=LOCATION, client_options=client_options)

# Example store name
example_store_name = f"projects/{PROJECT_ID}/locations/{LOCATION}/exampleStores/{EXAMPLE_STORE_ID}"

# Fetch examples with specific IDs
response = aiplatform.MatchingEngineIndex.list(
    filter="",
    order_by="",
)

# The following request returns examples that include the functions `flight_booking_tool` and `hotel_booking_tool`.
# Note that the `function_names` filter requires the `array_operator` to be `CONTAINS_ANY`.
response = example_store_name.fetch_examples( 
    example_ids=["exampleTypes/stored_contents_example/examples/09b1d383f92c47e7a2583a44ebbc7854"]
)

print(response)
```

--------------------------------

### Install and Import Vertex AI SDK for Python

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/quickstart

Installs the Vertex AI SDK for Python using pip and then imports and initializes the SDK for Example Store usage. Replace 'PROJECT_ID' and 'LOCATION' with your specific project details. 'us-central1' is the only supported location.

```bash
pip install --upgrade google-cloud-aiplatform>=1.87.0
```

```python
import vertexai
from vertexai.preview import example_stores

vertexai.init(
  project="PROJECT_ID",
  location="LOCATION"
)

```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (Ruby)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=fr

Installs the Google Cloud Pub/Sub client library for Ruby using the gem command. This command adds the necessary gem to your Ruby environment.

```bash
gem install google-cloud-pubsub

```

--------------------------------

### C# Example: Create Pub/Sub Topic

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=de

Demonstrates how to create a Google Cloud Pub/Sub topic using the C# client library. Handles cases where the topic already exists.

```csharp
using Google.Cloud.PubSub.V1;
using Grpc.Core;
using System;

public class CreateTopicSample
{
    public Topic CreateTopic(string projectId, string topicId)
    {
        PublisherServiceApiClient publisher = PublisherServiceApiClient.Create();
        var topicName = TopicName.FromProjectTopic(projectId, topicId);
        Topic topic = null;

        try
        {
            topic = publisher.CreateTopic(topicName);
            Console.WriteLine($"Topic {topic.Name} created.");
        }
        catch (RpcException e) when (e.Status.StatusCode == StatusCode.AlreadyExists)
        {
            Console.WriteLine($"Topic {topicName} already exists.");
        }
        return topic;
    }
}
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (PHP Composer)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

Installs the Google Cloud Pub/Sub client library for PHP using Composer.

```bash
composer require google/cloud-pubsub
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (C#)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=fr

Installs the Google Cloud Pub/Sub client library for C# using the NuGet package manager. This is a prerequisite for using Pub/Sub functionalities in C# applications.

```powershell
Install-Package Google.Cloud.PubSub.V1 -Pre

```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (Python)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=fr

Installs or upgrades the Google Cloud Pub/Sub client library for Python using pip. This command ensures you have the latest version installed.

```bash
pip install --upgrade google-cloud-pubsub

```

--------------------------------

### Install Google Cloud Pub/Sub Client Library (PHP)

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=fr

Installs the Google Cloud Pub/Sub client library for PHP using Composer. This command requires the Composer package manager to be installed.

```bash
composer require google/cloud-pubsub

```

--------------------------------

### Create Pub/Sub Topic in Go

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

This Go code example shows how to create a Google Cloud Pub/Sub topic. It utilizes the pubsub client library and requires the project ID and topic ID. Authentication is handled via Application Default Credentials.

```go
// Sample pubsub-quickstart creates a Google Cloud Pub/Sub topic.
package main

import (
	"context"
	"fmt"
	"log"

	"cloud.google.com/go/pubsub/v2"
	"cloud.google.com/go/pubsub/v2/apiv1/pubsubpb"
)

func main() {
	ctx := context.Background()

	// Sets your Google Cloud Platform project ID.
	projectID := "YOUR_PROJECT_ID"

	// Creates a client.
	client, err := pubsub.NewClient(ctx, projectID)
	if err != nil {
		log.Fatalf("Failed to create client: %v", err)
	}
	defer client.Close()

	// Sets the id for the new topic.
	topicID := "my-topic"
	pbTopic := &pubsubpb.Topic{
		Name: fmt.Sprintf("projects/%s/topics/%s", projectID, topicID),
	}

	// Creates the new topic.
	topic, err := client.TopicAdminClient.CreateTopic(ctx, pbTopic)
	if err != nil {
		log.Fatalf("Failed to create topic: %v", err)
	}

	fmt.Printf("Topic %v created.\n", topic)
}

```

--------------------------------

### Set up ADK Runtime with Session and Memory Services in Python

Source: https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/quickstart-adk_hl=pt-br

Illustrates the setup of an ADK Runtime, which orchestrates agents, tools, and callbacks. This example shows how to initialize a Vertex AI Session Service and use it along with the previously configured Memory Service to create an ADK Runner. The runner is then used to execute agent interactions based on user queries.

```python
from google.adk.sessions import VertexAiSessionService
from google.genai import types
import adk # Assuming 'adk' is the correct import for the Runner

# Assuming 'agent_engine_id' and 'memory_service' are already defined from the previous step.
# Replace "PROJECT_ID" and "LOCATION" with your actual project and location.
# Replace "APP_NAME" with your desired application name.

# Initialize Vertex AI Session Service.
session_service = VertexAiSessionService(
    project="PROJECT_ID",
    location="LOCATION",
    agent_engine_id=agent_engine_id
)

app_name="APP_NAME"

# Initialize the ADK Runner with the agent, app name, session service, and memory service.
# Ensure 'agent' is a valid Agent object passed into this context.
runner = adk.Runner(
    agent=agent, # 'agent' needs to be defined elsewhere
    app_name=app_name,
    session_service=session_service,
    memory_service=memory_service
)

# Define a function to call the agent.
def call_agent(query, session, user_id):
  content = types.Content(role='user', parts=[types.Part(text=query)])
  events = runner.run(user_id=user_id, session_id=session, new_message=content)

  for event in events:
      if event.is_final_response():
          final_response = event.content.parts[0].text
          print("Agent Response: ", final_response)
```

--------------------------------

### Install Google Cloud Pub/Sub Client Library

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=de

Installs the Google Cloud Pub/Sub client library for various programming languages using their respective package managers.

```csharp
Install-Package Google.Cloud.PubSub.V1 -Pre
```

```go
go get cloud.google.com/go/pubsub
```

```java-maven
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>libraries-bom</artifactId>
      <version>26.70.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-pubsub</artifactId>
  </dependency>

</dependencies>
```

```java-gradle
implementation platform('com.google.cloud:libraries-bom:26.71.0')

implementation 'com.google.cloud:google-cloud-pubsub'
```

```java-sbt
libraryDependencies += "com.google.cloud" % "google-cloud-pubsub" % "1.143.0"
```

```nodejs
npm install @google-cloud/pubsub
```

```php
composer require google/cloud-pubsub
```

```python
pip install --upgrade google-cloud-pubsub
```

```ruby
gem install google-cloud-pubsub
```

--------------------------------

### Create Example Store Instance using Python Client Library

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/create-examplestore

This Python code snippet demonstrates how to create an Example Store instance using the Vertex AI client library. It requires Vertex AI to be initialized with project and location, and configures the store with a specified embedding model. Ensure you have the Vertex AI client library installed and authenticated.

```python
import vertexai
from vertexai.preview import example_stores

vertexai.init(
    project="PROJECT_ID",
    location="LOCATION"
)

my_example_store = example_stores.ExampleStore.create(
    example_store_config=example_stores.ExampleStoreConfig(
        vertex_embedding_model="EMBEDDING_MODEL"
    )
)

```

--------------------------------

### Search and Use Examples with Google Agent Builder

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/quickstart_hl=fr

This snippet demonstrates how to search for relevant examples from an example store and use them to construct a prompt for the Gemini model. It shows initializing `ExampleStorePrompt`, searching for examples based on a query, and then generating content with a system instruction that includes the formatted examples. Dependencies include the `example_store` object and `genai_types`.

```python
query = "what's the fastest way to get to disney from lax"

# Search for relevant examples.
examples = example_store.search_examples(
  {"stored_contents_example_key": query}, top_k=3)

prompt = ExampleStorePrompt().get_prompt(examples.get("results", []))

model_response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="How do I get to LAX?",
    config=genai_types.GenerateContentConfig(
      system_instruction=prompt,
      tools=[
        genai_types.Tool(function_declarations=[track_flight_status_function])]
  )
)


```

--------------------------------

### Create Pub/Sub Topic - Ruby

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

This Ruby example illustrates creating a Pub/Sub topic using the `google-cloud-pubsub` gem. It requires authentication via Application Default Credentials. The code takes a topic ID, constructs the topic path using the project ID, and then creates the topic.

```ruby
# Imports the Google Cloud client library
require "google/cloud/pubsub"

# Instantiates a client
pubsub = Google::Cloud::PubSub.new
topic_admin = pubsub.topic_admin

# The name for the new topic
# topic_id = "your-topic-id"

# Creates the new topic
topic = topic_admin.create_topic name: pubsub.topic_path(topic_id)

puts "Topic #{topic.name} created."
```

--------------------------------

### Create Session and Call Agent using Agent Engine ADK

Source: https://docs.cloud.google.com/agent-builder/agent-engine/memory-bank/quickstart-adk_hl=de

This example illustrates creating a new session and then invoking the agent with a specific command and session details. It shows how the agent's response is captured.

```python
session = await session_service.create_session(
    app_name=app_name,
    user_id="USER_ID"
)

call_agent("Fix the temperature!", session.id, "USER_ID")
# Agent Response:  Setting temperature to 71 degrees.  Is that correct?
```

--------------------------------

### Create Pub/Sub Topic and Subscription in Node.js

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries

This Node.js code example creates a Google Cloud Pub/Sub topic and a subscription to it. It uses the '@google-cloud/pubsub' library. Remember to set up Application Default Credentials for authentication.

```javascript
// Imports the Google Cloud client library
const {PubSub} = require('@google-cloud/pubsub');

async function quickstart(
  projectId = 'your-project-id', // Your Google Cloud Platform project ID
  topicNameOrId = 'my-topic', // Name for the new topic to create
  subscriptionName = 'my-sub', // Name for the new subscription to create
) {
  // Instantiates a client
  const pubsub = new PubSub({projectId});

  // Creates a new topic
  const [topic] = await pubsub.createTopic(topicNameOrId);
  console.log(`Topic ${topic.name} created.`);

  // Creates a subscription on that new topic
  const [subscription] = await topic.createSubscription(subscriptionName);

  // Receive callbacks for new messages on the subscription
  subscription.on('message', message => {
    console.log('Received message:', message.data.toString());
    process.exit(0);
  });

  // Receive callbacks for errors on the subscription
  subscription.on('error', error => {
    console.error('Received error:', error);
    process.exit(1);
  });

  // Send a message to the topic
  await topic.publishMessage({data: Buffer.from('Test message!')});
}

```

--------------------------------

### Search and Use LLM Examples for Prompting (Python)

Source: https://docs.cloud.google.com/agent-builder/agent-engine/example-store/quickstart

This Python code demonstrates how to search for relevant examples from an example store and use them to construct a prompt for an LLM. It queries an `example_store`, formats the retrieved examples using `ExampleStorePrompt`, and then uses the generated prompt as a system instruction when calling the `generate_content` method.

```python
query = "what's the fastest way to get to disney from lax"

# Search for relevant examples.
examples = example_store.search_examples(
  {"stored_contents_example_key": query}, top_k=3)

prompt = ExampleStorePrompt().get_prompt(examples.get("results", []))

model_response = client.models.generate_content(
    model="gemini-2.0-flash",
    contents="How do I get to LAX?",
    config=genai_types.GenerateContentConfig(
      system_instruction=prompt,
      tools=[
        genai_types.Tool(function_declarations=[track_flight_status_function])]
  )
)

```

--------------------------------

### List and Get Vertex AI Sandboxes

Source: https://docs.cloud.google.com/agent-builder/agent-engine/code-execution/quickstart_hl=pt-br

These Python examples demonstrate how to list all sandboxes associated with an Agent Engine instance and retrieve details for a specific sandbox. The `list` method returns an iterable of sandbox objects, while the `get` method fetches a single sandbox by its name. The output includes creation time, display name, name, spec, state, and update time.

```python
sandboxes = client.agent_engines.sandboxes.list(name=agent_engine_name)

for sandbox in sandboxes:
    pprint.pprint(sandbox)

```

```python
sandbox = client.agent_engines.sandboxes.get(name=sandbox_name)

pprint.pprint(sandbox)

```

--------------------------------

### Initialize Pub/Sub Client in Ruby

Source: https://docs.cloud.google.com/agent-builder/quickstart-client-libraries_hl=fr

This Ruby snippet demonstrates how to initialize a Google Cloud Pub/Sub client and access its topic administration interface. It requires the `google/cloud/pubsub` gem. The code instantiates a client and then gets the `topic_admin` object, preparing for topic creation operations.

```ruby
# Imports the Google Cloud client library
require "google/cloud/pubsub"

# Instantiates a client
pubsub = Google::Cloud::PubSub.new
topic_admin = pubsub.topic_admin

# The name for the new topic
# topic_id = "your-topic-id"


```