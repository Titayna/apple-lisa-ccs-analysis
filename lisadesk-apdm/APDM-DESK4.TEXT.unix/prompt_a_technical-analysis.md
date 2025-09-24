# APDM-DESK4.TEXT.unix.pas - Technical Analysis

**Analysis Date**: January 2025  
**LLM Model**: ChatGPT-4o  
**Prompt Used**: Appendix A - Technical Software Analysis  
**Source**: Computer History Museum Apple Lisa Code Archive

## LLM Analysis Output

**📘 Full Technical Software Analysis: APDM-DESK4.TEXT.unix.txt**

**Date:** 5-Dec-1983

**System:** Apple Lisa Desktop

**Component:** Attribute Dialog Box Routines

**Language:** Pascal

**Focus:** GUI attribute dialog interactions, password logic, object metadata

---

**📄 File Header and Developer Comments**

**Lines 1–16:**

```
{apdm/desk4.text 5-Dec-83} {Copyright 1983, Apple Computer Co.}

{Contains the dialog box specific routines for putting up a particular dialog box}

{**** Modification Log ****}
{Created the new file : 12/5/83}
{Multiple mods : 12/6/83 to 12/13/83}
{Splitting stored on/stored in line, adding ClipDiName call : 12/14/83}
{Cleaning up interface, setting real diInfo fields, does split doc info : 12/15/83}
{Added password checking, new password record defs, real button code : 12/20/83}
{Re-wrote most of the display code (no notepad, what a drag) : 12/30/83}
{GetToolName call : 1/25/84}
{Calls to reset password values, cleanup of password states : 3/14/84}
```

**✅ Commentary:**

These comments document both the origin and iterative development of this source module. They:

•	Confirm its role: managing a GUI dialog box for attributes.

•	Highlight key feature evolution, such as:

•	Password dialog mechanics

•	Storage labeling (stored on vs stored in)

•	GUI cleanup and integration with naming utilities (e.g., ClipDiName)

•	Rewriting the display logic entirely by end of December 1983

•	Continued updates into 1984

Preserved formatting such as “(no notepad, what a drag)” offers rare insight into the developers’ frustrations and personalities — this is historically significant and has been preserved without modification.

---

**🔧 Section: {$S flrAtt} – Dialog Box Routines (Password Box)**

**🔹 Procedure: AttCancelBtn**

**Lines 19–28:**

```
{$S flrAtt}
PROCEDURE AttCancelBtn;
{called when the 'Cancel' button is pushed in the password box, cleans up password stuff}

BEGIN
{$IFC flrDebug } IF trcCalls THEN ALogCall; {$ENDC}
ShowDiBtn(diPassword.cancelBtnPanel, TRUE{selected});

IF (diPassword.state = Verified) THEN
   diPassword.state := safe;

ProtAttDiBox;                 { make sure that everything is displayed correctly }

ResetPwdFields;

ShowDiBtn(diPassword.cancelBtnPanel, FALSE);
END; {of PROC AttCancelBtn}
```

**🧠 Function Breakdown:**

•	**Name:** AttCancelBtn

•	**Role:** Handles GUI behavior and state cleanup when the “Cancel” button is pressed in a password dialog.

•	**Key Operations:**

•	Visually shows the cancel button as selected/deselected.

•	If the password dialog was in a Verified state, it reverts to safe.

•	Calls ProtAttDiBox to redraw appropriate dialog fields.

•	Clears the password field via ResetPwdFields.

**🗨 Comments:**

•	The inline and block comments are **faithfully preserved**.

•	Inline usage like TRUE{selected} and { make sure that everything is displayed correctly } exemplify Apple’s convention of placing logic cues directly inline for readability.

---

**🔹 Procedure: AttDoneBtn**

**Lines 31–45:**

```
PROCEDURE AttDoneBtn;
{called when the 'Done' button is pushed in the main dialog panel.  Dismisses the
 dialog box, should also check for inconsistant password change/set state }

BEGIN
{$IFC flrDebug } IF trcCalls THEN ALogCall; {$ENDC}
CASE (diPassword.state) OF
   Safe, Unprotected:
      BEGIN
      {verify that the post file is set correctly}
      IF (NOT VerifyPassword(diBox.obj^^.volHdl, diBox.obj^^.catRID)) THEN
         FixPassword(diBox.obj^^.volHdl, diBox.obj^^.catRID);
      END; {of CASE ITEM safe}
   OTHERWISE BEGIN END;
   END; {of CASE block}

StopDiBox;

END; {of PROC AttDoneBtn}
```

**🧠 Function Breakdown:**

•	**Name:** AttDoneBtn

•	**Role:** Triggered by the “Done” button; closes the attribute dialog and attempts to finalize password state.

•	**Key Logic:**

