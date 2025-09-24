# APDM-ALERT2.TEXT.unix.txt - Technical Analysis

**Analysis Date**: January 2025  
**LLM Model**: ChatGPT-4o  
**Prompt Used**: Appendix A - Technical Software Analysis  
**Source**: Computer History Museum Apple Lisa Code Archive

## LLM Analysis Output

**Apple Lisa Source Code Analysis – APDM-ALERT2.TEXT.unix.txt**

**Overview**

This file, APDM-ALERT2.TEXT.unix.txt, is a structured resource containing system alert messages used by the Apple Lisa Office System (LOS). These alerts are tied to various low-level and high-level operations such as disk I/O, document management, tool invocation, backups, and error handling. The alerts are indexed numerically and reflect different **types** of UI interactions such as stop, ask, wait, note, and caution alerts.

This analysis proceeds linearly, interpreting each block and citing code-related references, embedded variables, comments, and associated system functions.

---

**Structured Alert Entries**

Each alert follows a consistent format:

1.	**Alert Code and Type** (e.g., 200 stop alert)

2.	**Alert Text**: The message shown to the user.

3.	**Embedded Variables**: Denoted ^1, ^2, etc., replaced at runtime with dynamic values.

4.	**Developer Comments**: Begin with semicolons (;) and include technical descriptions and internal references.

5.	**Reference Functions**: Each block ends with internal Lisa system function references, e.g., FGlobals.SysErr.

---

**Lines 1–16: Alert 200 – Non-recoverable system error**

```
200 stop alert
The Lisa is having technical difficulties.  The system will restart.
^L^L
If the problem recurs, refer to the Lisa Office System manual,
Appendix A, Office System Error Messages, under System Restart.
^L^L
If you report this difficulty, please mention the number ^1^2.
```

**Developer Comments:**

```
; A non-recoverable Filer error was detected.
; 1) Filer err, 2) Low-level err from os, database, etc.
; References:
;   FGlobals.SysErr
```

**Analysis:**

•	**Purpose**: Signals a fatal error from the file system (Filer), causing an immediate restart.

•	**Runtime Behavior**: Embeds two dynamic values representing error codes.

•	**Reference Function**: FGlobals.SysErr handles global error tracking and is likely responsible for escalation and logging.

•	**Implication**: Catastrophic failure not recoverable by user action.

---

**Lines 17–33: Alert 201 – User-acknowledged system error**

```
201 ask alert
The Lisa is having technical difficulties.  You may continue at your
own risk by clicking Continue.  To restart The Lisa, click Restart.
```

**UI Options:**

```
^!Restart^?Continue
```

**Comments:**

```
; A possibly non-fatal Filer error was detected.
; 1) Filer err, 2) Low-level err from os, database, etc.
; References:
;   FGlobals.SysErr
```

**Analysis:**

•	**Purpose**: Informs the user of a fault with an option to continue or restart.

•	**Structure**: The ^! and ^? syntax designates UI buttons (Restart, Continue).

•	**System Behavior**: Allows user discretion; may be recoverable.

---

**Lines 34–44: Alert 202 – Powering down**

```
202 wait alert
The Lisa is turning off.
...
You may be able to terminate the operation by holding
down the Apple key while you type a period.
```

**Comments:**

```
; User pushed the power off button. Filer is shutting down the system.
; References:
;   FVolCtrl.PowerItDown (1902)
```

**Analysis:**

•	**System Function**: FVolCtrl.PowerItDown initiates a controlled shutdown.

•	**User Intervention**: Provides keyboard override.

•	**Hardware Dependency**: Tied to the Lisa’s hardware power controls.

---

**Lines 45–54: Alert 203 – File read error**

```
203 stop alert
The Lisa is having technical difficulties accessing ^1 on ^2.
```

**Comments:**

```
; Create/Copy/Move a document. OS cannot read from the source diskfile.
; 1) document name, 2) disk name.
; References:
;   FDocCtrl.TellUserWhy (2726) error fCantRead
```

**Analysis:**

•	**Context**: Fails during file operation due to disk read error.

•	**Core Subsystem**: FDocCtrl (Document Controller).

•	**Error Code**: fCantRead indicates read failure from storage.

---

The file continues with **incremental alert messages** structured in a similar way, each containing:

•	**Contextual UI messages**

•	**References to dynamic variables** (^1, ^2, etc.)

