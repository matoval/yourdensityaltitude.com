<script>
  import Topbar from './UI/Topbar.svelte';
  import Bottombar from './UI/Bottombar.svelte';
  import axios from 'axios';

  let densityAlt;
  let alt;
  let DaCalcStart = 0;
  let DaCalc;
  let start;

  function altCountUp() {
    DaCalcStart = Math.round(DaCalc / 1.2);
    console.log(DaCalc);
    let intervalCount = setInterval(() => {
      DaCalcStart += 1;
      if (DaCalcStart >= DaCalc) {
        clearInterval(intervalCount);
      }
    }, 1);
  }

  const getCoor = () => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(function(position) {
        const lat = position.coords.latitude;
        const lng = position.coords.longitude;
        const coor = [lat,lng];
        getAlt(coor);
      });
    };
  };
  getCoor();

  const getAlt = async function(getCoor) {
    const coor = getCoor;
    const res = await axios(`https://cors-anywhere.herokuapp.com/https://api.airmap.com/elevation/v1/ele/?points=${coor}`);
    const altMeters = res.data.data[0];
    metersToFeet(altMeters);
    getWx(coor);
  };


  const metersToFeet = (meters) => {
    if(meters < 0) {
      return 'meters equals zero';
    } else {
      alt = meters/0.3048;
    }
  };


  const getWx = async function(getCoor) {
    var pressMb, vapor, kelvin, r, pressMerc ;
    const coor = getCoor;
    const res = await axios(`https://cors-anywhere.herokuapp.com/https://api.darksky.net/forecast/1a6288f1a9bc1dabe707aab0011e0d56/${coor}`);
    const temp = res.data.currently.temperature;
    pressMb = res.data.currently.pressure;
    const dew = res.data.currently.dewPoint;


    const covertMillToMerc = (pressMb) => {
      const mill = pressMb;
      pressMerc = mill * 0.0295300;
    };

    const simpleDA = (pressMerc) => {
      const curPress = pressMerc;
      const calcPA = ((29.92 - curPress) * 930) + alt;
      const ISA = 15 - ((alt / 1000) + 2);
      densityAlt = calcPA + (120 * ((5/9 * (temp - 32)) - ISA));
      DaCalc = Math.round(densityAlt);
      altCountUp();
    }; 
    
    covertMillToMerc(pressMb);
    simpleDA(pressMerc);

  };

</script>

<style>
  header{
    margin: 0;
    padding: 0;
    top: 0;
    position: fixed;
  }

  main {
    margin: 0;
    padding-top: 60px;
    max-width: 100vw;
    height: 100%;
  }

  .subtitle {
    text-align: center;
  }

  .gauge-box {
    display: flex;
    margin: 0 auto;
    width: 260px;
    border: 3px solid blue;
    border-radius: 10px;
  }

  .gauge {
    height: 60px;
    padding: 15px;
    text-align: center;
    background-color: blue;
  }
  .DAfeet {
    color: rgb(216, 213, 197);
    font-size: 30px;
  }

  .disclaimer {
    margin: 0 auto;
    padding-left: 30px; 
    max-width: 500px;

  }
</style>

<header>
  <Topbar></Topbar>
</header>

<main>
  <div class="subtitle">
    <h2>Your Density Altitude</h2>
  </div>
  <div class="gauge-box">
    <div class="gauge" style="width: {(DaCalcStart / DaCalc) * 100}%;">
    {#if DaCalc >= 1}
      <div class="DAfeet">{DaCalcStart} Ft</div>
    {:else}
      <div class="DAfeet">Calculating...</div>
      {/if}
    </div>
  </div>
  <div class="disclaimer">
    <p>
      Density altitude is calulated using the following formula:<br>
      Pressure altitude(PA) = ((29.92 - current pressure) * 930) + alt<br>
      Density altitude = PA + (120 * ((5/9 * (temp - 32)) - ISA))<br>

      <b>Please always double check your math before using the numbers for your flight planning.</b>
    </p>
  </div>
</main>

<footer>
  <Bottombar></Bottombar>
</footer>