•	If in Safe or Unprotected state:

•	Verifies the password using VerifyPassword.

•	If verification fails, invokes FixPassword to restore a valid state.

•	Closes the dialog using StopDiBox.

**🗨 Comments:**

•	The original block comment documents both **UI intent** and **security concern** (“inconsistent password change/set state”).

•	Pascal case logic is followed with clear comment tags, e.g., {of CASE ITEM safe} — this was a common 1980s Pascal style for aiding manual traceability.

---

**🔹 Procedure: AttEnterBtn**

**Lines 48–118:**

```
PROCEDURE AttEnterBtn;
{called when the 'Enter' button is pushed in the password panel of the dialog box.
 Sets the global password string to 'Delts' (for testing only).  Should eventually
 set the password value, check for validity, call SetObjPwd, etc.}

VAR
   err      : INTEGER;
   numStr   : NumberStr;
   tempStr  : FmaxStr;

BEGIN
{$IFC flrDebug } IF trcCalls THEN ALogCall; {$ENDC}
ShowDiBtn(diPassword.enterBtnPanel, TRUE{selected});

{set up name to use for the alert, we know that diBox.obj exists}
GetStrg (diPassword.iconName, tempStr);
QuoteName (tempStr);
ArgAlert (1, tempStr);

CASE (diPassword.state) OF
   safe:
      BEGIN
      IF GetPassField THEN
         BEGIN
         {get the password entered here and stuff into the old field}
         diPassword.old := diPassword.entered;
         Verify_Password (err, diPassword.path, diPassword.old);
         IF (err > 0) THEN
            BEGIN
            StopAlert (flrAlert, 327);
            ResetPwdFields;
            END
         ELSE
            BEGIN
            {old password has been verified}
            diPassword.state := verified;
            END; {of IF ELSE block}
         END; {of IF THEN block}
      END; {of CASE ITEM safe}

   Verified, Unprotected:               {  doc/tool, enter of new password }
      BEGIN
      IF GetPassField THEN
         BEGIN
         diPassword.new := diPassword.entered;
         IF (diPassword.state = Unprotected) THEN
            diPassword.old := '';
         SetObjPwd (err, diBox.obj^^.volHdl, diBox.obj^^.catRID, dipassword.old, diPassword.new);
         IF (err > 0) THEN
            BEGIN
            IF (NOT VerifyPassword(diBox.obj^^.volHdl, diBox.obj^^.catRID)) THEN
               FixPassword(diBox.obj^^.volHdl, diBox.obj^^.catRID)
            ELSE
               BEGIN
               IntToStr (err, numStr);
               ArgAlert (2, '1220/');
               ArgAlert (3, numStr);
               StopAlert (flrAlert, 325);
               END;
            ResetPwdFields;
            END
         ELSE
            BEGIN
            diBox.obj^^.password := TRUE;
            diPassword.passworded := TRUE;
            diPassword.validpasswd := TRUE;
            diPassword.state := safe;
            END;
         END {of IF THEN block}
      ELSE {getting the password failed}
         ResetPwdFields;
      END; {of CASE ITEM verified, unprotected}
   END; {of CASE block}

ProtAttDiBox;

ShowDiBtn(diPassword.enterBtnPanel, FALSE{not selected});

IF (diPassword.state = Verified) THEN
   BEGIN
   ClearDiField (diPassword.passwdPanel);
   {now make sure that the field is ready for typing}
   diPassword.passwdPanel^^.info.active := TRUE;
   diPassword.passwdPanel^^.selected := TRUE;
   diPassword.passwdPanel^^.changed := TRUE;
   END
ELSE
   ResetPwdFields;

END; {of PROC AttEnterBtn}
```

**🧠 Function Breakdown:**

•	**Name:** AttEnterBtn

•	**Purpose:** Manages password entry confirmation, validation, and state transition.

•	**Behavior Summary:**

•	**State = safe**: Verifies the old password.

•	**State = Verified, Unprotected**: Accepts and sets a new password via SetObjPwd.

•	Handles error reporting via alerts and resets password UI if needed.

•	Adjusts GUI panel activation and redraws based on current password state.

**🗨 Comments:**

•	This procedure contains deeply detailed and **authentic 1983 developer guidance**, especially:

```
{called when the 'Enter' button is pushed in the password panel of the dialog box.
 Sets the global password string to 'Delts' (for testing only).  Should eventually
 set the password value, check for validity, call SetObjPwd, etc.}
```

•	It suggests **unfinished work** and use of **hardcoded test values**, reflecting iterative GUI development.

---

---

**🔹 Procedure: AttNoPassBtn**

**Lines 121–149:**