•	**Developer annotations explaining the context or the triggering subsystem**

•	**Function and file references** (e.g., FDocCtrl.TellUserWhy, FVolCtrl.JustMount, Filer.Initialization)

Each of these reference calls corresponds to low-level Pascal or assembly routines in the Lisa system libraries and file management tools.

---

**Architectural Insight**

The Lisa alert system is:

•	**Data-driven**: The alert messages are stored in a flat file and dynamically filled with runtime values.

•	**Localized**: Alerts reference Lisa-specific manuals for recovery steps.

•	**Structured**: Each alert maintains a strict syntax and pattern for both display and system behavior.

**Subsystem Mapping**

•	FDocCtrl: Document Controller – Handles documents, backups, transfers.

•	FVolCtrl: Volume Controller – Manages disks, volumes, and their state.

•	Filer: UI layer interfacing with file operations.

•	Desk, Vol, Apdm/Desk*: User interface and disk operations menus.

---

**Summary of Techniques**

•	**Embedded UI Markup**: ^!, ^? mark buttons. ^1, ^2 are dynamic fields.

•	**Structured Comments**: Every alert is thoroughly documented with cause and system reference.

•	**Clear System Hooks**: Each alert is connected to a handler or generator function, enabling traceability.

•	**ASCII Blocks for Instructions**: Uses ^L^L to indicate screen formatting (likely form feed or spacing for display clarity).

---

Would you like me to continue parsing the remaining alerts one by one into a full structured analysis document as specified?

Continuing the full technical breakdown of the Apple Lisa alert system file APDM-ALERT2.TEXT.unix.txt, we now proceed sequentially from:

---

**Lines 55–68: Alert 204 – Write Error to Disk**

```
204 stop alert
The Lisa is having technical difficulties writing ^1 to ^2.
...
Refer to the Lisa Office System manual... under
Difficulty Saving Document.
```

**Developer Comments:**

```
; Create/Copy/Move a document. OS cannot write to the destination diskfile.
; 1) document name, 2) disk name.
; References:
;   FDocCtrl.TellUserWhy (2732) error fCantWrite
```

**Analysis:**

•	**Dynamic Fields**: ^1 = document name, ^2 = disk name.

•	**Subsystem**: FDocCtrl.TellUserWhy presents error dialogs.

•	**Error Trigger**: Disk write operation failed.

•	**Error Code**: fCantWrite, likely tied to OS file system failure or full disk sectors.

---

**Lines 69–85: Alert 205 – No Disk Space or Too Many Documents**

```
205 stop alert
The Lisa cannot place ^1 on ^2 because either there is not enough room or there are too
many documents on the disk.
```

**Developer Comments:**

```
; Create/Copy/Move a document. No room on the destination disk.
; 1) document name, 2) disk name.
; References:
;   FDocCtrl.TellUserWhy (2755) error fNoSpace
;   FDocCtrl.XferObj (2947) error fNoSpace while back up in progress
```

**Analysis:**

•	**Multiple Error Sources**:

•	General: fNoSpace (destination full).

•	During backup: XferObj (transfer object fails).

•	**Design Detail**: Reflects both capacity and file count constraints.

---

**Lines 86–101: Alert 206 – Memory Limit for Open Documents**

```
206 stop alert
The Lisa is having technical difficulties displaying this many documents.  Try
putting away some documents...
```

**Developer Comments:**

```
; Pull, Mount. Window manager limit of 20(?) windows or OS needs memory.
; References:
;   FDocCtrl.TellUserWhy (2743) error fNoMemory
;   FVolCtrl.JustMount (699) OS Mount error 864, 870, 872
;   Filer.NewWindow () WM would not create a window, too many windows
```

**Analysis:**

•	**Error Code**: fNoMemory — either OS memory exhaustion or UI limit.

•	**Window Management**: Potential hard limit of ~20 simultaneous document windows.

•	**Fail Points**: Both mounting and UI window creation may independently trigger the alert.

---

**Lines 102–106: Alert 207 – System Restarting**

```
207 wait alert
The Lisa is restarting.
```

**Developer Comments:**

```
; Following alert 200.
; References:
;   Desk.LastWishes
```

**Analysis:**

•	**Trigger**: Follows fatal condition (alert 200).

•	**Function**: Desk.LastWishes likely handles shutdown state saving or cleanup.

•	**System Behavior**: Controlled system restart initiated by Filer or kernel.

