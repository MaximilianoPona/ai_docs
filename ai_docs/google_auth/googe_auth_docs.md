# Google Auth Python Library

## Introduction

The Google Auth Python Library provides a comprehensive authentication solution for applications that need to authenticate with Google services. It simplifies the complex process of obtaining and managing credentials for various Google Cloud Platform services and APIs, supporting multiple authentication mechanisms including service accounts, OAuth 2.0, API keys, external accounts (AWS, Azure, OIDC), and Application Default Credentials (ADC). The library handles token acquisition, automatic token refresh, credential verification, and provides transport adapters for making authenticated requests.

This library is the foundational authentication layer for Google Cloud client libraries in Python, enabling server-to-server communication and user-based authentication flows. It supports diverse deployment environments including Google Cloud Platform (Compute Engine, App Engine, Cloud Run), on-premises systems, and third-party cloud platforms. The library implements industry-standard protocols including OAuth 2.0, OpenID Connect, JWT signing/verification, and the OAuth 2.0 Token Exchange specification for workload identity federation.

## API Reference

### Application Default Credentials (ADC)

Automatically discover and load credentials from the environment using Google's Application Default Credentials flow.

```python
import google.auth
from google.auth.transport.requests import Request

# Automatically discover credentials from environment
# Searches in order: GOOGLE_APPLICATION_CREDENTIALS env var,
# gcloud CLI credentials, GCE metadata service, etc.
credentials, project_id = google.auth.default()

# Refresh credentials to obtain access token
request = Request()
credentials.refresh(request)

# Access token is now available
print(f"Access token: {credentials.token}")
print(f"Project ID: {project_id}")

# Use credentials with Google Cloud client libraries
from google.cloud import storage
storage_client = storage.Client(credentials=credentials, project=project_id)
buckets = storage_client.list_buckets()
for bucket in buckets:
    print(bucket.name)
```

### Service Account Credentials

Create credentials using a service account JSON key file for server-to-server authentication.

```python
from google.oauth2 import service_account
from google.auth.transport.requests import Request

# Load credentials from service account JSON file
credentials = service_account.Credentials.from_service_account_file(
    'path/to/service-account.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform']
)

# Or load from JSON data
import json
with open('service-account.json') as f:
    service_account_info = json.load(f)

credentials = service_account.Credentials.from_service_account_info(
    service_account_info,
    scopes=['https://www.googleapis.com/auth/devstorage.read_only']
)

# Refresh to get access token
request = Request()
credentials.refresh(request)
print(f"Access token: {credentials.token}")
print(f"Expires at: {credentials.expiry}")

# Create credentials with custom parameters
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform'],
    subject='user@example.com',  # For domain-wide delegation
    quota_project_id='my-quota-project'
)
```

### Service Account Domain-Wide Delegation

Impersonate users in a Google Workspace domain using domain-wide delegation.

```python
from google.oauth2 import service_account

# Load service account credentials
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/admin.directory.user']
)

# Create delegated credentials for a specific user
delegated_credentials = credentials.with_subject('user@example.com')

# Use delegated credentials to access user's data
from google.auth.transport.requests import Request
request = Request()
delegated_credentials.refresh(request)

# Now you can access APIs on behalf of user@example.com
from googleapiclient.discovery import build
service = build('admin', 'directory_v1', credentials=delegated_credentials)
results = service.users().list(customer='my_customer').execute()
print(f"Users: {results.get('users', [])}")
```

### OAuth 2.0 User Credentials

Authenticate as a user using OAuth 2.0 access and refresh tokens.

