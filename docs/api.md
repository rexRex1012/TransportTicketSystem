# API Routes
The backend API that the web app (staff, dispatcher, manager) and the mobile app (transporters) use. Based on `walkthroughs.md`, `roles.md`, and `lists.md`.

## Conventions
- All routes start with `/api` and send and receive JSON (camelCase field names).
- Every request needs a Keycloak login token (`Authorization: Bearer <token>`). The API reads the user's role, department, and sector from it.
- Times are ISO 8601 in UTC, like `2026-10-27T18:45:00Z`. Anything tied to the clock on the wall (routine run times, shifts, report days) uses hospital time, America/New_York.
- The mobile app sends an `Idempotency-Key` header on every action, so a retry after a bad signal doesn't do the same thing twice.
- Roles: **Staff** (department staff), **Transporter**, **Dispatcher**, **Manager**, **Admin**. "Dispatcher" in this doc always means Dispatcher **or** Manager, since the manager can do everything a dispatcher can.

### Errors
| Code | Meaning |
|---|---|
| 400 | Bad input (missing or invalid fields) |
| 401 | Not logged in |
| 403 | Logged in, but your role or department can't do this |
| 404 | Not found (or not in your department) |
| 409 | Conflict: invalid status change, call already assigned, or someone else edited it first |

Error body:
```json
{ "error": "INVALID_STATUS_CHANGE", "message": "Can't complete a call that hasn't been picked up" }
```

---

## Me
| Method | Route | Who | What it does |
|---|---|---|---|
| GET | `/api/me` | Everyone | Your name, role, department, and (for transporters) sector and status |

---

## Requests (calls)
Staff submit a **request**. Once it's sent to a transporter, it's a **call**. Same thing, same routes.

### Create, view, edit
| Method | Route | Who | What it does |
|---|---|---|---|
| POST | `/api/requests` | Staff, Dispatcher | Submit a request |
| GET | `/api/requests` | Staff (own department), Dispatcher | List requests. Filters: `status`, `priority`, `type`, `departmentId`, `sectorId`, `transporterId` |
| GET | `/api/requests/{id}` | Staff (own department), assigned Transporter, Dispatcher | One request with its full details |
| PATCH | `/api/requests/{id}` | Staff (own department), Dispatcher | Edit room, destination, priority, or details. Send the `version` you last saw, or you get a 409 if someone else changed it first |
| GET | `/api/requests/{id}/events` | Staff (own department), Dispatcher | Status timeline: who did what and when |

**Create request body (patient):**
```json
{
  "type": "PATIENT",
  "priority": "STAT",
  "departmentId": 3,
  "tripPurpose": "TEST",
  "fromLocationId": 12,
  "fromRoom": "N14",
  "toLocationId": 9,
  "patientId": 301,
  "transportMode": "STRETCHER",
  "transportBringsEquipment": false,
  "clinicalRequirements": ["NURSE_ASSIST", "CARDIAC_MONITOR"],
  "isolationType": "CONTACT",
  "scheduledFor": null,
  "notes": "Patient on heparin drip"
}
```

**`departmentId`:** staff can leave it out, since it's always their own department. A dispatcher has to set it when submitting for someone who called, so the request still belongs to that department.

**`tripPurpose`** (patient requests only):
- `TRANSFER`: the patient's room changes to the destination when the call completes (like 403 to 6 ICU, or the ED to a floor)
- `TEST`: a test or procedure. The room stays the same, and the trip back is a new request
- `DISCHARGE`: the patient is marked discharged when the call completes

**Equipment request:** `"type": "EQUIPMENT"` with `equipmentType` (`BED`, `STRETCHER`, `WHEELCHAIR`, `O2_TANK`, `CODE_CART`, `OTHER`), `quantity`, and either a `fromLocationId` (move a specific one) or no start point (bring the nearest one).

**Specimen request:** `"type": "SPECIMEN"` with `fromLocationId` (like Endo), `toLocationId` (Labs), and `notes`.

