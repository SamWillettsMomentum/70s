
2. Loading a Scene

Step 1: Install the Plugin
First, we'll need to install the Plattar plugin, which can be done by adding the following code into your website <head> tag:

<!-- install the plugin using a script tag -->
<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

Step 3: Add the Embed to Your Website
Once we get the embed code we can now add it into our website like so:

<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

<!-- the embed code obtained through the CMS -->
<plattar-embed id="embed" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="viewer"></plattar-embed>

Listening for Scene Load Events
An extra step you can do is add a function listener to wait for the scene to complete loading. You can use this to show your own loading screen or instructions until the Plattar scene has completed loading:

const embed = document.getElementById("embed");

embed.viewer.context.setLoading = function(params) {
  if (params.loading == false) {
    console.log('Scene has completed loading');
  }
}
CopyErrorCopied!
Note for Dynamic Loading Environments
In environments like CodePen where the embed loads asynchronously, you may need to add extra functionality:

let intv = setInterval(() => {
  if (!embed.viewer) {
    return;
  }
  clearInterval(intv);

  embed.viewer.context.setLoading = function(params) {
    if (params.loading == false) {
      console.log('Scene has completed loading');
    }
  }
}, 100);





3. Changing Between Scenes

In the previous step, we obtained the scene embed tag with scene-id and other mandatory attributes through the CMS. During runtime, we can change this scene-id to swap between different scenes. The renderer will automatically change scenes when the scene-id is updated.

Changing the Scene using Scene ID
Now we have the new Scene ID, we can change the scene-id attribute directly in the embed. The Plattar plugin will automatically change the scene whenever the scene-id attribute is modified.

Step 1: Prepare the HTML
Start by removing the show-ui attribute as we'll be creating our own buttons. We'll also add an ID to the HTML tag so we can select it using JavaScript:

<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

<!-- remove show-ui="true" and added id="embed" -->
<plattar-embed id="embed" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="viewer"></plattar-embed>
CopyErrorCopied!
Step 2: Add Control Buttons
Next, we'll add a section above the embed with buttons and text. We'll use these buttons to call the scene change functions:

<!-- Section containing buttons which will call the scene switching -->
<div>
  <p>Scene Selection</p>
  <button>Helmet</button>
  <button>Bed</button>
  <button>Coffee Machine</button>
</div>
CopyErrorCopied!
Step 3: Configure Button Attributes
Add a value and onclick attribute to these buttons. Each button should have a value equal to the scene-id from the CMS. The onclick attributes should call selectScene(this.value) which will pass over the value attribute we set:

<div>
  <p>Scene Selection</p>
  <!-- button value contains scene-id obtained through the CMS -->
  <!-- These buttons will use their value as parameter to call selectScene() -->
  <button value="d9331ec5-3292-4ba9-b632-fab49b29a9e8" onclick="selectScene(this.value)">Helmet</button>
  <button value="550b5585-7f0b-4a3a-ab01-dc5b27741372" onclick="selectScene(this.value)">Bed</button>
  <button value="95c7a7a0-c89d-42dc-9752-102c36475413" onclick="selectScene(this.value)">Coffee Machine</button>
</div>
CopyErrorCopied!
Step 4: Implement JavaScript Functions
Once we have the HTML setup we can move into the JavaScript. We'll first get the embed tag through an ID:

// Obtain the embed through the ID we've added
const embed = document.getElementById("embed");
CopyErrorCopied!
Now, we can add the selectScene(id) function which gets called by the buttons we've made earlier, passing over the scene-id:

const embed = document.getElementById("embed");

// create an empty function which will be called by the buttons we've added
function selectScene(id) {}
CopyErrorCopied!
Finally, to change scenes we'll be changing the scene-id attribute from the embed tag using setAttribute(attribute, value):

const embed = document.getElementById("embed");

function selectScene(id) {
  // using setAttribute(), we'll change the scene into the ones we've added as value for the buttons
  embed.setAttribute("scene-id", id);
}









4.Selecting by Variation ID


