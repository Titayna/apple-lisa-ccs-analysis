# APDM-DESK3.TEXT.unix.pas - Technical Analysis

**Analysis Date**: January 2025  
**LLM Model**: ChatGPT-4o  
**Prompt Used**: Appendix A - Technical Software Analysis  
**Source**: Computer History Museum Apple Lisa Code Archive

## LLM Analysis Output

**📘 Full Technical Analysis: apdm/desk3.text**

**System:** Apple Lisa

**Component:** Dialog Box Runtime

**Language:** Pascal

**Developer:** Ken Krugler

**Date:** December 1983 – March 1984

**Primary File Role:** Runtime drawing and control logic for modal dialog boxes in Lisa GUI.

---

**🔹 HEADER COMMENTS**

```
{apdm/desk3.text 5-Dec-83} { Copyright 1983, Apple Computer Co.}

{This file contains the procedures used to set up, display and monitor a dialog box.
 It is called exclusively by the routines in apdm/desk4.
                                                 - Ken Krugler}

{The data structures used by these routines are declared at the end of the standard type
 declarations in apdm/desk.}
```

**→ Analysis**

These comments establish the module’s **exclusive integration** with desk4. This isolates UI setup from high-level control logic, promoting maintainability.

---

**🔹 SYSTEM OVERVIEW COMMENT BLOCK**

```
{The following routines all draw the appropriate object from the info in PanelRec.
 The correct way to use these routines is to set the QD bottleneck procs to nil,
 start up picture recording, draw every panel (with DrawPanel, which will call one
 of these routines), close the picture and call UseDiBox.  UseDiBox will then handle
 events like keydown, mouseDown, etc., changing the panel states accordingly.  Any
 panel which has its state (visible/invisible) changed has its changed flag set to true.
 UseDiBox will then call ShowDiBox, which first sets the QD procs to be its own special
 procs, then replays the dialog picture.  If the panel being drawn by the replaced QD
 proc has changed, it is drawn and its changed flag is reset, otherwise the next panel
 is processed.  We know which panel corresponds to what QD proc call as the panels were
 all originally drawn in a linear order, which will match the ordering of the QD calls}
```

**→ Analysis**

Outlines the **dialog lifecycle**:

•	Panels are drawn and recorded as a QuickDraw picture.

•	During replay, only changed panels are redrawn.

•	Use of custom bottleneck procs allows **dialog-specific overrides**.

---

**🔹 DRAW FUNCTION LIST**

```
{DrawDiLabel, DrawDiButton, DrawDiField, DrawDiLine, DrawDiRect}
```

> These functions render individual dialog components from PanelRec.
> 

---

**🔹 MODIFICATION LOG**

```
{******** Modification Log ********)
{First skeleton written : 12/2/83}
{Filled out ShowAttributes to work with newly defined data structures : 12/3/83}
{Added diLine, diRect, made it actually put up a box (finally) : 12/4/83}
{Broke out routines that do dialog box specific stuff, into apdm/desk4 : 12/5/83}
{Does real picture recording : 12/5/83}
{General extension, first object specific (disk/folder/doc) panels : 12/10/83}
{Fixed problem with heap crash in QD pictures : 12/13/83}
{Modified QD picComment code to reference only a handle : 12/27/83}
{Added field editor code : 1/10-16/84}
{Changed field framing on drawPicture : 3/4/84}
{SelDiField change for use in desk4, DoDiUpdate fixes half-button problem : 3/14/84}
```

> Critical historical record; shows
> 
> 
> **progressive refinement**
> 

---

**🔸 PROCEDURE AND FUNCTION ANALYSIS**

---

**FUNCTION AddDiPanel**

```
{Create a new panel, link with the panel list and return it}
```

**Lines:** 42–58

**Behavior:**

•	Allocates a new panel via NewDiPanel

•	Links it into the circular doubly-linked list used for dialog panel storage

---

**PROCEDURE ClearDiField**

```
{resets an edit field, erases inside the box}
```

**Behavior:**

