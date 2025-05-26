# wuj
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Wake Up Genius - Alarm App</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;600&display=swap');
    body {
      margin: 0; padding: 0;
      background: linear-gradient(135deg, #667eea, #764ba2);
      font-family: 'Poppins', sans-serif; color: #fff; height: 100vh;
      display: flex; flex-direction: column; align-items: center; justify-content: center;
    }
    h1 {
      font-weight: 600; margin-bottom: 0.5em; font-size: 3em;
      text-shadow: 0 0 10px rgba(0,0,0,0.4);
    }
    .container {
      background: rgba(255, 255, 255, 0.1); border-radius: 12px; padding: 2em;
      width: 320px; box-shadow: 0 10px 30px rgba(0,0,0,0.3);
      text-align: center;
    }
    label {
      display: block; margin-top: 1em; font-weight: 600; font-size: 1.1em;
    }
    input[type="time"], select {
      margin-top: 0.5em; padding: 0.5em 1em; border-radius: 8px; border: none;
      font-size: 1.2em; width: 100%;
      box-sizing: border-box;
    }
    button {
      margin-top: 1.5em; padding: 0.75em 1.5em; font-size: 1.2em;
      border: none; border-radius: 30px; background: #f6d365; color: #333;
      font-weight: 600; cursor: pointer; transition: background 0.3s ease;
      box-shadow: 0 5px 15px rgba(246, 211, 101, 0.5);
    }
    button:hover {
      background: #fda085;
    }
    #status {
      margin-top: 1em; font-weight: 600; font-size: 1em;
    }
    #modal {
      position: fixed; top: 0; left: 0; width: 100vw; height: 100vh;
      background: rgba(0,0,0,0.85);
      display: flex; align-items: center; justify-content: center;
      visibility: hidden; opacity: 0; transition: opacity 0.3s;
      z-index: 1000;
    }
    #modal.show {
      visibility: visible; opacity: 1;
    }
    #modal-content {
      background: #fff; color: #333; padding: 2em 3em; border-radius: 14px;
      max-width: 400px; width: 90%; box-shadow: 0 10px 30px rgba(0,0,0,0.4);
      text-align: center;
      font-family: 'Poppins', sans-serif;
    }
    #modal h2 {
      margin-top: 0; margin-bottom: 1em; font-weight: 700;
    }
    #question {
      font-weight: 600; font-size: 1.3em; margin-bottom: 1em;
    }
    #answer-input {
      padding: 0.5em 1em; font-size: 1.2em; border-radius: 10px; border: 2px solid #764ba2;
      width: 100%; box-sizing: border-box;
    }
    #submit-answer {
      margin-top: 1.5em; padding: 0.5em 1.5em; border-radius: 30px;
      border: none; background: #764ba2; color: white;
      font-weight: 600; font-size: 1.1em;
      cursor: pointer; box-shadow: 0 5px 15px rgba(118,75,162,0.7);
      transition: background 0.3s ease;
    }
    #submit-answer:hover {
      background: #667eea;
    }
  </style>