Selecting Variation ID
While changing between scenes works for multiple different products, sometimes products are more a variation of each other rather than a different object entirely. In which case we'll change between variations rather than changing the scene entirely.

Changing Variation Using Attribute
There are two ways to change a variation in a scene: either by changing the attribute of the embed or using a JavaScript function.

In the first example, we'll use attributes to change between Variation IDs.

Step 1: Setup Basic HTML
First, just like in the basic example we'll get the embed tag through the CMS and install the plattar plugin through a script tag. (If you need a refresher, you can go back to the basic example)

<!-- Installing the Plattar Plugin -->
<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

<!-- The embed code acquired from the CMS -->
<plattar-embed id="embed" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="viewer" height="700px"></plattar-embed>
CopyErrorCopied!
Step 2: Add Selection Buttons
Next, let's add buttons to select the variations. In the scene we have for this example, we'll be changing the shell, lens, and mic between a handful of options:

<!-- Added a section containing all the needed buttons to change between variations -->
<section>
  <p>Shell Colour</p>
  <div>
    <button>White Gloss</button>
    <button>Black Matte</button>
    <button>Taheel Blue Gloss</button>
    <button>Carbon Fibre Matte</button>
  </div>

  <p>Lens Colour</p>
  <div>
    <button>Grey</button>
    <button>Clear</button>
    <button>Gold</button>
    <button>Ice</button>
  </div>

  <p>Microphone Choice</p>
  <div>
    <button>Flex Boom</button>
    <button>Wire Rail</button>
  </div>
</section>
CopyErrorCopied!
Step 3: Configure Button Values
Just like in the changing scene example we'll use the value in our buttons to hold our variation-id for changing between variations in shells, lenses and microphones:

<section>
  <p>Shell Colour</p>
  <div>
    <!-- The button value contains a variation-id obtained from the CMS -->
    <!-- On click, these buttons will use its value as a variable to call selectVariation() -->
    <button type="button" value="936639e0-3854-11ec-b8ed-8d91c3e5372d" onclick="selectVariation(this.value)">White Gloss</button>
    <button type="button" value="00d19f50-3855-11ec-ad9f-f7fd41a04008" onclick="selectVariation(this.value)">Black Matte</button>
    <button type="button" value="315beb90-990a-11ec-94ac-bb29596fb341" onclick="selectVariation(this.value)">Taheel Blue Gloss</button>
    <button type="button" value="41e79050-99dd-11ec-839d-974669311e2e" onclick="selectVariation(this.value)">Carbon Fibre Matte</button>
  </div>

  <p>Lens Colour</p>
  <div>
    <button type="button" value="11d66560-3856-11ec-ac17-edcbe6ab5f51" onclick="selectVariation(this.value)">Grey</button>
    <button type="button" value="008b84c0-3856-11ec-b231-53e63b05f28d" onclick="selectVariation(this.value)">Clear</button>
    <button type="button" value="08b1cdf0-3856-11ec-81ea-6db89c21ded2" onclick="selectVariation(this.value)">Gold</button>
    <button type="button" value="19ceb2f0-3856-11ec-b5fe-99005f0f10da" onclick="selectVariation(this.value)">Ice</button>
  </div>

  <p>Microphone Choice</p>
  <div>
    <button type="button" value="9e72b970-952f-11ec-b235-99fb2ff9be07" onclick="selectVariation(this.value)">Flex Boom</button>
    <button type="button" value="9e7431d0-952f-11ec-83d1-c1a06c185e57" onclick="selectVariation(this.value)">Wire Rail</button>
  </div>
</section>
CopyErrorCopied!
Step 4: Implement JavaScript
Moving into the JavaScript, as with the changing scene example, we'll first find the embed tag through its id:

// get the embed tag using id
const embed = document.getElementById("embed");
CopyErrorCopied!
Then, using setAttribute(attribute, value) to change the variation-id, which we've set to be called by the buttons when clicked:

// Function used to change between variation-ids
function selectVariation(id) {
  // using setAttribute()
  embed.setAttribute("variation-id", id);
}



Changing Variation Using Function
In this example, we'll use a function instead which offers better redundancy and error handling.

