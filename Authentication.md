# Authentication & Authorization System

This document provides a comprehensive guide to the JWT-based authentication and role-based authorization system implemented in the TalentHub backend.

## Overview

The authentication system provides:
- JWT-based authentication with access and refresh tokens
- Role-based authorization (USER and HR roles)
- Token refresh mechanism
- Secure logout functionality
- Route protection with guards and decorators

## Architecture

### Core Components

1. **JWT Token Service** (`src/auth/jwt/jwt.service.ts`)
   - Manages access and refresh token generation
   - Handles token verification and refresh
   - Stores tokens in database for security

2. **Authentication Guards**
   - `JwtAuthGuard`: Validates JWT tokens
   - `RolesGuard`: Enforces role-based access control

3. **Decorators**
   - `@Public()`: Marks routes as publicly accessible
   - `@Roles()`: Specifies required roles for route access
   - `@CurrentUser()`: Injects current user data into controllers

4. **Database Models**
   - `AccessToken`: Stores access tokens with expiration
   - `RefreshToken`: Stores refresh tokens with revocation support
   - `User`: Enhanced with role information

## Setup

### Environment Variables

Add these environment variables to your `.env` file:

```env
# JWT Configuration
JWT_ACCESS_SECRET=your-super-secret-access-key-here
JWT_REFRESH_SECRET=your-super-secret-refresh-key-here
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d
```

### Database Models

The system uses the following MongoDB collections:

- `users`: User information with roles
- `accesstokens`: Active access tokens
- `refreshtokens`: Active refresh tokens
- `otps`: Email verification OTPs
- `passwordresets`: Password reset tokens

## API Endpoints

### Authentication Endpoints

#### Register User
```http
POST /auth/register
Content-Type: multipart/form-data

{
  "email": "user@example.com",
  "password": "password123",
  "role": "user",
  "subRole": "jobseeker"
}
```

#### Login
```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "error": false,
  "msg": "Login successful",
  "statusCode": 200,
  "data": {
    "id": "user_id",
    "email": "user@example.com",
    "role": "user",
    "subRole": "jobseeker",
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "tokenType": "Bearer",
      "expiresIn": 900
    }
  }
}
```

#### Refresh Token
```http
POST /auth/refresh-token
Content-Type: application/json

{
  "refreshToken": "your-refresh-token"
}
```

#### Logout
```http
POST /auth/logout
Content-Type: application/json

{
  "refreshToken": "your-refresh-token"
}
```

#### Logout All Devices
```http
POST /auth/logout-all-devices
Authorization: Bearer your-access-token
```

### Protected Endpoints

#### User Profile Management
```http
# Create Profile (USER role only)
POST /user/profile/:userId
Authorization: Bearer your-access-token

# Get Profile (USER can view own, HR can view any)
GET /user/profile/:userId
Authorization: Bearer your-access-token

# Update Profile (USER role only, own profile)
PUT /user/profile/:userId
Authorization: Bearer your-access-token

# Delete Profile (USER can delete own, HR can delete any)
DELETE /user/profile/:userId
Authorization: Bearer your-access-token
```

#### HR Management (HR role only)
```http
# Get All Users
GET /hr/users?page=1&limit=10
Authorization: Bearer your-access-token

# Get User Details
GET /hr/users/:userId
Authorization: Bearer your-access-token

# Approve User Profile
POST /hr/users/:userId/approve
Authorization: Bearer your-access-token

# Reject User Profile
POST /hr/users/:userId/reject
Authorization: Bearer your-access-token
Content-Type: application/json

{
  "reason": "Incomplete information"
}

# HR Dashboard
GET /hr/dashboard
Authorization: Bearer your-access-token
```

## Usage Examples

### Protecting Routes

#### Public Route
```typescript
@Controller('public')
export class PublicController {
  @Get('info')
  @Public() // This route is accessible without authentication
  getPublicInfo() {
    return { message: 'This is public information' };
  }
}
```

#### Role-Based Protection
```typescript
@Controller('admin')
@UseGuards(JwtAuthGuard, RolesGuard)
@ApiBearerAuth()
export class AdminController {
  @Get('users')
  @Roles(ROLE_VALUES.HR) // Only HR can access
  getAllUsers(@CurrentUser() user: JwtPayload) {
    return this.userService.findAll();
  }

  @Get('profile')
  @Roles(ROLE_VALUES.USER, ROLE_VALUES.HR) // Both USER and HR can access
  getProfile(@CurrentUser() user: JwtPayload) {
    return this.userService.findById(user.sub);
  }
}
```

#### Resource Ownership Check
```typescript
@Put('profile/:userId')
@Roles(ROLE_VALUES.USER)
async updateProfile(
  @Param('userId') userId: string,
  @CurrentUser() user: JwtPayload,
) {
  // Users can only update their own profile
  if (user.sub !== userId) {
    throw new ForbiddenException('You can only update your own profile');
  }
  
  return this.userService.updateProfile(userId, updateData);
}
```

