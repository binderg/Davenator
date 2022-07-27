<main>

    <div class="box">
        <a class="button" href="#popup1"><button id="info"><i class="material-icons" style="color:#C8C8C8">info</i></button></a>
    </div>
    
    <div style="z-index:2147483647"  id="popup1" class="overlay">
        <div class="popup">
            <h5>Zmanim</h5>
            <p style="font-style: italic;">From hebcal.com</p>
            <a class="close" href="#"><span style="font-size:30px">&times;</span></a>
            <div class="content"				>
                <p style = "height: 30vw; overflow:auto;">
                    <span style="color:#e74c3c"><strong>Midnight &ndash; Chatzot</strong></span><br />
                    Sunset plus 6 halachic hours<br />
                    <span style="color:#e74c3c"><strong>Dawn &ndash; Alot haShachar</strong></span><br />
                    Sun is 16.1&deg; below the horizon in the morning<br />
                    <span style="color:#e74c3c"><strong>Earliest talis &amp; tefillin &ndash; Misheyakir</strong></span><br />
                    Sun is 11.5&deg; below the horizon in the morning<br />
                    <span style="color:#e74c3c"><strong>Earliest talis &amp; tefillin &ndash; Misheyakir Machmir</strong></span><br />
                    Sun is 10.2&deg; below the horizon in the morning<br />
                    <span style="color:#e74c3c"><strong>Civil dawn</strong></span><br />
                    Sun is 6&deg; below the horizon in the morning<br />
                    <span style="color:#e74c3c"><strong>Sunrise</strong></span><br />
                    Upper edge of the Sun appears over the eastern horizon in the morning (0.833&deg; above horizon)<br />
                    <span style="color:#e74c3c"><strong>Latest Shema (Gra)</strong></span><br />
                    Sunrise plus 3 halachic hours, according to the Gra<br />
                    <span style="color:#e74c3c"><strong>Latest Shema (MGA)</strong></span><br />
                    Sunrise plus 3 halachic hours, according to Magen Avraham<br />
                    <span style="color:#e74c3c"><strong>Latest Shacharit (Gra)</strong></span><br />
                    Sunrise plus 4 halachic hours, according to the Gra<br />
                    <span style="color:#e74c3c"><strong>Latest Shacharit (MGA)</strong></span><br />
                    Sunrise plus 4 halachic hours, according to Magen Avraham<br />
                    <span style="color:#e74c3c"><strong>Midday &ndash; Chatzot</strong></span><br />
                    Sunrise plus 6 halachic hours<br />
                    <span style="color:#e74c3c"><strong>Earliest Mincha &ndash; Mincha Gedola</strong></span><br />
                    Sunrise plus 6.5 halachic hours<br />
                    <span style="color:#e74c3c"><strong>Preferable earliest time to recite Minchah &ndash; Mincha Ketana</strong></span><br />
                    Sunrise plus 9.5 halachic hours<br />
                    <span style="color:#e74c3c"><strong>Plag haMincha</strong></span><br />
                    Sunrise plus 10.75 halachic hours<br />
                    <span style="color:#e74c3c"><strong>Sunset</strong></span><br />
                    When the upper edge of the Sun disappears below the horizon (0.833&deg; below horizon)<br />
                    <span style="color:#e74c3c"><strong>Civil dusk</strong></span><br />
                    Sun is 6&deg; below the horizon in the evening<br />
                    <span style="color:#e74c3c"><strong>Nightfall (3 medium stars) &ndash; Tzeit 7.083&deg;</strong></span><br />
                    When 3 medium stars are observable in the night sky with the naked eye (sun 7.083&deg; below the horizon)<br />
                    <span style="color:#e74c3c"><strong>Nightfall (3 small stars) &ndash; Tzeit 8.5&deg;</strong></span><br />
                    When 3 small stars are observable in the night sky with the naked eye (sun 8.5&deg; below the horizon)<br />
                    <span style="color:#e74c3c"><strong>Nightfall (3 medium stars) &ndash; Tzeit 42 minutes</strong></span><br />
                    When 3 medium stars are observable in the night sky with the naked eye (fixed 42 minutes after sunset)<br />
                    <span style="color:#e74c3c"><strong>Nightfall (3 small stars) &ndash; Tzeit 50 minutes</strong></span><br />
                    When 3 small stars are observable in the night sky with the naked eye (fixed 50 minutes after sunset)<br />
                    <span style="color:#e74c3c"><strong>Nightfall (Rabbeinu Tam) &ndash; Tzeit 72 minutes</strong></span><br />
                    When 3 small stars are observable in the night sky with the naked eye (fixed 72 minutes after sunset)
                </p>
            </div>
        </div>
    </div>

</main>

<style>
button, input[type="submit"], input[type="reset"] {
	background: none;
	color: inherit;
	border: none;
	padding: 0;
	font: inherit;
	cursor: pointer;
	outline: inherit;
}


 .box {
  margin: 10px 200px 0px 0px;
  width: 70%;
  /* margin: 0 auto;
  background: rgba(255,255,255,0.2);
  padding: 35px;
  border: 2px solid #fff;
  border-radius: 20px/50px;
  background-clip: padding-box;
  text-align: center;  */
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
  margin: 70px auto;
  padding: 20px;
  background: #fff;
  border-radius: 5px;
  width: 70%;
  position: relative;
  /* transition: all 3s ease-in-out; */
  padding: 10px 30px 10px 30px
}

.popup .close {
  position: absolute;
  top: 20px;
  right: 30px;
  /* transition: all 200ms; */
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

/* @media screen and (max-width: 700px){
  .box{
    width: 70%;
  }
  .popup{
    width: 70%;
  }
} */
</style>