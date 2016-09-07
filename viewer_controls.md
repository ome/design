# Interaction

Scope:
Updating and synchronization of rendering controls.

GUI Location:
Main window - Right Hand Pane - Preview Tab
Full Image Viewer - Rendering Settings Panel


### Rendering controls

### General controls

**Format:**
Control
- State
  - Condition

**Save**
- Disabled
 - default 
 - user permissions exclude action
- Enabled
 - user permissions allow
 - when the settings have been changed
- At Event
 - check against settings prior to action
- Action
 - saves settings for current image and current user

**Save to All**
- Disabled 
 - user permissions exclude action
- Enabled
 - default - if user permissions allow
- At Event
 - check against settings prior to action
- Action
 - saves settings for current image and current user to all images in dataset

**Undo**
- Disabled
 - default
- Enabled
 - when the settings have been changed
- At Event
 - check against settings prior to action
- Action
 - undo the last rendering change
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button

**Redo**
As above except:
- Action
 - redo the last rendering change

**Copy**
- Enabled
 - by default
- Action
 - copy the rendering settings

**Paste**
- Disabled
 - default
 - for the image the settings are copied from
- Enabled
 - when saved settings available to paste
- At Event
 - check against settings prior to action
- Action
 - undo the last rendering change
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button


### Rendering controls

Most of the controls are initialised using the rendering settings
retrieved from the server.

### Channel controls

The following change will trigger a "render" image event and trigger a modification of the
enabled flag of the save button. 

#### Controls per channel

For all:

- Default value
 - set using the retrieved settings

**Channel button**
- At Event
 - toggles channel on/off
- Action
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

**Slider**
- At Event - move left-right knobs
 - sets the input intensity interval
 - updates values in text fields on both side of the slider
- Action - when slider released
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

 **Text field**
- Action - when Return hit or focus lost
 - update slider's value
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

**Color picker**
- At Event
 - open color-picker dialog

#### Color picker dialog

**Toolbar button - colour wheel**
- Action
 - changes selector pane to colour wheel selector

 **Toolbar button - rgb**
- Action
 - changes selector pane to rgb selector

 **Toolbar button - default colour picker button**
- Action
 - changes selector pane to default colour picker selector

**Color/lut selection**
- Default value
 - set using the retrieved settings
- At Event
 - selects color/lut
- Action
 - enable Preview, Accept, Revert buttons

**Alpha slider**
- Default value
 - set using the retrieved settings
- At Event - move knob
  - sets alpha value
  - updates value in text fields on right of the slider
- Action
 - enable Preview, Accept, Revert buttons

**Preview button**
- Disabled
 - default
- Enabled
 - when the settings have been changed
- Action
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

**Accept button**
- Disabled
 - default
- Enabled
 - when the settings have been changed
- Action
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button
 - background of the button and the slider updated
 - close color picker dialog

**Cancel button**
- Enabled
 - all the time
- Action
 - revert to rendering settings at time of opening of color picker 
 - update color picker display
 - trigger rendering settings update - ? only if Preview had been used

#### Controls for all channels

**Greyscale checkbox**
- Unchecked
 - default
- Event - check
 - toggles all on channels off
- set rendering to grey scale
- Event - uncheck
 - toggles all channels that were on back on
 - set rendering to previous
- Action
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

For all:

- Enabled
 - all the time

**Min/Max button**
- Action
 - Set the start/end values of the intensity interval to the global min/max
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button
- Note
 - global min/max are calculated at import if ``calculate stats`` is on

**Full Range button**
- Action
 - Set the start/end values of the intensity interval to the min/max values of the pixels interval e.g. 0 to 65535 for 16 bit pixels type
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

**Imported button**
- Action
 - will request new rendering settings - this means that z/t, color, pixels intensity values will be changed, the mapping family used for each channel - all settings need to be re-initialized.
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

### Users settings

**User setting thumbnail and text**
- thumbnail displayed is generated using the last settings saved by the users who viewed the image
- current implementation
 - one thumbnail per user
- text displayed under the thumbnail
 - from string composed of the first name and last name of the user who viewed the image and saved settings
- thumbnail selected
 - all settings re-initialized
 - trigger a "render" image event
 - trigger a modification of the enabled flag of the Save button
 - trigger a modification of the enabled flag of the Undo button

### Useful links to see interactions:

 * [metadata_preview.html](<https://github.com/openmicroscopy/openmicroscopy/blob/develop/components/tools/OmeroWeb/omeroweb/webclient/templates/webclient/annotations/metadata_preview.html>)
 * [ome.viewport.js](<https://github.com/openmicroscopy/openmicroscopy/blob/develop/components/tools/OmeroWeb/omeroweb/webgateway/static/webgateway/js/ome.viewport.js>)
 * [omero_image.js](<https://github.com/openmicroscopy/openmicroscopy/blob/develop/components/tools/OmeroWeb/omeroweb/webgateway/static/webgateway/js/omero_image.js>)


## Histogram

Describe how the UI components are updated when the selection changes.

**Show Histogram checkbox**
- Unchecked
 - default
- Event - check
 - display the histogram just above the channels 
- Event - uncheck
 - hides the histogram
- Action
 - histogram displayed for the first active channel
 - if the first channel is inactive the histogram corresponding to the next active channel is displayed
 - if there is no active channel last channel is displayed

**Update of intensity values - slider knobs moved or values in the start/end text fields updated**
- vertical bars indicating the input intervals are updated
- if histogram displayed not the one corresponding to the channel slider
 - histogram updated first
- when field selected
 - histogram updated
- when color/lut and histogram update
 - if the histogram corresponding to the channel is displayed the color of the histogram is updated
