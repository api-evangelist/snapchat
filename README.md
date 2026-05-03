# Snapchat

Snap Inc. operates Snapchat, a visual messaging app and camera platform with developer tools including the Marketing API for programmatic advertising, Conversions API for server-side conversion tracking, Login Kit for OAuth-based user authentication, Creative Kit for content sharing to Snapchat, Camera Kit for embedding Snap AR technology into third-party apps, and Lens Studio for building augmented reality experiences.

## APIs

- [Snapchat Ads API](https://developers.snap.com/api/marketing-api/Ads-API/introduction) - Programmatic campaign management for organizations, ad accounts, campaigns, ad squads, ads, creatives, media, audience segments, and measurement.
- [Snapchat Conversions API](https://developers.snap.com/api/marketing-api/Conversions-API/Introduction) - Server-to-server conversion event tracking for web, app, and offline conversions (CAPI v3).
- [Snapchat Login Kit API](https://developers.snap.com/snap-kit/login-kit/overview) - OAuth 2.0 user authentication and profile retrieval including Bitmoji avatar.
- [Snapchat Creative Kit](https://developers.snap.com/snap-kit/creative-kit/overview) - Share content including lenses, stickers, videos, and links to Snapchat from third-party apps.
- [Snapchat Camera Kit](https://developers.snap.com/camera-kit/home) - Embed Snap AR camera technology into iOS, Android, and web applications.
- [Lens Studio](https://developers.snap.com/lens-studio/home) - Desktop AR lens creation platform for Snapchat and Spectacles.

## Properties

- [Developer Portal](https://developers.snap.com)
- [GitHub Organization](https://github.com/Snapchat)
- [Business Help](https://businesshelp.snapchat.com)
- [Terms of Service](https://snap.com/en-US/terms)
- [Engineering Blog](https://eng.snap.com)

## Artifacts

### OpenAPI
- [snapchat-ads-api-openapi.yml](openapi/snapchat-ads-api-openapi.yml) — Snapchat Ads API (Organizations, Ad Accounts, Campaigns, Ad Squads, Ads, Creatives, Media, Audience Segments, Measurement)
- [snapchat-conversions-api-openapi.yml](openapi/snapchat-conversions-api-openapi.yml) — Snapchat Conversions API v3 (Web, App, and Offline conversion events)
- [snapchat-login-kit-openapi.yml](openapi/snapchat-login-kit-openapi.yml) — Snapchat Login Kit OAuth 2.0 API (Authorization, Token Exchange, User Profile)

### Rules
- [snapchat-rules.yml](rules/snapchat-rules.yml) — Spectral ruleset enforcing Snapchat API conventions (camelCase operationIds, versioned servers, envelope responses)

### Capabilities
- [ad-campaign-management.yaml](capabilities/ad-campaign-management.yaml) — Unified workflow for Snapchat campaign management and conversion tracking (Ads API + CAPI)
- [user-authentication.yaml](capabilities/user-authentication.yaml) — Snapchat user identity and profile workflow via Login Kit OAuth

#### Shared Definitions
- [capabilities/shared/ads-api.yaml](capabilities/shared/ads-api.yaml) — Snapchat Ads API shared capability definition
- [capabilities/shared/conversions-api.yaml](capabilities/shared/conversions-api.yaml) — Snapchat Conversions API shared capability definition
- [capabilities/shared/login-kit.yaml](capabilities/shared/login-kit.yaml) — Snapchat Login Kit shared capability definition

### JSON Schema
- [snapchat-ad-campaign-schema.json](json-schema/snapchat-ad-campaign-schema.json) — Schema for Snapchat ad campaign entities (Organization, AdAccount, Campaign, AdSquad, Ad, Creative)
- [snapchat-conversion-event-schema.json](json-schema/snapchat-conversion-event-schema.json) — Schema for Snapchat CAPI v3 conversion event payloads

### JSON Structure
- [snapchat-campaign-structure.json](json-structure/snapchat-campaign-structure.json) — Hierarchical structure of Snapchat advertising objects from Organization to Ad

### JSON-LD
- [snapchat-context.jsonld](json-ld/snapchat-context.jsonld) — JSON-LD context for Snapchat advertising and social media linked data semantics

### Examples
- [snapchat-create-campaign-example.json](examples/snapchat-create-campaign-example.json) — Create a Snapchat awareness campaign via the Ads API
- [snapchat-send-conversion-events-example.json](examples/snapchat-send-conversion-events-example.json) — Send purchase conversion events via the Conversions API

### Vocabulary
- [snapchat-vocabulary.yml](vocabulary/snapchat-vocabulary.yml) — Domain vocabulary covering Snapchat advertising, AR, authentication, and platform concepts