**Request response:**
```json
{
  "id": 1042,
  "type": "PATIENT",
  "priority": "STAT",
  "status": "ACKNOWLEDGED",
  "escalated": false,
  "sectorId": 1,
  "departmentId": 3,
  "tripPurpose": "TEST",
  "from": { "locationId": 12, "name": "North Pod", "room": "N14" },
  "to": { "locationId": 9, "name": "ED CT" },
  "patient": { "id": 301, "name": "Test Patient", "mrn": "MRN0301" },
  "transportMode": "STRETCHER",
  "transportBringsEquipment": false,
  "clinicalRequirements": ["NURSE_ASSIST", "CARDIAC_MONITOR"],
  "isolationType": "CONTACT",
  "assignedTo": [{ "id": 7, "name": "Marcus T" }],
  "requestedBy": { "id": 22, "name": "Kim R", "department": "ED" },
  "createdAt": "2026-10-27T18:40:00Z",
  "assignedAt": "2026-10-27T18:40:05Z",
  "version": 3
}
```

### Transporter actions (moving a call through its statuses)
| Method | Route | Who | What it does |
|---|---|---|---|
| POST | `/api/requests/{id}/acknowledge` | Assigned Transporter | Acknowledge the call (must happen within 10 minutes) |
| POST | `/api/requests/{id}/start` | Assigned Transporter | Heading to the pickup (**en route**) |
| POST | `/api/requests/{id}/arrive` | Assigned Transporter | At the pickup, found the nurse (**arrived**) |
| POST | `/api/requests/{id}/pickup` | Assigned Transporter | Scan to confirm the right patient or item (**picked up**) |
| POST | `/api/requests/{id}/complete` | Assigned Transporter | Dropped off, call done (**completed**) |
| POST | `/api/requests/{id}/help` | Assigned Transporter | Get help: `{ "count": 1 }` (up to 3 more transporters) |

**Pickup body:**
```json
{ "scannedCode": "MRN0301" }
```
The scanned code has to match the patient's MRN (patient calls) or the equipment's asset tag (equipment calls), or you get a 409 and the call stays where it was. For specimen runs, send `{ "specimensTaken": 6, "specimensLeft": 1 }` instead, for any specimen that wasn't on the sign-off sheet.

### Delay, postpone, cancel
| Method | Route | Who | What it does |
|---|---|---|---|
| POST | `/api/requests/{id}/delay` | Assigned Transporter, Dispatcher | `{ "reasonCode": "PATIENT_USING_RESTROOM", "note": "" }`. The delay clears when the call moves to its next status |
| POST | `/api/requests/{id}/postpone` | Assigned Transporter, Dispatcher | `{ "minutes": 20 }` (10, 20, or 30). Takes the call off the transporter right away and puts it back in the queue when the time is up |
| POST | `/api/requests/{id}/cancel` | Staff (own department), assigned Transporter, Dispatcher | `{ "reasonCode": "DUPLICATE_REQUEST", "note": "" }` |

### Dispatcher actions
| Method | Route | Who | What it does |
|---|---|---|---|
| POST | `/api/requests/{id}/assign` | Dispatcher | Assign or reassign: `{ "transporterId": 7 }` |
| POST | `/api/requests/{id}/unassign` | Dispatcher | Pull the call back into the queue |

### Allowed status changes
| From | Can go to |
|---|---|
| new | assigned, postponed, cancelled |
| assigned | acknowledged, new (unassigned, or not acknowledged in 10 minutes), postponed, cancelled |
| acknowledged | en route, new (reassigned), postponed, cancelled |
| en route | arrived, new (reassigned), postponed, cancelled |
| arrived | picked up, postponed, cancelled |
| picked up | completed |
| postponed | new (automatically, when the time is up), cancelled |
| completed / cancelled | nothing, these are final |

Delayed and escalated are flags on a call, not statuses. Anything else returns a 409.

---

## Messages
| Method | Route | Who | What it does |
|---|---|---|---|
| GET | `/api/requests/{id}/messages` | Anyone on the call: requesting department, assigned transporters, Dispatcher | The call's message thread |
| POST | `/api/requests/{id}/messages` | Same as above | Post to the call's thread: `{ "body": "Patient not ready, coming back in 10" }` |
| GET | `/api/messages` | Transporter, Dispatcher | Your direct messages |
| POST | `/api/messages` | Transporter (to dispatchers only), Dispatcher (to anyone) | Send a direct message: `{ "toUserId": 7, "body": "Head to the ED, it's backing up" }` |

---

