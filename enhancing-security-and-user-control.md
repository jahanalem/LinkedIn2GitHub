# Enhancing Security and User Control: New Features in LiliShop

I'm excited to share some new features I've added to enhance security, user control, and maintain clean code. Here's a quick overview:

## 🔐 1. Refresh Token
- When a user's login session expires, they no longer need to log in again.
- A **Refresh Token** (stored in a secure HTTP-only cookie) automatically creates a new session.

## 📲 2. Logout from All Devices
- Users can now log out from all their devices with a single click.
- This feature clears login sessions and tokens across all active devices (e.g., mobile, desktop).
- It's especially useful for securing accounts or logging out from old sessions.

## 🔄 3. Role Change Update
- When a user's role changes (e.g., from **User** to **Admin**), their permissions are updated immediately.
- Old tokens are invalidated to ensure users only have access to what they should.

## 📘 4. RFC 9457 - Problem Details for HTTP APIs
- Implemented **RFC 9457** for providing clear and standardized error messages in the API.

---

### 🔗 Live Project
[LiliShop Live Project](https://lnkd.in/eAK8B3Ka)

### 📂 Source Code
- **Frontend:** [GitHub Repository](https://lnkd.in/ePhuYGhP)

---

### Tags
`#webdevelopment` `#security` `#dotnet` `#angular` `#cleancode` `#softwaredevelopment` `#programming`

![LogoutFromAllDevices](https://github.com/user-attachments/assets/83d9eb74-40b4-4e95-a72b-db29158115d5)

