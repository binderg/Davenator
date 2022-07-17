<script>
  import { location } from "./store.js";  
  google.maps.event.addDomListener(window, "load", initialize);
  async function initialize() {
    var input = document.getElementById("autocomplete_search");
    var autocomplete = new google.maps.places.Autocomplete(input);
    autocomplete.addListener("place_changed", function () {
      var place = autocomplete.getPlace();
      // place variable will have all the information you are looking for.
      getGeoInfo(place.place_id).then((geoInfo) => {
        $location = {lat: geoInfo.results[0].geometry.location.lat, long: geoInfo.results[0].geometry.location.lng  }// fetched geoInfo (lat and long)
        console.log()
      });
    });
  }

  async function getGeoInfo(place_id) {
    const res = await fetch(
      `https://maps.googleapis.com/maps/api/geocode/json?place_id=${place_id}&key=AIzaSyBnWc0u7o1Ey_P99m31Sf6Ucj3J7BGvr5U&libraries=places`
    );  

    if (res.ok) {
      return res.json();
    }
  }
  
async function initUserLocation() {
    // Try HTML5 geolocation.
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          const pos = {
            lat: position.coords.latitude,
            long: position.coords.longitude,
          };

          console.log("lat " +  pos.lat + " lon " + pos.long)    
          $location = pos
          
        },
        () => {
          handleLocationError(true)
        }
      );
      return true
    } else {
      // Browser doesn't support Geolocation
      handleLocationError(false)
      return false
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

async function getUserLocation(lat, lon) {
  const res = await fetch(
      `https://maps.googleapis.com/maps/api/geocode/json?latlng=${lat},${lon}&key=AIzaSyBnWc0u7o1Ey_P99m31Sf6Ucj3J7BGvr5U`
    );  

    if (res.ok) {
      return res.json();
    }
}

async function onMyLocationClick() {
  if (initUserLocation() && $location.lat !== undefined && $location.long !== undefined ) { 
    //TODO: the first time the location button is clicked, location.lat and location.long are undefined, so api cannot be called. push 
      getUserLocation($location.lat, $location.long).then((geoInfo) => {
        const results = geoInfo.results[4] //just one of the address options. results is an array of same addresses, different formattings
        const address = results.formatted_address
        document.getElementById("autocomplete_search").setAttribute("value", address)
        
      });
    } 
  }

</script>

<main>

  <div class="container">
    <div class="row">
      <div class="col-12 text-center text-3xl	"><h2>Davenator</h2></div>
      <div class="col-12">
        <div id="custom-search-input input-container">
          <div class="input-group">
            <a class="btn-flat" on:click={onMyLocationClick}><i class="material-icons right">my_location</i></a>
            <input
            id="autocomplete_search"
            name="autocomplete_search"
            type="text"
            class="form-control input"
            placeholder="Location"
            />
            
          </div>
        </div>
      </div>
    </div>
  </div>

</main>

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

  .input-container {
    display: -ms-flexbox;
    display:flex;
    width: 100%;
    flex-direction:column;
    position:relative;

  }
  .input-group {
  width: 100%;
  font-size:20px;
  padding: 5px;
  outline: none;
  border:1px solid #000;
  border-left:none;
  color:#000;
}
  .input {
    position: absolute;
    /* width: 75%; */
    /* padding-right: 50px; */
  }
  .btn-flat {
    /* position:absolute; */
    margin-right:30;
    padding: 10px;
  }
  

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