---

**Lines 107–119: Alert 208 – Not Enough Disk Space to Close a Document**

```
208 stop alert
^1 cannot be closed because there is not enough room left on ^2.
```

**Developer Comments:**

```
; Put Back/Twiggy button. Tool ran out of disk space while closing doc's diskfiles.
; 1) doc name, 2) disk name.
; References:
;   FDocCtrl.TellUserWhy (2751) error fNoSpace while putting doc away.
```

**Analysis:**

•	**Operation**: Finalizing/saving a document using “Put Back.”

•	**Trigger**: Destination disk lacks space to complete the close/save operation.

•	**Legacy Note**: “Twiggy” was the name for Lisa’s original 5¼” drive system.

---

**Lines 120–138: Alert 209 – Damaged Document Handling**

```
209 ask alert
^1 could not do as you asked...
...Release Doc
```

**UI Buttons**:

```
^!Cancel^?Release Doc
```

**Developer Comments:**

```
; Put Back/Twiggy button. Tool detected inconsistencies or crashed while closing doc.
; 1) tool name.
; References:
;   FDocCtrl.TellUserWhy (2772) error fToolErr
```

**Analysis:**

•	**Failure State**: Tool detected corruption or crashed mid-operation.

•	**User Control**: Can forcibly release doc from a hanging state, risking damage.

•	**Handler Function**: Uses fToolErr, marks tool or doc as corrupted/inconsistent.

---

**Lines 139–144: Alert 210 – Boot Splash Message**

```
210 wait alert
^1  7/7
...
Office System ^2
...
^31983, 1984  ^4apple computer, inc.
```

**Developer Comments:**

```
; Booting up.
; 1) version, 2) (c).
; References:
;   Filer.Initialization (4096) booting up the Filer, initing various intrinsics
```

**Analysis:**

•	**UI Text**: Displayed during Lisa OS boot.

•	**Key System Function**: Filer.Initialization—major boot-time setup.

•	**Historical Stamp**: This embeds the 1983–1984 copyright.

---

**Lines 145–170: Alert 211 – Disk Not in Lisa Format**

```
211 ask alert
The ^1 is not in the standard Lisa Office System format.
...
^!Cancel^?Continue^?Repair^?Initialize
```

**Developer Comments:**

```
; Diskette inserted. Filer could not open its catalog or system crashed when disk was
;                    last mounted.
; 1) device name.
; References:
;   FVolCtrl.JustMount (627) Error opening catalog
```

**Analysis:**

•	**System Failure**: Volume catalog could not be read — could indicate filesystem corruption or foreign format.

•	**User Options**:

•	Continue: Assume compatible.

•	Repair: Attempt restoration.

•	Initialize: Reformat completely.

•	**Subsystem**: FVolCtrl.JustMount handles mounting and error detection.

---

To maintain your instruction of a **full technical pass**, I will now proceed through **Alerts 212–220** in the next segment. Would you like to continue immediately or should I prepare a downloadable document of the full analysis as I proceed?

Continuing the **full structured technical pass** of APDM-ALERT2.TEXT.unix.txt:

---

**Lines 171–180: Alert 212 – Boot Disk Repair Failure**

```
212 note alert
The Lisa did not repair the ^1.  You will have to restart using the
Office System 1 diskette...
```

**Developer Comments:**

```
; Attempt to repair a broken boot disk failed or user said cancel to 247.
; 1) device name.
; References:
;   FVolCtrl.InitVolCtrl (1808) Error opening cat on boot disk, was not repaired
```

**Analysis:**

•	**Trigger**: Disk repair (boot volume) failed or user declined it.

•	**Reference Routine**: FVolCtrl.InitVolCtrl handles initialization and repair attempts during boot.

•	**Implication**: Recovery not possible — fallback to OS installation media.

---

**Lines 181–187: Alert 213 – Reuse Prompt After Failed Backup**

```
213 ask alert
Do you want to reuse the disk?
^!Don't Reuse^?Reuse
```

**Developer Comments:**

```
; Back up. Back up to a fixed disk has failed.
; References:
;   FVolCtrl.AnotherDiskette (280) Failure backingup disk to disk.
```

**Analysis:**

•	**Context**: Backup operation to a fixed (non-removable) disk failed.

•	**User Option**: May reuse the media or cancel the operation.

