# Roles and Permissions

## Roles
- **Department staff:** nurses, radiology techs, and other departments that request transport
- **Transporter:** works calls from the phone app
- **Dispatcher:** runs the queue and manages transporters
- **Manager:** transport manager / boss. Can do everything a dispatcher can, plus see reports and call history
- **Admin:** manages accounts, roles, and hospital setup data

## What each role can do
| Action | Dept staff | Transporter | Dispatcher | Manager | Admin |
|---|---|---|---|---|---|
| Submit a request | Yes | | Yes | Yes | |
| Edit a request (room, destination, priority, details) | Their department's | | Any | Any | |
| See status of their department's requests | Yes | | Yes | Yes | |
| Cancel a request (with a reason) | Their department's | Their own calls | Any | Any | |
| Message on a call's thread | Yes | Yes | Yes | Yes | |
| Message the dispatcher | | Yes | | | |
| Message anyone | | | Yes | Yes | |
| Receive calls | | Yes | | | |
| Update call status (en route, arrived, scan, complete) | | Yes | | | |
| Delay a call (with a reason) | | Yes | Yes | Yes | |
| Postpone a call (10/20/30 min, at the nurse's discretion) | | Yes | Yes | Yes | |
| Get help (up to 3 more transporters) | | Yes | | | |
| Pick a sector at sign-in | | Yes (from the dispatcher's schedule) | | | |
| Move transporters to another sector | | | Yes | Yes | |
| Go on break / come back from break | | Yes (themselves) | Yes (any transporter) | Yes (any transporter) | |
| See the full queue | | | Yes | Yes | |
| Assign and reassign calls | | | Yes | Yes | |
| See transporter activity since sign-in | | | Yes | Yes | |
| See reports and call history | | | | Yes | |
| Manage locations, sectors, and routine run times | | | | | Yes |
| Manage accounts and roles | | | | | Yes |

## Notes
- Requests are scoped to departments: staff only see, edit, and cancel their own department's requests.
- Postponing happens at the nurse's discretion. The transporter, dispatcher, or manager enters it. A postponed call is taken off the transporter right away and comes back into the queue automatically when the time is up, so nobody waits on it.
- Transporters pick their sector when they sign in, but it's set by the dispatcher's schedule, not their choice. The dispatcher or manager can move them anytime.
- Only transporters have a break status, since it affects who gets calls. Other staff don't need one.
- Equipment cleanliness is the transporter's judgment. Stretchers and wheelchairs can be wiped down before use, and dirty beds are usually stripped, so it's obvious. The system doesn't track clean or dirty.