•	Resets visual content of an input field using ClearField

•	Clears, clips, and redraws the rectangular boundary

---

**PROCEDURE ClipDiName**

```
{clips the name to length maxSize.  If greater, chops name to maxSize-3 and adds three '.'}
```

**Behavior:**

•	Ensures long file names fit inside constrained UI boxes

---

**PROCEDURE CloseDiBox**

```
{make the linked list of panels a circular one, draw all of the panels to set up the picture,
 then show the dialog window on the screen for the first time and reset the menu bar.}
```

**Behavior:**

•	Finalizes all panels (circular links)

•	Begins QuickDraw picture recording (StartDiPic)

•	Calls DrawPanel on each panel

•	Shows dialog using DialogHeight, unhighlights menus with HiLiteMenu(0)

---

**PROCEDURE DiCommandKey**

```
{needs to handle the Apple-x (cut, paste, etc) for scrap...Actually should put up an alert for
 'Cant do cut/paste/copy with passwords...', then ignore it}
```

**Behavior:**

•	Prevents inappropriate command key usage in password fields

•	Triggers StopAlert with specific error codes

---

**PROCEDURE DiFieldEdit**

```
{ called when a button down occurs in an editable field in the dialog box }
{ assumes that the grafport is set correctly }
```

**Behavior:**

•	Handles click-to-edit on text fields

•	Supports word selection, extension via mouse drag

•	Initiates caret blinking and redrawing if text changes

---

**PROCEDURE DispHandle**

```
{First checks to see if the handle is Nil, otherwise disposes of it (frees up the heap)}
```

**Behavior:**

•	A classic Pascal memory management guard: frees only if Handle is non-nil

---

**PROCEDURE DiTextKey**

```
{handles the key processing for the dialog box. If typing performance becomes a problem, create a
 global field edit structrue variable for the password panel, thus eliminating the need for the
 tempInfo record and accompanying assignments}
```

**Behavior:**

•	Keyboard input dispatcher

•	Handles special characters (enter, escape, backspace, arrows)

•	Redraws fields and maintains cursor state

•	Updates fStateHdl and re-binds it to the panel

---

**PROCEDURE DiCommentProc**

```
{Note: In this case dataHandle references yet another handle, this time the real one to the panel
 record}
```

**Behavior:**

•	Core logic to **draw or skip** panel rendering depending on mode:

•	DiNormal, DiUpdate, DiErase

•	Sets changed := FALSE based on visibility

•	Wraps each panel with PicLParen/PicRParen markers

---

**PROCEDURES: DiLineProc, DiRectProc, DiRRectProc, DiTextProc**

```
{of PROC DiLineProc}
{of PROC DiRectProc}
{of PROC DiRRectProc}
{of PROC DiTextProc}
```

**Behavior:**

•	QuickDraw passthroughs for standard primitives during replay

•	Controlled via doPanel flag

---

**PROCEDURE DoDiBtnDown**

```
{we were editing a field before button-down, so do some consistancy checking}
```

**Behavior:**

•	Mouse-down handler

•	Handles case of active field vs. button click

•	Updates current selection and inverts button/field visuals

---

**PROCEDURE DoDiBtnUp**

```
{called from DoBtnUpEvent (in Apdm/Desk), executes button code if a button is currently selected}
```

**Behavior:**

•	Invokes button procedure if a diButton was selected

•	Submits password if needed

---

**PROCEDURE DoDiFinished**

```
{terminates the dialog box, calls the terminating procedure}
{reQueue = TRUE => shouldn't have done GetEvent, so re-queue for the filer}
```

**Behavior:**

•	Ends the dialog loop

•	Optionally requeues event to the file manager

---

**PROCEDURE DoDiUpdate**

```
{handles a folder update sent to the dialog folder}
```

**Behavior:**

•	Handles UpdateEvent window message

•	Triggers redraw with ShowDiBox, resets password buttons

---

**PROCEDURES DrawDi<Button,Label,Line,Rect>**

```
{ Draw a dialog box label into the dialog box picture }
```

