# Mac: Getting rid of an old Microsoft Teams account that keeps coming back

I had an old work account stuck in Microsoft Teams on my Mac. Signing out
didn't help. Deleting things didn't help either: every time I relaunched
Teams, the account was back, like nothing happened.

It took a fair bit of digging, so here's what I found, in case you're
stuck in the same loop.

## Where Teams keeps your account

The new Teams client uses Microsoft's OneAuth for single sign-on, and
OneAuth keeps account details and refresh tokens in the macOS Keychain.

You can look for it with the `security` command. I found an entry with the
service name `OneAuthAccount`, and its account name was a UUID. The entry
also had a hex-encoded attribute (`gena`). Decoding that hex gave me a small
JSON blob with the account's profile:

- Login name (the old work email)
- Display name
- Account ID (a UUID, `<account-id>` from here on)
- Tenant ID (another UUID, `<tenant-id>` from here on)

Those two IDs turned out to be the useful bit. Once you have them, you can
search for them on disk.

## Where the cache files live

**OneAuth's shared store**

`~/Library/Group Containers/UBF8T346G9.com.microsoft.oneauth/BlobStore/`

This folder had files named straight after the IDs:

- `<account-id>_graph_organization_metadata` (account info)
- `<account-id>_avatar_metadata` (profile picture)
- `<tenant-id>_hrd` (home realm discovery for the old organization)

**Teams' own sandboxed containers**

- `~/Library/Containers/com.microsoft.teams2/` (the app)
- `~/Library/Containers/com.microsoft.teams2.agent/` (a background helper)

Teams shows its login pages in an embedded Edge WebView, so there are
cookies and local storage in here too:

- `.../Data/Library/Application Support/Microsoft/MSTeams/EBWebView/`
- `.../Data/Library/Cookies/Cookies.binarycookies`

## Why it kept coming back

I deleted the keychain entry and cleared these folders, and Teams happily
recreated all of it on the next launch. Two things were doing this.

**The background agent.** `com.microsoft.teams2.agent` keeps running after
you quit Teams. If you delete the keychain entry while it's alive, it just
writes the tokens back from memory.

**Keychain sync.** The entry also lived in the iCloud keychain, not just in
`login.keychain`. Deleting it locally wasn't enough: macOS saw it was still
in iCloud and quietly restored it.

## What finally worked

1. **Kill everything Teams.** Quit Teams, then kill every process with
   "teams" in the name (case-insensitive), so the background agent is gone
   too.
2. **Empty the containers.** macOS won't let you delete the container
   folders themselves, but you can clear what's inside their `Data/`
   folders, especially the cookies and the WebView storage.
3. **Delete the entry from Keychain Access.** Open the Keychain Access
   app, go to the iCloud keychain / *All Items*, find the OneAuth entries
   and delete them there. Doing it in the app made the deletion stick in
   iCloud too, and that was the step that finally made it go away.

Do them in that order. If the agent is still running, or the entry is still
in iCloud, you'll be right back where you started.