```python
from google.oauth2 import credentials

# Create credentials from existing OAuth 2.0 tokens
creds = credentials.Credentials(
    token='ya29.a0AfH6SMC...',  # Access token
    refresh_token='1//0gLNp...',  # Refresh token
    token_uri='https://oauth2.googleapis.com/token',
    client_id='your-client-id.apps.googleusercontent.com',
    client_secret='your-client-secret',
    scopes=['https://www.googleapis.com/auth/drive.readonly']
)

# Refresh if expired
from google.auth.transport.requests import Request
if creds.expired:
    request = Request()
    creds.refresh(request)
    print(f"Refreshed token: {creds.token}")

# Create from authorized user file (gcloud format)
creds = credentials.Credentials.from_authorized_user_file(
    'authorized-user.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform']
)

# Save credentials to file
import json
creds_data = {
    'type': 'authorized_user',
    'client_id': creds.client_id,
    'client_secret': creds.client_secret,
    'refresh_token': creds.refresh_token
}
with open('saved-creds.json', 'w') as f:
    json.dump(creds_data, f)
```

### ID Token Creation from Service Account

Generate OpenID Connect ID tokens for authenticating with IAP or other OIDC-secured services.

```python
from google.oauth2 import service_account
from google.auth.transport.requests import Request

# Create ID token credentials for specific audience
credentials = service_account.IDTokenCredentials.from_service_account_file(
    'service-account.json',
    target_audience='https://my-service-abc123.run.app'
)

# Refresh to obtain ID token
request = Request()
credentials.refresh(request)

# Use ID token in Authorization header
print(f"ID Token: {credentials.token}")

# Make authenticated request to IAP-protected service
import requests
headers = {'Authorization': f'Bearer {credentials.token}'}
response = requests.get('https://my-service-abc123.run.app', headers=headers)
print(f"Response: {response.status_code}")

# Create with custom claims
credentials = service_account.IDTokenCredentials.from_service_account_info(
    service_account_info,
    target_audience='https://example.com',
    additional_claims={'custom_claim': 'value'}
)
```

### ID Token Verification

Verify and decode Google-issued ID tokens for authentication.

```python
from google.oauth2 import id_token
from google.auth.transport import requests

# Create request object for fetching public keys
request = requests.Request()

# Verify OAuth 2.0 ID token from Google
try:
    id_info = id_token.verify_oauth2_token(
        'eyJhbGciOiJSUzI1...',  # The ID token string
        request,
        'your-client-id.apps.googleusercontent.com',  # Expected audience
        clock_skew_in_seconds=10  # Allow 10s clock skew
    )

    # Token is valid - extract claims
    user_id = id_info['sub']
    email = id_info.get('email')
    email_verified = id_info.get('email_verified', False)

    print(f"User ID: {user_id}")
    print(f"Email: {email} (verified: {email_verified})")

except ValueError as e:
    # Token is invalid
    print(f"Invalid token: {e}")

# Verify Firebase token
firebase_info = id_token.verify_firebase_token(
    'eyJhbGciOiJSUzI1...',
    request,
    audience='my-firebase-project'
)

# Verify with custom certificate URL
custom_info = id_token.verify_token(
    'eyJhbGciOiJSUzI1...',
    request,
    audience='https://iap.googleapis.com',
    certs_url='https://www.googleapis.com/oauth2/v3/certs'
)
```

### JWT Encoding and Decoding

Create and verify JSON Web Tokens with RSA or ES256 signatures.

```python
from google.auth import crypt
from google.auth import jwt
import time

# Create a signer from private key
with open('private-key.pem', 'rb') as f:
    private_key_data = f.read()
signer = crypt.RSASigner.from_string(private_key_data)

# Create JWT payload
now = int(time.time())
payload = {
    'iss': 'service@project.iam.gserviceaccount.com',
    'sub': 'service@project.iam.gserviceaccount.com',
    'aud': 'https://example.com',
    'iat': now,
    'exp': now + 3600,  # Expires in 1 hour
    'custom_claim': 'custom_value'
}

# Encode JWT
encoded_jwt = jwt.encode(signer, payload, key_id='key123')
print(f"Encoded JWT: {encoded_jwt.decode('utf-8')}")

# Decode without verification
header, claims, _, _ = jwt._unverified_decode(encoded_jwt)
print(f"Claims: {claims}")

# Verify and decode JWT with public certificates
public_certs = {
    'key123': 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A...'
}
try:
    verified_claims = jwt.decode(
        encoded_jwt,
        certs=public_certs,
        audience='https://example.com'
    )
    print(f"Verified claims: {verified_claims}")
except ValueError as e:
    print(f"Verification failed: {e}")
```