### Using Current User Data

```typescript
@Get('me')
@UseGuards(JwtAuthGuard)
async getCurrentUser(@CurrentUser() user: JwtPayload) {
  return {
    id: user.sub,
    email: user.email,
    role: user.role,
    subRole: user.subRole,
  };
}

// Get specific user property
@Get('role')
@UseGuards(JwtAuthGuard)
async getUserRole(@CurrentUser('role') role: string) {
  return { role };
}
```

## Security Features

### Token Management
- **Access Tokens**: Short-lived (15 minutes) for API access
- **Refresh Tokens**: Long-lived (7 days) for token renewal
- **Token Storage**: Tokens stored in database for revocation
- **Automatic Cleanup**: Expired tokens are automatically removed

### Role-Based Access Control
- **USER Role**: Can manage own profile and data
- **HR Role**: Can manage all users and access HR-specific features
- **Flexible Permissions**: Easy to extend with additional roles

### Security Best Practices
- Passwords hashed with Argon2
- JWT tokens signed with strong secrets
- Token revocation on logout
- Rate limiting ready (can be added)
- CORS configuration ready

## Error Handling

The system provides comprehensive error handling:

```typescript
// Authentication errors
{
  "statusCode": 401,
  "message": "Invalid or expired access token",
  "error": "Unauthorized"
}

// Authorization errors
{
  "statusCode": 403,
  "message": "Access denied. Required roles: hr",
  "error": "Forbidden"
}

// Validation errors
{
  "statusCode": 400,
  "message": ["email must be a valid email"],
  "error": "Bad Request"
}
```

## Testing

### Test Authentication Flow

1. **Register a user:**
```bash
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123","role":"user","subRole":"jobseeker"}'
```

2. **Verify email** (check your email for OTP)

3. **Login:**
```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'
```

4. **Use access token for protected routes:**
```bash
curl -X GET http://localhost:3000/user/profile/user_id \
  -H "Authorization: Bearer your-access-token"
```

5. **Refresh token when expired:**
```bash
curl -X POST http://localhost:3000/auth/refresh-token \
  -H "Content-Type: application/json" \
  -d '{"refreshToken":"your-refresh-token"}'
```

## File Structure

```
src/
├── auth/
│   ├── jwt/
│   │   ├── jwt.service.ts          # JWT token management
│   │   └── index.ts
│   ├── guards/
│   │   ├── jwt-auth.guard.ts       # JWT authentication guard
│   │   ├── roles.guard.ts          # Role-based authorization guard
│   │   └── index.ts
│   ├── decorators/
│   │   ├── public.decorator.ts     # Public route decorator
│   │   ├── roles.decorator.ts      # Role requirement decorator
│   │   ├── current-user.decorator.ts # Current user injection
│   │   └── index.ts
│   ├── dto/
│   │   ├── refresh-token.dto.ts    # Token refresh DTOs
│   │   ├── logout.dto.ts           # Logout DTOs
│   │   └── ...                     # Other auth DTOs
│   ├── auth.service.ts             # Authentication service
│   ├── auth.controller.ts          # Authentication controller
│   ├── auth.module.ts              # Authentication module
│   └── auth.config.ts              # JWT configuration
├── user/
│   ├── user.controller.ts          # User management (protected)
│   ├── user.service.ts
│   └── user.module.ts
├── hr/
│   ├── hr.controller.ts            # HR management (HR role only)
│   └── hr.module.ts
└── models/
    ├── auth/
    │   ├── access-token/
    │   │   └── accessToken.schema.ts
    │   └── refresh-token/
    │       └── refreshToken.schema.ts
    └── user/
        └── user.schema.ts          # User model with roles
```

## Extending the System

### Adding New Roles

1. Update the `ROLE_VALUES` enum in `models/user/user.schema.ts`:
```typescript
export enum ROLE_VALUES {
  USER = 'user',
  HR = 'hr',
  ADMIN = 'admin', // New role
}
```

2. Use the new role in controllers:
```typescript
@Roles(ROLE_VALUES.ADMIN)
async adminOnlyFunction() {
  // Admin-only functionality
}
```

### Adding New Protected Routes

1. Apply guards to controller or specific methods:
```typescript
@Controller('protected')
@UseGuards(JwtAuthGuard, RolesGuard)
@ApiBearerAuth()
export class ProtectedController {
  @Get('data')
  @Roles(ROLE_VALUES.USER)
  getData(@CurrentUser() user: JwtPayload) {
    return { data: 'protected data' };
  }
}
```

### Custom Authorization Logic

Create custom guards for complex authorization logic:

```typescript
@Injectable()
export class ResourceOwnerGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const user = request.user;
    const resourceId = request.params.id;
    
    // Custom logic to check if user owns the resource
    return user.sub === resourceId || user.role === ROLE_VALUES.HR;
  }
}
```

This authentication system provides a robust, scalable foundation for role-based access control in your TalentHub application.
