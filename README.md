# GTX Messenger 1.1 — Liquid Glass Edition

This package contains:
- `GTX_Messenger_v1.1.html` — upgraded client, preserving the existing Firebase project configuration and current database paths.
- `GTX_Messenger_v1.1_database.rules.json` — Realtime Database rules for the new friend-request and display-name index paths, plus conversation/message checks.
- `GTX_Messenger_v1.1_backend/` — Firebase Cloud Functions for friend requests and audited administrator conversation review.

## Important safety notes

1. Export/backup the existing Realtime Database before deploying rules or functions.
2. The client does not reset or migrate existing users, identities, conversations, groups, or messages.
3. The existing application source did not contain a dedicated `friends` collection. For backward compatibility, existing one-to-one conversations are treated as established friends in the new group picker and remain openable. They are not forced through a new friend request.
4. New friend requests and accepted friend records are written by Cloud Functions, not by browser code.
5. Administrator privileges use Firebase Authentication custom claims. Do not add an admin password or service-account private key to the HTML file.
6. The administrator conversation review endpoint requires an admin claim and a written reason, and records an audit entry. It returns up to the most recent 300 messages for the requested conversation.
7. Review the rules in a staging Firebase project first. The current application already has legacy rules and data conventions; do not publish rules without testing a normal account, an existing friend, a new friend request, and a group.

## Deploy the backend

Prerequisites:
- Node.js 20
- Firebase CLI
- Access to the existing Firebase project `gtxmessenger`

From this folder:

```bash
firebase login
firebase use gtxmessenger
npm install --prefix functions
firebase deploy --only functions,database
```

The Functions use the default `us-central1` region.

## Assign an administrator

1. Sign in to GTX Messenger using the account that should become the administrator.
2. In Firebase Console → Authentication → Users, copy that account's UID.
3. From this backend folder, authenticate the Firebase Admin SDK using Application Default Credentials (for example, `gcloud auth application-default login`) or set `GOOGLE_APPLICATION_CREDENTIALS` to a service-account file stored securely outside the web root.
4. Run:

```bash
node functions/set-admin.js YOUR_FIREBASE_AUTH_UID
```

5. Sign out of GTX Messenger and sign in again so Firebase refreshes the custom claim.
6. The shield button appears for accounts whose verified ID token contains `admin: true`.

Never commit service-account JSON or expose it to the browser.

## Enable display-name search for existing profiles

New/edited identities write to `displayNameIndex`. To index existing public identities without deleting or overwriting any other database data, run:

```bash
node functions/backfill-display-name-index.js
```

This uses Admin SDK privileges and adds only missing/current index entries. Run after backup.

## Client deployment

Upload `GTX_Messenger_v1.1.html` to the same hosting location currently serving GTX Messenger, after backend deployment and testing. Keep a copy of the previous HTML file so you can roll back the frontend without touching Firebase data.

## What is included in this client build

- Liquid Glass-inspired surfaces, translucent panels, blur, reflections, gradients and mobile-safe layouts.
- Five new theme choices are intended to be available alongside the existing themes.
- Friend request UI backed by `sendFriendRequest`, `respondFriendRequest`, and `cancelFriendRequest` Cloud Functions.
- Existing one-to-one chats are recognized as legacy friendships and are included in group-member selection.
- New accepted friends are shown in the group-member selector; search filters username and display name.
- Identity editing now includes a status line and maintains identity IDs.
- Version 1.1 release announcement, acknowledged per user in Realtime Database.
- Expanded quick emojis and categorized Emoji Studio.
- Admin console with custom-claim verification, dashboard counts, reason-required conversation review, and server-side audit logging.

## Testing checklist

- Existing account sign-in and Google sign-in
- Existing conversations and message history
- Existing group memberships
- Edit identity, username, avatar, bio and status
- New friend request → accept/reject/cancel
- Existing one-to-one chats remain openable
- New private conversation requires accepted friendship
- Group picker lists accepted friends and legacy private chats
- Search by username and display name
- Message send, reply, reactions and deletion
- Release notes appear once per user
- Admin claim is checked server-side
- Admin review creates an audit log
- Mobile layout on a narrow viewport
- Database rules deny unauthenticated access

## Limitations to verify before production

- Firebase Cloud Functions must be deployed before friend requests and admin calls work.
- Existing users' public display-name search requires the one-time backfill script.
- The admin dashboard currently provides aggregate user/conversation counts and audited conversation review; it does not yet implement account suspension, reports moderation, or a complete user-management console.
- Existing user profile photos are still handled using the source app's current image/data-URL approach. For a production-scale deployment, migrate image uploads to Firebase Storage with per-user Storage rules.
- A security rules deployment can affect existing workflows. Test the new rules in the Firebase Rules simulator and a staging project before production.
