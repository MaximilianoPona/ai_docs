# Google Gen AI Python SDK

The Google Gen AI Python SDK is a unified interface for interacting with Google's generative AI models through both the Gemini Developer API and Vertex AI platforms. It provides seamless access to advanced AI capabilities including text generation, vision understanding, function calling, embeddings, image generation, video generation, chat sessions, caching, batch processing, and model tuning. The SDK supports both synchronous and asynchronous operations with automatic retry logic, context managers for resource management, and flexible configuration options.

The SDK abstracts away platform differences between Gemini Developer API and Vertex AI, allowing developers to write code once and deploy across both platforms with minimal changes. It includes comprehensive type support through Pydantic models, automatic function calling for tool integration, streaming responses for real-time interactions, and built-in support for multimodal inputs including text, images, video, and audio. The architecture emphasizes developer productivity with intuitive APIs, robust error handling, and extensive documentation.

## Client Initialization

Initialize client for Gemini Developer API or Vertex AI

```python
from google import genai

# Gemini Developer API with explicit API key
client = genai.Client(api_key='your-api-key')

# Gemini Developer API with environment variable
# Set GEMINI_API_KEY or GOOGLE_API_KEY first
import os
os.environ['GEMINI_API_KEY'] = 'your-api-key'
client = genai.Client()

# Vertex AI with explicit credentials
client = genai.Client(
    vertexai=True,
    project='my-project-id',
    location='us-central1'
)

# Vertex AI with environment variables
os.environ['GOOGLE_GENAI_USE_VERTEXAI'] = 'true'
os.environ['GOOGLE_CLOUD_PROJECT'] = 'my-project-id'
os.environ['GOOGLE_CLOUD_LOCATION'] = 'us-central1'
client = genai.Client()

# Using context manager for automatic cleanup
with genai.Client(api_key='your-api-key') as client:
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents='Hello, world!'
    )
    print(response.text)
# Client is automatically closed here
```

## Text Generation

Generate text responses from prompts

```python
from google import genai
from google.genai import types

client = genai.Client(api_key='your-api-key')

# Simple text generation
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='Why is the sky blue?'
)
print(response.text)

# Advanced configuration with temperature, tokens, and safety
response = client.models.generate_content(
    model='gemini-2.0-flash-001',
    contents='Write a creative story about a robot',
    config=types.GenerateContentConfig(
        temperature=0.9,
        max_output_tokens=1000,
        top_p=0.95,
        top_k=40,
        seed=42,
        stop_sequences=['THE END'],
        safety_settings=[
            types.SafetySetting(
                category='HARM_CATEGORY_HATE_SPEECH',
                threshold='BLOCK_ONLY_HIGH'
            )
        ]
    )
)
print(response.text)
print(f"Tokens used: {response.usage_metadata.total_token_count}")
```

## Streaming Generation

Stream responses in real-time for long outputs

```python
from google import genai

client = genai.Client(api_key='your-api-key')

# Synchronous streaming
for chunk in client.models.generate_content_stream(
    model='gemini-2.5-flash',
    contents='Tell me a long story in 500 words'
):
    print(chunk.text, end='', flush=True)

# Asynchronous streaming
import asyncio

async def stream_async():
    async for chunk in await client.aio.models.generate_content_stream(
        model='gemini-2.5-flash',
        contents='Explain quantum physics'
    ):
        print(chunk.text, end='', flush=True)

asyncio.run(stream_async())
```

## Multimodal Input with Images

Process images with text prompts

```python
from google import genai
from google.genai import types

client = genai.Client(api_key='your-api-key')

# Using Google Cloud Storage URI
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents=[
        'What is in this image?',
        types.Part.from_uri(
            file_uri='gs://generativeai-downloads/images/scones.jpg',
            mime_type='image/jpeg'
        )
    ]
)
print(response.text)

# Using local file
with open('local_image.jpg', 'rb') as f:
    image_bytes = f.read()

response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents=[
        'Describe this image in detail',
        types.Part.from_bytes(
            data=image_bytes,
            mime_type='image/jpeg'
        )
    ]
)
print(response.text)
```

