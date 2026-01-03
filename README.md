# Serverless Users API with JWT Authorization

A serverless REST API for user management with JWT-based authentication using AWS Lambda, API Gateway, DynamoDB, and Cognito.

## Architecture Overview

```
┌─────────────┐   JWT Token   ┌──────────────┐   Authorize   ┌─────────────────┐
│   Client    │──────────────▶│ API Gateway  │──────────────▶│ Lambda          │
│             │               │              │               │ Authorizer      │
└─────────────┘               └──────────────┘               └─────────────────┘
                                      │                              │
                                      │ Authorized                   │ Validate JWT
                                      ▼                              ▼
                              ┌──────────────┐                ┌─────────────────┐
                              │ Users Lambda │                │ Cognito         │
                              │ Function     │                │ User Pool       │
                              └──────────────┘                └─────────────────┘
                                      │                              │
                                      │ CRUD Operations              │ User Groups
                                      ▼                              │ (apiAdmins)
                              ┌──────────────┐                      │
                              │ DynamoDB     │◀─────────────────────┘
                              │ Users Table  │
                              └──────────────┘
```

## Request Flow

```
1. Client ──JWT──▶ API Gateway
2. API Gateway ──Token──▶ Lambda Authorizer
3. Lambda Authorizer ──Validate──▶ Cognito User Pool
4. Cognito User Pool ──Policy──▶ Lambda Authorizer
5. Lambda Authorizer ──Allow/Deny──▶ API Gateway
6. API Gateway ──Request──▶ Users Lambda Function
7. Users Lambda Function ──Query/Update──▶ DynamoDB
8. DynamoDB ──Response──▶ Users Lambda Function
9. Users Lambda Function ──Response──▶ API Gateway
10. API Gateway ──Response──▶ Client
```

## Components

- **API Gateway**: REST API with custom Lambda authorizer
- **Lambda Authorizer**: Validates JWT tokens and user permissions
- **Users Lambda**: Handles CRUD operations for users
- **DynamoDB**: Stores user data with pay-per-request billing
- **Cognito User Pool**: Manages authentication and user groups
- **User Pool Client**: OAuth2 configuration for token generation

## API Endpoints

| Method | Path | Description | Authorization |
|--------|------|-------------|---------------|
| GET | `/users` | List all users | JWT Required |
| PUT | `/users` | Create new user | JWT Required |
| GET | `/users/{userid}` | Get specific user | JWT Required |
| PUT | `/users/{userid}` | Update user | JWT Required |
| DELETE | `/users/{userid}` | Delete user | JWT Required |

## Deployment

```bash
sam build
sam deploy --guided
```

## Authentication Flow

1. **Get JWT Token**:
   ```bash
   aws cognito-idp initiate-auth \
     --auth-flow USER_PASSWORD_AUTH \
     --client-id <CLIENT_ID> \
     --auth-parameters USERNAME=<username>,PASSWORD=<password>
   ```

2. **Use Token in API Calls**:
   ```bash
   curl -H "Authorization: Bearer <JWT_TOKEN>" \
        https://<API_ID>.execute-api.<REGION>.amazonaws.com/Prod/users
   ```

## Configuration

- **Runtime**: Python 3.10
- **Memory**: 128MB per function
- **Timeout**: 100 seconds
- **Admin Group**: `apiAdmins` (configurable parameter)
- **Tracing**: X-Ray enabled
- **Billing**: Pay-per-request for DynamoDB