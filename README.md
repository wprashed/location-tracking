# Let’s make a form that puts current location to use in a map!
Getting started and setting up
To build this app, we’re going to use the Materialize CSS framework to save us some time fussing with styles. Materialize is a modern responsive front-end framework based on Google’s Material Design system. The beta version works with vanilla JavaScript.

A basic setup using Materialize’s CSS and JavaScript files in a document is like this:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Address Locator</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0-beta/css/materialize.min.css">
  <link rel="stylesheet" href="css/main.css">
</head>
<body>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0-beta/js/materialize.min.js"></script>
  <script src="js/main.js"></script>
</body>
</html>
```

We’re also going to use the Google Maps API for displaying the map and getting the user’s human-readable address. We’ll need an API key to do this. Here’s how to get one:

Sign in to your Google Developer’s Console Account
Create a new project or select an existing one
Click on “Enable APIs and Services”
Select the “Maps Javascript API” option
Click “Enable” on the new page that comes up. Go back to the previous page and do a search for “Geocoding API,” click on it and enable it as well
Then, on the right nav of the page, click on Credentials, copy the API key on the page and save it to a file
Now, let’s update our document to show the map and let the user know it can be used to get their current location as the address. Also, we will add a form that’s pre-filled with the address the user selects.

```
<body>
  <div class="container">
    <h3>Shipping Address</h3>
    <p>You can click the button below to use your current location as your shipping address</p>
    <div id="map">
    </div>
    <button id="showMe" class="btn">Use My Location</button>
    <form id="shippingAddress">
      <div id="locationList"></div>
      <br>
      <div class="input-field">
        <textarea class="input_fields materialize-textarea" id="address" type="text"></textarea>
        <label class="active" for="address">Address (Area and Street)</label>
      </div>
      <div class="input-field">
        <input class="input_fields" id="locality" type="text">
        <label class="active" for="locality">Locality</label>
      </div>
      <div class="input-field">
        <input class="input_fields" id="city" type="text">
        <label class="active" for="city">City/District/Town</label>
      </div>
      <div class="input-field">
        <input class="input_fields" id="postal_code" type="text">
        <label class="active" for="pin_code">Pin Code</label>
      </div>
      <div class="input-field">
        <input class="input_fields" id="landmark" type="text">
        <label class="active" for="landmark">Landmark</label>
      </div>
      <div class="input-field">
        <input class="input_fields" id="state" type="text">
        <label class="active" for="State">State</label>
      </div>
    </form>
    <!-- You could add a fallback address gathering form here -->
  </div>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0-beta/js/materialize.min.js"></script>
  <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY"></script>
  <script src="js/main.js"></script>
</body>

</html>
```

While we are here, let’s style things a bit to make this look a little better:

```
.container {
  width: 50%;
  max-width: 800px;
}

#map {
  height: 50vh;
  margin-bottom: 10px;
  display: none;
}

#locationList .card {
  padding: 10px;
}

#toast-container {
  top: 50%;
  bottom: unset;
}

.toast {
  background-color: rgba(0, 0, 0, 0.8);
}

@media only screen and (max-width: 768px) {
  .container {
    width: 80%;
  }
}
```

This CSS hides the map until we are ready to view it. Our app should look like this:

Let’s plan this out
Our app will be making use of the HTML5 Geolocation API to determine our user’s current location as well as Google’s Geocode API with a technique called Reverse Geocoding. The Geocode API takes a human-readable address and changes it into geographical (latitudinal and longitudinal) coordinates and marks the spot in the map.

Reverse Geocoding does the reverse. It takes the latitude and longitude and converts them to human-readable addresses. Geocoding and Reverse Geocoding are well documented.

Here’s how our app will work:

The user clicks on the “Use My Location” button
The user is located with the HTML5 Geolocation API (navigator.geolocation)
We get the user’s geographic coordinates
We pass the coordinates to the geocode request API
We display the resulting addresses to the user
Most times, the geocode returns more than one address, so we would have to show the user all the returned addresses and let them choose the most accurate one.

Phew! Finally, we can get to the fun part of actually writing the JavaScript. Let’s go through each of the steps we outlined.

Step 1: Clicking the button
In our main.js file, let’s get a reference to the HTML button. While we are at it, we’ll set up some other variables we’ll need, like our API key.

```
//This div will display Google map
const mapArea = document.getElementById('map');

//This button will set everything into motion when clicked
const actionBtn = document.getElementById('showMe');