Step 1: Use Previous HTML
We'll first start from the HTML we've used in the previous example:

<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

<section>
  <div>
    <p>Shell Colour</p>
    <button type="button" value="936639e0-3854-11ec-b8ed-8d91c3e5372d" onclick="selectVariation(this.value)">White Gloss</button>
    <button type="button" value="00d19f50-3855-11ec-ad9f-f7fd41a04008" onclick="selectVariation(this.value)">Black Matte</button>
    <button type="button" value="315beb90-990a-11ec-94ac-bb29596fb341" onclick="selectVariation(this.value)">Taheel Blue Gloss</button>
    <button type="button" value="41e79050-99dd-11ec-839d-974669311e2e" onclick="selectVariation(this.value)">Carbon Fibre Matte</button>
  </div>

  <div>
    <p>Lens Colour</p>
    <button type="button" value="11d66560-3856-11ec-ac17-edcbe6ab5f51" onclick="selectVariation(this.value)">Grey</button>
    <button type="button" value="008b84c0-3856-11ec-b231-53e63b05f28d" onclick="selectVariation(this.value)">Clear</button>
    <button type="button" value="08b1cdf0-3856-11ec-81ea-6db89c21ded2" onclick="selectVariation(this.value)">Gold</button>
    <button type="button" value="19ceb2f0-3856-11ec-b5fe-99005f0f10da" onclick="selectVariation(this.value)">Ice</button>
  </div>

  <div>
    <p>Mic Choice</p>
    <button type="button" value="9e72b970-952f-11ec-b235-99fb2ff9be07" onclick="selectVariation(this.value)">Flex Boom</button>
    <button type="button" value="9e7431d0-952f-11ec-83d1-c1a06c185e57" onclick="selectVariation(this.value)">Wire Rail</button>
  </div>
</section>

<section>
  <div>
    <plattar-embed id="embed" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="viewer" height="700px"></plattar-embed>
  </div>
</section>
CopyErrorCopied!
Step 2: Use Messenger Function
Unlike the previous example which uses setAttribute(attribute, value), we'll instead use a function from the plugin, viewer.messenger.selectVariationID(variationID:string|Array<string>):

const embed = document.getElementById("embed");

function selectVariation(id) {
  // instead of changing attribute we can use plattar plugin built-in selectVariationID()
  embed.viewer.messenger.selectVariationID(id);
}




5. Selecting Variation SKU
Variation IDs are generated by Plattar and are guaranteed to be unique. However, as they can be long and cumbersome, the Plattar Plugin allows the use of user-defined SKUs instead to help sync with an existing product database.

Changing Variation Using Attribute
There are two ways to change a variation in a scene using the SKU: either by updating the attribute of the embed node or through a JavaScript function.

In the first example, we'll use attributes to change between SKU values.

Step 1: Setup Basic HTML
First, just like in the basic example we'll get the embed tag through the CMS and install the plattar plugin through a script tag. (If you need a refresher, you can go back to the basic example)

<!-- Installing the Plattar Plugin -->
<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

<!-- The embed code acquired from the CMS -->
<plattar-embed id="embed" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="viewer" height="700px"></plattar-embed>
CopyErrorCopied!
Step 2: Add Selection Buttons
Next, let's add buttons we'll use to change between the variations. In the scene we have for this example, we'll be changing the shell, lens, and mic between a handful of options:

<!-- install script tag above -->
<!-- Added a section containing all the needed buttons to change between variations -->
<section>
  <div>
    <p>Shell Colour</p>
    <button>White Gloss</button>
    <button>Black Matte</button>
    <button>Taheel Blue Gloss</button>
    <button>Carbon Fibre Matte</button>
  </div>

  <div>
    <p>Lens Colour</p>
    <button>Grey</button>
    <button>Clear</button>
    <button>Gold</button>
    <button>Ice</button>
  </div>

  <div>
    <p>Mic Choice</p>
    <button>Flex Boom</button>
    <button>Wire Rail</button>
  </div>