**Behavior:**

•	Renders specific UI elements based on PanelRec

•	Uses text metrics from FontInfo

•	Applies padding, frames, and vertical alignment

---

**PROCEDURE DrawPanel**

```
{stuffs the picture being formed with a left paren, the QD commands to draw the panel, and a right
 paren.  Point of order, the QDHandle of the left paren points to a copy of what handleHdl points
 to.  This value is the actual handle to the designated panel -- all this because PicComment mades
 a copy of the data structure that you pass it a handle to.}
```

**Behavior:**

•	Encloses panel drawing in PicComment calls

•	Allows replay of individual panels during ShowDiBox

---

**PROCEDURE InitDiBox**

```
{Set up the variables used for building a dialog box.
 (Note: First panel reference number is set to 1)}
```

**Behavior:**

•	Initializes the diBox master record

•	Creates the first panel

•	Sets outline, height, and picture handles

---

**FUNCTION NewDiPanel**

```
{Returns a handle to a newly allocated panel record (with the record fields set to defaults).
 Also bumps the reference number by one.}
```

**Behavior:**

•	Allocates and zero-initializes a PanelHandle

•	Prepares bounding region and marks changed := TRUE

---

**PROCEDURE RmvDiBox**

```
{Removes the dialog box from the desktop, also deallocate the panels from the heap}
```

**Behavior:**

•	Disposes of panel memory (field text, buttons, labels)

•	Deallocates all QD regions

•	Ends picture and removes dialog visuals

---

**PROCEDURE SelDiField**

```
{sets field as selected, it required shows a selected field w/blinking cursor, etc.}
```

**Behavior:**

•	Initializes caret position and selection

•	Starts field edit state

---

**PROCEDURES: SetDi<Button,Field,Label,Line,Rect>**

```
{stuffs panel with the appropriate info}
```

**Behavior:**

•	These procedures assign configuration to panel structures:

•	Fonts

•	Styles

•	Rectangles

•	Text handles

•	Visibility

•	Creates bounding regions for hit testing

---

**PROCEDURE SetNullProcs**

```
{Sets all of the quickdraw bottleneck procedures that draw dialog box
 panels to be Nil (dialogFolder is thePort)}
```

**Behavior:**

•	Assigns all QDProcs to nil stubs to prevent drawing during testing

---

**PROCEDURE ShowDiBox**

```
{Display the dialog box.  Use the linked panel list from diBox}
```

**Behavior:**

•	Assigns custom QDProcs

•	Replays the stored QuickDraw picture to draw the full dialog state

---

**PROCEDURE StartDiPic, StopDiPic**

```
{ start recording all QD calls }
{ stop recording QD calls }
```

**Behavior:**

•	Start/stop QuickDraw picture recording of the dialog panels

---

**PROCEDURE TestDiBox, TestDiPanel**

```
{draws picture, only non-nil bottleneck routine is TestDiPanel.  TestDiPanel sets hitDiPanel non-nil
 of button up occured in a valid selectable panel (button/field).  TestDiPanel also inverts the
 current panel (PtInRgn check)}
```

**Behavior:**

•	Performs hit-testing

•	Highlights selected panels

•	Used to detect button clicks during dialog interaction

---

**PROCEDURE UseDiBox**

```
{main processing loop, co-exists uncomfortably with the main filer loop.  Basically calls ProcessTheEvent,
 which then calls the appropriate sub-event (ie. DoBtnDownEvent).  At the lower level routine is where
 the branch to either main filer code or dialog box code occurs}
```

**Behavior:**

•	Main loop of the dialog system

•	Intercepts events and routes them to dialog handlers

•	Allows idle time for caret blinking, etc.

---

**✅ CONCLUSION**

This file delivers a sophisticated event-driven dialog manager built atop **QuickDraw picture recording**, QDProcs interception, and low-level memory management via handles.

The **developer comments are an extraordinary primary source**, detailing:

•	System design strategy

•	Historical constraints (heap crashes, field bugs)

•	Evolution over time (12/83–3/84)