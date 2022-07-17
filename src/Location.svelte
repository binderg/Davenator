<script>
  import { location } from "./store.js";
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
      //   let lat = $('#lat').val(place.geometry['location'].lat());
      //   let long =$('#long').val(place.geometry['location'].lng());
      //let location = getGeoInfo(place.place_id);
      getGeoInfo(place.place_id).then((geoInfo) => {
        $location = geoInfo.results[0].geometry.location; // fetched geoInfo (lat and long)
      });
    });
  }

  async function getGeoInfo(place_id) {
    const res = await fetch(
      `https://maps.googleapis.com/maps/api/geocode/json?place_id=${place_id}&key=AIzaSyBvQBjBzOIihWhd9_dUPqhhlseMCJAeCIc&libraries=places`
    );

    if (res.ok) {
      return res.json();
    }
  }
</script>

<main>
  <div class="container">
    <div class="row">
      <div class="col-12 text-center text-3xl	"><h2>Davenator</h2></div>
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
            <input type="hidden" id="lat" name="lat" />
            <input type="hidden" id="long" name="long" />
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

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
