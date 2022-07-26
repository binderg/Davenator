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
  let previewBool = false;
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
      createFinalJSON("xlsx");
    }
  }

  function preview() {
    previewBool = false;
    if (requiredFields()) {
      previewBool = true;
      createFinalJSON("preview");
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

  function datestringFormatter(string) {
    let date = moment(string);
    return `${date.format("M")}/${date.format("D")}/${date.format("YYYY")}`;
  }

  function createFinalJSON(state) {
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
          if (element.title.includes("Parashat")) {
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
              console.log("po dates: " + JSON.stringify(parshiyosObj.dates));
              for (const dateKey in res.times[element.time]) {
                console.log("date " + res.times[element.time][dateKey]);
                if (
                  // if date is one of the saturday day we want
                  parshiyosObj.dates.includes(
                    datestringFormatter(res.times[element.time][dateKey])
                  )
                ) {
                  console.log("in");
                  result[element.name].push(
                    timeMath(
                      res.times[element.time][dateKey],
                      element.minutes,
                      element.beforeAfter
                    )
                  );
                }
              }
            }
          });

          /*-----------check state-------------*/
          if (state == "xlsx") {
            createXLSXfromResult(result);
          } else if (state == "preview") {
            createTable(resultToArrays(result));
          }
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

    const finalMatrix = resultToArrays(object);

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

  function createTable(tableData) {
    var modal = document.getElementById("popupContent");
    modal.innerHTML="";
    var table = document.createElement("table");
    var tableBody = document.createElement("tbody");
    tableData.forEach(function (rowData) {
      var row = document.createElement("tr");

      rowData.forEach(function (cellData) {
        var cell = document.createElement("td");
        cell.appendChild(document.createTextNode(cellData));
        row.appendChild(cell);
      });

      tableBody.appendChild(row);
    });

    table.appendChild(tableBody);
    modal.appendChild(table);

  }
</script>

<main>
  <div class="row">
    <img
      style="float: left text-align: center;"
      src="assets/ZmanTime_Banner.png"
      width="400"
    />
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
  <div class="box">
    <a on:click={preview} class="waves-effect waves-light btn ibutton" href={previewBool? "#popupPreview" : "#"}
    ><i class="material-icons left">preview</i>Preview</a
  >

</div>

<div style="z-index:2147483647"  id="popupPreview" class="overlay">
    <div class="popup">
        <h5>Preview</h5>
        <a class="close" href="#"><span style="font-size:30px">&times;</span></a>
        <div class="content"				>
            <p style = "height: 30vw; overflow:auto;" id="popupContent">
               
            </p>
        </div>
    </div>
</div>

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

  .ibutton{
	color:white
}


 .box {
  margin: 10px 200px 0px 0px;
  /* width: 40%;
  margin: 0 auto;
  background: rgba(255,255,255,0.2);
  padding: 35px;
  border: 2px solid #fff;
  border-radius: 20px/50px;
  background-clip: padding-box;
  text-align: center; */
} 


.overlay {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.7);
  transition: opacity 500ms;
  visibility: hidden;
  opacity: 0;
}
.overlay:target {
  visibility: visible;
  opacity: 1;
}

.popup {
  margin: 10px auto;
  padding: 20px;
  background: #fff;
  border-radius: 5px;
  width: 80%;
  position: relative;
  transition: all 5s ease-in-out;
  padding: 10px 30px 10px 30px
}

.popup .close {
  position: absolute;
  top: 20px;
  right: 30px;
  transition: all 200ms;
  font-size: 12px;
  font-weight: bold;
  text-decoration: none;
  color: #333;
}
.popup .close:hover {
  color: #06D85F;
}
.popup .content {
  max-height: 30%;
}
</style>