## Transporters
### For the transporter (mobile app)
| Method | Route | Who | What it does |
|---|---|---|---|
| POST | `/api/transporters/me/sign-in` | Transporter | Start your shift in a sector: `{ "sectorId": 1 }` |
| POST | `/api/transporters/me/sign-out` | Transporter | End your shift. Any open calls get reassigned |
| GET | `/api/transporters/me/calls` | Transporter | Your current call(s). You never see the open queue |
| POST | `/api/transporters/me/break` | Transporter | Go on break |
| POST | `/api/transporters/me/break/end` | Transporter | Come back from break |
| POST | `/api/transporters/me/resume` | Transporter | Come back from inactive (after missing an acknowledge) |

### For the dispatcher (web app)
| Method | Route | Who | What it does |
|---|---|---|---|
| GET | `/api/transporters` | Dispatcher | The transporter board: everyone signed in, with status, sector, current call, and last known location |
| GET | `/api/transporters/{id}/activity` | Dispatcher | Everything that transporter has done since signing in |
| PATCH | `/api/transporters/{id}/sector` | Dispatcher | Move them to another sector: `{ "sectorId": 2 }` |
| POST | `/api/transporters/{id}/break` | Dispatcher | Put them on break |
| POST | `/api/transporters/{id}/break/end` | Dispatcher | Take them off break |

---

## Hospital data and lists
| Method | Route | Who | What it does |
|---|---|---|---|
| GET | `/api/sectors` | Everyone | ED, Main Hospital |
| GET | `/api/locations` | Everyone | All 45 locations with floor, sector, and flags (equipment only, no equipment parking) |
| GET | `/api/locations/{id}/rooms` | Everyone | Room numbers for a unit, like 601 to 616 |
| GET | `/api/lists` | Everyone | Every dropdown: priorities, transport modes, equipment types, clinical requirements, isolation types, delay reasons, cancel reasons |
| GET | `/api/patients?search=` | Staff (own department), Dispatcher | Find a (fake) patient by name, MRN, or room |
| GET | `/api/equipment?type=&near=` | Dispatcher | Equipment by type, nearest first (stretch: needs equipment tracking) |

### Admin
| Method | Route | Who | What it does |
|---|---|---|---|
| POST / PATCH | `/api/locations` | Admin | Add or change a location, its sector, or its flags |
| POST / PATCH | `/api/sectors` | Admin | Add or change sectors and which ones neighbor each other |
| GET / POST / PATCH | `/api/routine-runs` | Admin | Routine runs (O2 restock, Endo specimen runs): times, rotation, on or off |

Accounts and roles are managed in Keycloak (and later Active Directory), not through this API.

---

## Reports (Manager)
| Method | Route | Who | What it does |
|---|---|---|---|
| GET | `/api/reports/response-times?from=&to=` | Manager | Average time from request to assigned, acknowledged, picked up, and completed, by priority |
| GET | `/api/reports/volume?from=&to=` | Manager | Calls per shift, per department, and per hour |
| GET | `/api/reports/delays?from=&to=` | Manager | Delays and cancels by reason and by department |
| GET | `/api/reports/transporters?from=&to=` | Manager | Calls completed, average times, and escalations per transporter |

---

## Live updates (WebSocket)
The apps connect once to `/ws` (STOMP) and listen on the topics they're allowed to see:

| Topic | Who listens | What comes through |
|---|---|---|
| `/topic/requests` | Dispatcher | Every new request and every status change |
| `/topic/departments/{id}/requests` | Staff | Changes to their department's requests |
| `/user/queue/calls` | Transporter | A new call assigned to you, or one taken away |
| `/user/queue/messages` | Everyone | New messages for you |
| `/topic/transporters` | Dispatcher | Transporter status changes (break, inactive, signed in or out) |

---

## Background jobs (no route, the server runs these)
- **Assignment engine:** assigns new requests to the next available transporter in the right sector (STAT first)
- **Acknowledge timer:** unassigns calls not acknowledged within 10 minutes, reassigns them, and sets that transporter to inactive
- **Escalation timer:** flags calls as escalated after 5 minutes (STAT) or 10 minutes (regular) from when they were assigned, and alerts the dispatcher
- **Postpone timer:** puts postponed calls back in the queue when their time is up
- **Routine runs:** creates the O2 restock and Endo specimen requests on schedule
