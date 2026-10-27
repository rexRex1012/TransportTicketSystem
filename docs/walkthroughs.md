# Job Walkthroughs
How real transport works, step by step. These double as our user stories and drive the statuses, rules, and screens.

**Terms:** staff submit a **request**. Once it's sent to a transporter, it's a **call**. They're the same thing, just seen from different sides.

## 1. Patient trip (standard flow)
1. A department submits a request to move a patient from their room to a destination.
2. The call goes to a transporter in the pickup's sector, who has to acknowledge it (**acknowledged**). See `lists.md` for the timing rules.
3. The transporter heads to the room (**en route**), finds the nurse, and makes sure they know and that the patient is okay to go (**arrived**).
4. The transporter scans the wristband to confirm the right patient (**picked up**).
5. The nurse or tech removes anything that needs to come off before the trip.
6. The transporter brings the patient to the destination, parks them, and lets the staff there know.
7. The transporter completes the call (**completed**).

**Example:** ED N14 → ED CT. The pickup is in the ED, so an ED Sector transporter gets the call, and the patient is already on a stretcher (default for ED pickups).

## 2. STAT calls
- A STAT request goes to the top of the queue and is assigned to the next available transporter.
- It doesn't pull a transporter off a call they're already on, unless the dispatcher reassigns it.
- The phone clearly shows the call is STAT so the transporter knows to move faster.

**Example:** a STAT from North Pod to ED CT jumps ahead of a routine discharge that came in first.

## 3. Patient needs equipment from transport
1. The call shows what equipment the patient needs (wheelchair, stretcher, or bed) and whether the floor has it or transport has to bring it.
2. If transport has to bring it, the transporter grabs it first, then goes to the room.
3. From there it follows the standard flow.

**Example:** 4 East 412 → ED X-Ray. The pickup is on 4 East, so a Main Hospital Sector transporter gets the call.

## 4. Return trips
- A call ends when the patient is dropped off and the transporter completes it, even if the patient will need to go back.
- When the patient is ready, the department they're at submits a new request.
- The transporter scans the wristband again at pickup, like any other pickup.

**Example:** 6 East 612 → Radiology CT. When the scan is done, Radiology submits a new request, Radiology CT → 6 East 612.

## 5. Discharge to the Lobby
1. The nurse submits the request, or calls the dispatcher, who submits it for them.
2. The patient should already be all set when the call comes in.
3. The transporter grabs a wheelchair from the Lobby, goes to the patient's room, scans the wristband, and leaves.
4. The transporter brings the patient to the Lobby and completes the call.

**Example:** 7 West 721 → Lobby.

## 6. Equipment
Equipment requests come in two kinds:
- **Bring one:** staff request a type of equipment for a room. The start point is where that equipment is kept (like Bed Park) or left open, and the transporter grabs the nearest one. Beds must be clean.
  - **Example:** "bring a bed to 6 ICU 675."
- **Move a specific one:** staff request a specific piece of equipment from one place to another, with a fixed start and end.
  - **Example:** "bring the bed from 403 to 6 ICU."

## 7. O2 restock (routine run)
1. The request is created automatically at 12:00am and 7:00pm for the next floor in the rotation.
2. The transporter takes the O2 carrier (holds 12 tanks, 3 x 4) and fills it with full tanks at the O2 Room.
3. Each department on the floor should have 6 full tanks. Most already have some, so the transporter just tops them up to 6 and swaps out any empty ones.
4. 12 tanks is usually more than enough. If the floor needs more, the transporter makes another trip as part of the same call.
5. The transporter brings the empty tanks back to the O2 Room and completes the call.

## 8. Endo specimen run (routine run)
1. The request is created automatically at 12:00am, 2:00pm, and 4:30pm.
2. At Endo, the specimens are in a bin, each labeled with a patient sticker (first and last name).
3. Next to the bin is a binder with sign-off sheets. For each specimen, the transporter signs with the time picked up and their signature.
4. If a patient's info isn't on the sheet, the transporter does **not** take that specimen.
5. The transporter brings the specimens to Labs and completes the call.

## 9. Getting help (multi-transporter calls)
- On any call, the transporter can hit **Get help** and ask for up to 3 more transporters.
- The next transporters who finish a call or are free get the same call right away, and everyone works it together.
- This covers bed moves and anything else that needs more than one person.

## 10. Nurse assist
- Some trips need a nurse or one-to-one to come along. The call is marked **nurse assist necessary**.
- The transporter has to wait for the nurse before leaving.

**Example:** a patient with blood running needs a nurse on the trip.

## 11. Isolation patients
- The transporter gowns up with gloves and an N95.
- The patient must wear a mask.

## 12. Between calls
- When a transporter has no calls, they can sit wherever they want or go to the office (Transport Home Base). They can't leave the hospital.
- The system uses their last known location (where they completed their last call) when assigning the next one.

## Delay, postpone, cancel
| Action | When | Reasons / options |
|---|---|---|
| **Delay** | The call is held up but still happening | Provider with patient, patient using restroom, patient receiving meds, etc. |
| **Postpone** | The nurse asks to push it back | 10, 20, or 30 minutes, at the nurse's discretion. The call goes back in the queue when the time is up |
| **Cancel** | The call isn't happening | Patient declined, transport not needed, patient not ready, duplicate request, etc. |

## Statuses
new → assigned → acknowledged → en route → arrived → picked up (wristband scan) → completed

Side states: delayed (with a reason), postponed (back in the queue after 10/20/30 minutes), cancelled (with a reason). A call can also be flagged **escalated** (see `lists.md`).
