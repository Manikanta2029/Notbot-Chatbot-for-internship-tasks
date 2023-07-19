# Notbot-Chatbot-for-internship-tasks
const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');

const app = express();
const PORT = 3000;

// In-memory database to store chat sessions
const sessionsDB = {};

// Parse incoming request bodies
app.use(bodyParser.json());

// Handle incoming messages from WhatsApp
app.post('/webhook', (req, res) => {
  const { from, body } = req.body;

  // Check if the incoming message is a session message and if the user has an active session
  if (from && body && sessionsDB[from]) {
    processIncomingMessage(from, body);
  }

  res.status(200).end();
});

// Process incoming messages and generate responses
function processIncomingMessage(from, message) {
  const session = sessionsDB[from];

  switch (session.step) {
    case 'start':
      handleStepStart(from, message);
      break;
    case 'name':
      handleStepName(from, message);
      break;
    case 'email':
      handleStepEmail(from, message);
      break;
    case 'experience':
      handleStepExperience(from, message);
      break;
    default:
      sendSessionMessage(from, 'Invalid session. Please start again.');
      break;
  }
}

// Step 1: Handle the initial message
function handleStepStart(from, message) {
  if (message.toLowerCase().includes('yes')) {
    sessionsDB[from] = { step: 'name' };
    sendSessionMessage(from, 'Please enter your Name');
  } else if (message.toLowerCase().includes('no')) {
    delete sessionsDB[from];
    sendSessionMessage(from, 'Thank you for your response. Chat session ended.');
  } else {
    sendSessionMessage(from, 'Please select "Yes" or "No" to proceed.');
  }
}

// Step 2: Handle the Name input
function handleStepName(from, message) {
  // Perform validation here if needed
  sessionsDB[from].name = message;
  sessionsDB[from].step = 'email';
  sendSessionMessage(from, 'Please enter your email ID?');
}

// Step 3: Handle the Email input
function handleStepEmail(from, message) {
  // Perform email validation here if needed
  sessionsDB[from].email = message;
  sessionsDB[from].step = 'experience';
  sendSessionMessage(
    from,
    'Please select how many years of experience you have with Python/JS/Automation Development.\nShow List:\n1 year\n2 years\n3 years\n4 years\n5 years'
  );
}

// Step 4: Handle the Experience selection
function handleStepExperience(from, message) {
  // Perform validation here if needed
  sessionsDB[from].experience = message;
  // Save all the data in the database here
  saveToDatabase(from, sessionsDB[from]);

  // End the chat session and thank the user
  delete sessionsDB[from];
  sendSessionMessage(from, 'Thanks for connecting. We will get back to you shortly.');
}

// Send session message to the specified number
function sendSessionMessage(to, message) {
  // Customize this function based on the WhatsApp API provider you're using
  // This example uses Axios to make an HTTP POST request to the API
  const YOUR_API_KEY = 'YOUR_WHATSAPP_API_KEY';
  const YOUR_API_URL = 'https://api.whatsapp.com/message/send';
  
  axios.post(YOUR_API_URL, {
    phone: to,
    body: message
  }, {
    headers: {
      'Authorization': `Bearer ${YOUR_API_KEY}`
    }
  })
  .then(response => {
    console.log('Message sent successfully:', response.data);
  })
  .catch(error => {
    console.error('Error sending message:', error.message);
  });
}

// Function to save the chat session data to a database (customize this as needed)
function saveToDatabase(from, sessionData) {
  // Implement database saving logic here
  console.log(`Session data for ${from} saved:`, sessionData);
}

// Start the server
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
