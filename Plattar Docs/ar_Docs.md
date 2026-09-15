One of the features that the Plattar Plugin supports is the ability to open your scenes in Augmented Reality (AR) on supported mobile devices.

To help users transport between the web platform to mobile, the Plattar plugin can display a QR Code which links to the AR Launcher on mobile. The Plattar AR Launch page contains all the code to turn your scene/configuration into an AR compatible model and open the Augmented Reality experience.

Swap from Viewer to a QR Code (Legacy Code)
The Plattar Embed supports displaying a QR Code on desktop devices for users to scan with their AR supported devices in order to see the AR Experience.

Plattar supports swapping between the 3D viewer and the QRCode, but due to the load times it can be easier to maintain two embeds, one for the Viewer and one for the QRCode. The next doc will explain how to do that, but for now we will use the legacy functions to do the switch.

Step 1: Setup Basic Elements
First, we'll get the embed tag through the CMS and install the Plattar plugin through a script tag. (If you need a refresher, you can go back to the basic example)

Step 2: Add Toggle Button
Then add a function and button to do the switch:

<section>
  <div>
    <button onclick="toggleQRCode()">Toggle Plattar QRCode</button>
  </div>
</section>
CopyErrorCopied!
Step 3: Implement Toggle Function
And some JavaScript to handle the button press:

// getting the embed Tag
const embed = document.getElementById("embed");

// A function that gets called when the button is pressed
function toggleQRCode() {

}
CopyErrorCopied!
Swap from Viewer to the Plattar Launch Page
The Plattar Launch page is a more advanced version of the QR Code, which contains some instructions to the user on how to interact with the AR experience and will either display the QR Code or a button depending on whether they are on an AR supported device.

Step 1: Add Launch Button
After setting up the embed tag, we can add a button which we'll use to launch the AR experience:

<section>
  <div>
    <button onclick="toggleLaunchPage()">Toggle Plattar Launch Page</button>
  </div>
</section>
CopyErrorCopied!
Step 2: Create Toggle Function
Now we can start adding some functionality into the script. We can start by getting the embed tag from the DOM and creating a simple function that will be called when the button is pressed:

// getting the embed Tag
const embed = document.getElementById("embed");

// A function that gets called when the button is pressed
function toggleLaunchPage() {

}
CopyErrorCopied!
Step 3: Implement Launch Logic
There's multiple ways to launch the AR Launcher, we'll use the embed-type for this example. All we need to do is change the embed-type inside the embed tag attribute to launcher.

Since we only have one button, we want it to both function as a way to trigger the AR Launcher and return back to the 360 Viewer. We can use the function getAttribute("embed-type") to find out the current state of the embed node:

function toggleLaunchPage() {
  // Prints out what the current state of the embed-type is
  console.log(embed.getAttribute("embed-type"));
}
CopyErrorCopied!
Step 4: Add Conditional Logic
From here we can add a simple if statement which only triggers when the renderer isn't in launcher mode and to switch between them:

const embed = document.getElementById("embed");

function toggleLaunchPage() {
  // Depending on the embed-type trigger one of these lines
  // Notably, this works even if the embed type starts as null instead of viewer
  if ((embed.getAttribute("embed-type")) != "launcher") {
    console.log("Launch AR");
  }
  else {
    console.log("Return to Viewer");
  }
}
CopyErrorCopied!
Step 5: Set Attributes
Finally, using a similar method to changing variants, we'll use setAttribute() to change between embed-types:

const embed = document.getElementById("embed");

function toggleLaunchPage() {
  if ((embed.getAttribute("embed-type")) != "launcher") {
    // uses setAttribute() to change between launcher and viewer
    embed.setAttribute("embed-type", "launcher");
  }
  else {
    embed.setAttribute("embed-type", "viewer");
  }
}
CopyErrorCopied!
Launching AR Directly
The Plattar Embed supports launching AR directly on supported devices, by using the startAR() function. It also has a helper function to query whether the device supports AR or not, to allow a fallback to show the launch page or an error message.

Step 1: Add Launch Button
Firstly we set up a button to launch the AR experience:

<button onclick="launchAR()">Launch AR</button>
CopyErrorCopied!
Step 2: Create Launch Function
Then we create a function to call when the launch button is pressed:

function launchAR() {
  console.log('Launch AR Experience');
}
CopyErrorCopied!
Step 3: Check AR Support
We can use the Plattar utility to test if the device supports AR:

if (PlattarARAdapter.Util.canAugment()) {
  console.log('Supports AR');
}
else {
  console.log('AR Unavailable');
}
CopyErrorCopied!
Step 4: Implement Full Launch Logic
And finally we can launch the AR, with a fallback option to open the Plattar Launch page if it fails or if AR is unavailable:

function launchAR() {
  if (PlattarARAdapter.Util.canAugment()) {
    embed.startAR()
      .catch(function(err) {
        console.error('Error starting AR:', err);
        toggleLaunchPage(true);
      });
  }
  else {
    // fallback to rendering QR Code
    toggleLaunchPage(true);
  }
}




--------------------




Using a 3D Viewer and QR Code Node Together
Jump to final result

In a custom-built integration, you may want to display a QR Code to your users at the same time as displaying the Plattar Viewer, for ease of use. This is very easy to set up and will allow the users to configure at will and scan the QR code for each configuration without having to reload the scene.

Display Two Embeds
Step 1: Copy Previous Example
First, we'll copy the variation selection using attribute example from a previous tutorial.

Step 2: Add Second Embed
We will then use two embed nodes, one for the viewer and one for the QR code, and give them ids to identify them. Note the difference in the init attribute for the QR Code embed:

<section>
  <div>
    <!-- The embed code acquired from the CMS -->
    <plattar-embed id="embed-viewer" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="viewer" height="700px"></plattar-embed>

    <plattar-embed id="embed-qrcode" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="qrcode" height="500px"></plattar-embed>
  </div>
</section>
CopyErrorCopied!
Step 3: Update Selection Function
Then we will edit the selectVariation(id) function to set the attribute on both embed nodes:

const embedViewer = document.getElementById("embed-viewer");
const embedQrcode = document.getElementById("embed-qrcode");

// Function used to change between variations
function selectVariation(id) {
  embedViewer.setAttribute("variation-id", id);
  embedQrcode.setAttribute("variation-id", id);
}




--------------------


More Docs available at: https://docs.plattar.com/#/