•	**System Function**: FVolCtrl.AnotherDiskette appears to iterate over diskettes or volumes for backup.

---

**Lines 188–193: Alert 214 – Not Formatted**

```
214 ask alert = 211
```

**Developer Comments:**

```
; Diskette inserted. OS reports that it is not initialized/formatted.
; 1) device name
; References:
;   FVolCtrl.JustMount (683) OS Mount err 606,1197. Bad formatting
```

**Analysis:**

•	**Alias Alert**: Same logic as alert 211, but specifically triggered by unformatted media.

•	**OS Error Codes**: 606, 1197 (likely internal Lisa OS codes for unreadable format).

•	**Response Option**: Presents same Cancel, Continue, Repair, Initialize choices.

---

**Lines 194–199: Alert 215 – Disk Initialization in Progress**

```
215 wait alert
The Lisa is initializing the ^1.
```

**Developer Comments:**

```
; User responded affirmatively to 214.
; 1) device name
; References:
;   FVolCtrl.JustMount (687), (726)
```

**Analysis:**

•	**Runtime State**: Engaged only after user chooses to Initialize a disk from alert 214.

•	**Dual References**:

•	(687): Execution of initialization.

•	(726): Possibly finalizing formatting.

•	**Implied Subroutine Flow**: Internal OS state machine routes through alerts 214 → 215.

---

**Lines 200–209: Alert 216 – Manual File Recovery Required**

```
216 stop alert
You will have to open ^1 and then put it away through the File/Print menu.
```

**Developer Comments:**

```
; PutBack/in/copy an open document. App says it may be inconsistent.
; User must open the window and force consistency.
; 1) doc name
; References:
;   FDocCtrl.TellUserWhy (2737) error fDirtyDoc
```

**Analysis:**

•	**Error Code**: fDirtyDoc — document flagged as possibly inconsistent.

•	**Recovery Flow**: User intervention required; tool cannot safely auto-close or file the document.

•	**Implication**: Lisa OS prioritizes data integrity, avoiding silent data loss.

---

**Lines 210–213: Alert 217 – Unknown Tool Error**

```
217 stop alert
The Lisa was unable to do as you asked.
```

**Developer Comments:**

```
; Received an 'internal error' from a tool but there is no info about what was
; being done.
```

**Analysis:**

•	**Minimal Info**: Catch-all fallback error when no contextual metadata is available.

•	**Likely Trigger**: A tool crashes without logging error context.

•	**Design Note**: Prevents crash-to-desktop, still provides user with UI feedback.

---

**Lines 214–221: Alert 218 – Unusable Disk or Drive Failure**

```
218 stop alert
The Lisa cannot use the ^1.  The disk might be damaged or the disk drive might
not be working.
```

**Developer Comments:**

```
; Disk could not be initialized.
; 1) device name.
; References:
;   FDocCtrl.XferSysFiles (3279)
;   FVolCtrl.InitNewVol (464)
;   FVolCtrl.DocBackup (1128)
;   FVolCtrl.GetNextDiskette (1744, 1755)
```

**Analysis:**

•	**Failure Sources**:

•	Write: XferSysFiles

•	Initialize: InitNewVol

•	Erase/Backup: DocBackup, GetNextDiskette

•	**Hardware Triggered**: Includes physical failures (e.g., unreadable magnetic media or controller fault).

---

**Lines 222–233: Alert 219 – Disk Initialization Failure**

```
219 ask alert
The Lisa cannot initialize the ^1 for use by the Lisa Office System.
...
^?Cancel^!Continue
```

**Developer Comments:**

```
; Broken disk was not repaired.
; 1) device name
; References:
;   FVolCtrl.JustMount (742) OS Scavenge failed
;   FVolCtrl.RepairVol (2028) Filer catalog could not be repaired
```

**Analysis:**

•	**Catalog Repair Failure**: Core structure (filer catalog) could not be fixed.

•	**System Path**: JustMount → RepairVol

•	**Final Option**: User can opt to skip or ignore the disk.

---

**Lines 234–245: Alert 220 – Repair Unsuccessful**

```
220 ask alert
The Lisa cannot repair the ^1 for use by the Lisa Office System.
...
^?Cancel^!Continue
```

**Developer Comments:**

```
; Broken disk was not repaired.
; 1) device name
; References:
;   FVolCtrl.JustMount (742)
;   FVolCtrl.RepairVol (2028)
```

**Analysis:**