## Function Calling with Automatic Execution

Integrate Python functions as tools for the model

```python
from google import genai
from google.genai import types

client = genai.Client(api_key='your-api-key')

def get_current_weather(location: str, unit: str = "fahrenheit") -> str:
    """Returns the current weather for a location.

    Args:
        location: The city and state, e.g. San Francisco, CA
        unit: Temperature unit (fahrenheit or celsius)
    """
    # Simulate API call
    return f"Sunny and 72 degrees {unit} in {location}"

def get_stock_price(symbol: str) -> dict:
    """Get current stock price for a symbol.

    Args:
        symbol: Stock ticker symbol (e.g., GOOGL)
    """
    return {"symbol": symbol, "price": 142.50, "change": "+2.3%"}

# Automatic function calling (default behavior)
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='What is the weather in Boston and the stock price of GOOGL?',
    config=types.GenerateContentConfig(
        tools=[get_current_weather, get_stock_price]
    )
)
print(response.text)  # Model calls functions and responds with results

# Manual function calling (disable automatic execution)
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='What is the weather in Boston?',
    config=types.GenerateContentConfig(
        tools=[get_current_weather],
        automatic_function_calling=types.AutomaticFunctionCallingConfig(
            disable=True
        )
    )
)

# Execute function manually
if response.function_calls:
    for fc in response.function_calls:
        result = get_current_weather(**fc.function_call.args)
        print(f"Function {fc.name} returned: {result}")
```

## JSON Schema Response

Force structured JSON output with schema validation

```python
from google import genai
from google.genai import types
from pydantic import BaseModel

client = genai.Client(api_key='your-api-key')

# Using Pydantic model
class CountryInfo(BaseModel):
    name: str
    population: int
    capital: str
    continent: str
    gdp: int
    official_language: str
    total_area_sq_mi: int

response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='Give me information for Japan',
    config=types.GenerateContentConfig(
        response_mime_type='application/json',
        response_schema=CountryInfo
    )
)
print(response.text)
country_data = response.parsed  # Automatically parsed to CountryInfo

# Using JSON schema dict
user_schema = {
    'type': 'object',
    'properties': {
        'username': {'type': 'string', 'description': "User's unique name"},
        'age': {'type': 'integer', 'minimum': 0, 'maximum': 120},
        'email': {'type': 'string', 'format': 'email'}
    },
    'required': ['username', 'age']
}

response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='Generate a random user profile',
    config=types.GenerateContentConfig(
        response_mime_type='application/json',
        response_json_schema=user_schema
    )
)
print(response.parsed)
```

## Chat Sessions

Multi-turn conversations with history management

```python
from google import genai
from google.genai import types

client = genai.Client(api_key='your-api-key')

# Create chat session
chat = client.chats.create(
    model='gemini-2.5-flash',
    config=types.GenerateContentConfig(
        temperature=0.7,
        system_instruction='You are a helpful coding assistant'
    )
)

# Send messages
response1 = chat.send_message('How do I sort a list in Python?')
print(response1.text)

response2 = chat.send_message('Can you show me a reverse sort example?')
print(response2.text)

# Access history
history = chat.get_history(curated=True)
for item in history:
    print(f"{item.role}: {item.parts[0].text[:50]}...")

# Streaming in chat
for chunk in chat.send_message_stream('Explain decorators'):
    print(chunk.text, end='', flush=True)
```

## File Upload and Management (Gemini API Only)

Upload files for processing

