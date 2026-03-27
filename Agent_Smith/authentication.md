# Redis
Redis is used in this project for server-side session management.

## 1. Configuration
- In `app/config.py`, the `Config` class sets up `SESSION_REDIS` using:
```python
     SESSION_REDIS = redis.from_url('redis://localhost:6379')
```  
This tells Flask to use a Redis server running locally (on the default port 6379) as the backend for storing session data.
## 2. Flask App Initialization:
- In `app/__init__.py`, the `create_app` function loads this configuration:
```python
app.config.from_object('app.config.Config')
```  
- The `flask_session` extension is initialized with:
```python
from flask_session import Session
Session(app)
```  
This sets up Flask to store session data in Redis, rather than in cookies or on the filesystem.
## 3. Session Storage:
- When interacting with this app, the session data (such as login state, OAuth tokens, etc.) is stored in Redis. This is more secure and scalable than storing session data in client-side cookies.
## Summary:
- **Redis is not used for general-purpose caching** (like caching API responses or database queries) in the current database.
- **Redis is specifically used as the backend for Flask session storage** via the `flask_session` extension, configured in `app/config.py` and initialized in `app/__init__.py`.  
To use Redis for other types of caching (e.g., for expensive computations or API results), additional code would be needed for that purpose.

# Google OAuth Authentication Process
## 1. Initial Setup:
- The application uses Google OAuth 2.0 for authentication
- It requires a `CLIENT_SECRETS_FILE` (specified in environment variables) containing Google OAuth credentials.
- The application requests several Google API scopes:
    - Google Drive read-only access
    - Gmail read-only access
    - Google Calendar read-only access
    - Google Tasks read and write access
    - Google Docs read-only access
## 2. Authentication Flow:
### a. Starting the Auth Process:
- When a user needs to authenticate, they are directed to the `/authorize` endpoint.
- The endpoint creates a Google OAuth flow and generates an authorization URL
- The current state is stored in the session for security
- If there's a `next` URL parameter, it's stored in the session to redirect back after authentication
### b. Google OAuth Consent Screen:
- The user is redirected to Google's consent screen
- They can choose which permissions to grant to the application
- The consent screen shows the requested scopes (Drive, Gmail, Calendar, etc.)
### c. OAuth Callback
- After the user grants permission, Google redirects back to the `/oauthcallback` endpoint
- The application verifies the state parameter to prevent CSRF attacks.
- The flow exchanges the authorization code for access and refresh tokens.
- The credentials are stored in the session.
### d. JWT Token Generation:
- The application generates a JWT token containing:
    - User ID
    - Email address
    - Expiration time (1 hour)
- The tokwn is signed using the application's secret key
- The token is stored in an HTTP-only cookie for security
## 3. Session Management
- The application uses Flask-Session for server-side session management
- Session data (including OAuth tokens) is stored securely
- The application supports CORS for the frontend application running on localhost:3000
## 4. User Verification
- The `/me` endpoint allows the frontend to verify the user's authentication status
- It checks the JWT token in the cookie
- Returns the user's email and ID if the token is valid
- Returns appropriate error responses for expired or invalid tokens
## 5. Security Features:
- Uses HTTPS (SSL/TLS) for all communications
- Implements CSRF protection through state parameter
- Stores sensitive tokens in HTTP-only cookies
- Uses secure session management
- Implements proper CORS configuration
# To use this authentication system:
1. The frontend should redirect unauthenticated users to `/authorize`
2. After successful authentication, the user will be redirected back with a valid JWT token
3. The frontend can verify the authentication status using the `/me` endpoint
4. The JWT token should be included in subsequent API requests  
The system is designed to be secure and follows OAuth 2.0 best practices while providing a seamless authentication experience for users.
## AWS Functionality
### 1. Configuration:
- The application uses the AWS SDK for Python (boto3) to interact with S3
- AWS credentials are automatically loaded from the `~/.aws/credentials` file using IAM credentials
- Environment variables are used to configure the S3 bucket and prefix:
    - `S3_BUCKET_NAME`: The name of the S3 bucket
    - `S3_PREFIX`: The prefix/folder path within the bucket (currently set to 'uploads', but this is configurable and may change over time)

### 2. Implementation:
- The AWS functionality is implemented in `app/aws.py` as a Flask blueprint (`aws_bp`)
- The blueprint is registered in `app/__init__.py` and provides the `/aws` endpoint

