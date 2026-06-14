# Snapchat GraphQL Schema

This is a conceptual GraphQL schema for the Snapchat platform, covering the Snap Kit developer APIs (Login Kit, Creative Kit, Camera Kit), the Snapchat Ads API (Marketing API), the Conversions API, and Lens Studio. Snapchat does not publicly expose a GraphQL endpoint; this schema represents the logical data model derivable from their REST APIs and developer documentation.

## Source APIs

- **Snapchat Ads API (Marketing API)**: https://developers.snap.com/api/marketing-api/Ads-API/introduction
- **Snapchat Conversions API**: https://developers.snap.com/api/marketing-api/Conversions-API/Introduction
- **Login Kit**: https://developers.snap.com/snap-kit/login-kit/overview
- **Creative Kit**: https://developers.snap.com/snap-kit/creative-kit/overview
- **Camera Kit**: https://developers.snap.com/camera-kit/home
- **Lens Studio**: https://developers.snap.com/lens-studio/home

## Domain Coverage

### User and Identity

Types covering Snapchat user identity via Login Kit OAuth: `User`, `SnapUser`, `Bitmoji`, `BitmojiAvatar`, `BitmojiScene`, `Friend`, `FriendRequest`, `FriendStatus`, `OAuthApp`, `LoginKit`, `APIKey`, `Token`.

### Content and Media

Types representing Snaps, Stories, Spotlight, and Discover content: `Snap`, `SnapMedia`, `Photo`, `Video`, `Filter`, `Lens`, `LensStudio`, `LensAnalytics`, `Story`, `UserStory`, `PublicStory`, `StoryMedia`, `SpotlightVideo`, `DiscoverShow`, `Publisher`, `PublisherProfile`, `StoryKit`, `BitmojiFriendship`.

### Advertising and Campaigns

Types derived from the Ads API representing the full campaign hierarchy: `AdAccount`, `Campaign`, `AdSquad`, `Ad`, `AdMedia`, `StoryAd`, `CollectionAd`, `DynamicAd`, `Creative`, `CreativeMedia`, `CreativeElement`.

### Audience and Targeting

Types covering audience segmentation and ad targeting: `Audience`, `PixelAudience`, `LALAudience`, `CustomerList`, `SavedAudience`, `RetargetingAudience`, `Targeting`, `GeoTarget`, `DemographicTarget`, `InterestTarget`.

### Measurement and Conversions

Types for pixel tracking and conversion reporting from the Conversions API: `SnapPixel`, `PixelEvent`, `ConversionLog`, `AppInstall`, `AppOpen`, `Purchase`, `Revenue`, `ROAS`.

### Device

`Device` — device-level targeting and attribution context.

## Schema File

See `snapchat-schema.graphql` for the full type definitions.

## Notes

- Snapchat's public APIs are REST-based; this GraphQL schema is a conceptual translation of the REST resource model into GraphQL types and relationships.
- The Ads API uses OAuth 2.0 client credentials; Login Kit uses OAuth 2.0 authorization code flow.
- The Conversions API (CAPI) supports server-side event ingestion at v3 and does not have a GraphQL analogue.
- Camera Kit and Lens Studio expose SDK/scripting interfaces rather than HTTP APIs; the types here reflect their logical data models.
