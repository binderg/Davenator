<script>
  // import { onMount } from 'svelte';
  // import M from 'materialize-css'
  import ParshaPicker from "./ParshaPicker.svelte";
  import Location from "./Location.svelte";
  import Rule from "./Rule.svelte";
  import { startEndParsha, location, columns, idIncrement } from "./store.js";
  import moment from "moment";

  /*------------data-------------*/
  let parshiyosJSON = {};
  $columns = [];
  $startEndParsha = {
    start: { date: "", parsha: "" },
    end: { date: "", parsha: "" },
  };
  $location = {
    address: "",
    lat: "40.13",
    long: "55.14",
    tzone: "America/New_York",
  };
  idIncrement.set(0); // this is a crude way to increment the id for new items

  function addItem(type) {
    var l = $columns.length; // get our current items list count
    $columns[l] = {
      id: $idIncrement,
      name: "",
      minutes: null,
      beforeAfter: "before",
      time: "",
      text: "",
    };
    if (type == "rule") {
      $columns[l].type = "rule";
    } else {
      $columns[l].type = "textbox";
    }
    $idIncrement++; // increment our id to add additional items
  }
  //------------------------------
  let promise = Promise.resolve([]);

  //get zmanim for set dates
  async function getParshiyosList() {
    let startDate = $startEndParsha.start.date;
    let endDate = $startEndParsha.end.date;
    let lat = $location.lat;
    let long = $location.long;
    let tzone = $location.tzone;
    const res = await fetch(
      //tzone is hardcoded
      `https://www.hebcal.com/hebcal?v=1&cfg=json&s=on&leyning=off&latitude=${lat}&longitude=${long}&tzid=America/New_York&start=${startDate}&end=${endDate}`
    );

    if (res.ok) {
      return await res.json();
    } else {
      throw new Error(404);
    }
  }

  //------------XLS-------------

  function save() {
    if (requiredFields()) {
      createFinalJSON();
    }
  }

  function requiredFields() {
    if ($startEndParsha.start.parsha == "") {
      alert("Please select a start Parsha");
      return false;
    } else if ($startEndParsha.end.parsha == "") {
      alert("Please select an end Parsha");
      return false;
    } else if ($columns.length < 1) {
      alert("Please add a column");
      return false;
    } else if (
      moment($startEndParsha.end.date).isBefore(
        moment($startEndParsha.start.date)
      )
    ) {
      alert("Please choose an end date that is after the start date");
    } else if (
      $location && // 👈 null and undefined check
      Object.keys($location).length === 0 &&
      Object.getPrototypeOf($location) === Object.prototype
    ) {
      alert("Please select a location");
    } else {
      return true;
    }
  }

  //get zmanim for set dates
  async function getZmanim(zmanimURL) {
    const res = await fetch(zmanimURL);

    if (res.ok) {
      return await res.json();
    } else {
      throw new Error(404);
    }
  }

  function datestringFormatter(string){
    let date = moment(string);         
    return `${date.format("M")}/${date.format("D")}/${date.format("YYYY")}`
          

  }

  function createFinalJSON() {
    /*-----------get Zmanim info-------------*/
    //get start and end dates
    let startDate = $startEndParsha.start.date;
    let endDate = $startEndParsha.end.date;
    let result = {};
    let lat = $location.lat;
    let long = $location.long;
    //let tzone = $location.tzone;
    let tzone = "america/new_york";
    let zmanimURL = `https://www.hebcal.com/zmanim?cfg=json&start=${startDate}&end=${endDate}&latitude=${lat}&longitude=${long}&tzid=${tzone}`;
    ////////////////////////////////////////////////////
    let parshiyosObj = { parshiyos: [], dates: [] };
    getParshiyosList()
      .then((res) => {
        for (const element of res.items) {
          if (element.title.includes("Parashat")){
            parshiyosObj.parshiyos.push(element.title.substr(9));
          parshiyosObj.dates.push(datestringFormatter(element.date));
        }
        }
        //console.log("createParshiyosObj " + JSON.stringify(parshiyosObj))
      })
      .then((r) => {
        getZmanim(zmanimURL).then((res) => {
          /*-----------Populate result with Parshiyos and Dates-------------*/
          //let parshiyosObj = createParshiyosObj(); // parshiyosObj has {parshiyos:[], dates:[]};
          result.Dates = parshiyosObj.dates; //Push Dates
          result.Parshiyos = parshiyosObj.parshiyos; //Push Parsha
          //console.log("parsh push " +  result.Parshiyos)
          /*-----------Populate result with Times-------------*/
          //console.log("sp " + $startEndParsha.start.parsha)
          $columns.forEach((element) => {
            // element has {id:null, type:"rule", name:"", minutes:"", beforeAfter:"", time:"", text:"" }
            result[element.name] = [];
            //--------------if textbox - for amount of parshiyos in parsha column, add to textcolumn
            if (element.type == "textbox") {
              for (const parsha of parshiyosObj.parshiyos) {
                //console.log("includes " + parsha)
                result[element.name].push(element.text);
              }
            }

            //--------------if rule - for dates in given zman, if date is in the list of Saturday dates, add time (with math)
            else if (element.type == "rule") {
              console.log("po dates: " + JSON.stringify(parshiyosObj.dates))
              for (const dateKey in res.times[element.time]) {
                console.log("date " + res.times[element.time][dateKey])
                if ( // if date is one of the saturday day we want
                  parshiyosObj.dates.includes(datestringFormatter(res.times[element.time][dateKey]))
                ) {
                  console.log("in")
                  result[element.name].push(
                    timeMath(res.times[element.time][dateKey], element.minutes, element.beforeAfter)
                  );
                }
              }
            }
          });
          /*-----------Create xlsx-------------*/
          createXLSXfromResult(result);
        });
      });
  }

  function createXLSXfromResult(object) {
    var wb = XLSX.utils.book_new();
    wb.Props = {
      Title: "Zmanim Table",
      Subject: "Zmanim",
      Author: "ZmanTime",
      CreatedDate: new Date(),
    };

    const finalMatrix =resultToArrays(object);
    
    console.log(finalMatrix);
    const worksheet = XLSX.utils.aoa_to_sheet(finalMatrix);
    //format column width
    //const max_width = finalMatrix.reduce((w, r) => Math.max(w, r[0].length), 10);
    //worksheet["!cols"] = [ { wch: max_width } ];
    XLSX.utils.book_append_sheet(wb, worksheet, "Dates");

    XLSX.writeFile(wb, "Zmanim Table.xlsx");
  }

  function timeMath(dateString, minutes, beforeAfter) {
    let date = moment(dateString);
    if (beforeAfter == "before") {
      date = moment(date).subtract(minutes, "minutes");
    } else {
      date = moment(date).add(minutes, "minutes");
    }
    return moment(date).format("h:mm a");
  }

  function resultToArrays(object) {
    let excelArrays = [];
    Object.keys(object).forEach((element) => {
      //   let tempArray = [];
      //   tempArray.push(element);
      //   console.log("te " + object[element])
      //   tempArray.concat(object[element]);
      excelArrays.push([element, ...object[element]]);
    });
    // console.log("result " + excelArrays[0].map((_, colIndex) =>
    //  excelArrays.map((row) => row[colIndex])))
    return excelArrays[0].map((_, colIndex) =>
      excelArrays.map((row) => row[colIndex])
    );
  }
</script>

<main>
  <div class="row">
    <div class="col s6">
      <img
        style="margin-right:-150px; margin-top:25px"
        src="assets/ZmanTime_500.png"
        height="80"
      />
    </div>
    <div class="col s6 pull-s3 text-center text-3xl	"><h2>Davenator</h2></div>
  </div>

  <Location />

  <div class="row">
    <div class="col s6">
      <ParshaPicker isStart={true} />
    </div>
    <div class="col s6">
      <ParshaPicker isStart={false} />
    </div>
  </div>

  <a on:click={() => addItem("rule")} class="waves-effect waves-light btn"
    ><i class="material-icons left">code</i>Add Rule Column</a
  >
  <a on:click={() => addItem("textbox")} class="waves-effect waves-light btn"
    ><i class="material-icons left">text_fields</i>Add Text Column</a
  >
  <a on:click={save} class="waves-effect waves-light btn"
    ><i class="material-icons left">cloud</i>Save</a
  >
  {#each $columns as item}
    <svelte:component this={Rule} objAttributes={item} />
  {/each}
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