//This will display all the available addresses returned by Google's Geocode Api
const locationsAvailable = document.getElementById('locationList');

//Let's bring in our API_KEY
const __KEY = 'YOUR_API_KEY';

//Let's declare our Gmap and Gmarker variables that will hold the Map and Marker Objects later on
let Gmap;
let Gmarker;

//Now we listen for a click event on our button
actionBtn.addEventListener('click', e => {
  // hide the button 
  actionBtn.style.display = "none";
  // call Materialize toast to update user 
  M.toast({ html: 'fetching your current location', classes: 'rounded' });
  // get the user's position
  getLocation();
});
```

When the click handler for our button runs, it:

Hides the button
Alerts the user that we are getting their current location (a “toast” in Materialize is like a popup notification)
Calls the getLocation function
Step 2: Get the user’s location (latitude and longitude)
When our getLocation function is invoked, we need to do some more work. First, let’s check if we can even use the Geolocation API:

```
getLocation = () => {
  // check if user's browser supports Navigator.geolocation
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(displayLocation, showError, options);
  } else {
    M.toast({ html: "Sorry, your browser does not support this feature... Please Update your Browser to enjoy it", classes: "rounded" });
  }
}
```

When we have support, it calls geolocation’s getCurrentPosition method. If it doesn’t have support, then the user is alerted that there’s no browser support.

If there is support, then the getCurrentLocation method is used to get the current location of the device. The syntax is like this:

navigator.geolocation.getCurrentPosition(*success, error, [options]*)
success : This is a callback function that takes a position as its only parameter. For us, our success callback function is the displayLocation function.
error : [optional] This is a callback function that takes a PositionError as its sole input parameter. You can read more about this here. Our error callback function is the showError function.
options : [optional] This is an object which describes the options property to be passed to the getCurrentPosition method. You can read more about this here. Our options parameter is the options object.
Before writing our displayLocation function, let’s handle the showError function and options object:

```
// Displays the different error messages
showError = (error) => {
  mapArea.style.display = "block"
  switch (error.code) {
    case error.PERMISSION_DENIED:
      mapArea.innerHTML = "You denied the request for your location."
      break;
    case error.POSITION_UNAVAILABLE:
      mapArea.innerHTML = "Your Location information is unavailable."
      break;
    case error.TIMEOUT:
      mapArea.innerHTML = "Your request timed out. Please try again"
      break;
    case error.UNKNOWN_ERROR:
      mapArea.innerHTML = "An unknown error occurred please try again after some time."
      break;
  }
}
//Makes sure location accuracy is high
const options = {
  enableHighAccuracy: true
}
```

Now, let’s write the code for our displayLocation function inside our main.js file:

```
displayLocation = (position) => {
  const lat = position.coords.latitude;
  const lng = position.coords.longitude;
}
```

We now have our user’s latitude and longitude and we can view them in the console by writing the code below inside displayLocation:

```
console.log( `Current Latitude is ${lat} and your longitude is ${lng}` );
```

Step 3: Show the user’s current location on a Google Map
To do this, we will be adding these lines of code to our displayLocation function.

```
const latlng = {lat, lng}
showMap(latlng);
createMarker(latlng);
mapArea.style.display = "block";
```

The first line takes our lat and lng values and encapsulates it in the latlng object literal. This makes it easy for us to use in our app.

The second line of code calls a showMap function which accepts a latlng argument. In here, we get to instantiate our Google map and render it in our UI.

The third line invokes a createMarker function which also accepts our object literal (latlng) as its argument and uses it to create a Google Maps Marker for us.

The fourth line makes the mapArea visible so that our user can now see the location.

```
displayLocation = (position) => {
  const lat = position.coords.latitude;
  const lng = position.coords.longitude;
  const latlng = { lat, lng }
  showMap(latlng);
  createMarker(latlng);
  mapArea.style.display = "block";
}
```

Now, let’s get to creating our functions. We will start with the showMap function.

```
showMap = (latlng) => {
  let mapOptions = {
    center: latlng,
    zoom: 17
  };
  Gmap = new google.maps.Map(mapArea, mapOptions);
}
```

The showMap function creates a mapOptions objects that contain the map center (which is the lat and lng coordinates we got from displayLocation) and the zoom level of the map. Finally, we create an instance of the Google Maps class and pass it on to our map. In fewer words, we instantiate the Google Maps class.

To create a map instance, we specify two parameters in the constructor: the div the map will be displayed and the mapOptions. In our case, our div is called mapArea and our mapOptions is called mapOptions. After this, our created map will show up, but without a marker. We need a marker so the user can identify their current position on the map.

Let’s create our marker using the createMarker function:

```
createMarker = (latlng) => {
  let markerOptions = {
    position: latlng,
    map: Gmap,
    animation: google.maps.Animation.BOUNCE,
    clickable: true
  };
  Gmarker = new google.maps.Marker(markerOptions);
}
```

A few things to note in this code:

The position property positions the marker at the specified latlng
The map property specifies the map instance where the marker should be rendered (in our case, it’s Gmap)
The animation property adds a little BOUNCE to our marker
The clickable property set to true means our marker can be clicked
Finally, we instantiate the Marker class in our Gmarker instance variable
So far, our user’s location has been fetched, the map has rendered and the user can see their current location on the map. Things are looking good! 🕺


Step 4: Pass the latitude and longitude to the Geocode API
Google’s Geocoding API will be used to convert our user’s numeric geographical coordinates to a formatted, human-readable address using the reverse geocoding process we covered earlier.

The URL takes this form:

https://maps.googleapis.com/maps/api/geocode/outputFormat?parameters
…where the outputFormat may either be a json or xml which determines the the format used to deliver the data. The parameters part is a list of parameters needed for the request.

Our request URL will look like this:

https://maps.googleapis.com/maps/api/geocode/json?latlng=${latlng}&key=${__KEY}
Let’s go ahead and connect to the API. We would do this in a function called getGeolocation.

```
getGeolocation = (lat, lng) => {
  const latlng = lat + "," + lng;
  fetch( `https://maps.googleapis.com/maps/api/geocode/json?latlng=${latlng}&key=${__KEY}` )
    .then(res => res.json())
    .then(data => console.log(data.results));
}
```

The getGeolocation function takes two arguments ( lat and lng } concatenates them to form a new latlng variable that is passed to the URL.

Using the Fetch API (more on this here), we add the new latlng and __KEY into the Geocode request URL. Then, on the response object we get back, we pass the .json method to resolve the promise with JSON. Finally, we log the response in our console.

To make use of our newly created function, we have to call it in the displayLocation function. So let’s update our displayLocation function to contain the getGeolocation function call:

```
displayLocation = (position) => {
  const lat = position.coords.latitude;
  const lng = position.coords.longitude;
  const latlng = { lat, lng }
  showMap(latlng);
  createMarker(latlng);
  mapArea.style.display = "block";
  getGeolocation(lat, lng)// our new function call
}
```

The returned data should look something like this:

```
{
  "results" : 
    {
      "address_components": 
        {
          "long_name": "1600",
          "short_name": "1600",
          "types": ["street_number"]
        },
        {
          "long_name": "Amphitheatre Pkwy",
          "short_name": "Amphitheatre Pkwy",
          "types": ["route"]
        },
        {
          "long_name": "Mountain View",
          "short_name": "Mountain View",
          "types": ["locality", "political"]
        },
        {
          "long_name": "Santa Clara County",
          "short_name": "Santa Clara County",
          "types": ["administrative_area_level_2", "political"]
        },
        {
          "long_name": "California",
          "short_name": "CA",
          "types": ["administrative_area_level_1", "political"]
        },
        {
          "long_name": "United States",
          "short_name": "US",
          "types": ["country", "political"]
        },
        {
          "long_name": "94043",
          "short_name": "94043",
          "types": ["postal_code"]
        }
      ],
      "formatted_address": "1600 Amphitheatre Parkway, Mountain View, CA 94043, USA",
      "geometry": {
        "location": {
          "lat": 37.4224764,
          "lng": -122.0842499
        },
        "location_type": "ROOFTOP",
        "viewport": {
          "northeast": {
            "lat": 37.4238253802915,
            "lng": -122.0829009197085
          },
          "southwest": {
            "lat": 37.4211274197085,
            "lng": -122.0855988802915
          }
        }
      },
      "place_id": "ChIJ2eUgeAK6j4ARbn5u_wAGqWA",
      "types": ["street_address"]
    }
  ],
  "status" : "OK"
}
```

Step 5: Display the returned address(es) for the user to select
At this stage, we have made a request to Google’s Geocoding API and have gotten our result logged in the console. Now, we have to display the results in a UI for our user. This requires two things:

Create a new function that handles creating HTML elements
Update our getGeolocation function to make the function call
Let’s create the function that would take care of creating the HTML elements and updating the DOM.

```
populateCard = (geoResults) => {
  geoResults.map(geoResult => {
    // first create the input div container
    const addressCard = document.createElement('div');
    // then create the input and label elements
    const input = document.createElement('input');
    const label = document.createElement('label');
    // then add materialize classes to the div and input
    addressCard.classList.add("card");
    input.classList.add("with-gap");
    // add attributes to them
    label.setAttribute("for", geoResult.place_id);
    label.innerHTML = geoResult.formatted_address;
    input.setAttribute("name", "address");
    input.setAttribute("type", "radio");
    input.setAttribute("value", geoResult.formatted_address);
    input.setAttribute("id", geoResult.place_id);
    addressCard.appendChild(input);
    addressCard.appendChild(label)
    return (
      // append the created div to the locationsAvailable div
      locationsAvailable.appendChild(addressCard)
    );
  })
}
```

In this function, we iterate through our results and create some HTML elements (div , input and a label), append the input and the label to the div and finally append the new div to a parent div (which is locationsAvailable). Once we get the result from our API call, our DOM will be created and displayed to the user.

Next, we update our getGeolocation function to call our populateCard function by replacing the last line of getGeolocation with this:

.then(data => populateCard(data.results));
…which means our updated function should look this:

```
getGeolocation = (lat, lng) => {
  const latlng = lat + "," + lng;
  fetch( `https://maps.googleapis.com/maps/api/geocode/json?latlng=${latlng}&key=${__KEY}` )
    .then(res => res.json())
    .then(data => populateCard(data.results));
}
```

At this point, everything should be working fine. Our user clicks a button, gets a location displayed on the map along with a list of addresses that match the current location.


Step 6: Listen for map events and repeat steps 4 and 5
If our user decides to move the map or marker, nothing happens to the UI — new addresses aren’t not displayed and everything remains static. We’ve got to fix this, so let’s make our app dynamic by listening for map events. You can read all about Google Map Events here.

There are three events we want to listen for:

drag: This is fired once the user starts dragging and continues to drag the map
dragend: This is fired once the user stops dragging the map
idle: This is fired once every event has been fired and the map is idle
Quick Question: Why are these events best suited for our app?

Quick Answer: The first two events will make sure that our map marker stays in the center of the map during the drag event while the idle event will make a geocoding request with the new coordinates.

To listen for these events we have to update the showMap function with the following:

```
Gmap.addListener('drag', function () {
  Gmarker.setPosition(this.getCenter()); // set marker position to map center
});
Gmap.addListener('dragend', function () {
  Gmarker.setPosition(this.getCenter()); // set marker position to map center
});
Gmap.addListener('idle', function () {
  Gmarker.setPosition(this.getCenter()); // set marker position to map center
  if (Gmarker.getPosition().lat() !== lat || Gmarker.getPosition().lng() !== lng) {
    setTimeout(() => {
      updatePosition(this.getCenter().lat(), this.getCenter().lng()); // update position display
    }, 2000);
  }
});
```

As explained above, the first two event listeners simply ensure that the marker remains in the center of our map. Pay closer attention to the idle event listener because that is where the action is.

Once the idle event is fired, the marker goes to the center, then a check is done to find out if the current position of the marker is the same with the lat or lng values received by the displayLocation function. If it is not the same, then we call the updatePosition function after two seconds of idleness.

Having said that, we have to make a few updates to the showMap function. First, on the function header, we have to include more parameters and on the showMap function call. We need to add the new arguments there, too. Our showMap function should look like this:

```
showMap = (latlng, lat, lng) => {
  let mapOptions = {
    center: latlng,
    zoom: 17
  };
  Gmap = new google.maps.Map(mapArea, mapOptions);
  Gmap.addListener('drag', function () {
    Gmarker.setPosition(this.getCenter()); // set marker position to map center
  });
  Gmap.addListener('dragend', function () {
    Gmarker.setPosition(this.getCenter()); // set marker position to map center
  });
  Gmap.addListener('idle', function () {
    Gmarker.setPosition(this.getCenter()); // set marker position to map center
    if (Gmarker.getPosition().lat() !== lat || Gmarker.getPosition().lng() !== lng) {
      setTimeout(() => {
        updatePosition(this.getCenter().lat(), this.getCenter().lng()); // update position display
      }, 2000);
    }
  });
}
```

And our displayLocation function should look like this:

```
displayLocation = (position) => {
  const lat = position.coords.latitude;
  const lng = position.coords.longitude;
  const latlng = { lat, lng }
  showMap(latlng, lat, lng); //passed lat and lng as the new arguments to the function
  createMarker(latlng);
  mapArea.style.display = "block";
  getGeolocation(lat, lng);
}
```

Having listened for the map events, let’s repeat Step 4 and Step 5.

We start by writing our updatePosition function. This function will perform only one action for now, which is to pass the new lat and lng values to the getGeolocation function:

```
updatePosition = (lat, lng) => {
  getGeolocation(lat, lng);
}
After getting the new position and fetching addresses, our DOM should re-render for the user, right? Well, it doesn’t. And to fix that, we create a function that will force the DOM to re-render:

