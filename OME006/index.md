# Interaction

Scope:
ROI Drawing controls.

GUI Location:
Full Image Viewer - Rendering Settings Panel

### General controls

---

**Format:**

**Set-up:** ...

**Control**
- State
  - Condition
     - Clarification
- Implementation

---

**Save**
- Disabled
  - default
  - user permissions exclude action
- Enabled
  - when ROI added to image
  - when position or parameter of existing ROI changed
  - user permissions allow
     - this should be checked using canAnnotate on the image (@jburel  - confirm call)
- At Event
  - disable
- Action
  - saves ROIs for current image

**Delete**
- Disabled
  - no ROIs on image
  - user permissions exclude action

- At Event (Drop-down menu options)- need confirmatory dialog
  - Delete ROIs added by others
     - delete ROIs not on image owned by current user
         - need confirmatory dialog

**Show Comments**  - checkbox
- Unckecked
  - default
- Checked
  - comments on ROIs displayed

**Open as Palette**
Closes RH panel and opens ROI tab in palette
(probably not first step)

---

**Line colour**
- At Event (Drop-down menu options)
  - colour options  - Black, Blue, Green, Yellow, White, Magenta, More Colours... (opens color picker)
- Action
  - update colour of line
  - ? automatic save

**Line thickness**
- At Event (Drop-down menu options)
  - line thickness options 1-7, 10, 15, 20, 30 px
  - ? automatic save
- Action
  - update colour of line
  - ? automatic save

**Edit**
- At Event (Drop-down menu options)
  - Copy Shape
     - copy selected shape to clipboard
  - Paste Shape
     - paste copied shape to image
  - Delete Shape
     - delete all shapes from image regardless of owner
         - need confirmatory dialog
  - Select All Shapes
- Action
  - Copy Shape
     - none
  - Others
     - redraw ROIs on image
  - ? automatic save

**Clone**
- Unselected
  - default
- Selected
  - toggles on/off
- Toggle on
  - copy selected shape to clipboard
     - if no selected shape
         - dialog "Select shape for cloning"
              - OK - deselect button
- Action
  - click on image pastes copied shape to image
     - shape centred on location clicked
- Toggle off
  - clear clipboard

**Copy**
- Unselected
  - default
  - ROI de-selected
- Selected
  - when ROI selected
- Action
  - copy selected shape to clipboard

**Paste**
- Unselected
  - default
- Selected
  - when ROI copied
- Action
  - paste copied shape to image
     - same coordinates as copied shape
     - unless coordinates not in image (too small)
         - then paste top left corner


**Save to All**
- Disabled
  - default
  - user permissions exclude action
- Enabled
  - if ROIs present on image
  - user permissions allow
         - this should be checked using canAnnotate on the image (@jburel  - confirm call)
- At event
  - dialog box explaining action
     - OK  - proceed with action
     - Cancel  - no action
- Action
  - saves ROIs for current image to all images in dataset
     - of same size
     - with appropriate Z and T

---

**Selector Tool**
- Disabled
  - other tool selected
- Enabled
  - default
- At event
  - deselect any selected tool
- Action
  - ?

**Rectangle Tool**
- Disabled
  - default
- Enabled
  - when selected
- At event
  - deselect any selected tool
- Action
  - draws rectangular ROI when mouse clicked on image

**Elipse Tool**

Same as above except:
- Action
  - draws elipse ROI when mouse clicked on image

**Point Tool**

Same as above except:
- Action
  - draws point ROI when mouse clicked on image

**Line Tool**
Same as above except:
- Action
  - draws line ROI when mouse clicked on image


**Polygon Tool**

Same as above except:
- Action
  - draws polygon ROI when mouse clicked on image

**Text Tool**

Same as above except:
- Action
  - draws text ROI when mouse clicked on image

**Show Intensity Checkbox**
- Unchecked
  - default
- At event
  - select checkbox
- Action
  - enables display of intensity values for pixel cursor is on

---

**Add ROI Folder**
- Disabled
  - user permissions exclude action
- Enabled
  - default
  - user permissions allow
     - this should be checked using canAnnotate on the image (@jburel  - confirm call)
- At event
  - opens Displayed Folders Window
     - interactions for this window shown seperately
- Action
  - adds folders returned by Displayed Folders Window to display pane

**Filter Folders Text Box**
- At event
  - use text entered to filter ROI Folder names shown in ROI table
- Action
  - display only folders with names matching filter term in table

---

### Folder-ROI Table

**ROI Folder Expansion Arrow**
- Enabled
  - default
- Action
  - expand ROI Folder

**ROI Folder**
- Enabled
  - default
- At event
  - highlight

 **ROI Expansion Arrow**
- Enabled
  - default
- Action
  - expand ROI

**ROI**
- Enabled
  - default
- At event
  - highlight

 **Shape**
- Enabled
  - default
- At event
  - highlight
- Action
  - select shape on image

**Show Checkbox  - ROI Folder**
- Enabled
  - default
- At event
  - highlight ROI Folder
- Action  - deselect
  - disable Show Checkbox for all child object 
  - hide all child shapes on image
- Action  - select
  - display all child shapes on image

**Show Checkbox  - ROI**
- Enabled
  - default
  - on click if disabled
  - after Save event
- Disabled
  - on click if enabled
- At event
  - highlight ROI
- Action  - deselect
  - disable Show Checkbox for all child objects 
  - hide all child Shapes on image
- Action  - select
  - display all child Shapes on image

 **Show Checkbox  - Shape**
- Enabled
  - default
  - on click if disabled
  - after Save event