```python
from google import genai
from google.genai import types

client = genai.Client(api_key='your-api-key')

# Upload a file
file = client.files.upload(
    file='document.pdf',
    config=types.UploadFileConfig(
        display_name='My Document',
        mime_type='application/pdf'
    )
)
print(f"Uploaded: {file.name}, URI: {file.uri}")

# Wait for processing if needed
import time
while file.state == 'PROCESSING':
    time.sleep(2)
    file = client.files.get(name=file.name)
print(f"File state: {file.state}")

# Use file in generation
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents=['Summarize this document', file]
)
print(response.text)

# List all files
for f in client.files.list(config={'page_size': 10}):
    print(f"{f.name}: {f.display_name} ({f.size_bytes} bytes)")

# Delete file
client.files.delete(name=file.name)
```

## Context Caching

Cache large contexts for cost-effective repeated queries

```python
from google import genai
from google.genai import types

client = genai.Client(api_key='your-api-key')

# Create cache with multiple documents
cached_content = client.caches.create(
    model='gemini-2.5-flash',
    config=types.CreateCachedContentConfig(
        contents=[
            types.Content(
                role='user',
                parts=[
                    types.Part.from_uri(
                        file_uri='gs://samples/doc1.pdf',
                        mime_type='application/pdf'
                    ),
                    types.Part.from_uri(
                        file_uri='gs://samples/doc2.pdf',
                        mime_type='application/pdf'
                    )
                ]
            )
        ],
        system_instruction='Answer questions about these documents',
        display_name='Document cache',
        ttl='3600s'  # 1 hour
    )
)
print(f"Cache created: {cached_content.name}")

# Use cached content (saves tokens and cost)
response = client.models.generate_content(
    model='gemini-2.5-flash',
    contents='What are the main topics in the documents?',
    config=types.GenerateContentConfig(
        cached_content=cached_content.name
    )
)
print(response.text)

# Update cache TTL
updated = client.caches.update(
    name=cached_content.name,
    config=types.UpdateCachedContentConfig(ttl='7200s')
)

# Delete cache
client.caches.delete(name=cached_content.name)
```

## Batch Processing

Process multiple requests efficiently

```python
from google import genai
from google.genai import types
import time

client = genai.Client(
    vertexai=True,
    project='my-project-id',
    location='us-central1'
)

# Vertex AI: Create batch job with GCS input
batch_job = client.batches.create(
    model='gemini-2.5-flash',
    src='gs://my-bucket/batch-input.jsonl',
    config=types.CreateBatchJobConfig(
        display_name='My Batch Job',
        dest=types.BatchJobDestination(
            gcs_uri='gs://my-bucket/output/'
        )
    )
)
print(f"Batch job created: {batch_job.name}")

# Poll for completion
while batch_job.state not in ['JOB_STATE_SUCCEEDED', 'JOB_STATE_FAILED']:
    print(f"Status: {batch_job.state}")
    time.sleep(30)
    batch_job = client.batches.get(name=batch_job.name)

print(f"Final state: {batch_job.state}")
if batch_job.dest:
    print(f"Output at: {batch_job.dest.gcs_uri}")

# Gemini API: Inline requests
gemini_client = genai.Client(api_key='your-api-key')
batch_job = gemini_client.batches.create(
    model='gemini-2.5-flash',
    src=[
        {'contents': 'Tell me a joke', 'config': {}},
        {'contents': 'What is 2+2?', 'config': {}},
        {'contents': 'Explain gravity', 'config': {}}
    ]
)
```

## Image Generation with Imagen

Generate and edit images

