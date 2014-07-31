Description
===========
 ARem is a software tool enabling simulation of Augmented Reality
 by allowing an AR developer to record a 360 degree view of a
 location using the devices camera and orientation sensors (this
 functionality is provided by the ARemRecorder app). The ARCamera
 class which provides an impostor of the Android camera class
 can then be used to preview the recorded scene instead of the live
 camera preview provided by the Android Camera class. The ARCamera
 preview callback is analogous to the standard Camera preview
 callback except that the preview bytes provided in the callback
 are extracted from a file created by the recorder application
 based on the current bearing returned by the orientation
 sensor(s). These preview bytes are passed to the development code
 via the same preview callback as provided by the standard Camera
 classes and can thus be processed by Computer Vision algorithms
 before being displayed by the client application. The frames are
 stored as individual video frames in RGBA, RGB or RGB565 format
 and not as video so the preview can be accessed in both
 clockwise and anti-clockwise directions.

 The tool is aimed at developers of outdoor mobile AR application
 as it allows the developer to record one or more 360 degree
 panoramas of a given location and then debug and test the AR
 application in the comfort of a office or home without having to
 make extensive changes to the programming
 code.

Components
==========
ARem comprises the following components (or modules in Android Studio parlance)
ARemRecorder
------------
<h3>Description and Usage</h3>
Used to record a 360 degree view at a given location (will also be
available on Google Play).

The recorder displays the camera output in full screen mode with a interface drawer on the left border
of the display which can be dragged out. To start recording drag the drawer out and click the recording
button. At start of recording the user is asked to provide a name for the recording files, a file format,
resolution, recording increment and which orientation sensor implementation to use.

The file format can currently be one of RGBA, RGB, RGB565, NV21 and YV12.
While resulting in larger files RGBA is preferred as GPU texture units
work best with 4 byte aligned textures and most OpenGL implementations
convert to RGBA internally anyway. As RGB is not aligned some resolutions
may have issues (see
[OpenGL Mistakes](http://www.opengl.org/wiki/Common_Mistakes#Texture_upload_and_pixel_reads),
although most of the resolutions provided by Android seem to work (the RGB buffer is also internally
expanded to align to 4 bytes in the recorder code). If NV21 or YV12 is chosen then it is the responsibility
of the implementation using the recording to convert to RGBA with an obvious time penalty
(see the source for how to use Renderscript intrinsics to convert).


The resolution can be selected in a spinner which provides all of the resolutions
supported by the device. The recording increment specifies the bearing increment
between which frames are saved. The Rotation sensor specifies which orientation sensor
fusion method to use for calculating the device orientation and bearing. The sensor fusion
code is based on work done by Alexander Pacha for his Masters thesis (see Credits below).

Once recording the interface drawer displays the current bearing and the target bearing. At the start of the
recording the target is set to 355 in order to start at 0 approaching in a clockwise direction. The camera
output surface displays an overlaid arrow with the direction of movement which is red if correcting and
green if recording. Once the user moves to 355 the target is set to 0 and on rotating to 0 the arrow becomes
green and the target bearing is set to 0 plus the recording increment. During recording if a frame is missed
for a bearing increment then the target bearing is reset to 5 degrees before the missed bearing and the arrow
color changes to red until the user corrects. At completion of recording the recording frame file which was
temporarily saved in NV21 format is converted into the final format.

<h3>Issues, Caveats and Recommendations</h3>
The two main issues are the stability of the compass bearing sensors and ensuring that the frame that gets
saved for a given bearing is the correct frame for that bearing. Even when using enhanced fusion sensor
techniques the bearings can sometimes drift which can give the appearance of "frame jump" in a recording. When
recording keeping the <b>device at a constant vertical angle</b> and <b>rotating slowly and smoothly</b> to
reduce the number of corrections required is important. The direction of movement can seemingly also
sometimes affect bearing accuracy which is why during correction the target bearing is set to 5 degrees
before the target bearing to ensure that the target bearing is always approached from the same side.
The recording options allow 0.5, 1, 1,5 and 2 degree increments, however 0.5 degree recordings can be
difficult possibly because the accuracy required is just within hardware limits which is why the default
is 1 degree increments.

In order to attempt to ensure that the correct frame is assigned to a bearing interval a timestamp is
attached to each frame added to a ring buffer and every bearing event also has a custom timestamp (as
opposed to the Android provided event timestamp) attached. On recent versions of Android the timestamp
used is provided by SystemClock.elapsedRealtimeNanos(). Then for each bearing increment the frame ring
buffer is cleared, then the recording thread blocks on a conditional variable until frames are available. The
frame with the closest timestamp to the current bearing timestamp is then selected.

ARemu
-----
<h3>Description and Usage</h3>
This module provides the ARCamera impostor class as well as  supporting classes and is thus the primary module
to be used by AR applications wishing to emulate an AR environment. In addition to the Camera API methods,
ARCamera also provides supplementary API methods specific to playing back recordings such as setting the recording files. The
render mode can also be set to dirty where frames are previewed only when the bearing changes, or continuous
where the preview continuously receives frames. In the latter case the frame rate specified in the
Camera.Parameters returned by ARCamera.getParameters is respected. When creating applications it is possible to
emulate the C/C++ preprocessor #ifdef statements in Java by using the Just in Time (JIT) feature of the Java
runtime to optimise out unused code (unfortunately unlike for C, it does not reduce code size, just speed):

<code>
static final boolean EMULATION = true;<br>
if (EMULATION)<br>
&nbsp;&nbsp;&nbsp;cameraEm.startPreview();<br>
else<br>
&nbsp;&nbsp;&nbsp;camera.startPreview();<br>
</code>
In the code above the unused branch will be optimised out.

Common
------
Provides various classes shared across different modules such as the orientation fusion implementation and
various OpenGL and math helper classes.

ARemSample
----------
This is a sample application which can be used to play back a recording made with ARemRecorder. It also
makes use iof the ARemu review supplementary API which allows 360 degree playback without having to move
the device. Will also be available on Google Play.

DisplayFrames
-------------
This is another sample application which can also be used to play back a recording made with ARemRecorder but
provides an option to set a start degree and single step forwards or backwards through the frames.

AROpenCVemu
-----------
Provides an implementation of a CameraBridgeViewBase OpenCV4Android camera preview UI view class which uses
ARCamera instead of the Android Camera class. This could have been included in ARemu, however this
would have resulted in a dependency on OpenCV for ARem.

ARemOpenCVSample
----------------
Provides a sample using the CameraBridgeViewBase derived OpenCV4Android camera preview UI view class
implemented in AROpenCVemu.

OpenCV
------
OpenCV4Android (currently version 2.4.9) as used by the OpenCV modules.

Credits
=======

ADraweLayout which is used to provide a sliding drawer UI is copyright DoubleTwist and is licensed under the
Apache license. See their [Github site](https://github.com/doubletwist/adrawerlayoutlib).
</p>
<p>
    Much of the sensor code is based on  <a href="https://bitbucket.org/apacha/sensor-fusion-demo">work</a> done by
    Alexander Pacha for his <a href="http://my-it.at/media/MasterThesis-Pacha.pdf">Masters thesis</a>. The fused
    Gyroscope/Accelerometer/Magnet sensor is based on code by Kaleb Kircher
    (See his <a href="http://www.kircherelectronics.com/blog/index.php/11-android/sensors/16-android-gyroscope-fusion">blog</a>).
</p>
</body>
</html>
