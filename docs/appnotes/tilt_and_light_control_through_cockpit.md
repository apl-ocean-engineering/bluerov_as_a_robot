---
sidebar_position: 1
---

# Controlling camera tilt and lights through Cockpit

It's possible to control the camera tilt and lights through GUI elements in Cockpit, rather than with the joystick.

**Note:** As of recent versions of BlueOS, these instructions will break joystick control of the lights, but joystick control of camera tilt will still work.

The following is based on [this BR forum post](https://discuss.bluerobotics.com/t/cockpit-slider-to-control-light-brightness/20722/2) and were tested under BlueOS [1.5.0-beta.35](https://github.com/bluerobotics/BlueOS/releases/tag/1.5.0-beta.35) with Cockpit Lite [v1.18.0-beta.15](https://github.com/bluerobotics/cockpit/releases/tag/v1.18.0-beta.15) running in the browser.

These instructions will describe setting up *both* a lights control and a camera tilt control.  The instructions are very similar for the most part, but we will highlight some differences.

## Add controls to the UI

 First, edit the Cockpit interface using the "Edit interface" option.   For this test, I created a new view called "Camera Control" which let me customize the view without discarding the existing configuration.

In this view I deleted the widgets I didn't need, and added two slider widgets, one each for the lights and camera tilt.

[![](/img/appnotes/camera_and_lights_through_cockpit/edit_ui.png)](/img/appnotes/camera_and_lights_through_cockpit/edit_ui.png)

2. Select each slider, and "Create" a new data lake variable, giving it a name.  We used `lights1control` and `camtiltcontrol`.   For `lights1control` the default range of 0-100 is fine;  for `camtiltcontrol` we set the range to -45 to +45 so we could control angle directly.

[![](/img/appnotes/camera_and_lights_through_cockpit/configure_light_slider.png)](/img/appnotes/camera_and_lights_through_cockpit/configure_light_slider.png)  [![](/img/appnotes/camera_and_lights_through_cockpit/configure_tilt_slider.png)](/img/appnotes/camera_and_lights_through_cockpit/configure_tilt_slider.png)


## Create the light control compound variable

 Close the UI editor, and select "Tools -> Data lake" in the menu.   Each of the sliders is connected to a data lake variable, but we need to do math on those variables before we can send them to the ROV.

  1. For the light control, we will convert the percentage to a servo PWM value from 1100-1900.   BlueOS controls lumen lights like servos, so we will send a raw PWM value to control the lights.
  2. Search for `lights1control`.  The variable from the slider should appear.

[![](/img/appnotes/camera_and_lights_through_cockpit/light_datalake_before.png)](/img/appnotes/camera_and_lights_through_cockpit/light_datalake_before.png)

  1. Press the "copy" button to the left of its name to copy the full name to the clipboard.
  2. Select "New Compound Variable"
  3. Give the variable a name like `lights1pwm` and set its expression as follows.  The name in the double braces `{{ }}` should be the variable name copied in step 3.    This expression does two things -- it converts the percentage 0-100 to a PWM value from 1100-1900.   It also produces an integer (whole number) value, rather than a floating point value.

```
return 1100 + round( 800 * {{user/inputs/light1control}}/100 )
```

[![](/img/appnotes/camera_and_lights_through_cockpit/light_datalake_variable.png)](/img/appnotes/camera_and_lights_through_cockpit/light_datalake_variable.png)

   6. Save the compound value and return to the data lake screen.   Both the `lights1control` and `lights1pwm` values should be shown.   This screen is "live" so if you move the lights slider, the two values should chance together.

[![](/img/appnotes/camera_and_lights_through_cockpit/light_datalake_after.png)](/img/appnotes/camera_and_lights_through_cockpit/light_datalake_after.png)

## Create the camera tilt compound variable

Repeat the same for the camera tilt value.  This time, name the variable something like `camtiltcontrol_int`.   The expression is much simpler, we just need to convert the floating point angle to an integer.   As above, ensure the name in the braces matches the variable associated with the slider.

```
return round( {{user/inputs/camtiltcontrol}} )
```

[![](/img/appnotes/camera_and_lights_through_cockpit/cam_datalake_variable.png)](/img/appnotes/camera_and_lights_through_cockpit/cam_datalake_variable.png)

**(Note I didn't follow my own advice and named the variable `camtiltcontroller` ... oops)**

## Set up the light action

First, we'll set up the lights.   This is the step which disconnects joystick control of the lights.   These instructions assume the lights are connected to Navigator channel 13 (the default per the BlueROV manual) but will work for any other channel.

[![](/img/appnotes/camera_and_lights_through_cockpit/blueos_servo_parameters.png)](/img/appnotes/camera_and_lights_through_cockpit/blueos_servo_parameters.png)

    1. Go to the BlueOS web page at `http://192.168.2.2/` and select `Autopilot Parameters`.   Search for `SERVO13` and find the entry for `SERVO13_FUNCTION`.   Set it to "RCPassThru", then "Save and Reboot".   At this point BlueOS no longer knows that channel is a "light".
    1. Return to Cockpit and select "Tools -> Datalake".   Search for `lights1pwm` and copy its full name into the clipboard.
    2. Navigate to "Settings -> Actions"
    3. Create a new action, select "Mavlink message" and configure it as shown below.
       1. The "Message Type" should be `COMMAND_LONG`
       2. The "Command ID" should be `MAV_CMD_DO_SET_SERVO`
       3. "Param 1" should be `13` (or the channel for your lights)
       4. "Param 2" should be `{{user/compoung/lights1pwm}}` (or the full name of your data lake variable)

[![](/img/appnotes/camera_and_lights_through_cockpit/create_light_action.png)](/img/appnotes/camera_and_lights_through_cockpit/create_light_action.png)


   5.  Close the new action dialog, then select the "chain" icon
   6.  Configure the automatic trigger for the `lights1pwm` value.

[![](/img/appnotes/camera_and_lights_through_cockpit/light_action_trigger.png)](/img/appnotes/camera_and_lights_through_cockpit/light_action_trigger.png)

   7. The slider should now control the lights.

## Set up the camera tilt action

Finally, set up an action for camera tilt.  The instructions are the same as for lights, however the Mavlink message is different:

[![](/img/appnotes/camera_and_lights_through_cockpit/create_cam_action.png)](/img/appnotes/camera_and_lights_through_cockpit/create_cam_action.png)

    1. The "Command ID" should be `MAV_CMD_DO_MOUNT_CONTROL`
    2. "Param1" should be `{{user/compound/camtiltcontrol-int}}` (or whatever your datalake variable is named)
    3. "Param7" should be "2" (which corresponds to [`MAV_MOUNT_MODE_MAVLINK_TARGETING`](https://mavlink.io/en/messages/common.html#MAV_MOUNT_MODE_MAVLINK_TARGETING))

An automatic trigger must also be set up for the camera

[![](/img/appnotes/camera_and_lights_through_cockpit/cam_action_trigger.png)](/img/appnotes/camera_and_lights_through_cockpit/cam_action_trigger.png)