- Disabled
  - on click if enabled
- At event
  - highlight ROI
- Action  - deselect
  - disable Show Checkbox for all child Shapes 
  - hide all child Shapes on image
- Action  - select
  - display all child Shapes on image

---

### Displayed Folders Window

**Filter Available Textbox**

- At event
  - use text entered to filter ROI Folder names shown in ROI table
- Action
  - display only folders with names matching filter term in table

**Folder Selection Pane**

- At event
  - highlight ROI Folder/s
- Action
  - select ROI Folder/s
  - enable
     - Add Button (Arrow --> Right)
     - Add All Button (Double Arrow --> Right)
  - disable
     - Remove Button (Arrow --> Left)
     - Remove All Button (Double Arrow --> Left)

**Selected Pane**
- At event
  - highlight ROI Folder/s
- Action
  - select ROI Folder/s

**Add Button (Arrow --> Right)**
- Enabled
  - selection of ROI Folder/s in Folder Selection Pane
  - click of:
     - Remove Button (Arrow --> Left)
     - Remove All Button (Double Arrow --> Left)
- Disabled
  - selection of ROI Folder/s in Selected Pane
  - click of:
     - Add Button (Arrow --> Right)
     - Add All Button (Double Arrow --> Right)
- Action
  - move selected ROI Folder/s to Selected Pane

**Remove Button (Arrow --> Left)**
- Enabled
  - selection of ROI Folder/s in Selected Pane
  - click of:
     - Add Button (Arrow --> Right)
     - Add All Button (Double Arrow --> Right)
- Disabled
  - selection of ROI Folder/s in Folder Selection Pane
  - click of:
     - Remove Button (Arrow --> Left)
     - Remove All Button (Double Arrow --> Left)
- Action
  - move selected ROI Folder/s to Folder Selection Pane

**Add All Button (Double Arrow --> Right)**
- Enabled
  - default
- Disabled
  - no ROI Folder/s in Folder Selection Pane
- Action
  - move all ROI Folder/s in Folder Selection Pane to Selected Pane

**Remove All Button (Double Arrow --> Left)**
- Enabled
  - ROI Folder/s in Selected Pane
- Disabled
  - no ROI Folder/s in Selected Pane
- Action
  - move all ROI Folder/s in Selected Pane to Folder Selection Pane

**Filter By Dropdown**
- Start of name
- Anywhere in name

**Filter by Owner**
- All
- Owned by me
- Owned by others

**Create Folder Textbox**
- Enabled
  - default
- Event
  - entered text is name of new ROI Folder
- Action
  - text entry enables Add Button
  - all text deleted disables Add Button

**New Folder Description Textbox**
- Enabled
  - default
- Event
  - entered text is description of new ROI Folder

**Add Button**
- Disabled
  - default
- Enabled
  - text in Create Folder Textbox
- Event
  - entered text is name of new ROI Folder
- Action
  - text entry enables Add Button
  - all text deleted disables Add Button

**Save Button**
- Disabled
  - default
- Enabled
  - change to status of either pane
- Action
  - save added/removed folders to database
  - refresh ROI Folders-ROIs display pane with added/removed folders

**Cancel Button**
- Enabled
  - default
- Action
  - close window without saving any changes

**Reset Button**
- Disabled
  - default
- Enabled
  - change to status of either pane or text boxes
- Action
  - revert to state at opening of window

---

### Drawing Mode Tab

**Propagate Checkbox**
- Disabled
  - default
  - on click if enabled
- Enabled
  - on click if disabled
- Action
  - check
    - ROI drawn automatically propogated to Z and T as defined by Attach to Radio Buttons
  - uncheck
    - ROI drawn only on current Z and T

**Attach to Radio Buttons**

**Attach to All Z and T**
- Selected
  - default
  - on click
- Deselected
  - selection of other radio button
- Action
  - set status for ROI drawn to automatically propogated to all Z and all T

**Attach to all Z**
- Selected
  - on click
- Deselected
  - default
  - selection of other radio button
- Action
  - set status for ROI drawn to automatically propogated to all Z and current T

**Attach to all T**
- Selected
  - on click
- Deselected
  - default
  - selection of other radio button
- Action
  - set status for ROI drawn to automatically propogated to current Z and all T

**Attach to**
- Selected
  - on click
  - on click into Z Text box or Y Text box
- Deselected
  - default
  - selection of other radio button

**Z Text box**
- Event
  - click in
    - select Attach to radio button
- Action
  - set status for entered values determine which Z ROI drawn automatically propogated to

**T Text box**
- Event
  - click in
     - select Attach to radio button
- Action
  - set status for entered values determine which T ROI drawn automatically propogated to

**Do not attach to Z or T**
- Selected
  - on click
- Deselected
  - default
  - selection of other radio button
- Action
  - set status for ROI drawn to not be attached to any Z or T (equivalent of overlay)

**Attach to viewed Z and/or T Checkbox**
- Selected
  - on click
- Deselected
  - default
- Action
  - set status for ROI drawn to be attached to any Z or T subsequently viewed

**Note** - not sure if this should be a checkbox or a radio button as part of thegroup above

**Do Graphically**
? not implementing yet
- if do do not show

**Cancel**
- Disabled
  - default
  - Save clicked
- Enabled
  - ROI drawn
- Action
  - set status for ROI drawn to be attached to any Z or T subsequently viewed

**Save**
- Disabled
  - default
  - Save clicked
- Enabled
  - ROI drawn
- Action
  - set status for ROI drawn to be attached to any Z or T subsequently viewed


**Analysis Tab**

To follow