</section>
<!-- embed tag below -->
CopyErrorCopied!
Step 3: Configure Button Values
Just like in the changing scene example we'll use the value in our buttons to hold our sku for changing between variations in shells, lenses and microphones:

<!-- install script tag above -->
<section>
  <div>
    <p>Shell Colour</p>
    <!-- The button value contains a variation SKU obtained from the CMS -->
    <button type="button" value="8z4m4" onclick="selectSKU(this.value)">White Gloss</button>
    <!-- On click, these buttons will use its value as a variable to call selectSKU() -->
    <button type="button" value="f1ora" onclick="selectSKU(this.value)">Black Matte</button>
    <button type="button" value="bluey" onclick="selectSKU(this.value)">Taheel Blue Gloss</button>
    <button type="button" value="ge3d8" onclick="selectSKU(this.value)">Carbon Fibre Matte</button>
  </div>

  <div>
    <p>Lens Colour</p>
    <button type="button" value="pmk51" onclick="selectSKU(this.value)">Grey</button>
    <button type="button" value="he5mx" onclick="selectSKU(this.value)">Clear</button>
    <button type="button" value="uxzs1" onclick="selectSKU(this.value)">Gold</button>
    <button type="button" value="ogcu7" onclick="selectSKU(this.value)">Ice</button>
  </div>

  <div>
    <p>Mic Choice</p>
    <button type="button" value="zl33i" onclick="selectSKU(this.value)">Flex Boom</button>
    <button type="button" value="ogqdu" onclick="selectSKU(this.value)">Wire Rail</button>
  </div>
</section>
<!-- embed tag below -->
CopyErrorCopied!
Step 4: Implement JavaScript
Moving into the JavaScript, in the same way as the previous example, we'll first find the embed tag through its id:

// get the embed tag using id
const embed = document.getElementById("embed");
CopyErrorCopied!
Then, using setAttribute(attribute, value) to change the SKU, which we've set to be called by the buttons when clicked:

const embed = document.getElementById("embed");

// Function used to change between SKUs
function selectSKU(sku) {
  embed.setAttribute("variation-sku", sku);
}



6. Changing Variation Using Function
In this example, we'll use a function instead which offers better redundancy and error handling.

Step 1: Use Previous HTML
We'll first start from the HTML we've used in the previous example:

<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

<section>
  <div>
    <p>Shell Colour</p>
    <!-- The button value contains a variation SKU obtained from the CMS -->
    <button type="button" value="8z4m4" onclick="selectSKU(this.value)">White Gloss</button>
    <!-- On click, these buttons will use its value as a variable to call selectVariation() -->
    <button type="button" value="f1ora" onclick="selectSKU(this.value)">Black Matte</button>
    <button type="button" value="bluey" onclick="selectSKU(this.value)">Taheel Blue Gloss</button>
    <button type="button" value="ge3d8" onclick="selectSKU(this.value)">Carbon Fibre Matte</button>
  </div>

  <div>
    <p>Lens Colour</p>
    <button type="button" value="pmk51" onclick="selectSKU(this.value)">Grey</button>
    <button type="button" value="he5mx" onclick="selectSKU(this.value)">Clear</button>
    <button type="button" value="uxzs1" onclick="selectSKU(this.value)">Gold</button>
    <button type="button" value="ogcu7" onclick="selectSKU(this.value)">Ice</button>
  </div>

  <div>
    <p>Mic Choice</p>
    <!-- For demonstration we use SKU instead of variation-id for microphone -->
    <button type="button" value="zl33i" onclick="selectSKU(this.value)">Flex Boom</button>
    <button type="button" value="ogqdu" onclick="selectSKU(this.value)">Wire Rail</button>
  </div>
</section>
CopyErrorCopied!
Step 2: Use Messenger Function
Unlike the previous example which uses setAttribute(attribute, value), we'll instead use a function from the plugin, viewer.messenger.selectVariationSKU(variationSKU:string|Array<string>):

const embed = document.getElementById("embed");

function selectSKU(sku) {
  embed.viewer.messenger.selectVariationSKU(sku);
}





7. Selecting Multiple Variations
In certain scenarios, you might want to change multiple variations at the same time.

