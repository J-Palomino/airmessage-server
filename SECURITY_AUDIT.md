# AirMessage Server - Telemetry and External Communications Security Audit

This document provides a comprehensive audit of all external communications, telemetry, and data collection mechanisms in the AirMessage Server application.

## Executive Summary

AirMessage Server communicates with several external services for authentication, error reporting, software updates, and message relay functionality. All communications use secure protocols (HTTPS/WSS) and most external data collection can be configured or disabled.

## External Services and Communications

### 1. Sentry Error Reporting

**Purpose**: Crash reporting and error analytics for debugging and improving application stability.

**Endpoints**:
- Configurable via `SENTRY_DSN` environment variable
- Default: Not configured (empty in Secrets.default.xcconfig)

**Data Transmitted**:
- Error messages and stack traces
- Application crash reports
- Debug information and context
- Device/OS information (automatically collected by Sentry SDK)

**Files Involved**:
- `AppDelegate.swift` (lines 28-42): Sentry initialization
- `LogManager.swift`: Error capture throughout the application
- Various files: ~30+ `SentrySDK.capture()` calls for error reporting

**Configuration**:
- Disabled in DEBUG builds
- Enabled in production builds only when `SENTRY_DSN` is configured
- Can be completely disabled by leaving `SENTRY_DSN` empty

**Security Measures**:
- Uses custom URL session delegate for certificate validation on older macOS versions
- Only activates when explicitly configured
- No user data collected beyond error context

### 2. Firebase Authentication Service

**Purpose**: User authentication and identity management for AirMessage Connect functionality.

**Endpoints**:
- `https://identitytoolkit.googleapis.com/v1/accounts:signInWithIdp` - IDP token exchange
- `https://identitytoolkit.googleapis.com/v1/accounts:lookup` - User data retrieval  
- `https://securetoken.googleapis.com/v1/token` - Token refresh

**Data Transmitted**:
- ID tokens and refresh tokens
- User email addresses
- Google OAuth authorization data
- Firebase project identifiers

**Files Involved**:
- `FirebaseAuthHelper.swift`: All Firebase API interactions
- `AccountConnectViewController.swift`: OAuth flow integration

**Configuration**:
- `FIREBASE_API_KEY`: API key for Firebase project
- Default configured to use open-source test environment

**Security Measures**:
- All communications over HTTPS
- Tokens have expiration times
- Uses secure token exchange protocols
- Custom certificate validation for older macOS compatibility

### 3. Google OAuth Service

**Purpose**: User authentication via Google accounts for AirMessage Connect.

**Endpoints**:
- `https://accounts.google.com/o/oauth2/auth` - Authorization endpoint
- `https://oauth2.googleapis.com/token` - Token endpoint
- `https://www.googleapis.com/auth/userinfo.email` - Email scope

**Data Transmitted**:
- User authorization codes
- Access and refresh tokens
- User profile information (email, display name)
- OAuth client credentials

**Files Involved**:
- `AccountConnectViewController.swift`: OAuth flow implementation

**Configuration**:
- `GOOGLE_OAUTH_CLIENT_ID`: OAuth client identifier
- `GOOGLE_OAUTH_CLIENT_SECRET`: OAuth client secret
- Default configured for open-source test environment

**Security Measures**:
- Standard OAuth 2.0 authorization code flow
- HTTPS for all communications
- Temporary local HTTP server for redirect handling
- Prompt for account selection to prevent silent authentication

### 4. AirMessage Connect Relay Service

**Purpose**: WebSocket-based message relay service for connecting clients to the server.

**Endpoints**:
- Configurable via `CONNECT_ENDPOINT` 
- Default: `wss://connect-open.airmessage.org`

**Data Transmitted**:
- Encrypted message data between clients and server
- Connection management and client IDs
- Push notification tokens (FCM)
- Installation IDs for device linking

**Files Involved**:
- `DataProxyConnect.swift`: WebSocket communication implementation
- `ConnectConstants.swift`: Protocol definitions

**Configuration**:
- `CONNECT_ENDPOINT`: WebSocket server URL
- Installation ID automatically generated per device
- Optional app-level encryption using user-configured password

**Security Measures**:
- WebSocket Secure (WSS) protocol
- TLS encryption for all communications
- Optional app-level message encryption
- Client authentication via Firebase tokens
- Ping/pong keepalive mechanism
- Automatic reconnection with exponential backoff

