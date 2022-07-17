<!-- <link rel="stylesheet"
                  href=
              "https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
                <style>
                  .input-icons i {
                    position: absolute;
                  }
                  
                  .input-icons {
                    width: 100%;
                    margin-bottom: 10px;
                  }
                  
                  .icon {
                    padding: 10px;
                    min-width: 40px;
                  }
                  
                  .input-field {
                    width: 100%;
                    padding: 10px;
                    text-align: center;
                  }
                </style> -->

<script>
  import { location } from "./store.js";
  import dotenv from "dotenv"
  dotenv.config() // inject the content of the .env file into 'process.env'
  
  google.maps.event.addDomListener(window, "load", initialize);
  function initialize() {
    var input = document.getElementById("autocomplete_search");
    var autocomplete = new google.maps.places.Autocomplete(input);
    autocomplete.addListener("place_changed", function () {
      var place = autocomplete.getPlace();
      // place variable will have all the information you are looking for.
      let lat = document
        .getElementById("lat")
        .setAttribute("value", place.geometry["location"].lat());
      let long = document
        .getElementById("long")
        .setAttribute("value", place.geometry["location"].lng());

      
      getGeoInfo(place.place_id).then((geoInfo) => {
        $location = geoInfo.results[0].geometry.location; // fetched geoInfo (lat and long)
      });
    });
  }

  async function getGeoInfo(place_id) {
    const res = await fetch(
      `https://maps.googleapis.com/maps/api/geocode/json?place_id=${place_id}&key=${process.env.GOOGLE_MAPS_API_KEY}&libraries=places`
    );

    if (res.ok) {
      return res.json();
    }
  }

async function getPlaceId(lat, lon) {
  const res = await fetch(
    `https://maps.googleapis.com/maps/api/geocode/json?latlng=40.714224,-73.961452&key=AIzaSyBnWc0u7o1Ey_P99m31Sf6Ucj3J7BGvr5U`);

    if (res.ok) {
      return res.json();
    }
}

function initUserLocation() {
    // Try HTML5 geolocation.
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          const pos = {
            lat: position.coords.latitude,
            lng: position.coords.longitude,
          };

          console.log("lat " +  pos.lat + " lon " + pos.lng)    
          $location = pos
          
        },
        () => {
          handleLocationError(true);
        }
      );
    } else {
      // Browser doesn't support Geolocation
      handleLocationError(false);
    }
}

function handleLocationError(
  browserHasGeolocation
) {
  const message = browserHasGeolocation
      ? "Error: The Geolocation service failed."
      : "Error: Your browser doesn't support geolocation."
  alert(message);
  
}

document.addEventListener("DOMContentLoaded", initUserLocation());

</script>

  <div class="container">
    <div class="row">
      <div class="col-12 text-center text-3xl	">
        <h2>Davenator</h2>
      </div>
      <div class="col-12">
        <div id="custom-search-input">
          <div class="input-group">
            <input
              id="autocomplete_search"
              name="autocomplete_search"
              type="text"
              class="form-control"
              placeholder="Location"
            />
              <i class="material-symbols-outlined">
                my_location
              </i>

              
             
                
              
              <body>
                <h3>
                Icons inside the input element
              </h3>
                <div style="max-width:400px;margin:auto">
                  <div class="input-icons">
                    <i class="fa fa-user icon"></i>
                    <input class="input-field" type="text">
                    <i class="fa fa-instagram icon"></i>
                    <input class="input-field" type="text">
                    <i class="fa fa-envelope icon"></i>
                    <input class="input-field" type="text">
                    <i class="fa fa-youtube icon"></i>
                    <input class="input-field" type="text">
                    <i class="fa fa-facebook icon"></i>
                    <input class="input-field" type="text">
                  </div>
              </div>
              
              
          </div>
        </div>
      </div>
    </div>
  </div>


<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