### API Key Credentials

Authenticate using a Google API key for public APIs.

```python
from google.auth import api_key
from google.auth.transport.requests import Request
import requests as req

# Create API key credentials
credentials = api_key.Credentials('AIzaSyD..._your_api_key')

# API keys never expire
print(f"Expired: {credentials.expired}")  # False
print(f"Valid: {credentials.valid}")  # True

# Apply credentials to request headers
headers = {}
credentials.apply(headers)
print(f"Headers: {headers}")  # {'x-goog-api-key': 'AIzaSyD...'}

# Make authenticated request
url = 'https://maps.googleapis.com/maps/api/geocode/json'
params = {'address': '1600 Amphitheatre Parkway, Mountain View, CA'}
response = req.get(url, params=params, headers=headers)
print(f"Response: {response.json()}")

# Use with Google Cloud client (if supported)
credentials.before_request(
    Request(),
    'GET',
    'https://example.googleapis.com/v1/resource',
    headers
)
```

### Impersonated Service Account Credentials

Impersonate a service account using existing credentials with proper IAM permissions.

```python
from google.auth import impersonated_credentials
import google.auth
from google.auth.transport.requests import Request

# Get source credentials (from ADC or other method)
source_credentials, _ = google.auth.default()

# Impersonate target service account
target_scopes = ['https://www.googleapis.com/auth/cloud-platform']
target_credentials = impersonated_credentials.Credentials(
    source_credentials=source_credentials,
    target_principal='target-sa@project.iam.gserviceaccount.com',
    target_scopes=target_scopes,
    lifetime=1800,  # 30 minutes (max 3600)
    delegates=['intermediate-sa@project.iam.gserviceaccount.com']  # Optional delegation chain
)

# Refresh to get token
request = Request()
target_credentials.refresh(request)
print(f"Impersonated token: {target_credentials.token}")

# Use impersonated credentials
from google.cloud import storage
client = storage.Client(credentials=target_credentials)
buckets = list(client.list_buckets())
print(f"Accessible buckets: {len(buckets)}")

# Sign blobs using impersonated credentials
signature = target_credentials.sign_bytes(b'data to sign')
print(f"Signature: {signature.hex()}")
```

### External Account Credentials (Workload Identity Federation)

Authenticate using credentials from external identity providers like AWS, Azure, or OIDC.

```python
import google.auth

# Load external account credentials from JSON file
# For AWS workload identity
credentials, project = google.auth.load_credentials_from_file(
    'aws-external-account.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform']
)

# For Azure workload identity
credentials, project = google.auth.load_credentials_from_file(
    'azure-external-account.json'
)

# For OIDC identity pool
credentials, project = google.auth.load_credentials_from_file(
    'oidc-external-account.json'
)

# Refresh to exchange external token for Google access token
from google.auth.transport.requests import Request
request = Request()
credentials.refresh(request)
print(f"Access token: {credentials.token}")
print(f"Project: {project}")

# Create from configuration dictionary
import json
with open('external-account.json') as f:
    config = json.load(f)

credentials, project = google.auth.load_credentials_from_dict(
    config,
    scopes=['https://www.googleapis.com/auth/devstorage.read_only']
)

# Use with Google Cloud services
from google.cloud import storage
client = storage.Client(credentials=credentials, project=project)
```

### AWS Credentials (External Account)

Authenticate using AWS credentials through workload identity federation.

