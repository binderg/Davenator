<script>
  import Flatpickr from "svelte-flatpickr";
  import { startEndParsha, location } from "./store.js";
  let lat = 40.3;
  let long = 42.3;
  // let tzone = $location.tzone;
  let tzone = "America/New_York";
  export let isStart = true;
  let parsha = isStart ? $startEndParsha.start.parsha : $startEndParsha.end.parsha;
  let date = isStart ? $startEndParsha.start.date : $startEndParsha.end.date;


  async function parshiyos() {

    const res = await fetch(
      `https://www.hebcal.com/hebcal?v=1&cfg=json&s=on&leyning=off&latitude=${lat}&longitude=${long}&tzid=${tzone}&start=${date}&end=${date}`
    );

    if (res.ok) {
      return res.json();
    } else {
      console.log("error");
    }
  }

  const options = {
    element: "#my-picker",
    monthSelectorType:"static",
    enableTime: false,
    disable: [
      function (date) {
        // return true to disable
        return date.getDay() != 6;
      },
    ],

    onChange(selectedDates, dateStr) {
      console.log("flatpickr hook", selectedDates, dateStr);
    },
  };

  function startHandleChange(event) {
    const [selectedDates, dateStr] = event.detail;
    date = dateStr;
    console.log({ selectedDates, dateStr });
    console.log(parsha);
    parshiyos(date).then((res) => {
      parsha = "Parshas " + res.items[0].title.substr(9);
      if(isStart) {
        $startEndParsha.start.parsha = parsha;
        $startEndParsha.start.date = dateStr;
      } else{
        $startEndParsha.end.parsha = parsha;
        $startEndParsha.end.date = dateStr;
      }
    });
  }
</script>

<main>
  {isStart?
  ($startEndParsha.start.date==""?`Select a start date`:parsha):($startEndParsha.end.date==""?`Select a start date`:parsha)
  }
  <Flatpickr
    {options}
    class="flatpickr"
    on:change={startHandleChange}
    name="date" placeholder={isStart?"Choose start date":"Choose end date"}
  />

</main>
