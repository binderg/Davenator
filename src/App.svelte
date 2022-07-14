<script>
	import Rule from "./Rule.svelte";
	import { columns, idIncrement } from "./store.js";
	import moment from "moment";

	/*------------data-------------*/
	let startParsha = null;
	let endParsha = null;
	$columns = [];

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
		console.log($columns);
		$idIncrement++; // increment our id to add additional items
	}
	//------------------------------
	let promise = Promise.resolve([]);

	let parshiyosObj = {};
	let parshaURL =
		"https://www.hebcal.com/hebcal?v=1&cfg=json&year=now&month=x&s=on&leyning=off";

	async function parshiyos() {
		const res = await fetch(parshaURL);
		let parshiyosJSON = {};
		parshiyosJSON = await res.json();

		if (res.ok) {
			parshiyosJSON.items.forEach((element) => {
				//Remove "Parshat"
				let parsha = element.title.substr(9)
				//Add unformatted date
				parshiyosObj[parsha] = {};
				parshiyosObj[parsha].unformatted = element.date;
				//Reformat date
				const dateArray = element.date.split("-");
				//add to parshiyos array
				const date = `${dateArray[1]}/${dateArray[2]}/${dateArray[0]}`;
				parshiyosObj[parsha].formatted = date;
			});
		} else {
			throw new Error(users);
		}
	}
	promise = parshiyos();

	//------------XLS-------------

	function save() {
		if (requiredFields()) {
			createXLS();
		}
	}

	function requiredFields() {

		if (startParsha == null) {
			alert("Please select a start Parsha");
			return false;
		} else if (endParsha == null) {
			alert("Please select an end Parsha");
			return false;
		} else if ($columns.length < 1) {
			alert("Please add a column");
			return false;
		} else if (moment(parshiyosObj[endParsha].unformatted).isBefore(parshiyosObj[startParsha].unformatted)) {
			alert("Please choose an end parsha that is after the start parsha")
		} else {
			return true;
		}
	}

	function createXLS() {
		/*-----------get Zmanim info-------------*/
		//get start and end dates
		let startDate = parshiyosObj[startParsha].unformatted;
		let endDate = parshiyosObj[endParsha].unformatted;
		let zmanimURL = "";
		let zmanim = {};
		zmanimURL = `https://www.hebcal.com/zmanim?cfg=json&geonameid=3448439&start=${startDate}&end=${endDate}`;
		//get zmanim for set dates
		async function getZmanim() {
			const res = await fetch(zmanimURL);

			if (res.ok) {
				zmanim = await res.json();
				let result = {};
				result.Parshiyos = [];
				result.Dates = [];
				/*-----------Populate result with Times-------------*/
				$columns.forEach((element) => {
					result[element.name] = [];
					//-------------if rule---------------
					//get times
					let bool = false;
					//Key - date, location, time
					for (const [key, value] of Object.entries(parshiyosObj)) {
						let data =
							zmanim.times[element.time][value.unformatted];
						if (data != undefined) {
							console.log("final kz " + JSON.stringify(data));
							result[element.name].push(
								timeMath(
									data,
									element.minutes,
									element.beforeAfter
								)
							);
							result.Parshiyos.push(key);
							result.Dates.push(value.formatted);
							console.log(
								`minutes ${element.minutes} be/af ${element.beforeAfter}`
							);
						}
					}
					//--------------if textbox - for amount of parshiyos in parsha column, add to textcolumn
					for (const parsha in result.Parshiyos) {
						result[element.name].push(element.text);
					}
				});
				console.log("Result: " + JSON.stringify(result));
			} else {
				throw new Error(404);
			}
		}
		getZmanim();
	}

	function timeMath(dateString, minutes, beforeAfter) {
		let date = moment(dateString);
		console.log(date.format("h:mm a"));
		if (beforeAfter == "before") {
			date = moment(date).subtract(minutes, "minutes");
		} else {
			date = moment(date).add(minutes, "minutes");
		}
		console.log(date.format("h:mm a"));
		return moment(date).format("h:mm a");
	}

	function save1() {
		$columns.forEach((element) => {
			//if rule

			//if textbox - for amount of parshiyos in parsha column, add to textcolumn
			for (const parsha in result[element.Parshiyos]) {
				result[element.name] = [element.text];
			}

			/* flatten objects */
			const rows = results.map((row) => ({}));
		});
		/* generate worksheet and workbook */
		const worksheet = XLSX.utils.json_to_sheet(rows);
		const workbook = XLSX.utils.book_new();
		XLSX.utils.book_append_sheet(workbook, worksheet, "Times");

		XLSX.writeFile(workbook, "Times.xlsx");
	}
</script>

<main>
	<div class="row">
		<div class="col s6">
			<label>Start Parsha</label>
			<select bind:value={startParsha} class="browser-default">
				{#await promise}
					...
				{:then}
					<option value="" disabled selected> Choose your starting Parsha </option>
					{#each Object.entries(parshiyosObj) as [parsha, date], i}
						<option value={parsha}>
							{parsha}: {date.formatted}</option
						>
					{/each}
				{/await}
			</select>
		</div>
		<div class="col s6">
			<label>End Parsha</label>
			<select bind:value={endParsha} class="browser-default">
				{#await promise}
					...
				{:then}
					<option value="" disabled selected> Choose your starting Parsha</option>
					{#each Object.entries(parshiyosObj) as [parsha, date], i}
						<option value={parsha}> {parsha}: {date.formatted}</option>
					{/each}
				{/await}
			</select>
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
