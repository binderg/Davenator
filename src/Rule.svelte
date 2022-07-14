<script>
	import { columns } from './store.js';
	export let objAttributes = {id:null, type:"rule", name:"", minutes:"", beforeAfter:"", time:"", text:"" };
	let zmanim = [
		{ value:"chatzotNight", text:"Chatzos"},
		{ value:"alotHaShachar", text:"Alos HaShachar"},
		{ value:"misheyakir", text:"Misheyakir"},
		{ value:"misheyakirMachmir", text:"Misheyakir Machmir"},
		{ value:"dawn", text:"Civil dawn"},
		{ value:"sunrise", text:"Sunrise"},
		{ value:"sofZmanShma", text:"Latest Shema (Gra)"},
		{ value:"sofZmanShmaMGA", text:"Latest Shema (MGA)"},
		{ value:"sofZmanTfilla", text:"Latest Shacharis (Gra)"},
		{ value:"sofZmanTfillaMGA", text:"Latest Shacharit (MGA)"},
		{ value:"chatzot", text:"Chatzos"},
		{ value:"minchaGedola", text:"Mincha Gedola"},
		{ value:"minchaKetana", text:"Mincha Ketana"},
		{ value:"plagHaMincha", text:"Plag HaMincha"},
		{ value:"sunset", text:"Sunset"},
		{ value:"dusk", text:"Civil Dusk"},
		{ value:"tzeit7083deg", text:"Nightfall (3 medium stars) Tzeit 7.083°"},
		{ value:"tzeit85deg", text:"Nightfall (3 small stars) Tzeit 8.5°"},
		{ value:"tzeit42min", text:"Nightfall (3 medium stars) Tzeit 42 minutes"},
		{ value:"tzeit50min", text:"Nightfall (3 small stars) Tzeit 50 minutes"},
		{ value:"tzeit72min", text:"Nightfall (Rabbeinu Tam) Tzeit 72 minutes"}
	]




	function removeComponent() {
		$columns = $columns.filter(function(value, index, arr){ 
			if (value.id != objAttributes.id) return value;
		});
		}
	
</script>

<main>
	<div class="container">

	{#if objAttributes.type=="rule"}
      <div class="row">
		<div class="input-field col s2">
			<button on:click={removeComponent} class="btn-floating waves-effect waves-light"><i class="material-icons">close</i></button>			
		  </div>
		<div class="input-field col s2">
			<input bind:value={objAttributes.name}  id="name" type="text" class="validate">
			<label for="name">Name</label>
		  </div>
        <div class="input-field col s2">
          <input bind:value={objAttributes.minutes}  id="minutes" type="number" min="0" class="validate">
          <label for="minutes">Minutes</label>
        </div>

		<div class="col s2">
		<label>Befor/After</label>
		<select bind:value={objAttributes.beforeAfter} class="browser-default">
		  <option value="before">Before</option>
		  <option value="after">After</option>
		</select>
	</div>
	<div class="col s2">
		<label>Time</label>
		<select bind:value={objAttributes.time} class="browser-default">
			<option value="" disabled selected>Time</option>
			{#each zmanim as zman}
			<option value={zman.value}>{zman.text}</option>
	        {/each}

		</select>
      

</div>	
</div>	
{:else}

	<div class="row">
		<div class="input-field col s4 pull-s1">
			<button on:click={removeComponent} class="btn-floating waves-effect waves-light"><i 	class="material-icons">close</i></button>			
		  </div>
		  <div class="input-field col s4 pull-s2">
			<input bind:value={objAttributes.name} id="name" type="text" class="validate">
			<label for="name">Name</label>
		  </div>
		  <div class="input-field col s4 pull-s2">
			<input bind:value={objAttributes.text} id="text" type="text" class="validate">
			<label for="name">Text</label>
		  </div>
		</div>  
{/if}

	
	
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