const quickReplies = [
  { label: "Compliment her", text: "You look really beautiful today, Aiko.", affinity: 8, response: "Aiko blushes and smiles. \"T-thank you... I was hoping you'd notice.\"" },
  { label: "Ask about her day", text: "How was your day, Aiko?", affinity: 6, response: "Aiko leans closer. \"It was better because you messaged me.\"" },
  { label: "Be sweet", text: "I’m really glad you’re in my life.", affinity: 10, response: "Aiko’s eyes sparkle. \"I’m glad too... more than I can say.\"" },
  { label: "Flirt", text: "You make my heart race when you smile at me.", affinity: 12, response: "Aiko covers her face, giggling. \"You’re so bold... but I like it.\"" },
  { label: "Confess", text: "I think I’m falling for you, Aiko.", affinity: 16, response: "Aiko’s face turns pink and she whispers, \"Then stay with me a little longer...\"" }
];

const state = {
  affection: 62,
  mood: "Blushing",
  day: 1,
  messages: [],
  autoVoice: true,
  listening: false
};

const chatLog = document.getElementById("chatLog");
const chatForm = document.getElementById("chatForm");
const chatInput = document.getElementById("chatInput");
const quickRepliesContainer = document.getElementById("quickReplies");
const affectionFill = document.getElementById("affectionFill");
const affectionValue = document.getElementById("affectionValue");
const moodValue = document.getElementById("moodValue");
const sceneQuote = document.getElementById("sceneQuote");
const restartBtn = document.getElementById("restartBtn");
const voiceBtn = document.getElementById("voiceBtn");
const voiceToggleBtn = document.getElementById("voiceToggleBtn");
const voiceStatus = document.getElementById("voiceStatus");
const bedSceneBtn = document.getElementById("bedSceneBtn");

let recognition = null;

function clampAffection(value) {
  return Math.max(0, Math.min(100, value));
}

function setVoiceStatus(message, isError = false) {
  voiceStatus.textContent = message;
  voiceStatus.classList.toggle("error", isError);
}

function getMood(affection) {
  if (affection >= 85) return "Head over heels";
  if (affection >= 70) return "Blushing";
  if (affection >= 50) return "Happy";
  if (affection >= 30) return "Shy";
  return "Nervous";
}

function updateUI() {
  const clamped = clampAffection(state.affection);
  state.mood = getMood(clamped);
  affectionFill.style.width = `${clamped}%`;
  affectionValue.textContent = `${clamped}%`;
  moodValue.textContent = state.mood;

  if (clamped >= 90) {
    sceneQuote.textContent = "“You are the only one I ever want to see when I wake up...”";
  } else if (clamped >= 75) {
    sceneQuote.textContent = "“Your voice makes my heart feel so warm...”";
  } else if (clamped >= 55) {
    sceneQuote.textContent = "“I’m happy just being close to you...”";
  } else if (clamped >= 30) {
    sceneQuote.textContent = "“I’m still learning how to be brave around you...”";
  } else {
    sceneQuote.textContent = "“I’m a little nervous, but I really like talking to you...”";
  }
}

function addMessage(from, text) {
  const wrapper = document.createElement("div");
  wrapper.className = `message ${from}`;
  wrapper.textContent = text;
  chatLog.appendChild(wrapper);
  chatLog.scrollTop = chatLog.scrollHeight;
}

function getReplyForMessage(message) {
  const lower = message.toLowerCase();

  if (/(love|ador|heart|fall for|miss)/i.test(lower)) {
    return {
      response: "Aiko smiles shyly. \"I... I feel the same way. I’ve been thinking about you too.\"",
      delta: 14
    };
  }

  if (/(beautiful|pretty|cute|gorgeous|amazing|sweet|special)/i.test(lower)) {
    return {
      response: "Aiko blushes deeply. \"You really know how to make me feel shy... thank you.\"",
      delta: 10
    };
  }

  if (/(day|school|work|busy)/i.test(lower)) {
    return {
      response: "Aiko leans in. \"I was hoping you'd ask. I’m glad I get to talk to you after all that.\"",
      delta: 7
    };
  }

  if (/(date|spend time|hang out|walk|movie|coffee)/i.test(lower)) {
    return {
      response: "Aiko’s eyes light up. \"I’d love that... maybe we can do it soon, just the two of us.\"",
      delta: 12
    };
  }

  if (/(kiss|hug|hold|close|touch|bed|sleep)/i.test(lower)) {
    return {
      response: "Aiko turns pink instantly. \"I-I think I’d like to stay close to you... let me make sure you’re comfortable.\"",
      delta: 13
    };
  }

  if (/(hello|hi|hey|good morning|good evening)/i.test(lower)) {
    return {
      response: "Aiko softly smiles. \"Hi... I’m glad you’re here.\"",
      delta: 4
    };
  }

  if (/(jealous|angry|sad|alone|lonely)/i.test(lower)) {
    return {
      response: "Aiko lowers her gaze. \"I don’t like seeing you sad... I want to be the one who makes you smile.\"",
      delta: 9
    };
  }

  if (/(thank you|you’re kind|care|support)/i.test(lower)) {
    return {
      response: "Aiko gently laughs. \"You make it easy to feel safe around you.\"",
      delta: 8
    };
  }

  const moodResponses = [
    "Aiko tilts her head and smiles. \"Hehe, that’s kind of adorable.\"",
    "Aiko giggles softly. \"I like hearing your voice.\"",
    "Aiko blushes and looks away. \"You always manage to make me feel special.\"",
    "Aiko leans closer. \"Tell me more... I like when you talk to me.\""
  ];

  return {
    response: moodResponses[Math.floor(Math.random() * moodResponses.length)],
    delta: 5
  };
}