// check if the container has a child node to force re-render of dom
function removeAddressCards(){
  if (locationsAvailable.hasChildNodes()) {
    while (locationsAvailable.firstChild) {
      locationsAvailable.removeChild(locationsAvailable.firstChild);
    }
  }
}
```
It checks if the locationsAvailable div has any childNodes, and if it does, it deletes them before creating new address cards. The populateCard function is now updated to this:

```
populateCard = (geoResults) => {
  // check if a the container has a child node to force re-render of dom
  removeAddressCards();
  
  geoResults.map(geoResult => {
    // first create the input div container
    const addressCard = document.createElement('div');
    // then create the input and label elements
    const input = document.createElement('input');
    const label = document.createElement('label');
    // then add materialize classes to the div and input
    addressCard.classList.add("card");
    input.classList.add("with-gap");
    // add attributes to them
    label.setAttribute("for", geoResult.place_id);
    label.innerHTML = geoResult.formatted_address;
    input.setAttribute("name", "address");
    input.setAttribute("type", "radio");
    input.setAttribute("value", geoResult.formatted_address);
    input.setAttribute("id", geoResult.place_id);
    addressCard.appendChild(input);
    addressCard.appendChild(label);
    return (
      locationsAvailable.appendChild(addressCard);
    );
  })
}
```

We are done and are now able to fully get and display the user’s address!

Step 7: Pre-fill the form with the address data the user selects
The final step is to fill the form with the address the user selects. We need to add a click event listener to the address card and pass the address as argument to the callback function.

Here’s how we add the event listener in the populateCard function:

```
input.addEventListener('click', () => inputClicked(geoResult));
You should note that the geoResult argument in the above callback is the selected address object from the results array. That said, update the populateCard function to accommodate our new line of code.

