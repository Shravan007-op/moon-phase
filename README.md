<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Your Moon Phase 🌙</title>
  <link href="https://fonts.googleapis.com/css2?family=Pacifico&family=Poppins:wght@300;500&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'Poppins', sans-serif;
      background: linear-gradient(to right, #000428, #004e92);
      color: white;
      text-align: center;
      padding: 50px 20px;
      min-height: 100vh;
    }

    .container {
      max-width: 800px;
      margin: 0 auto;
    }

    h1 {
      font-family: 'Pacifico', cursive;
      font-size: 3em;
      margin-bottom: 10px;
      text-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
    }

    p {
      font-size: 1.2em;
      margin-bottom: 30px;
    }

    input[type="date"] {
      padding: 10px;
      font-size: 1em;
      border-radius: 10px;
      border: none;
      margin-bottom: 20px;
      background-color: rgba(255, 255, 255, 0.9);
    }

    button {
      padding: 12px 25px;
      background-color: #ff4081;
      color: white;
      font-size: 1em;
      border: none;
      border-radius: 25px;
      cursor: pointer;
      margin-left: 10px;
      transition: all 0.3s ease;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    }

    button:hover {
      background-color: #e91e63;
      transform: translateY(-2px);
      box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
    }

    #moonImage {
      margin-top: 30px;
      max-width: 300px;
      border-radius: 20px;
      box-shadow: 0 0 30px rgba(255, 255, 255, 0.3);
    }

    .message {
      margin-top: 20px;
      font-size: 1.4em;
      padding: 15px;
      background-color: rgba(255, 255, 255, 0.1);
      border-radius: 15px;
    }

    .moon-info {
      margin-top: 20px;
      background-color: rgba(255, 255, 255, 0.1);
      padding: 15px;
      border-radius: 15px;
      max-width: 500px;
      margin-left: auto;
      margin-right: auto;
    }

    .loading {
      display: none;
      margin: 20px auto;
      width: 50px;
      height: 50px;
      border: 5px solid rgba(255, 255, 255, 0.3);
      border-radius: 50%;
      border-top-color: #fff;
      animation: spin 1s ease-in-out infinite;
    }

    @keyframes spin {
      to { transform: rotate(360deg); }
    }

    footer {
      margin-top: 50px;
      font-size: 0.9em;
      opacity: 0.8;
    }

    .error {
      color: #ff6b6b;
      font-weight: bold;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Your Moon 🌙</h1>
    <p>Enter your birthdate and see how the moon looked from India</p>
    <input type="date" id="birthdate" min="1900-01-01" max="">
    <button onclick="showMoon()">Reveal</button>

    <div class="loading" id="loading"></div>

    <div id="result" style="display:none;">
      <div class="message" id="moonMessage"></div>
      <img id="moonImage" src="" alt="Moon Phase">
      <div class="moon-info" id="moonInfo"></div>
    </div>

    <footer>
      <p>Moon phase data calculated for India's timezone (IST)</p>
    </footer>
  </div>

  <script>
    // Set max date to today
    document.getElementById("birthdate").max = new Date().toISOString().split("T")[0];
    
    // Set default date to today's date 20 years ago (typical birthdate scenario)
    const defaultDate = new Date();
    defaultDate.setFullYear(defaultDate.getFullYear() - 20);
    document.getElementById("birthdate").valueAsDate = defaultDate;

    async function showMoon() {
      const dateInput = document.getElementById("birthdate").value;
      if (!dateInput) {
        alert("Please select a valid date.");
        return;
      }

      // Show loading spinner
      document.getElementById("loading").style.display = "block";
      document.getElementById("result").style.display = "none";

      try {
        const date = new Date(dateInput);
        const moonData = await getMoonPhase(date);
        
        document.getElementById("moonMessage").innerHTML = 
          `This is how the moon blessed the sky on <strong>${date.toLocaleDateString('en-IN')}</strong> 🌌`;
        
        document.getElementById("moonImage").src = moonData.imageUrl;
        document.getElementById("moonImage").alt = moonData.phaseName;
        
        document.getElementById("moonInfo").innerHTML = `
          <p><strong>Moon Phase:</strong> ${moonData.phaseName}</p>
          <p><strong>Illumination:</strong> ${moonData.illumination}%</p>
          <p><strong>Zodiac Sign:</strong> ${moonData.zodiacSign}</p>
        `;
        
        document.getElementById("result").style.display = "block";
      } catch (error) {
        console.error("Error fetching moon data:", error);
        document.getElementById("moonMessage").className = "message error";
        document.getElementById("moonMessage").textContent = 
          "Failed to load moon data. Please try again later.";
        document.getElementById("result").style.display = "block";
      } finally {
        document.getElementById("loading").style.display = "none";
      }
    }

    async function getMoonPhase(date) {
      // Convert date to UTC and adjust for India's timezone (IST is UTC+5:30)
      const utcDate = new Date(date);
      utcDate.setMinutes(utcDate.getMinutes() + utcDate.getTimezoneOffset() + 330); // +5:30 hours
      
      // Calculate moon phase (simplified calculation)
      const phase = calculateMoonPhase(utcDate);
      
      // Get zodiac sign
      const zodiacSign = getMoonZodiacSign(utcDate);
      
      return {
        phaseName: phase.name,
        illumination: Math.round(phase.illumination * 100),
        imageUrl: getMoonImageUrl(phase.value),
        zodiacSign: zodiacSign
      };
    }

    function calculateMoonPhase(date) {
      // Based on simplified astronomical calculations
      // Returns phase value between 0 and 1 (0 = new moon, 0.5 = full moon)
      
      // Start with known new moon date (2020-01-24)
      const knownNewMoon = new Date('2020-01-24T21:42:00Z');
      const lunarCycle = 29.53058867; // days in lunar cycle
      
      // Calculate days since known new moon
      const daysSince = (date - knownNewMoon) / (1000 * 60 * 60 * 24);
      
      // Calculate current phase (0-1)
      let phase = (daysSince % lunarCycle) / lunarCycle;
      if (phase < 0) phase += 1; // handle dates before known new moon
      
      // Determine phase name and illumination
      let name, illumination;
      
      if (phase < 0.03 || phase > 0.97) {
        name = "New Moon";
        illumination = phase < 0.03 ? phase * 33.33 : (1 - phase) * 33.33;
      } else if (phase < 0.22) {
        name = "Waxing Crescent";
        illumination = (phase - 0.03) / 0.19 * 50 + 1;
      } else if (phase < 0.28) {
        name = "First Quarter";
        illumination = 50 + (phase - 0.22) / 0.06 * 10;
      } else if (phase < 0.47) {
        name = "Waxing Gibbous";
        illumination = 60 + (phase - 0.28) / 0.19 * 39;
      } else if (phase < 0.53) {
        name = "Full Moon";
        illumination = 99 - (phase - 0.47) / 0.06 * 18;
      } else if (phase < 0.72) {
        name = "Waning Gibbous";
        illumination = 81 - (phase - 0.53) / 0.19 * 40;
      } else if (phase < 0.78) {
        name = "Last Quarter";
        illumination = 41 - (phase - 0.72) / 0.06 * 10;
      } else {
        name = "Waning Crescent";
        illumination = 31 - (phase - 0.78) / 0.19 * 30;
      }
      
      return {
        value: phase,
        name: name,
        illumination: Math.min(100, Math.max(0, illumination))
      };
    }

    function getMoonImageUrl(phaseValue) {
      // Use NASA moon phase images or generated SVG based on phase
      const phaseImages = [
        {min: 0, max: 0.03, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0001_print.jpg"},
        {min: 0.03, max: 0.22, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0020_print.jpg"},
        {min: 0.22, max: 0.28, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0040_print.jpg"},
        {min: 0.28, max: 0.47, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0060_print.jpg"},
        {min: 0.47, max: 0.53, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0080_print.jpg"},
        {min: 0.53, max: 0.72, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0100_print.jpg"},
        {min: 0.72, max: 0.78, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0120_print.jpg"},
        {min: 0.78, max: 1, url: "https://svs.gsfc.nasa.gov/vis/a000000/a004600/a004675/moon.0140_print.jpg"}
      ];
      
      const matchingPhase = phaseImages.find(p => phaseValue >= p.min && phaseValue < p.max);
      return matchingPhase ? matchingPhase.url : phaseImages[0].url;
    }

    function getMoonZodiacSign(date) {
      // Simplified zodiac sign calculation based on moon position
      // This is a rough approximation - real moon zodiac changes every ~2.5 days
      
      const year = date.getFullYear();
      const month = date.getMonth() + 1;
      const day = date.getDate();
      
      // This is a very simplified approach - for accurate results you'd need
      // to calculate the moon's actual position in the ecliptic
      
      // Approximation: moon moves through all signs in ~27.3 days
      // Start with known moon position (Aries on Jan 1, 2000)
      const knownMoonSignDate = new Date('2000-01-01T00:00:00Z');
      const daysInLunarOrbit = 27.321661;
      
      const daysSince = (date - knownMoonSignDate) / (1000 * 60 * 60 * 24);
      const signIndex = Math.floor((daysSince % daysInLunarOrbit) / daysInLunarOrbit * 12);
      
      const signs = [
        "Aries", "Taurus", "Gemini", "Cancer", 
        "Leo", "Virgo", "Libra", "Scorpio", 
        "Sagittarius", "Capricorn", "Aquarius", "Pisces"
      ];
      
      return signs[(signIndex + 0) % 12]; // +0 is offset adjustment if needed
    }
  </script>
</body>
</html>
