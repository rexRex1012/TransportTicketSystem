# Lists
The fixed options used across the system. These become dropdowns in the request form and seed data in the database.

## Request types
- **Patient:** move a patient from one place to another
- **Equipment:** bring a type of equipment to a room, or move a specific piece of equipment
- **Specimen:** pick up specimens and bring them to Labs (like the Endo specimen runs)

## Priorities and timing
| Priority | Escalated after |
|---|---|
| **STAT** | 5 minutes from when it's assigned |
| **Regular** | 10 minutes from when it's assigned |

- When a call is **escalated**, the dispatcher is alerted and texts the transporter to check that everything's alright.
- When a transporter gets a call, they have **10 minutes to acknowledge** it. If they don't, the call is unassigned and reassigned to someone else, and the transporter goes **inactive** (stops getting calls).
- An inactive transporter taps **Resume** to start getting calls again.

## Transport modes
How the patient is moved:
- **Wheelchair**
- **Stretcher** (default for ED pickups)
- **Bed**

The call also shows whether the floor already has it or transport has to bring it.

## Equipment types
- Bed (must be clean)
- Stretcher
- Wheelchair
- O2 tank (full or empty)
- Code cart
- Other (described in the notes)

## Clinical requirements
- **Nurse assist / RN escort:** a nurse has to come along, like when the patient has blood running. The transporter waits for the nurse
- **One-to-one (sitter):** confused or high-risk patients who need someone with them
- **Cardiac monitor / telemetry**
- **O2:** on oxygen, so a tank goes with them
- **IV pump**
- **Ventilator:** usually needs a respiratory therapist to come along
- **Fall risk**
- **Bariatric:** may need special equipment or the Get help option
- **Isolation:** with the type from the table below

## Isolation types
| Isolation type | Used for (examples) | Transporter PPE | Patient during transport |
|---|---|---|---|
| **Contact** | MRSA, VRE, wounds | Gown and gloves | Clean gown and linens |
| **Enteric Contact** (Contact Plus) | C. diff, norovirus | Gown and gloves, wash hands with soap and water | Clean gown and linens, equipment wiped with bleach after |
| **Droplet** | Flu, RSV, whooping cough | Surgical mask (plus eye protection if close) | Surgical mask |
| **Airborne** | TB, measles, chickenpox | N95 | Surgical mask |
| **Special / Enhanced Respiratory** | COVID-19 and similar | Gown, gloves, N95, eye protection | Surgical mask |
| **Protective** (Neutropenic) | Patients with weak immune systems | Mask and clean hands, to protect the patient | Mask |

## Delay reasons
- Provider with patient
- Patient using restroom
- Patient receiving meds
- Other (with a note)

## Postpone options
- 10, 20, or 30 minutes, at the nurse's discretion

## Cancel reasons
- Patient declined
- Transport not needed
- Patient not ready
- Duplicate request
- Other (with a note)

## Call statuses
new → assigned → acknowledged → en route → arrived → picked up (wristband scan) → completed

Side states: delayed, postponed, cancelled. A call can also be flagged **escalated** while it's in progress.

## Transporter statuses
- **Available:** signed in and can get calls
- **On a call:** working a call
- **On break:** set by the transporter or the dispatcher
- **Inactive:** missed acknowledging a call. Taps Resume to come back
- **Signed out**
