# Obscura Cookie Manager

Shared cookie refresh mechanism for CLI tools that need persistent authentication.

## Features

- **Multi-source cookie storage**: File, browser profile, environment variables
- **Automatic validation**: Periodic cookie validation with configurable interval
- **Auto re-extraction**: On validation failure, automatically re-extract from browser
- **Auth invalidation**: On persistent failure, invalidate auth and trigger re-login
- **Thread-safe**: Async-safe operations with proper locking

## Installation

```bash
pip install obscura-cookie-manager
```

## Usage

```python
from obscura_cookie_manager import (
    ObscuraCookieManager,
    FileCookieStorage,
    BrowserCookie3Extractor,
    CookieSource,
)

# Define storage
storage = FileCookieStorage(Path("~/.my-cli/cookies.json"))

# Define extractor
extractor = BrowserCookie3Extractor("brave")

# Define validator
def validate_cookies(cookies: dict[str, str]) -> bool:
    # Make an API call to verify cookies work
    return True  # or False

# Create manager
manager = ObscuraCookieManager(
    storage=storage,
    extractor=extractor,
    validator=validate_cookies,
    required_cookies=["token_v2", "reddit_session"],
    validation_interval=300,  # 5 minutes
)

# Get valid cookies (auto-validates, re-extracts if needed)
result = await manager.get_cookies()
if result.valid:
    cookies = result.cookies
    # Use cookies for API calls
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    ObscuraCookieManager                      │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Storage   │  │  Extractor  │  │      Validator      │  │
│  │             │  │             │  │                     │  │
│  │ • File      │  │ • browser_  │  │ • API call          │  │
│  │ • Env var   │  │   cookie3   │  │ • Custom function   │  │
│  │ • Browser   │  │ • Subprocess│  │                     │  │
│  │   profile   │  │ • SQLite    │  │                     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## Integration

Designed for use with:
- `reddit-httpx` - Reddit API client
- `twitter-cli` - Twitter/X CLI
- `instagram-httpx` - Instagram API client

## License

MIT