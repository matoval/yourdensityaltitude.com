<script>
  import Topbar from './UI/Topbar.svelte';
  import Bottombar from './UI/Bottombar.svelte';
  import axios from 'axios';

  let densityAlt;
  let alt;
  let DaCalcStart = 0;
  let DaCalc;
  let start;
  let oneHand;
  let tenHand;
  let oneDegree;
  let tenDegree;

  function altCountUp() {
    DaCalcStart = Math.round(DaCalc / 1.2);
    console.log(DaCalc);
    let intervalCount = setInterval(() => {
      DaCalcStart += 1;
      if (DaCalcStart >= DaCalc) {
        clearInterval(intervalCount);
      }
    }, 1);
    if (DaCalc.toString().length === 3) {
      tenDegree = 0
    } else if (DaCalc.toString().length === 4) {
      tenDegree = Math.abs(DaCalc / 1000).toFixed(2);
      oneDegree = Math.abs(DaCalc / 100).toFixed(2);
    } else if (DaCalc.toString().length === 5) {
      tenDegree = Math.abs(DaCalc / 1000).toFixed(2);
      oneDegree = Math.abs(DaCalc / 100).toFixed(2);
    } 
    oneHand = 180 + (parseInt(oneDegree) * 3.6);
    tenHand = 180 + (parseInt(tenDegree) * 3.6);
    console.log(tenHand);
    console.log(oneHand);
    document.documentElement.style.setProperty('--oneHand',  `${oneHand}deg`);
    document.documentElement.style.setProperty('--tenHand', `${tenHand}deg`);
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
  :root {
    --oneHand: 180deg;
    --tenHand: 180deg;
  }

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

  .altimeter {
    display: flex;
    justify-content: center;
    position: relative;
    height: 437px;
    width: 410px;
    margin: 0 auto;
  }

  .tenHands {
    position: absolute;
    top: 105px;
    left: 174px;
    z-index: 2;
    transform: rotate(var(--tenHand));
    transition: 7500ms linear all;
  }

  .oneHands {
    position: absolute;
    top: 70px;
    left: 174px;
    z-index: 2;
    transform: rotate(var(--oneHand));
    transition: 7500ms linear all; 
  }

  .DAfeet {
    color: rgb(0, 0, 0);
    font-size: 30px;
    text-align: center;
  }

  .disclaimer {
    margin: 0 auto;
    padding-left: 30px; 
    max-width: 500px;

  }

   @media only screen and (max-device-width: 400px) {
    .altimeter {
      width: 300px;
    }

    .altimeter img {
      width: 300px;
      height: 350px;
    }

    .tenHands {
      top: 65px;
      left: 0px;
    }

    .tenHands img {
      height: 170px;
    }

    .oneHands {
      top: 70px;
      left: 0px;
    }
    .oneHands img {
      height: 170px;
    }
  }

</style>

<header>
  <Topbar></Topbar>
</header>

<main>
  <div class="subtitle">
    <h2>Your Density Altitude</h2>
  </div>
  <div class="altimeter">
    <img src="altimeter.png" alt="altimeter">
    <div class="tenHands">
      <img src="ShortHand.svg" alt="Short hand">
    </div>
    <div class="oneHands">
      <img src="LongHand.svg" alt="Long Hand">
    </div>
  </div>
  {#if DaCalc >= 1}
    <div class="DAfeet">{DaCalcStart} Ft</div>
  {:else}
    <div class="DAfeet">Calculating...</div>
  {/if}
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