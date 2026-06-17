# Security Specification: Volunteer Registration System

## 1. Data Invariants
- **Unverified Users Blocked**: Standard users must have verified emails (`request.auth.token.email_verified == true`) to perform standard writes.
- **Admin Access Level**: Admins are authenticated through `/admins/{uid}` presence.
- **Bootstrapping Overrides**: The user email `yashodarani2022014@gmail.com` can self-assign as an admin.
- **PII Isolation Policy**: Personally Identifiable Information is maintained inside `/users_private/{userId}`. Only the authenticated owner or admins can read/write.
- **Immortal Fields**: Fields like `createdAt` and `joinedAt` are immutable.
- **Type Safety and Bounds**: All strings, arrays, and numbers are constrained by length or value boundaries.

## 2. The "Dirty Dozen" Payloads (Attacks to Block)
1. **Unsigned-In Write**: Non-logged in user attempting to read/write event data.
2. **Unverified Account Creation**: A user with `email_verified == false` trying to create a volunteer profile.
3. **Identity Spoofing**: `User-A` trying to create or edit a public profile under `/users/User-B`.
4. **PII Breach**: `User-A` attempting to read `User-B`'s private record under `/users_private/User-B`.
5. **Self-Elevating Admin Role**: A non-admin user passing `role: "admin"` inside `/users/userId` to force administrator privileges.
6. **Admin Collection Tampering**: A user other than `yashodarani2022014@gmail.com` trying to register themselves inside `/admins/temp_id`.
7. **Orphaned Registration**: Creating a registration to a non-existent `eventId`.
8. **Malicious Event Spoofing**: A non-admin creation of a new opportunity under `/events`.
9. **Event Cap Bypass**: Injecting arbitrary counts in `registeredCount` when registering.
10. **Shadow Field Injection**: Saving a registration with a ghost field `isVIPConfirmed: true` to bypass typical fields.
11. **Hours Logging Spoof**: Logging hours of `9999` for an event to inflate metrics.
12. **Status Shortcutting**: Directly writing `status: "approved"` to one's own hours log bypassing administrator approval.

## 3. Test Runner Definition (Verification Concept)
All the above payloads MUST resolve to a strict `PERMISSION_DENIED` inside firebase rules testing.