### 3. Functionality:
- **`GET /aws`**: Fetches all objects from the configured S3 bucket with the specified prefix
    - Uses boto3's paginator to handle large numbers of objects efficiently
    - Retrieves each object's content and parses it as JSON
    - Returns a JSON array containing objects with their keys and parsed data
    - Currently configured to pull objects from the 'uploads' prefix, though this prefix can be changed via environment variables

### 4. AWS Credentials Management:
- boto3 automatically loads AWS credentials from the standard AWS credentials file located at `~/.aws/credentials`
- This follows AWS best practices for credential management
- No hardcoded credentials are required in the application code
- The credentials file should contain the appropriate IAM credentials with permissions to read from the specified S3 bucket

### 5. Usage:
- The endpoint can be called to retrieve all JSON objects stored in the S3 bucket under the configured prefix
- The prefix can be updated via the `S3_PREFIX` environment variable to point to different folders/prefixes as needed
- This design allows for flexible data organization within the S3 bucket structure

## Authentication System for Google Services (`app/auth.py`)

Implements Google OAuth 2.0 to securely authenticate users and authorize access to Google services, including Gmail, Drive, Calendar, and Tasks.

### Overview

The `auth.py` file contains the authentication blueprint (`auth_bp`) that handles:
- Google OAuth 2.0 authentication flow
- JWT token generation and validation
- Session management
- User information retrieval

### Key Components

#### 1. **OAuth Scopes**
The application requests the following Google API scopes:
- `https://www.googleapis.com/auth/drive.readonly` - Read-only access to Google Drive
- `https://www.googleapis.com/auth/gmail.readonly` - Full access to Gmail
- `https://www.googleapis.com/auth/calendar.readonly` - Read-only access to Google Calendar
- `https://www.googleapis.com/auth/tasks` - Full access to Google Tasks
- `https://www.googleapis.com/auth/documents.readonly` - Read-only access to Google Docs
- `https://www.googleapis.com/auth/spreadsheets` - Full access to Google Sheets

#### 2. **Authentication Endpoints**

**`GET /authorize`**
- Initiates the OAuth flow
- Creates a Google OAuth flow instance
- Generates an authorization URL
- Stores state in session for CSRF protection
- Optionally stores a `next` URL parameter for post-authentication redirect
- Redirects user to Google's consent screen

**`GET /oauth2callback`**
- Handles the OAuth callback from Google
- Validates the state parameter to prevent CSRF attacks
- Exchanges authorization code for access and refresh tokens
- Stores OAuth credentials in the session
- Generates a JWT token containing user information
- Sets an HTTP-only, secure cookie with the JWT token
- Redirects to the intended destination or default route

**`GET /me`**
- Validates the JWT token from cookies
- Returns user information (email and user ID) if authenticated
- Returns 401 error if token is missing, expired, or invalid

#### 3. **Security Features**

- **CSRF Protection**: Uses state parameter in OAuth flow
- **Secure Cookies**: JWT tokens stored in HTTP-only, secure cookies
- **Session Management**: Server-side sessions stored in Redis
- **HTTPS**: All communication over SSL/TLS
- **Token Expiration**: JWT tokens expire after 1 hour

#### 4. **Authentication Flow**

1. User navigates to `/authorize` endpoint
2. Application redirects to Google's OAuth consent screen
3. User grants permissions
4. Google redirects back to `/oauth2callback` with authorization code
5. Application exchanges code for tokens
6. Application generates JWT token and stores it in a secure cookie
7. User is redirected to their intended destination
8. Subsequent requests include the JWT token in cookies for authentication

### Usage Example

```python
# Frontend initiates authentication
GET /authorize?next=/tasks

# After OAuth callback, user is redirected with JWT cookie set
# Frontend can verify authentication status
GET /me
# Returns: {"email": "user@example.com", "user_id": "123456"}
```

## Configuration

### Session Management

The application uses Flask-Session with Redis for server-side session storage:
- Sessions are stored in Redis (not client-side cookies)
- More secure and scalable than cookie-based sessions
- Configured in `app/config.py`

### CORS Configuration

CORS is configured to allow requests from:
- `https://localhost:3000`
- `https://127.0.0.1:3000`

This enables the frontend React application to communicate with the Flask backend.

## Development Notes

- The application runs in debug mode for development
- SSL certificates are required for HTTPS (configured in `app.py`)
- All session data is stored in Redis
- OAuth credentials are stored in the session after successful authentication
- JWT tokens are used for stateless authentication after initial OAuth flow