### 5. Software Update Service

**Purpose**: Automatic checking and downloading of software updates.

**Endpoints**:
- `https://update.airmessage.org/server.json` - Stable updates
- `https://update.airmessage.org/server-beta.json` - Beta updates

**Data Transmitted**:
- Current application version information
- OS compatibility data
- Update metadata requests
- Downloaded update packages

**Files Involved**:
- `UpdateHelper.swift`: Update checking and download logic
- `SoftwareUpdateViewController.swift`: Update UI

**Configuration**:
- Automatic checking can be disabled in preferences
- Beta updates opt-in available
- Check interval: 24 hours

**Security Measures**:
- HTTPS for all communications
- Version compatibility checking
- Digital signature validation (via macOS code signing)
- Automatic update installation with user consent

### 6. Certificate Validation Services

**Purpose**: Enhanced certificate trust for older macOS versions that don't trust modern root certificates.

**Implementation**:
- Custom URL session delegate with additional root certificates
- Let's Encrypt root certificate bundled with application
- Used for all external HTTPS communications

**Files Involved**:
- `ForwardCompatURLSessionDelegate.swift`: Custom certificate validation
- `CertificateTrust.swift`: Certificate trust evaluation
- `URLSessionCompat.swift`: Compatible URL session wrapper
- `Security/Certificates/`: Bundled root certificates

**Security Measures**:
- Maintains compatibility with older macOS versions
- Uses legitimate root certificates
- Fallback to system certificate validation

## Data Collection Summary

### Personal Data Collected:
- **User email addresses** (via Google OAuth, only for authentication)
- **Device installation IDs** (locally generated, used for device linking)

### Technical Data Collected:
- **Error logs and crash reports** (via Sentry, if configured)
- **Application version and OS information** (for updates and error context)
- **Message metadata** (for relay functionality, not message content unless unencrypted)

### Data Not Collected:
- **Message content** (when app-level encryption is enabled)
- **Contact information** (beyond authenticated email)
- **Usage analytics** (no behavioral tracking)
- **Location data**
- **Device identifiers** (beyond installation ID)

## Privacy and Security Recommendations

### Current Strengths:
1. All external communications use secure protocols (HTTPS/WSS)
2. Telemetry (Sentry) can be completely disabled
3. Message content can be encrypted end-to-end
4. Minimal personal data collection
5. Open-source transparent implementation

### Recommendations for Enhanced Security:

1. **Configuration Audit**:
   - Review all production configuration endpoints
   - Ensure test/development endpoints are not used in production builds
   - Consider making telemetry opt-in rather than opt-out

2. **Certificate Pinning**:
   - Consider implementing certificate pinning for critical endpoints
   - Regular review of bundled root certificates

3. **Data Minimization**:
   - Review error reporting to ensure no sensitive data in logs
   - Implement log sanitization for message content

4. **User Control**:
   - Provide clear user controls for all external communications
   - Allow granular control over telemetry and error reporting

5. **Audit Logging**:
   - Consider local audit logging of external communications
   - Transparency reports for data collection

## Configuration Reference

All external communications can be controlled via these configuration variables:

| Variable | Purpose | Default | Can Disable |
|----------|---------|---------|-------------|
| `SENTRY_DSN` | Error reporting | Empty | Yes |
| `FIREBASE_API_KEY` | Authentication | Test config | No* |
| `GOOGLE_OAUTH_CLIENT_ID` | OAuth | Test config | No* |
| `GOOGLE_OAUTH_CLIENT_SECRET` | OAuth | Test config | No* |
| `CONNECT_ENDPOINT` | Message relay | Test server | No* |

*Required for AirMessage Connect functionality

## Dependencies and Third-Party Libraries

### External Dependencies (via Swift Package Manager):
- **Sentry SDK**: Error reporting and crash analytics
- **WebSocketKit**: WebSocket communication for AirMessage Connect
- **AppAuth**: OAuth authentication flow implementation (Google)

### Bundled Libraries:
- **OpenSSL**: Cryptographic operations (custom build)
- **Zlib**: Compression utilities

### System Dependencies:
- **Foundation/URLSession**: Network communications
- **AuthenticationServices**: OAuth web authentication
- **Security Framework**: Certificate validation and trust

## Testing and Validation

### Automated Endpoint Testing