•	**Identical to 219** in logic and reference but likely differentiated by context (e.g., different menu path or user action).

•	**Purpose**: Reinforces user-facing clarity — same backend, distinct UI phrasing.

---

**Summary of Alerts 212–220**

| **Alert** | **Type** | **Purpose** | **Key Function** |
| --- | --- | --- | --- |
| 212 | Note | Boot disk not repaired | InitVolCtrl |
| 213 | Ask | Ask user to reuse failed backup media | AnotherDiskette |
| 214 | Ask | Non-initialized disk detected | JustMount |
| 215 | Wait | Disk is being initialized | JustMount |
| 216 | Stop | Document must be manually put away | TellUserWhy |
| 217 | Stop | Unclassified tool error | (none) |
| 218 | Stop | Disk or drive error | InitNewVol, XferSysFiles |
| 219 | Ask | Failed to initialize repaired disk | JustMount, RepairVol |
| 220 | Ask | Repair attempt failed | same as 219 |

---

Next segment will continue from **Alert 221 – Disk Repair In Progress** onward. Shall I proceed?

Continuing the structured, full technical breakdown of APDM-ALERT2.TEXT.unix.txt, this segment covers:

---

**Lines 246–260: Alert 221 – Disk Repair In Progress**

```
221 wait alert
The Lisa is repairing the ^1.  When the icons reappear, some may have returned
to former locations.
```

**Developer Comments:**

```
; Attempting to repair a broken disk.
; 1) device name
; References:
;   FVolCtrl.JustMount (630), (734)
;   FVolCtrl.InitVolCtrl (1100)
;   Filer.RepairDisk ()
```

**Analysis:**

•	**Execution**:

•	Disk has been flagged as needing repair.

•	The system enters a modal wait state while repairs are underway.

•	**Functions**:

•	FVolCtrl.JustMount: Triggered on mount failure, initiates repair.

•	InitVolCtrl: Handles boot volume repair.

•	Filer.RepairDisk: Explicit user action via Housekeeping menu.

•	**Runtime Note**: Repaired folder icons may shift — indicates index rebuild or catalog regeneration.

---

**Lines 261–267: Alert 222 – Unexpected Mount Error**

```
222 ask alert = 211
```

**Developer Comments:**

```
; Unexpected error mounting a disk.
; 1) device name, 2) OS error number
; References:
;   FVolCtrl.JustMount (721)
```

**Analysis:**

•	**Alias Alert**: Reuses Alert 211, but specifically for “unexpected” mount errors.

•	**Distinguishing Factor**: Captures system-level errors not tied to known formatting issues.

•	**Error Handling**: Passed to user for decision (initialize, repair, etc.).

---

**Lines 268–276: Alert 223 – Tool Redisplay Attempt**

```
223 ask alert
The ^1 tool is having technical difficulties displaying ^2.
...
^?Don't Redisplay^?Redisplay
```

**Developer Comments:**

```
; Tool crash. Filer has cleaned up and reclaimed associated doc windows.
; 1) tool name, 2) doc name
; References:
;   FDocCtrl.SonTerminated (2558)
```

**Analysis:**

•	**Subsystem**: FDocCtrl.SonTerminated indicates the “tool” process (child) terminated unexpectedly.

•	**UI Path**:

•	Recovered doc windows via Filer.

•	User can choose to attempt reinitialization of rendering.

•	**Design Insight**: Lisa’s Filer supports graceful degradation and recovery from process-level faults.

---

**Lines 277–284: Alert 225 – Disk Unmount Dependency**

```
225 wait alert
The Lisa must shut down ^1 before the ^2 can be released...
```

**Developer Comments:**

```
; Unmounting a disk that has open diskfiles.
; 1) process/tool name, 2) device name
; References:
;   FVolCtrl.UnmountVol (923)
```

**Analysis:**

•	**Constraint**: OS cannot unmount a disk with active open files.

•	**System Behavior**:

•	Begins clean shutdown of dependent processes.

•	Some UI artifacts (blank windows) may appear due to forced closure.

•	**Function**: UnmountVol manages teardown; checks file locks and live handles.

---

**Lines 285–297: Alert 228 – Confirm Disk Erase**

```
228 ask alert
WARNING: Everything on ^1, the ^2, is about to be erased...
^!Cancel^?Erase
```

**Developer Comments:**