```python
from google.auth import aws

# Create AWS credentials from configuration
credentials = aws.Credentials.from_info({
    "type": "external_account",
    "audience": "//iam.googleapis.com/projects/123/locations/global/workloadIdentityPools/pool-id/providers/provider-id",
    "subject_token_type": "urn:ietf:params:aws:token-type:aws4_request",
    "token_url": "https://sts.googleapis.com/v1/token",
    "credential_source": {
        "environment_id": "aws1",
        "region_url": "http://169.254.169.254/latest/meta-data/placement/availability-zone",
        "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials",
        "regional_cred_verification_url": "https://sts.{region}.amazonaws.com?Action=GetCallerIdentity&Version=2011-06-15"
    }
})

# Refresh to exchange AWS credentials
from google.auth.transport.requests import Request
request = Request()
credentials.refresh(request)
print(f"Access token from AWS credentials: {credentials.token}")

# Get project ID
project_id = credentials.get_project_id(request)
print(f"Project ID: {project_id}")
```

### Loading Credentials from File

Load credentials from various JSON credential file formats.

```python
import google.auth

# Load any supported credential type from file
# Supports: service_account, authorized_user, external_account,
# external_account_authorized_user, impersonated_service_account
credentials, project_id = google.auth.load_credentials_from_file(
    'credentials.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform'],
    quota_project_id='billing-project'
)

# Check credential type
from google.oauth2 import service_account, credentials as oauth2_credentials
from google.auth import impersonated_credentials

if isinstance(credentials, service_account.Credentials):
    print("Service account credentials")
    print(f"Service account email: {credentials.service_account_email}")
elif isinstance(credentials, oauth2_credentials.Credentials):
    print("OAuth 2.0 user credentials")
    print(f"Client ID: {credentials.client_id}")
elif isinstance(credentials, impersonated_credentials.Credentials):
    print("Impersonated credentials")
    print(f"Target principal: {credentials._target_principal}")

# Load with request object for external accounts
from google.auth.transport.requests import Request
request = Request()
credentials, project_id = google.auth.load_credentials_from_file(
    'external-account.json',
    request=request
)
```

### Credentials with Quota Project

Add a quota project to existing credentials for billing and quota purposes.

```python
from google.oauth2 import service_account

# Create credentials
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform']
)

# Add quota project
credentials_with_quota = credentials.with_quota_project('my-quota-project')

# Use credentials with quota project header
from google.auth.transport.requests import Request
request = Request()
credentials_with_quota.refresh(request)

# Check quota project
print(f"Quota project: {credentials_with_quota.quota_project_id}")

# Works with all credential types
import google.auth
creds, _ = google.auth.default()
creds = creds.with_quota_project('billing-project')
```

### Credential Scopes Management

Manage OAuth 2.0 scopes for credentials that support scoping.

```python
from google.oauth2 import service_account

# Create credentials with scopes
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/devstorage.read_only']
)

# Check if credentials have required scopes
has_scope = credentials.has_scopes(['https://www.googleapis.com/auth/devstorage.read_only'])
print(f"Has scope: {has_scope}")

# Create new credentials with different scopes
new_credentials = credentials.with_scopes([
    'https://www.googleapis.com/auth/cloud-platform',
    'https://www.googleapis.com/auth/devstorage.full_control'
])

# Check if credentials require scopes
if credentials.requires_scopes:
    print("Credentials require scopes to be specified")
    scoped_credentials = credentials.with_scopes([
        'https://www.googleapis.com/auth/cloud-platform'
    ])
else:
    print("Credentials don't require scopes")

# Use default scopes (library-provided) vs user scopes
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/drive.file'],  # User scopes
    default_scopes=['https://www.googleapis.com/auth/drive.readonly']  # Fallback
)
```

### Custom HTTP Transport

Use custom HTTP transport implementations for making authenticated requests.

