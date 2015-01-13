Nano Shortcuts Reference
========================

Movements
---------


+--------+----------+------------------------------+
|M-\     |  M-x     |Go to the first line of the   |
|        |          |file                          |
|        |          |                              |
+--------+----------+------------------------------+
|M-/     |   M-?    |Go to the last line of the    |
|        |          |file                          |
+--------+----------+------------------------------+
|^_      |  F13 M-G |Go to line and column number  |
|        |          |                              |
+--------+----------+------------------------------+
|^F      |          |Go forward one character      |
|        |          |                              |
+--------+----------+------------------------------+
|^B      |          |Go back one character         |
|        |          |                              |
+--------+----------+------------------------------+
|^Space  |          |Go forward one word           |
|        |          |                              |
+--------+----------+------------------------------+
|M-Space |          |Go back one word              |
|        |          |                              |
+--------+----------+------------------------------+
|^P      |          |Go to previous line           |
|        |          |                              |
+--------+----------+------------------------------+
|^N      |          |Go to next line               |
|        |          |                              |
+--------+----------+------------------------------+
|^A      |          |Go to beginning of current    |
|        |          |line                          |
|        |          |                              |
+--------+----------+------------------------------+
|^E      |          |Go to end of current line     |
|        |          |                              |
|        |          |                              |
+--------+----------+------------------------------+
|M-(     |   M-9    |Go to beginning of paragraph; |
|        |          |then of previous paragraph    |
|        |          |                              |
+--------+----------+------------------------------+
|M-)     | M-0      |Go just beyond end of         |
|        |          |paragraph; then of next       |
|        |          |paragraph                     |
|        |          |                              |
+--------+----------+------------------------------+
|M-]     |          |Go to the matching bracket    |
|        |          |                              |
+--------+----------+------------------------------+
|^Y      |  F7      |Go to previous screen         |
|        |          |                              |
+--------+----------+------------------------------+
|^V      | F8       |Go to next screen             |
|        |          |                              |
+--------+----------+------------------------------+
|M--     |  M-_     |Scroll up one line without    |
|        |          |scrolling the cursor          |
|        |          |                              |
+--------+----------+------------------------------+
|M-+     |  M-=     |Scroll down one line without  |
|        |          |scrolling the cursor          |
|        |          |                              |
+--------+----------+------------------------------+
|^C      |  F11     |Display the position of the   |
|        |          |cursor                        |
+--------+----------+------------------------------+
|M-D     |          |Count the number of words,    |
|        |          |lines, and characters         |
|        |          |                              |
+--------+----------+------------------------------+
|        |          |                              |
|^G      |  F1      |   Display help screen        |
|        |          |                              |
+--------+----------+------------------------------+
|^L      |          |Refresh (redraw) the current  |
|        |          |screen                        |
|        |          |                              |
+--------+----------+------------------------------+


Insertion/deletion
------------------

+--------+------+------------------------------+
|M-V     |      |Insert the next keystroke     |
|        |      |verbatim                      |
+--------+------+------------------------------+
|^I      |      |Insert a tab at the cursor    |
|        |      |position                      |
+--------+------+------------------------------+
|^M      |      |Insert a newline at the cursor|
|        |      |position                      |
+--------+------+------------------------------+
|^D      |      |Delete the character under the|
|        |      |cursor                        |
+--------+------+------------------------------+
|^H      |      |Delete the character to the   |
|        |      |left of the cursor            |
+--------+------+------------------------------+
|M-T     |      |Cut from the cursor position  |
|        |      |to the end of the file        |
+--------+------+------------------------------+
|^K      |  F9  |Cut the current line and store|
|        |      |it in the cutbuffer           |
+--------+------+------------------------------+
|^U      |  F10 |Uncut from the cutbuffer into |
|        |      |the current line              |
+--------+------+------------------------------+
|M-^     | M-6  |Copy the current line and     |
|        |      |store it in the cutbuffer     |
+--------+------+------------------------------+
|^J      | F4   |Justify the current paragraph |
+--------+------+------------------------------+
|M-J     |      |Justify the entire file       |
+--------+------+------------------------------+
|M-}     |      |Indent the current line       |
+--------+------+------------------------------+
|M-{     |      |Unindent the current line     |
+--------+------+------------------------------+


Files/Buffers
-------------

+--------+------+------------------------------+
|^X      |  F2  |Close the current file buffer |
|        |      |/ Exit from nano              |
+--------+------+------------------------------+
|^O      |  F3  |Write the current file to disk|
+--------+------+------------------------------+
|^0 ^T   |      |choose a file to write the    |
|        |      |buffer with the file browser. |
+--------+------+------------------------------+
|^R      |  F5  |Insert another file into the  |
|        |      |current buffer                |
+--------+------+------------------------------+
| ^R ^T  |      | insert with file browser     |
|        |      |                              |
+--------+------+------------------------------+
|M-<     |  M-, |Switch to the previous file   |
|        |      |buffer                        |
+--------+------+------------------------------+
|M->     |  M-. |Switch to the next file buffer|
+--------+------+------------------------------+
|^Z      |      |Suspend the editor            |
+--------+------+------------------------------+

Search
------
+--------+--------+------------------------------+
|^W      |  F6    |Search for a string or a      |
|        |        |regular expression            |
+--------+--------+------------------------------+
|^T      |  F12   |Invoke the spell checker, if  |
|        |        |available                     |
+--------+--------+------------------------------+
|^\      | F14 M-R|Replace a string or a regular |
|        |        |expression                    |
+--------+--------+------------------------------+
|M-W     |   F16  |Repeat last search            |
+--------+--------+------------------------------+
|^^      | F15 M-A|Mark text at the cursor       |
|        |        |position                      |
+--------+--------+------------------------------+


Toggles
-------

+----------+------------------------------+
|M-X       |Help mode enable/disable      |
+----------+------------------------------+
|M-C       |Constant cursor position      |
|          |display enable/disable        |
+----------+------------------------------+
|M-O       |Use of one more line for      |
|          |editing enable/disable        |
+----------+------------------------------+
|M-C       |Constant cursor position      |
|          |display enable/disable        |
+----------+------------------------------+
|M-O       |Use of one more line for      |
|          |editing enable/disable        |
+----------+------------------------------+
|M-S       |Smooth scrolling              |
|          |enable/disable                |
+----------+------------------------------+
|M-P       |Whitespace display            |
|          |enable/disable                |
+----------+------------------------------+
|M-Y       |Color syntax highlighting     |
|          |enable/disable                |
+----------+------------------------------+
|M-H       |Smart home key enable/disable |
+----------+------------------------------+
|M-I       |Auto indent enable/disable    |
+----------+------------------------------+
|M-K       |Cut to end enable/disable     |
+----------+------------------------------+
|M-L       |Long line wrapping            |
|          |enable/disable                |
+----------+------------------------------+
|M-Q       |Conversion of typed tabs to   |
|          |spaces enable/disable         |
+----------+------------------------------+
|M-B       |Backup files enable/disable   |
+----------+------------------------------+
|M-F       |Multiple file buffers         |
|          |enable/disable                |
+----------+------------------------------+
|M-M       |Mouse support enable/disable  |
+----------+------------------------------+
|M-N       |No conversion from DOS/Mac    |
|          |format enable/disable         |
+----------+------------------------------+
|M-Z       |Suspension enable/disable     |
+----------+------------------------------+
|M-$       |Soft line wrapping            |
|          |enable/disable                |
+----------+------------------------------+


`Nano reference <http://www.nano-editor.org/dist/latest/nano.html>`_ is
also available as an *info* manual.