```
PROCEDURE AttNoPassBtn;

{procedure to be executed when the user is changing the password on a protected file and
 clicks on the 'No Password' button}

VAR
   err      : INTEGER;
   numStr   : NumberStr;
   tempStr  : FmaxStr;

BEGIN
{$IFC flrDebug } IF trcCalls THEN ALogCall; {$ENDC};

ShowDiBtn(diPassword.noPassBtnPanel, TRUE{selected});

{set up name to use for the alert, we know that diBox.obj exists}
GetStrg (diPassword.iconName, tempStr);
QuoteName (tempStr);
ArgAlert (1, tempStr);

{we know that the password state is (verified), otherwise button would not have
 been visible, so take it from there}
SetObjPwd (err, diBox.obj^^.volHdl, diBox.obj^^.catRID, dipassword.old, '');
IF (err > 0) THEN
   IF (NOT VerifyPassword(diBox.obj^^.volHdl, diBox.obj^^.catRID)) THEN
      FixPassword(diBox.obj^^.volHdl, diBox.obj^^.catRID)
   ELSE
      BEGIN
      IntToStr (err, numStr);
      ArgAlert (2, '1221/');
      ArgAlert (3, numStr);
      StopAlert (flrAlert, 333);
      END
ELSE
   BEGIN
   diBox.obj^^.password := FALSE;
   diPassword.state := UnProtected;
   END;

ProtAttDiBox;                 { make sure that everything is displayed correctly }

ResetPwdFields;

ShowDiBtn(diPassword.noPassBtnPanel, FALSE{not selected});

END; {of PROC AttNoPassBtn}
```

**🧠 Function Breakdown:**

•	**Name:** AttNoPassBtn

•	**Role:** Removes the password from a protected object.

•	**Behavior:**

•	Assumes Verified state.

•	Calls SetObjPwd with an empty string to clear the password.

•	If error occurs, attempts fix and shows alert.

•	Updates UI and password state to UnProtected.

---

**🔹 Function: AttError**

**Lines 152–181:**

```
FUNCTION AttError {* myError, error: INTEGER) : BOOLEAN *};

{standard OS error check for attributes code}

VAR
   numStr   : NumberStr;
   tempStr  : FmaxStr;

BEGIN
{$IFC flrDebug } IF trcCalls THEN ALogCall; {$ENDC}
IF trcAttribute THEN
   IF (error <> 0) THEN WRITELN ('[AttError], entry: error = ',error:1);

AttError := TRUE;          {make it a major error to start with}
IF (error <= 0) THEN
   AttError := FALSE
ELSE
   BEGIN
   IF (myError = AttCatError) THEN
      DisplayError (myError, error, diBox.obj^^.volHdl^^.devHdl)
   ELSE
      BEGIN                                  {display the error in a StopAlert}
      GetStrg (diBox.obj^^.nameHdl, tempStr);
      QuoteName (tempStr);
      ArgAlert (1, tempStr);
      IntToStr (myError, numStr);
      tempStr := CONCAT(numStr,'/');
      ArgAlert (2, tempStr);
      IntToStr (error, numStr);
      ArgAlert (3, numStr);
      StopAlert (flrAlert, 326);
      END;
   END;

END; {of FUNCTION AttError}
```

**🧠 Function Breakdown:**

•	**Name:** AttError

•	**Role:** Standardized error handling for attribute-related routines.

•	**Behavior:**

•	Distinguishes between catalog errors (AttCatError) and others.

•	Uses DisplayError or fallback alert dialog (StopAlert) with 3 alert arguments.

•	Diagnostic logging via WRITELN.

---

**🔹 Function: AttObjInfo**

**Lines 184–743:**

•	This massive function retrieves and constructs all dialog metadata (diInfo) from an ObjectHandle.

•	Inline nested procedures (each with full comments) manage:

•	Tool path construction

•	Directory chain building

•	Disk/document structure checks

•	Volume metadata

•	Protection state detection

**Start of Header:**

```
FUNCTION AttObjInfo {* obj: ObjectHandle; VAR diInfo: dialogInfo) : BOOLEAN *};

{loads diInfo with as much relevant information as possible, available from obj & the structures it
 references.  Returns TRUE only if no errors were encountered (ie. OS calls, etc).}
```

**Subprocedures Preserved in Order:**

•	MakeToolName

•	SetInChain

•	SetSplit

•	SetStoredOn

**Example Developer Comment (from SetSplit):**

```
{is the object split? Could be doc/docpad/tool/calculator/clock.  Remember, must have set up
 diInfo.kindLbl1/2 first, otherwise bombs out...}
```

**Object classification logic:**

```
CASE kind OF
   profileKind, diskKind, disk1Kind, disk2Kind, drawerKind, priamKind: ...
   folderKind, folderPad: ...
   ...
   OTHERWISE
      BEGIN
      topLines := 1;
      infoLines := 1;
      tempStr := 'is an unknown icon object.';
      kindLbl1 := NewStrg (tempStr);
      END; {of CASE ITEM OTHERWISE}
```