```python
from google import genai
from google.genai import types

client = genai.Client(
    vertexai=True,
    project='my-project-id',
    location='us-central1'
)

# Generate image
response = client.models.generate_images(
    model='imagen-3.0-generate-002',
    prompt='A serene mountain landscape at sunset with a lake',
    config=types.GenerateImagesConfig(
        number_of_images=2,
        include_rai_reason=True,
        output_mime_type='image/jpeg'
    )
)

# Display or save images
for i, img in enumerate(response.generated_images):
    img.image.show()  # Display
    img.image.save(f'generated_{i}.jpg')  # Save

# Upscale image (Vertex AI only)
upscaled = client.models.upscale_image(
    model='imagen-3.0-generate-001',
    image=response.generated_images[0].image,
    upscale_factor='x2',
    config=types.UpscaleImageConfig(
        include_rai_reason=True,
        output_mime_type='image/jpeg'
    )
)
upscaled.generated_images[0].image.show()

# Edit image with mask (Vertex AI only)
edited = client.models.edit_image(
    model='imagen-3.0-capability-001',
    prompt='Clear blue sky',
    reference_images=[
        types.RawReferenceImage(
            reference_id=1,
            reference_image=response.generated_images[0].image
        ),
        types.MaskReferenceImage(
            reference_id=2,
            config=types.MaskReferenceConfig(
                mask_mode='MASK_MODE_BACKGROUND',
                mask_dilation=0
            )
        )
    ],
    config=types.EditImageConfig(
        edit_mode='EDIT_MODE_INPAINT_INSERTION',
        number_of_images=1
    )
)
```

## Video Generation with Veo

Generate videos from text or images

```python
from google import genai
from google.genai import types
import time

client = genai.Client(
    vertexai=True,
    project='my-project-id',
    location='us-central1'
)

# Text to video
operation = client.models.generate_videos(
    model='veo-2.0-generate-001',
    prompt='A cat playing piano in a jazz club',
    config=types.GenerateVideosConfig(
        number_of_videos=1,
        duration_seconds=5,
        enhance_prompt=True
    )
)

# Poll for completion (video generation takes time)
while not operation.done:
    print(f"Generating video... {operation.name}")
    time.sleep(20)
    operation = client.operations.get(operation)

# Get and display video
video = operation.response.generated_videos[0].video
video.show()  # Open in default player
video.save('output_video.mp4')  # Save to file

# Image to video
image = types.Image.from_file('sunset.jpg')
operation = client.models.generate_videos(
    model='veo-2.0-generate-001',
    prompt='Clouds moving across the sky',
    image=image,
    config=types.GenerateVideosConfig(
        number_of_videos=1,
        duration_seconds=5
    )
)
```

## Model Fine-Tuning (Vertex AI Only)

Tune models with custom data

```python
from google import genai
from google.genai import types
import time

client = genai.Client(
    vertexai=True,
    project='my-project-id',
    location='us-central1'
)

# Create tuning job
tuning_job = client.tunings.tune(
    base_model='gemini-2.5-flash',
    training_dataset=types.TuningDataset(
        gcs_uri='gs://my-bucket/training-data.jsonl'
    ),
    config=types.CreateTuningJobConfig(
        epoch_count=3,
        tuned_model_display_name='My Tuned Model'
    )
)
print(f"Tuning job started: {tuning_job.name}")

# Wait for completion
completed_states = {'JOB_STATE_SUCCEEDED', 'JOB_STATE_FAILED', 'JOB_STATE_CANCELLED'}
while tuning_job.state not in completed_states:
    print(f"State: {tuning_job.state}")
    time.sleep(30)
    tuning_job = client.tunings.get(name=tuning_job.name)

if tuning_job.state == 'JOB_STATE_SUCCEEDED':
    # Use tuned model
    response = client.models.generate_content(
        model=tuning_job.tuned_model.endpoint,
        contents='Test the tuned model'
    )
    print(response.text)

    # List tuned models
    for model in client.models.list(config={'query_base': False}):
        print(f"{model.name}: {model.display_name}")
```

## Embeddings

Generate text embeddings for similarity search