function speakText(text) {
  if (!state.autoVoice || !("speechSynthesis" in window)) {
    return;
  }

  window.speechSynthesis.cancel();
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.rate = 1.1;
  utterance.pitch = 1.2;
  utterance.lang = "en-US";

  const voices = window.speechSynthesis.getVoices();
  const preferred = voices.find((voice) => /female|girl|samantha|zira|susan|aria|karen/i.test(voice.name)) || voices[0];

  if (preferred) {
    utterance.voice = preferred;
  }

  window.speechSynthesis.speak(utterance);
}

function processUserMessage(message) {
  const result = getReplyForMessage(message);
  state.affection = clampAffection(state.affection + result.delta);
  updateUI();
  addMessage("ai", result.response);
  speakText(result.response);
  return result;
}

function runChoice(choice) {
  addMessage("user", choice.text);
  state.affection = clampAffection(state.affection + (choice.affinity || 0));
  updateUI();
  addMessage("ai", choice.response);
  speakText(choice.response);
}

function renderQuickReplies() {
  quickRepliesContainer.innerHTML = "";

  quickReplies.forEach((choice) => {
    const btn = document.createElement("button");
    btn.type = "button";
    btn.className = "reply-btn";
    btn.textContent = choice.label;
    btn.addEventListener("click", () => runChoice(choice));
    quickRepliesContainer.appendChild(btn);
  });
}

function triggerBedScene() {
  if (state.affection < 80) {
    const message = "Aiko blushes and looks away. \"Not yet... I want to be closer to you before we do that.\"";
    addMessage("ai", message);
    speakText(message);
    sceneQuote.textContent = "“Maybe later... when we are even closer.”";
    return;
  }

  const message = "Aiko gently tucks you in, smoothing your hair and whispering, \"Sleep well... I’ll be right here when you wake up.\"";
  state.affection = clampAffection(state.affection + 8);
  updateUI();
  addMessage("ai", message);
  speakText(message);
  sceneQuote.textContent = "“Rest now... I’ll stay beside you, just for a little while.”";
}

function initConversation() {
  const opening = [
    "Aiko: Hi... I was waiting for you.",
    "Aiko: I’m glad you came back. I’ve been thinking of you."
  ];

  opening.forEach((text) => addMessage("ai", text));
  updateUI();
}

function startVoiceInput() {
  const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

  if (!SpeechRecognition) {
    setVoiceStatus("Voice chat is not supported in this browser. Try Chrome or Edge.", true);
    return;
  }

  if (!recognition) {
    recognition = new SpeechRecognition();
    recognition.lang = "en-US";
    recognition.interimResults = false;
    recognition.continuous = false;

    recognition.onstart = () => {
      state.listening = true;
      voiceBtn.classList.add("listening");
      voiceBtn.textContent = "🎙️ Listening...";
      setVoiceStatus("Listening... say something to Aiko.");
    };

    recognition.onresult = (event) => {
      const transcript = event.results[0][0].transcript.trim();
      if (!transcript) {
        return;
      }

      chatInput.value = transcript;
      addMessage("user", transcript);
      processUserMessage(transcript);
      setVoiceStatus("Voice message sent.");
    };

    recognition.onerror = (event) => {
      state.listening = false;
      voiceBtn.classList.remove("listening");
      voiceBtn.textContent = "🎙️ Talk to Aiko";
      setVoiceStatus(`Voice input error: ${event.error}.`, true);
    };

    recognition.onend = () => {
      state.listening = false;
      voiceBtn.classList.remove("listening");
      voiceBtn.textContent = "🎙️ Talk to Aiko";
      if (!voiceStatus.textContent.includes("error")) {
        setVoiceStatus("Voice chat ready.");
      }
    };
  }

  try {
    recognition.start();
  } catch (error) {
    setVoiceStatus("Voice input already active. Try again in a moment.", true);
  }
}

chatForm.addEventListener("submit", (event) => {
  event.preventDefault();
  const value = chatInput.value.trim();

  if (!value) return;

  addMessage("user", value);
  chatInput.value = "";
  processUserMessage(value);
});

voiceBtn.addEventListener("click", () => {
  if (state.listening) {
    recognition.stop();
    return;
  }

  startVoiceInput();
});

voiceToggleBtn.addEventListener("click", () => {
  state.autoVoice = !state.autoVoice;
  voiceToggleBtn.classList.toggle("active", state.autoVoice);
  voiceToggleBtn.textContent = state.autoVoice ? "🔊 Voice on" : "🔇 Voice off";
  setVoiceStatus(state.autoVoice ? "Aiko will speak back to you." : "Voice replies are off.");
});

restartBtn.addEventListener("click", () => {
  state.affection = 62;
  state.day = 1;
  chatLog.innerHTML = "";
  initConversation();
  updateUI();
  setVoiceStatus("Voice chat ready.");
});

bedSceneBtn.addEventListener("click", triggerBedScene);

renderQuickReplies();
initConversation();
updateUI();
voiceToggleBtn.classList.add("active");
if ("speechSynthesis" in window) {
  window.speechSynthesis.getVoices();
}