**Notable Details:**

•	**Protectable object logic** carefully examines disk version, file presence, and if the object was “set aside” (a historical Lisa concept).

•	**Split object support**: documents and tools could be split across volumes.

•	**String management:** QuoteName, ClipDiName, GetString, and NewStrg abstract complex UI string formatting.

•	**DiInfo fields:** name, size, created, modified, kindLbl1, kindLbl2, storedOn, storeIn1, storeIn2, etc.

**🧠 Function Breakdown:**

•	**Name:** AttObjInfo

•	**Purpose:** Extracts and structures all displayable metadata from an object handle for use in the attribute dialog.

•	**Return:** TRUE on success, FALSE on error.

•	**Role:** This is the *core data aggregator* for all attribute panels.

---

**🔹 Procedure: BreakInTwo**

**Lines 746–772:**

```
PROCEDURE BreakInTwo {* source: FmaxStr; maxSize: INTEGER; VAR firstStr, secondStr: FmaxStr *};

{takes the source string and splits into two parts (on a space character), with the first part
 shorter in length than maxSize}
```

**🧠 Function Breakdown:**

•	Splits a long string into two at the last safe space under a maximum length.

•	Used to wrap text in the GUI.

---

**🔹 Procedure: GetADocName**

**Lines 775–798:**

```
{If ok, pathname is set to the OS pathname for some part of the object,
 eg. -slot3chan1-(D114T6)$R might be returned  if the $R extension file was the
 first one found on slot3chan1}
```

**🧠 Function Breakdown:**

•	Builds an operating system path for a document object based on tool ID and slot.

•	Emphasizes specific device pathing like -slot3chan1-...$R.

---

**🔹 Procedure: GetAFldrName**

**Lines 801–829:**

```
{If ok, pathname is set to the OS pathname for the Folder, eg. -slot3chan1-(F118) might be returned}
```

**🧠 Function Breakdown:**

•	Creates the OS pathname for a folder based on ID.

•	Uses Reset_Catalog and Get_Next_Entry for directory traversal.

---

**🔹 Function: GetPassField**

**Lines 832–864:**

```
{clears the password field, resets the diPassword strings}
```

**🧠 Function Breakdown:**

•	Reads the password field from the dialog UI.

•	Validates ASCII character constraints (!' through ~).

•	Displays an alert for invalid input.

---

**🔹 Function: GetPassword**

**Lines 868–916:**

```
{ Displays the password dialog box, queries the user, tests the entered password for validity.
  If the password is good, sets password and returns TRUE; if bad or the user decides to 'Cancel'
  the password dialog, the password is set to nil and the function returns FALSE.  The password
  is tested with the OS Valid_Password call.}
```

**🧠 Function Breakdown:**

•	Displays password prompt, sets up UI (SetPassDiBox), and handles events.

•	Performs OS-level password verification.

•	Returns success/failure and populates passwd.

---

**🔹 Procedure: InitPwdFields**

**Lines 920–932:**

```
{resets the diPassword record fields to a null state}
```

**🧠 Function Breakdown:**

•	Clears and resets all dialog and password UI handles and values.

---

**🔹 Procedure: PassCancelBtn**

**Lines 935–942**

```
{called if user 'pushed' the 'Cancel' button in the password dialog box.  Should set password flag
 to nil (for testing in main GetPassword routine)}
```

---

**🔹 Procedure: PassEnterBtn**

**Lines 945–975**

```
{password is entered (either through the user typing <Return>, <Enter>
 or pushing the 'Enter' button).}
```

---

**🔹 Procedure: ProtAttDiBox**

**Lines 978–1072**

```
{makes sure that the correct panels in the protection box are visible, based on diPassword.state}
```

**🧠 Function Breakdown:**

•	UI state manager for password dialog visibility.

•	Transitions between Safe, Verified, Unprotected, and NoProt GUI layouts.

---

**🔹 Procedure: ResetPwdFields**

**Lines 1075–1097**

---

**🔹 Procedure: SetAttDiBox**

**Lines 1100–1372**

**Nested Procedures:**

•	SetAttTop

•	SetAttInfo

•	SetAttProt

These generate the layout and UI logic for the full attribute dialog, invoking AttObjInfo and using all earlier utility functions.

---

**🔹 Procedure: SetPassDiBox**

**Lines 1375–1453**

---

**🔹 Procedure: ShowAttributes**

**Lines 1456–1476**

```
{ Displays the attributes dialog box for obj)}
```

Finalizes the dialog lifecycle:

•	Initialize

•	Layout with SetAttDiBox

•	Draw

•	Event loop (UseDiBox)

•	Cleanup

---