```python
from google import genai
from google.genai import types

client = genai.Client(api_key='your-api-key')

# Single text embedding
response = client.models.embed_content(
    model='text-embedding-004',
    contents='Machine learning is fascinating'
)
print(f"Embedding dimensions: {len(response.embeddings[0].values)}")
print(f"First 5 values: {response.embeddings[0].values[:5]}")

# Multiple texts with configuration
response = client.models.embed_content(
    model='text-embedding-004',
    contents=[
        'Artificial intelligence',
        'Deep learning models',
        'Neural networks'
    ],
    config=types.EmbedContentConfig(
        task_type='RETRIEVAL_DOCUMENT',
        output_dimensionality=256
    )
)

# Calculate similarity
import numpy as np
emb1 = np.array(response.embeddings[0].values)
emb2 = np.array(response.embeddings[1].values)
similarity = np.dot(emb1, emb2) / (np.linalg.norm(emb1) * np.linalg.norm(emb2))
print(f"Cosine similarity: {similarity}")
```

## Token Counting

Count tokens before making requests

```python
from google import genai

client = genai.Client(api_key='your-api-key')

# Count tokens for text
response = client.models.count_tokens(
    model='gemini-2.5-flash',
    contents='How many tokens is this text?'
)
print(f"Total tokens: {response.total_tokens}")

# Compute tokens with detailed breakdown (Vertex AI only)
vertex_client = genai.Client(
    vertexai=True,
    project='my-project-id',
    location='us-central1'
)
response = vertex_client.models.compute_tokens(
    model='gemini-2.5-flash',
    contents='Why is the sky blue?'
)
print(f"Tokens: {response.tokens_info}")

# Local tokenization (offline, fast)
tokenizer = genai.LocalTokenizer(model_name='gemini-2.5-flash')
result = tokenizer.count_tokens("What is your name?")
print(f"Local count: {result.total_tokens}")
```

## Error Handling

Handle API errors gracefully

```python
from google import genai
from google.genai import errors

client = genai.Client(api_key='your-api-key')

try:
    response = client.models.generate_content(
        model='invalid-model-name',
        contents='Hello'
    )
except errors.APIError as e:
    print(f"Error code: {e.code}")
    print(f"Error message: {e.message}")
    print(f"Status: {e.status}")
    if e.code == 404:
        print("Model not found")
    elif e.code == 429:
        print("Rate limit exceeded")
    elif e.code == 400:
        print("Invalid request")
```

## Asynchronous Operations

Use async/await for concurrent requests

```python
from google import genai
import asyncio

async def main():
    # Create async client
    async with genai.Client(api_key='your-api-key').aio as aclient:
        # Concurrent requests
        tasks = [
            aclient.models.generate_content(
                model='gemini-2.5-flash',
                contents='What is AI?'
            ),
            aclient.models.generate_content(
                model='gemini-2.5-flash',
                contents='What is ML?'
            ),
            aclient.models.generate_content(
                model='gemini-2.5-flash',
                contents='What is DL?'
            )
        ]

        results = await asyncio.gather(*tasks)
        for i, result in enumerate(results):
            print(f"Response {i+1}: {result.text[:100]}...")

asyncio.run(main())
```

## Summary

The Google Gen AI Python SDK provides a comprehensive, production-ready interface for building AI-powered applications. Primary use cases include chatbots and conversational AI with multi-turn dialogue support, content generation for marketing and creative writing with fine-tuned control over output, document analysis and summarization with large context caching for efficiency, code generation and assistance with function calling capabilities, visual understanding and image processing through multimodal inputs, and enterprise-scale batch processing for high-volume operations. The SDK also supports advanced workflows like RAG (Retrieval-Augmented Generation) with embeddings and vector search, custom model fine-tuning for domain-specific tasks, and video/image generation for creative applications.

Integration patterns emphasize flexibility and scalability. Use synchronous operations for simple scripts and prototypes, asynchronous operations for concurrent processing and web servers, context managers for automatic resource cleanup, batch processing for cost-effective large-scale operations, and context caching for repeated queries on large documents. The SDK integrates seamlessly with popular Python frameworks like FastAPI, Flask, Django, and Streamlit, supports both Pydantic and dict-based configurations for type safety, and provides comprehensive error handling with retry logic. Whether building a simple chatbot or a complex enterprise AI system, the Google Gen AI Python SDK offers the tools and flexibility needed for success.
