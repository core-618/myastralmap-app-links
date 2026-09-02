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

## Android: which certificate goes in `assetlinks.json`

**The Play App Signing certificate. Not the EAS keystore.** This README used to
say the opposite, and it cost a broken password reset for every Android user
(MAM-0500).

Play App Signing is enabled for this app: you upload an AAB signed with your
*upload* key, and Google **re-signs** it with the *app signing* key before
distributing it. The app on the user's phone therefore carries the app signing
certificate, and that is the one Android compares against `assetlinks.json`.
Publishing the upload fingerprint means verification fails **silently** — no
error anywhere, the link just opens in the browser instead of the app.

Get it from Play Console → *Test and release* → **App signing**. That page shows
both fingerprints plus a ready-made JSON snippet; copy the snippet, do not
hand-pick a fingerprint from further up the page (the *upload key certificate*
block sits right next to it and is the easy mistake).

    upload key      59:EE:9C:42:…  ← wrong, this is what was published
    app signing key 23:77:22:E7:…  ← right, this is what Android checks

Both are listed in the file now. The app signing one is what matters for Play
installs; the upload one keeps EAS-built APKs (installed by hand, e.g. the
`preview` profile) verifying too.

**Verification is cached.** Android checks at install/update time, so a fixed
file does not retroactively repair phones that already failed. Reinstalling, or
the next app update, re-runs it.

## TODO

- [x] Replace `TEAMID_PLACEHOLDER` in apple-app-site-association with real Apple Team ID.
- [x] Replace the Android fingerprint with the **Play App Signing** certificate (see above).
- [ ] Verify AASA in https://app-site-association.cdn-apple.com/a/v1/app.myastralmap.com
- [ ] Verify assetlinks in https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://app.myastralmap.com&relation=delegate_permission/common.handle_all_urls

## Related specs

- MAM-0115 (notificações grandes ciclos — email CTA usa este universal link)
- MAM-0113 (mobile RC integration — paywall e customer center deep link via mesmo domínio futuramente)
