# myastralmap-app-links

Universal Links / App Links verification files for the MyAstralMap mobile app.
Deployed as a Cloudflare Pages site bound to `app.myastralmap.com`.

## What lives here

```
public/
├── .well-known/
│   ├── apple-app-site-association   # iOS Universal Links (no extension, served as application/json)
│   └── assetlinks.json              # Android App Links
├── abrir-audio/
│   └── index.html                   # Fallback when app isn't installed (redirects to stores)
└── index.html                       # Generic redirect to myastralmap.com
```

## Why a dedicated subdomain (`app.myastralmap.com`)

- AASA is fetched by iOS at install time. CDN edge serving is faster than the Hetzner backend.
- Domain ownership = "this domain verifies the app". Mixing it with marketing/landing risks accidental breakage.
- HTTPS auto-managed by Cloudflare; no nginx config.
- Updates to the keystore SHA256 (Android) or Team ID (iOS) are one PR away.

## Deploy

```bash
npx wrangler pages deploy public --project-name=myastralmap-app-links
```

Custom domain `app.myastralmap.com` configured in the Cloudflare Pages dashboard
(requires DNS:Edit permission, not in the default OAuth token scope).

## TODO

- [ ] Replace `TEAMID_PLACEHOLDER` in apple-app-site-association with real Apple Team ID (blocked on Apple Developer Program approval).
- [ ] Replace `SHA256_PLACEHOLDER_GET_FROM_EAS_KEYSTORE` in assetlinks.json with the production keystore fingerprint (`npx eas-cli credentials --platform android --profile production` and extract from the displayed cert).
- [ ] Verify AASA in https://app-site-association.cdn-apple.com/a/v1/app.myastralmap.com after iOS Team ID is set.
- [ ] Verify assetlinks in https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://app.myastralmap.com&relation=delegate_permission/common.handle_all_urls

## Related specs

- MAM-0115 (notificações grandes ciclos — email CTA usa este universal link)
- MAM-0113 (mobile RC integration — paywall e customer center deep link via mesmo domínio futuramente)
