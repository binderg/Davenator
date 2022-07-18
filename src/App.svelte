<script>
  import { onMount } from 'svelte';
  // import M from 'materialize-css'
  import ParshaPicker from './ParshaPicker.svelte'
  import Location from "./Location.svelte";
  import Rule from "./Rule.svelte";
  import { startEndParsha, location, columns, idIncrement } from "./store.js";
  import moment from "moment";

  /*------------data-------------*/
  let parshiyosJSON = {};
  $columns = [];
  $startEndParsha = {start: {date:"", parsha:""}, end: {date:"", parsha:""}}
  $location = {lat:"40.13", long:"55.14", tzone:"America/New_York"}
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

  let parshiyosObj = {};

  async function parshiyosList() {
    let parshiyosObj ={parshiyos:[], dates:[]};
    const res = await fetch(`https://www.hebcal.com/hebcal?v=1&cfg=json&s=on&leyning=off&latitude=${lat}&longitude=${long}&tzid=${tzone}&start=${startEndParsha.start.date}&end=${startEndParsha.end.date}`);

    parshiyosJSON = await res.json();

    if (res.ok) {

      for(const element of parshiyosJSON.items){
        if(element.title.includes("Parashat"))
        parshiyosList.parshiyos.push(element.title.substr(9));
        let date = moment(element.date).format("M-DD-YYYY");
        parshiyosList.dates.push(`${date.format('M')}/${date.format('D')}/${date.format('YYYY')}`);
      }
      return parshiyosObj;
    } else {
      console.log("API error")
    }
  }
  // promise = parshiyos();

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
    } else if (endParsha == null) {
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
      alert("Please choose an end parsha that is after the start parsha");
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

  function createFinalJSON() {
    /*-----------get Zmanim info-------------*/
    //get start and end dates
    let startDate = $startEndParsha.start.date;
    let endDate = $startEndParsha.end.date;
    let result = {};
    let zmanim = {};
    let lat = $location.lat;
    let long = $location.long;
    let tzone = $location.tzone;
    let zmanimURL = `https://www.hebcal.com/zmanim?cfg=json&start=${startDate}&end=${endDate}&latitude=${lat}&longitude=${long}&tzid=${tzone}`;

    //get zmanim for set dates
    async function getZmanim() {
      const res = await fetch(zmanimURL);

      if (res.ok) {
        zmanim = await res.json();
        result.Parshiyos = [];
        result.Dates = [];

        /*-----------Populate result with Parshiyos and Dates-------------*/
        let parshiyosObj = parshiyosList();
        let parshiyosObjKeysArray = Object.keys(parshiyosObj);
        let startParshaIndex = parshiyosObjKeysArray.indexOf(startParsha);
        let endParshaIndex = parshiyosObjKeysArray.indexOf(endParsha);
        for (let index = startParshaIndex; index <= endParshaIndex; index++) {
          const parshiyosObjKey = parshiyosObjKeysArray[index]; //Parsha
        }
        result.Dates.push(parshiyosObj.dates); //Dates
        result.Parshiyos.push(parshiyosObj.parshiyos); //Parsha
        
        /*-----------Populate result with Times-------------*/
        $columns.forEach((element) => {
          result[element.name] = [];
          //--------------if textbox - for amount of parshiyos in parsha column, add to textcolumn
          if (element.type != "rule") {
            for (
              let index = startParshaIndex;
              index <= endParshaIndex;
              index++
            ) {
              result[element.name].push(element.text);
            }
          }
          // element has {id:null, type:"rule", name:"", minutes:"", beforeAfter:"", time:"", text:"" }
          //get times
          //key - parsha
          if (element.type == "rule") {
            for (const [key, value] of Object.entries(parshiyosObj)) {
              //-------------if rule - do math and add the modified date
              let data = zmanim.times[element.time][value.unformatted]; // data from zmanim object
              if (data != undefined) {
                //add chosen parshiyos and info-------------
                // result.Parshiyos.push(key); //Parsha
                // result.Dates.push(value.formatted); //Dates
                result[element.name].push(
                  timeMath(data, element.minutes, element.beforeAfter)
                ); //Time
              }
            }
          }
        });
        /*-----------Create xlsx-------------*/
        createXLSXfromResult(result);
      } else {
        throw new Error(404);
      }
    }
    //getZmanim();
  }
  function createXLSXfromResult(object) {
    var wb = XLSX.utils.book_new();
    wb.Props = {
      Title: "Zmanim Table",
      Subject: "Zmanim",
      Author: "Davenator",
      CreatedDate: new Date(),
    };
    wb.SheetNames.push("Test Sheet");
    var ws = XLSX.utils.aoa_to_sheet(resultToArrays(object));
    wb.Sheets["Test Sheet"] = ws;
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
    return excelArrays[0].map((_, colIndex) =>
      excelArrays.map((row) => row[colIndex])
    );
  }
</script>
 

<main>  

  <div class="row">
    <div class="col s6" ><img style="margin-right:-150px; margin-top:25px" src="assets/ZmanTime_500.png" height="80" /> </div>
     <div class="col s6 pull-s3 text-center text-3xl	"><h2>Davenator</h2></div>
     </div>

  
  <Location />


  <div class="row">
    <div class="col s6">
        <ParshaPicker isStart={true}/>
    </div>
    <div class="col s6">
      <ParshaPicker isStart={false}/>
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
