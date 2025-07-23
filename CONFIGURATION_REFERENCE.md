# AirMessage Server Configuration Reference

This file provides a quick reference for all configuration variables that control external communications and telemetry in AirMessage Server.

## Configuration File Location
`AirMessage/Secrets.xcconfig`

## Configuration Variables

### Telemetry and Error Reporting

| Variable | Default | Purpose | Can Disable |
|----------|---------|---------|-------------|
| `SENTRY_DSN` | Empty | Crash reporting and error analytics | ✅ Yes (leave empty) |

**Privacy Impact**: When configured, sends error reports and crash data to Sentry for debugging.
**Recommendation**: Leave empty for maximum privacy.

### Authentication Services (Required for Connect Mode)

| Variable | Default | Purpose | Can Disable |
|----------|---------|---------|-------------|
| `FIREBASE_API_KEY` | Test config | Firebase authentication API | ❌ No (required for Connect) |
| `GOOGLE_OAUTH_CLIENT_ID` | Test config | Google OAuth client ID | ❌ No (required for Connect) |
| `GOOGLE_OAUTH_CLIENT_SECRET` | Test config | Google OAuth client secret | ❌ No (required for Connect) |

**Privacy Impact**: Enables user authentication via Google accounts.
**Recommendation**: Use production values for production deployments.

### Message Relay Service

| Variable | Default | Purpose | Can Disable |
|----------|---------|---------|-------------|
| `CONNECT_ENDPOINT` | `wss://connect-open.airmessage.org` | WebSocket relay server | ❌ No (required for Connect) |

**Privacy Impact**: Routes encrypted messages through AirMessage Connect relay.
**Recommendation**: Use official production endpoints for production use.

## Operating Modes

### Connect Mode (Default)
- **Requires**: All authentication and relay configuration
- **External Services**: Firebase, Google OAuth, AirMessage Connect
- **Benefits**: Easy remote access, push notifications
- **Privacy**: Data routed through external relay (encrypted)

### Direct Mode (Alternative)
- **Requires**: No external configuration (can set all to empty except auth if needed)
- **External Services**: Only update service (optional)
- **Benefits**: No external relay, direct connections only
- **Privacy**: Maximum privacy, no message relay

## Privacy-Focused Configuration

For maximum privacy, use Direct mode:

```
SENTRY_DSN=
FIREBASE_API_KEY=
GOOGLE_OAUTH_CLIENT_ID=
GOOGLE_OAUTH_CLIENT_SECRET=
CONNECT_ENDPOINT=
```

And disable automatic updates in the application preferences.

## Testing Configuration

The default `Secrets.default.xcconfig` uses test/development endpoints that are safe for testing but should not be used in production.

## Security Notes

1. **Never commit production secrets** to version control
2. **Use strong, unique values** for production configurations
3. **Regularly rotate OAuth credentials** for production deployments
4. **Monitor external communications** in production environments
5. **Keep configuration files secure** with appropriate file permissions