A test script has been provided (`/tmp/test_endpoints.sh`) to validate connectivity to external endpoints:

```bash
#!/bin/bash
# Test external endpoint connectivity
curl -s --connect-timeout 5 https://identitytoolkit.googleapis.com/
curl -s --connect-timeout 5 https://accounts.google.com/
curl -s --connect-timeout 5 https://update.airmessage.org/
curl -s --connect-timeout 5 https://connect-open.airmessage.org/
```

### Manual Validation Steps:

1. **Network Traffic Analysis**: 
   - Use tools like Wireshark or Charles Proxy to monitor outbound connections
   - Verify all communications use HTTPS/WSS protocols
   - Check for unexpected data transmission

2. **Configuration Testing**: 
   - Test with empty `SENTRY_DSN` (should disable error reporting)
   - Test with different `CONNECT_ENDPOINT` values
   - Verify behavior with invalid/expired OAuth tokens

3. **Offline Testing**: 
   - Disconnect from internet and verify graceful degradation
   - Check error handling for unreachable endpoints
   - Validate local functionality continues without external services

4. **Update Testing**: 
   - Monitor update check requests to update.airmessage.org
   - Verify update preferences disable checking
   - Test manual update checks vs automatic checks

5. **Privacy Testing**:
   - Review logs for sensitive data leakage
   - Test message encryption/decryption paths
   - Verify no unintended data collection

### Network Monitoring Commands:

```bash
# Monitor DNS lookups
sudo dtruss -n AirMessage 2>&1 | grep "getaddrinfo\|connect"

# Monitor network connections (macOS)
netstat -an | grep ESTABLISHED

# Log network activity
sudo tcpdump -i any host update.airmessage.org
sudo tcpdump -i any host connect-open.airmessage.org
```

## Summary of Security Findings

### Total External Endpoints: 6 main categories
1. **Sentry Error Reporting** (optional/configurable)
2. **Firebase Authentication APIs** (required for Connect mode)
3. **Google OAuth Services** (required for Connect mode)  
4. **AirMessage Connect Relay** (required for Connect mode)
5. **Software Update Service** (can be disabled)
6. **Certificate Validation** (automatic, enhances security)

### Data Collection Minimal:
- ✅ No behavioral analytics or tracking
- ✅ No location data collection
- ✅ No contact information beyond email (for auth only)
- ✅ Optional error reporting (can be disabled)
- ✅ Message content encrypted end-to-end (when enabled)

### Security Strengths:
- ✅ All external communications use HTTPS/WSS
- ✅ Open source transparency
- ✅ Minimal data collection
- ✅ User control over telemetry
- ✅ Strong encryption capabilities
- ✅ Certificate validation for older systems

### Areas for Consideration:
- ⚠️ Sentry enabled by default in production (though not configured in defaults)
- ⚠️ Error logs could potentially contain sensitive data
- ⚠️ Default configuration uses test/development endpoints

## Configuration for Enhanced Privacy

To maximize privacy and minimize external communications:

### Minimal Configuration:
```
SENTRY_DSN=                    # Empty - disables error reporting
FIREBASE_API_KEY=              # Required for Connect functionality
GOOGLE_OAUTH_CLIENT_ID=        # Required for Connect functionality  
GOOGLE_OAUTH_CLIENT_SECRET=    # Required for Connect functionality
CONNECT_ENDPOINT=              # Required for Connect functionality
```

### Direct Mode Only (No External Services):
- Use TCP Direct mode instead of Connect mode
- Disable automatic updates in preferences
- Leave SENTRY_DSN empty
- Configure local port forwarding/VPN for remote access

### Production Hardening:
- Review all production endpoint configurations
- Implement network-level monitoring
- Enable app-level message encryption
- Regular security audits of external endpoints
- Consider certificate pinning for critical services

## Conclusion

AirMessage Server implements responsible data collection practices with minimal personal data collection and strong security measures. All external communications serve legitimate functional purposes and can be audited or configured as needed. The application provides good user control over data sharing while maintaining essential functionality.

**Key Takeaways:**
- External communications are limited, well-defined, and serve specific purposes
- All communications use secure protocols (HTTPS/WSS) 
- Telemetry can be disabled entirely by configuration
- Message content can be encrypted end-to-end
- The codebase is transparent and auditable
- Users have control over their data and privacy settings

This audit provides a complete accounting of all telemetry and external routes as requested for the initial security review.