The inputClicked function uses a series of if statements to assign values to our form elements. so before working on it, let’s bring our form elements into the equation:

const inputAddress = document.getElementById('address'),
  inputLocality = document.getElementById('locality'),
  inputPostalCode = document.getElementById('postal_code'),
  inputLandmark = document.getElementById('landmark'),
  inputCity = document.getElementById('city'),
  inputState = document.getElementById('state');
  
//Having done this, let us now work on pre-filling the form with the address_components in the inputClicked function.

inputClicked = result => {
  result.address_components.map(component => {
    const types = component.types
    if (types.includes('postal_code')) {
      inputPostalCode.value = component.long_name
    }
    if (types.includes('locality')) {
      inputLocality.value = component.long_name
    }
    if (types.includes('administrative_area_level_2')) {
      inputCity.value = component.long_name
    }
    if (types.includes('administrative_area_level_1')) {
      inputState.value = component.long_name
    }
    if (types.includes('point_of_interest')) {
      inputLandmark.value = component.long_name
    }
  });
  inputAddress.value = result.formatted_address;
  // to avoid labels overlapping pre-filled input contents
  M.updateTextFields();
  // removes the address cards from the UI
  removeAddressCards();
}
```

The above block of code iterates over the clicked (or selected) address component, checks the types of components and finally assigns them to the input fields if they match.


M.updateTextFields() function is from Materialize and it ensures that the label does not overlap with the input fields values and the removeAddressCards() function removes the address cards from the UI.

With this, we are done with our app and have saved our users lots of typing and headaches! Surely they will thank us for implementing such a hassle free solution.

This whole UX experiment can be seen as a shortcut that will help the user pre-fill the shipping address form. We should clearly state here that the returned addresses isn’t always 100% accurate. But that’s why we allow the address to be manually edited in the UI.