```
; Disk reinitialization.
; 1) name of disk.
; 2) user name of device.
; References:
;   FVolCtrl.BackupVol (1321)
;   FVolCtrl.GetNextDiskette (1738)
;   Filer.EraseDisk (3004)
```

**Analysis:**

•	**Pre-Destructive Operation Warning**: Confirms full erasure before formatting or backup.

•	**Multiple Use Cases**:

•	Direct user erase via Filer.EraseDisk

•	Disk swapping for backup (GetNextDiskette)

•	**Human-Centered Design**: Reiterates irreversibility despite UI affordances like Undo.

---

**Lines 298–304: Alert 229 – Prompt for Diskette Insertion**

```
229 caution ok alert
Please insert the next diskette into the ^1 drive.
```

**Developer Comments:**

```
; disk backup/restore to/from diskettes.
; 1) UPPER or LOWER.
; References:
;   FVolCtrl.AskForDiskette (291)
```

**Analysis:**

•	**Modular Disk Operations**: Lisa OS supports multi-disk backups with user-driven media swapping.

•	**Slot Identifier**: ^1 is either “UPPER” or “LOWER” — hard-coded drive bay references.

•	**Routine**: AskForDiskette orchestrates human-computer interaction for physical device operations.

---

**Lines 305–313: Alert 230 – Deleting Master Tool Warning**

```
230 ask alert
You are about to discard the master copy of the ^1 tool...
^!Keep^?Discard
```

**Developer Comments:**

```
; Shredding the master copy of a tool.
; References:
;   Filer.MoveToTrash ()
```

**Analysis:**

•	**Protection Mechanism**: Prevents accidental deletion of critical system components (master tools).

•	**Subsystem**: Filer.MoveToTrash identifies special-case objects requiring confirmation.

•	**UI Design Note**: Differentiates between normal icons and core system artifacts.

---

**Lines 314–323: Alert 231 – Backup Operation Failure**

```
231 ask alert
The Lisa cannot complete the backup operation.
...
^!Cancel^?Retry
```

**Developer Comments:**

```
; Backup between two identical media failed.
; References:
;   FVolCtrl.BackupVol (1094)
```

**Analysis:**

•	**Typical Failure Case**: Two devices with same volume labels, incompatible capacities, or format mismatch.

•	**Behavior**: Alert prompts retry or cancellation.

•	**Routine**: BackupVol checks preconditions and volume metadata before committing.

---

**Summary of Alerts 221–231**

| **Alert** | **Type** | **Purpose** | **Subsystem** |
| --- | --- | --- | --- |
| 221 | Wait | Repairing disk | Filer.RepairDisk |
| 222 | Ask | Unexpected mount error | JustMount |
| 223 | Ask | Tool crash, retry display | SonTerminated |
| 225 | Wait | Tool must close before unmount | UnmountVol |
| 228 | Ask | Confirm full disk erase | BackupVol, EraseDisk |
| 229 | Caution | Insert next diskette | AskForDiskette |
| 230 | Ask | Discarding master tool | MoveToTrash |
| 231 | Ask | Retry failed backup | BackupVol |

---

---

**Lines 324–335: Alert 232 – Backup Scope Confirmation**

```
232 ask alert
The last full backup of ^1 was ^2.
...
^!Cancel^?Full Backup^?Changes Only
```

**Developer Comments:**

```
; Backup. Asked if the destination media is smaller than the source.
; 1) name of source disk, 2) date of last complete back up.
; References:
;   FVolCtrl.BackupVol (1341)
```

**Analysis:**

•	**Purpose**: Prompts user to select full or incremental backup.

•	**Conditions**: Typically appears when media capacity is questioned.

•	**System Routine**: BackupVol performs decision logic for backup size & type.

---

**Lines 336–339: Alert 234 – Disk Erasure Message**

```
234 wait alert
The Lisa is erasing the ^1.
```

**Developer Comments:**

```
; Recycle. Unmounting disk before beginning initialization sequence.
; 1) device name.
; References:
;   Filer.EraseDisk (3009)
```

**Analysis:**

•	**Context**: Confirmed disk erase in progress.

•	**Lifecycle**: Unmount → Format → Ready.

•	**UI Behavior**: Non-interruptible. Purely status feedback.

---

**Lines 340–348: Alert 236 – Backup In Progress (Disk to Disk)**

```
236 wait alert
The Lisa is backing up ^1, the ^2, to the ^3.
...
Apple key + period to terminate.
```

