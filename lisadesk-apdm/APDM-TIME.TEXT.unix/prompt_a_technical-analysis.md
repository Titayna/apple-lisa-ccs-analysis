# APDM-TIME.TEXT.unix.pas - Technical Analysis

**Analysis Date**: January 2025  
**LLM Model**: ChatGPT-4o  
**Prompt Used**: Appendix A - Technical Software Analysis  
**Source**: Computer History Museum Apple Lisa Code Archive

## LLM Analysis Output

Here’s a **complete technical software analysis** of the DmTime unit from the Apple Lisa (1983) source code. This module handles **date and time formatting and conversion** for the Lisa Filer interface. The code is written in Pascal, reflecting structured programming idioms and early GUI conventions.

---

**📦 UNIT DmTime**

**Purpose:**

This unit supports **date/time formatting, display string generation**, and **Julian date conversion**. It is explicitly designed to be called by the **Lisa Filer** (file manager), a key UI component in the Lisa environment.

**Date:** June 30, 1983

**Authors:** Apple Computer Inc. (1983, 1984)

**Language:** Pascal

**Location in Lisa System:** Application Date/Time Module for Filer (APDM)

---

**🔗 INTERFACE SECTION**

**USES Clause**

**Lines 4–23:**

Lists imported units required by this module. These include:

•	**System and hardware units**: HwInt, UnitStd, UnitHz, Storage, SysCall, PSysCall

•	**UI and Graphics**: QuickDraw, FontMgr, Events, Folders, Menus

•	**Window management**: WmlStd, WmlCrs, WmlGrow, WmlSb

•	**User interaction**: AlertMgr

•	**Print and dialog modules**: PMDecl, PMM, PrStdInfo, PrPublic

•	**Globals and communications**: DmGlobals, FilerComm

•	**Debugging**: tracecalls is conditionally included with flrDebug

---

**Type Definition**

**TTimeStr**

**Line 26:**

```
TTimeStr = STRING[20];
```

•	A **fixed-length string** used to store time/date display strings, such as “04/08/25 AM”.

---

**Procedure Declarations**

**InitTime**

**Line 29:**

```
PROCEDURE InitTime(timeAlert, dateAlert: TTimeStr);
```

•	Parses and analyzes user/localized **time/date format strings** (timeAlert, dateAlert).

•	Calculates string **offsets** for graphical/text rendering.

•	Stores parsed components into global state variables.

**TimeToStr**

**Line 32:**

```
PROCEDURE TimeToStr(currentTime:BOOLEAN; theTime:LONGINT; VAR dateStr,timeStr: TTimeStr);
```

•	Converts a LONGINT or current system time into **formatted date/time strings**.

•	Uses formatting rules set up by InitTime.

---

**🧠 IMPLEMENTATION SECTION**

---

**💡 Compiler Flags**

**Lines 37–45:**

Sets compiler/debugging flags:

•	$D+/-: Symbol table debug info

•	$R+/-: Runtime range-checking

•	FTSymbols, FTDebug: Compile-time constants for enabling diagnostics

---

**📌 CONSTANTS**

**Lines 47–59: Julian Calendar Month End Days**

```
endJa = 31;
endFe = 59;
...
endDe = 365;
```

These constants define the **cumulative Julian day totals** for the end of each month in a non-leap year.

**Line 61:**

```
dbgFTime = TRUE or FALSE
```

Toggle debugging output.

---

**📦 GLOBAL VARIABLES**

**Lines 64–93: Display Offsets**

Variables like hourOffset, minOffset, ampmOffset, etc., define **text cursor positions** for graphical placement of date/time fields.

**Lines 95–98: Current Time and Error State**

```
curTime: Time_Rec;
osErr: INTEGER;
err: INTEGER;
```

Hold current time and any errors from OS time calls.

**Lines 101–102: Calendar Support**

```
julianDays: ARRAY [0..12] OF INTEGER;
leapYear: BOOLEAN;
```

Used to convert Julian days to month/day values.

---

**🔄 PROCEDURE ANALYSIS**

---

**🔁 JulianToMoDay**

**Lines 107–126**

**Role:**

•	Converts a Julian day number into **month and day**, taking leap years into account.

**Key Logic:**

•	Checks for February 29 if leap year and day = 60

•	Scans julianDays[] to find correct month

•	Computes day-of-month

**Example:**

```
mo := i+1;
day := julian - julianDays[i];
```

---

**⚙️ InitTime**

**Lines 130–250**

**Role:**

•	Parses time/date format strings (e.g., /://am/pm/12, mdy0/)

•	Determines clock format (12-hour or 24-hour)

•	Sets display **offsets** for formatting output strings

**Internal Procedure:**

```
PROCEDURE GetPart(VAR str:TTimeStr; VAR part:TTimeStr);
```

Extracts substrings between / delimiters.

**Constants:**

```
defTimeStr = '/://am/pm/12';
defDateStr = 'mdy0/';
```

**Format Extraction:**

•	leadTStr, sepTStr, trailT24Str, amStr, pmStr

•	Format type: 12 or 24 (→ ampm24Switch)

•	Show-zero flag (showZero): Enables padded values like “04” instead of “4”

•	Sequence flag (sequ): Defines field order (MDY, DMY, YMD)

**Offset Calculation:**

Computes exact character positions for formatting timeStr and dateStr.

---

**🧾 TimeToStr**

**Lines 253–325**

**Role:**

•	Generates **formatted date/time output** using stored offsets and layout from InitTime

•	Reads current system time (via Get_Time) or uses theTime value

•	Converts time to local using Convert_Time

•	Uses JulianToMoDay to get month/day

•	Applies InsertStr to inject values into offset positions

**Inner Logic:**

•	Handles 12-hour vs. 24-hour formatting

•	Selects AM/PM string

•	Pads with zeroes if needed

•	Formats output into timeStr and dateStr

---

**🏗 STRUCTURE & DESIGN INSIGHT**

•	**Structured Pascal:** Modular, strongly-typed design, clear procedure boundaries.

•	**Separation of Concerns:** Time parsing, formatting, and calendar conversion are cleanly separated.

•	**Early Internationalization:** Supports multiple date formats (MDY, DMY, YMD), zero-padding, and AM/PM localization.

•	**Debugging Support:** Compile-time toggles for debug output (dbgFTime) and call tracing (tracecalls.obj).

•	**Graphical UI-Aware:** Offset values help align text in GUI elements of the Lisa Filer.

---

**🗃 Summary of Key Procedures**

| **Procedure** | **Role** |
| --- | --- |
| JulianToMoDay | Converts Julian day to month/day (leap-aware) |
| InitTime | Parses format strings, sets offsets |
| TimeToStr | Formats a time value into a GUI-ready string |

---

Would you like a follow-up breakdown of the supporting types like Time_Rec, DateArray, or library dependencies used in this unit?