Prerequisites
To follow this guide you need to know how to:

Install the Plattar plugin
Get the embed tag through the CMS
Get the variation-id or SKU
Change between them through attribute or function
Changing Multiple Variations Using Attribute
Step 1: Setup Basic HTML
First, we'll get the embed tag through the CMS and install the Plattar plugin through a script tag. (If you need a refresher, you can go back to the basic example)

Step 2: Add Control Buttons
Next we'll add a couple of buttons to change between the variations:

<!-- install script tag above -->
<!-- Added a section containing all the needed buttons to change between variations -->
<section>
  <div>
    <p>Combinations</p>
    <button type="button">Base</button>
    <button type="button">Black and Gold</button>
    <button type="button">All Blue</button>
  </div>
</section>
<!-- embed tag below -->
CopyErrorCopied!
Step 3: Configure Button Values
Just like in other examples, we'll use this button to call a function on click. As for the value attribute, we'll use a comma-separated value to change multiple variations at the same time:

<section>
  <div>
    <p>Combinations</p>
    <!-- Add value as a comma separated list of multiple SKUs -->
    <button type="button" value="8z4m4,pmk51,zl33i" onclick="selectSKU(this.value)">Base</button>
    <!-- Button calls selectSKU() which we will define in later steps -->
    <button type="button" value="f1ora,uxzs1,ogqdu" onclick="selectSKU(this.value)">Black and Gold</button>
    <button type="button" value="y6j1e,ogcu7,zl33i" onclick="selectSKU(this.value)">All Blue</button>
  </div>
</section>
CopyErrorCopied!
Note: When changing multiple variations at the same time using the attributes, the variation-id or SKU has to be separated using a comma without space

Step 4: Implement JavaScript
Moving into the JavaScript, we'll start by getting the tag through an id search and defining the function we made for the buttons to call on click:

// get the embed tag through its id
const embed = document.getElementById("embed");

// define selectSKU() function
function selectSKU(sku) {}
CopyErrorCopied!
Using setAttribute(attribute, value) we'll now change the SKU attribute. The Plattar Plugin will detect the attribute change and swap to the selected variations in the scene:

const embed = document.getElementById("embed");

function selectSKU(sku) {
  // directly use setAttribute() to change the value of the variation-sku
  // no changes needed to be made since the value is already in the correct format
  embed.setAttribute("variation-sku", sku);
}



8. Changing Multiple Variations Using Function
Step 1: Use Previous HTML
Just like in the previous example, we'll start off from the same HTML:

<script src="https://sdk.plattar.com/plattar-plugin.min.js"></script>

<section>
  <div>
    <p>Combinations</p>
    <button type="button" value="8z4m4,pmk51,zl33i" onclick="selectSKU(this.value)">Base</button>
    <button type="button" value="f1ora,uxzs1,ogqdu" onclick="selectSKU(this.value)">Black and Gold</button>
    <button type="button" value="y6j1e,ogcu7,zl33i" onclick="selectSKU(this.value)">All Blue</button>
  </div>
</section>

<section>
  <div>
    <plattar-embed id="embed" scene-id="d9331ec5-3292-4ba9-b632-fab49b29a9e8" init="viewer" height="700px"></plattar-embed>
  </div>
</section>
CopyErrorCopied!
Step 2: Split and Process Values
In the JavaScript, instead of using setAttribute(attribute, value) we'll first have to separate the values into an array of strings based on where the comma is placed using value.split(character) function:

const embed = document.getElementById("embed");

function selectSKU(sku) {
  // Using split(",") to split the value into a comma separated list
  console.log(sku.split(","));
}
CopyErrorCopied!
Step 3: Use Messenger Function
Once we have the array, we can input them directly into viewer.messenger.selectVariationSKU(variationSKU:string|Array<string>) or viewer.messenger.selectVariationID(variationID:string|Array<string>):

const embed = document.getElementById("embed");

function selectSKU(sku) {
  // Directly putting the comma separated list into the viewer.messenger.selectVariationSKU()
  embed.viewer.messenger.selectVariationSKU(sku.split(","));
}