```python
from google.auth.transport.requests import Request, AuthorizedSession
from google.oauth2 import service_account
import requests

# Create credentials
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json',
    scopes=['https://www.googleapis.com/auth/cloud-platform']
)

# Use Request for manual token management
request = Request()
credentials.refresh(request)

# Make manual authenticated request
headers = {}
credentials.before_request(request, 'GET', 'https://example.com/api', headers)
response = requests.get('https://example.com/api', headers=headers)

# Use AuthorizedSession for automatic credential management
authed_session = AuthorizedSession(credentials)

# Session automatically applies credentials and refreshes tokens
response = authed_session.get('https://storage.googleapis.com/storage/v1/b')
print(f"Response: {response.json()}")

# Configure session with custom settings
session = AuthorizedSession(
    credentials,
    refresh_timeout=30,  # Timeout for refresh requests
    max_refresh_attempts=3
)
response = session.post(
    'https://example.googleapis.com/v1/resource',
    json={'key': 'value'},
    timeout=60
)
```

### Compute Engine Credentials

Obtain credentials from the GCE metadata service when running on Google Compute Engine.

```python
from google.auth import compute_engine
from google.auth.transport.requests import Request

# Create credentials from GCE metadata service
credentials = compute_engine.Credentials()

# Refresh to get access token from metadata service
request = Request()
credentials.refresh(request)
print(f"Access token: {credentials.token}")

# Get service account email from metadata
print(f"Service account: {credentials.service_account_email}")

# Create with custom service account
credentials = compute_engine.Credentials(
    service_account_email='custom-sa@project.iam.gserviceaccount.com'
)

# Use with specific scopes
credentials = compute_engine.Credentials(
    scopes=['https://www.googleapis.com/auth/devstorage.read_only']
)

# Create ID token credentials for GCE
id_credentials = compute_engine.IDTokenCredentials(
    request,
    target_audience='https://example.com',
    use_metadata_identity_endpoint=True
)
```

### Signing and Verification

Sign and verify data using credential signing capabilities.

```python
from google.oauth2 import service_account
from google.auth import crypt

# Create credentials with signing capability
credentials = service_account.Credentials.from_service_account_file(
    'service-account.json'
)

# Sign bytes
data_to_sign = b'important message'
signature = credentials.sign_bytes(data_to_sign)
print(f"Signature: {signature.hex()}")

# Get signer key ID
print(f"Signer key ID: {credentials.signer.key_id}")

# Get service account email (used as key ID)
print(f"Service account: {credentials.signer_email}")

# Verify signature using RSA verifier
from google.auth.crypt import RSAVerifier

# Load public key
with open('public-key.pem', 'rb') as f:
    public_key = f.read()

verifier = RSAVerifier.from_string(public_key)
is_valid = verifier.verify(data_to_sign, signature)
print(f"Signature valid: {is_valid}")

# Create custom signer
from google.auth.crypt import RSASigner
with open('private-key.pem', 'rb') as f:
    private_key = f.read()
signer = RSASigner.from_string(private_key, key_id='custom-key-id')
signature = signer.sign(b'data')
```

## Summary

The Google Auth Python Library provides a unified and robust authentication framework for Python applications interacting with Google Cloud Platform and Google APIs. It supports a comprehensive range of credential types including service accounts, OAuth 2.0 user credentials, API keys, and workload identity federation for external cloud providers. The library's Application Default Credentials (ADC) mechanism intelligently discovers credentials from the runtime environment, making it easy to write portable code that works across development, staging, and production environments without modification.

For production deployments, the library excels at enabling secure patterns such as service account impersonation, domain-wide delegation for Google Workspace, and workload identity federation that eliminates the need for long-lived service account keys. All credential types support automatic token refresh, proper expiration handling, and integration with popular HTTP libraries through transport adapters. Whether building microservices on Cloud Run, batch jobs on Compute Engine, or hybrid cloud applications integrating AWS or Azure identities, this library provides the authentication foundation with battle-tested implementations of OAuth 2.0, OpenID Connect, JWT, and token exchange protocols.