**Developer Comments:**

```
; Backup. Transferring diskfiles from source to destination.
; 1) source disk name, 2) source device name, 3) dest device name
; References:
;   FDocCtrl.XferObj (2971)
;   FVolCtrl.BackupVol (1138), (1359)
```

**Analysis:**

•	**Process-Oriented**: Ongoing file transfer.

•	**Interruption Support**: Apple + . command escape.

•	**Subsystems**:

•	XferObj: Transfers doc/tool files.

•	BackupVol: Coordinates bulk copy with error tracking.

---

**Lines 349–358: Alert 237 – Backup Failure for Specific File**

```
237 ask alert
The Lisa cannot copy ^1.  To back up the remaining documents and tools, click Continue.
^?Cancel^!Continue
```

**Developer Comments:**

```
; Backup. A diskfile was not transferred. Also see :fd296.
; 1) doc/tool name.
; References:
;   FDocCtrl.XferObj (2971)
```

**Analysis:**

•	**Partial Failure**: A specific file fails mid-backup, user can continue without it.

•	**Consistency Concern**: Allows user to skip known bad data but retain rest of archive.

•	**UI Pattern**: Same handling appears in restore flow (alert 296).

---

**Lines 359–364: Alert 238 – Generic Continue After Failure**

```
238 ask alert
Do you want to continue this operation?
^?Cancel^!Continue
```

**Developer Comments:**

```
; Backup. User did not mount a diskette when asked to or
;         a diskfile could not be transferred.
; References:
;   FVolCtrl.AnotherDiskette (259)
```

**Analysis:**

•	**Recovery Handling**: General case prompt for failure to insert media or transfer a file.

•	**State-Agnostic**: Used for both file-level and device-level interruptions.

•	**References**: Wraps around AnotherDiskette, which supports interactive restore/backup.

---

**Lines 365–376: Alert 239 – Disk Repair Failure (Scavenger)**

```
239 stop alert
The ^1 was damaged, and the Lisa is unable to repair it.
...
under Damaged Disk.
```

**Developer Comments:**

```
; Error returned from scavenger.  There might be hardware problems.
; 1) device name
; References:
;   FVolCtrl.JustMount (608)
```

**Analysis:**

•	**Scavenger Role**: Internal subsystem that attempts to auto-repair a volume’s catalog/data structures.

•	**Failure Classification**: If scavenger fails, repair must be attempted externally.

•	**Behavior**: Disk is unusable until further recovery or reinitialization.

---

**Lines 377–381: Alert 241 – Disk Successfully Repaired**

```
241 note alert
The ^1 has been checked for damage and is now ready to use.
```

**Developer Comments:**

```
; Following scavenging. The scavenger made changes to the disk.
; 1) device name
; References:
;   FVolCtrl.JustMOunt (656)
;   FVolCtrl.InitVolCtrl (1828)
```

**Analysis:**

•	**Repair Outcome**: Confirmation that automated repair succeeded.

•	**Mount Behavior**: Flags disk as safe to use again.

•	**Visual Reassurance**: Aimed at restoring user confidence after a technical scare.

---

**Lines 382–390: Alert 242 – Crash-Detected Disk Reinserted**

```
242 ask alert
The ^1 was in use when the Lisa failed...
^?Don't Check^?Check
```

**Developer Comments:**

```
; Mount.
; 1) device name
; References:
;   FVolCtrl.JustMount (595)
```

**Analysis:**

•	**Crash Handling**: Lisa tracks disk activity during crashes.

•	**Disk State Marker**: Likely uses volume metadata to flag “unclean unmounts.”

•	**Option to Scavenge**: User may inspect disk for possible catalog corruption.

---

**Summary of Alerts 232–242**

| **Alert** | **Type** | **Purpose** | **Key Function(s)** |
| --- | --- | --- | --- |
| 232 | Ask | Choose backup type | BackupVol |
| 234 | Wait | Erasing disk | EraseDisk |
| 236 | Wait | Backup in progress | BackupVol, XferObj |
| 237 | Ask | File backup failure | XferObj |
| 238 | Ask | Retry/continue backup | AnotherDiskette |
| 239 | Stop | Scavenger failed | JustMount |
| 241 | Note | Disk successfully repaired | JustMount, InitVolCtrl |
| 242 | Ask | Crash recovery disk check | JustMount |

---