</head>
<body>
  <h1>Wake Up Genius</h1>
  <div class="container">
    <label for="alarm-time">Set Alarm Time:</label>
    <input type="time" id="alarm-time" aria-label="Alarm Time" />
    <label for="ringtone">Select Ringtone:</label>
    <select id="ringtone" aria-label="Ringtone Selection">
      <option value="beep">Beep</option>
      <option value="chime">Chime</option>
      <option value="classic">Classic</option>
      <option value="alarm">Alarm</option>
    </select>
    <button id="set-alarm">Set Alarm</button>
    <div id="status"></div>

    <!-- Audio Elements -->
    <audio id="audio-beep" src="https://actions.google.com/sounds/v1/alarms/beep_short.ogg" preload="auto"></audio>
    <audio id="audio-chime" src="https://actions.google.com/sounds/v1/alarms/digital_watch_alarm_long.ogg" preload="auto"></audio>
    <audio id="audio-classic" src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg" preload="auto"></audio>
    <audio id="audio-alarm" src="https://actions.google.com/sounds/v1/alarms/analog_watch_alarm.ogg" preload="auto"></audio>
  </div>

  <div id="modal" role="dialog" aria-modal="true" aria-labelledby="modal-title" aria-describedby="modal-desc">
    <div id="modal-content">
      <h2 id="modal-title">Wake Up Genius - Answer To Stop Alarm</h2>
      <div id="question"></div>
      <input type="number" id="answer-input" aria-label="Answer input" autocomplete="off" />
      <button id="submit-answer">Submit</button>
      <div id="modal-desc" style="margin-top: 1em; font-size: 0.9em; color: #555;">
        Solve the question correctly to stop the alarm.
      </div>
    </div>
  </div>

  <script>
    (function(){
      const alarmTimeInput = document.getElementById('alarm-time');
      const ringtoneSelect = document.getElementById('ringtone');
      const setAlarmBtn = document.getElementById('set-alarm');
      const statusDiv = document.getElementById('status');
      const modal = document.getElementById('modal');
      const questionDiv = document.getElementById('question');
      const answerInput = document.getElementById('answer-input');
      const submitAnswerBtn = document.getElementById('submit-answer');

      const audios = {
        beep: document.getElementById('audio-beep'),
        chime: document.getElementById('audio-chime'),
        classic: document.getElementById('audio-classic'),
        alarm: document.getElementById('audio-alarm')
      };

      let alarmTime = null;
      let alarmTimeout = null;
      let currentAudio = null;
      let currentAnswer = null;

      function loadSettings(){
        const savedTime = localStorage.getItem('wakeUpGeniusAlarmTime');
        const savedRingtone = localStorage.getItem('wakeUpGeniusRingtone');
        if(savedTime){
          alarmTimeInput.value = savedTime;
          alarmTime = savedTime;
        }
        if(savedRingtone && audios[savedRingtone]){
          ringtoneSelect.value = savedRingtone;
        }
      }

      function saveSettings(time, ringtone){
        localStorage.setItem('wakeUpGeniusAlarmTime', time);
        localStorage.setItem('wakeUpGeniusRingtone', ringtone);
      }

      function clearAlarm(){
        if(alarmTimeout) clearTimeout(alarmTimeout);
        if(currentAudio){
          currentAudio.pause();
          currentAudio.currentTime = 0;
          currentAudio = null;
        }
      }

      function msTillAlarm(timeStr){
        const now = new Date();
        const [hours, minutes] = timeStr.split(':').map(Number);
        let alarmDate = new Date(now.getFullYear(), now.getMonth(), now.getDate(), hours, minutes, 0, 0);
        if(alarmDate <= now){
          alarmDate.setDate(alarmDate.getDate() + 1);
        }
        return alarmDate.getTime() - now.getTime();
      }

      function generateQuestion(){
        const ops = ['+', '-', '*'];
        const op = ops[Math.floor(Math.random() * ops.length)];
        let a = Math.floor(Math.random()*20)+1;
        let b = Math.floor(Math.random()*20)+1;
        if(op === '-' && b > a) [a,b] = [b,a];
        let questionStr = `${a} ${op} ${b} = ?`;
        let ans;
        switch(op){
          case '+': ans = a + b; break;
          case '-': ans = a - b; break;
          case '*': ans = a * b; break;
        }
        return {questionStr, ans};
      }

      function triggerAlarm(){
        const ringtone = ringtoneSelect.value;
        if(audios[ringtone]){
          currentAudio = audios[ringtone];
          currentAudio.loop = true;
          currentAudio.play();
        }

        const q = generateQuestion();
        currentAnswer = q.ans;
        questionDiv.textContent = q.questionStr;
        answerInput.value = '';
        modal.classList.add('show');
        answerInput.focus();

        if('wakeLock' in navigator){
          navigator.wakeLock.request('screen').catch(() => {});
        }
      }

      function stopAlarm(){
        clearAlarm();
        modal.classList.remove('show');
        statusDiv.textContent = 'Alarm stopped. Set another alarm!';
        currentAnswer = null;
        if('wakeLock' in navigator){
          navigator.wakeLock.release().catch(() => {});
        }
      }

      function setAlarm(){
        const timeVal = alarmTimeInput.value;
        if(!timeVal){
          statusDiv.textContent = "Please select a valid alarm time.";
          return;
        }
        saveSettings(timeVal, ringtoneSelect.value);
        alarmTime = timeVal;

        clearAlarm();

        const ms = msTillAlarm(alarmTime);
        alarmTimeout = setTimeout(() => {
          triggerAlarm();
        }, ms);

        statusDiv.textContent = `Alarm set for ${alarmTime}`;
      }

      submitAnswerBtn.addEventListener('click', ()=>{
        const val = answerInput.value.trim();
        if(val === '') return;
        if(Number(val) === currentAnswer){
          stopAlarm();
        } else {
          alert('Wrong answer. Please try again!');
          answerInput.select();
        }
      });

      answerInput.addEventListener('keypress', (e)=>{
        if(e.key === 'Enter'){
          submitAnswerBtn.click();
        }
      });

      setAlarmBtn.addEventListener('click', setAlarm);

      loadSettings();
      if(alarmTimeInput.value){
        setAlarm();
      }
    })();
  </script>
</body>
</html>
