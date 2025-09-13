<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Malla Reddy University - Smart Assistant</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
        .chat-container {
            max-width: 800px;
            margin: 0 auto;
            height: calc(100vh - 6rem);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            border-radius: 1.5rem;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
        }
        .chat-messages {
            flex-grow: 1;
            padding: 1.5rem;
            overflow-y: auto;
            -webkit-mask-image: linear-gradient(to bottom, transparent 0%, black 5%, black 95%, transparent 100%);
            mask-image: linear-gradient(to bottom, transparent 0%, black 5%, black 95%, transparent 100%);
        }
        .message {
            margin-bottom: 1rem;
            display: flex;
            max-width: 85%;
        }
        .message.user {
            justify-content: flex-end;
            margin-left: auto;
        }
        .message.assistant {
            justify-content: flex-start;
        }
        .message-bubble {
            padding: 0.75rem 1rem;
            border-radius: 1.25rem;
            word-wrap: break-word;
            white-space: pre-wrap;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }
        .user .message-bubble {
            background-color: #3b82f6;
            color: white;
            border-bottom-right-radius: 0.25rem;
        }
        .assistant .message-bubble {
            background-color: #ffffff;
            color: #1f2937;
            border-bottom-left-radius: 0.25rem;
        }
        .typing-indicator {
            width: 3rem;
            height: 1.5rem;
            display: flex;
            justify-content: space-around;
            align-items: center;
            background-color: #e5e7eb;
            border-radius: 9999px;
            padding: 0 0.5rem;
            animation: fadeIn 0.5s ease-in-out;
        }
        .typing-indicator span {
            width: 0.5rem;
            height: 0.5rem;
            background-color: #6b7280;
            border-radius: 50%;
            animation: pulse 1.5s infinite ease-in-out;
        }
        .typing-indicator span:nth-child(2) {
            animation-delay: 0.2s;
        }
        .typing-indicator span:nth-child(3) {
            animation-delay: 0.4s;
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        @keyframes pulse {
            0%, 100% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.25); opacity: 0.7; }
        }
    </style>
</head>
<body class="bg-gray-100 p-4 sm:p-6 lg:p-8 flex items-center justify-center min-h-screen">
    <div class="chat-container bg-white flex flex-col rounded-3xl shadow-lg">
        <!-- Header -->
        <div class="bg-blue-600 text-white p-4 sm:p-6 rounded-t-3xl flex items-center justify-between shadow-md">
            <h1 class="text-xl sm:text-2xl font-bold flex items-center">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 mr-3 text-white" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M8 12h.01M12 12h.01M16 12h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                </svg>
                Malla Reddy University Assistant
            </h1>
        </div>

        <!-- Chat Messages -->
        <div id="chat-messages" class="chat-messages overflow-y-auto p-4 sm:p-6 bg-gray-50 flex-grow">
            <!-- Initial welcome message from the assistant -->
            <div class="message assistant">
                <div class="message-bubble">
                    <p>Hello! I am your Smart Campus Assistant. How can I help you today?</p>
                </div>
            </div>
        </div>

        <!-- Input Area -->
        <div class="p-4 sm:p-6 bg-white border-t border-gray-200 flex items-center space-x-2 sm:space-x-4 rounded-b-3xl shadow-inner">
            <input type="text" id="user-input" placeholder="Type your query here..." class="flex-grow p-3 sm:p-4 border border-gray-300 rounded-full focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200">
            <button id="send-btn" class="p-3 sm:p-4 bg-blue-600 text-white rounded-full transition-all duration-200 shadow-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M5 12h14m-7-7l7 7-7 7" />
                </svg>
            </button>
        </div>
    </div>

    <script>
        // --- Hard-coded Campus Database (for demonstration) ---
        // In a real application, this data would be fetched from a database like Firestore.
        const campusData = {
            "facilities": {
                "general": "Malla Reddy University has modern facilities including spacious seminar halls, amphitheater classrooms, a central library, a food court, a fitness center, and sports grounds.",
                "library": "The Central Library has a collection of over 25,000 books, subscribed national and international journals, and a digital library. It is open from 9:00 AM to 6:30 PM on weekdays.",
                "hostel": "Separate hostels for boys and girls with 24/7 security, Wi-Fi, and hygienic dining facilities.",
                "labs": "Departmental-based laboratories for various disciplines, including chemistry, computer science, physics, and more.",
                "sports": "The campus offers excellent sports facilities for cricket, football, basketball, and indoor games like carrom and chess."
            },
            "dining": {
                "food_court": "The university has a well-established cafeteria and several kiosks and food courts across the campus. They provide a wide range of hygienic and nutritious food, including North and South Indian cuisines."
            },
            "schedules": {
                "academic_calendar": "The academic calendar is released annually and is available on the university's official website. It outlines important dates for semesters, exams, and holidays. You can find specific schedules for B.Tech, M.Tech, and MBA programs.",
                "exams": "Exam schedules are typically announced a few weeks before the examination period. Please check the official examination section on the university website for the latest timetable."
            },
            "administrative": {
                "admissions": "Information about the admission process, eligibility criteria, and fee structure is available on the official website. You can also contact the admissions office for detailed procedures.",
                "grievance": "For any administrative issues or grievances, students can contact the Student Grievance Redressal Committee or the concerned department head.",
                "fees": "Fee payment procedures can be found on the university's official admissions and finance portals. You can also visit the administrative block for assistance."
            },
            "contact": {
                "general": "For general queries, you can visit the Administrative Block or check the 'Contact Us' section on the Malla Reddy University website."
            }
        };

        // --- Chatbot Logic ---
        const userInput = document.getElementById('user-input');
        const sendBtn = document.getElementById('send-btn');
        const chatMessages = document.getElementById('chat-messages');

        // Function to create and append a message bubble
        function appendMessage(sender, message) {
            const messageDiv = document.createElement('div');
            messageDiv.classList.add('message', sender);

            const bubbleDiv = document.createElement('div');
            bubbleDiv.classList.add('message-bubble', sender === 'user' ? 'bg-blue-600' : 'bg-white');
            bubbleDiv.innerHTML = `<p>${message}</p>`;

            messageDiv.appendChild(bubbleDiv);
            chatMessages.appendChild(messageDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight; // Auto-scroll to the latest message
        }

        // Simulates an AI response generation
        async function generateResponse(query) {
            // Placeholder for a real LLM API call
            // For a real app, this is where you would call a service like the Gemini API.
            // For example:
            /*
            const payload = {
                contents: [{ parts: [{ text: query }] }],
                tools: [{ "google_search": {} }],
                systemInstruction: { parts: [{ text: "Act as a helpful campus assistant for Malla Reddy Deemed University. Use only the provided campus data to answer questions." }] },
            };
            const response = await fetch("YOUR_GEMINI_API_ENDPOINT", {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });
            const result = await response.json();
            const generatedText = result.candidates[0].content.parts[0].text;
            return generatedText;
            */

            // Simple keyword-based logic for this demo
            const lowerCaseQuery = query.toLowerCase();
            let response = "I'm sorry, I couldn't find information about that. Please try asking about schedules, facilities, dining, library, or administrative procedures.";

            // Check for keywords and provide a relevant response from the hard-coded data
            if (lowerCaseQuery.includes('schedule') || lowerCaseQuery.includes('calendar') || lowerCaseQuery.includes('exams')) {
                response = campusData.schedules.academic_calendar + " " + campusData.schedules.exams;
            } else if (lowerCaseQuery.includes('facility') || lowerCaseQuery.includes('building') || lowerCaseQuery.includes('classrooms')) {
                response = campusData.facilities.general;
            } else if (lowerCaseQuery.includes('library') || lowerCaseQuery.includes('books')) {
                response = campusData.facilities.library;
            } else if (lowerCaseQuery.includes('dining') || lowerCaseQuery.includes('food') || lowerCaseQuery.includes('canteen')) {
                response = campusData.dining.food_court;
            } else if (lowerCaseQuery.includes('administrative') || lowerCaseQuery.includes('admission') || lowerCaseQuery.includes('fees')) {
                response = campusData.administrative.admissions + " " + campusData.administrative.fees;
            } else if (lowerCaseQuery.includes('hello') || lowerCaseQuery.includes('hi')) {
                response = "Hello there! How can I assist you with your queries about Malla Reddy University?";
            } else if (lowerCaseQuery.includes('hostel')) {
                 response = campusData.facilities.hostel;
            } else if (lowerCaseQuery.includes('sports')) {
                 response = campusData.facilities.sports;
            } else if (lowerCaseQuery.includes('contact')) {
                 response = campusData.contact.general;
            }

            return new Promise(resolve => setTimeout(() => resolve(response), 1000)); // Simulate a delay
        }

        async function handleUserInput() {
            const query = userInput.value.trim();
            if (query === '') return;

            // Display user's message
            appendMessage('user', query);
            userInput.value = '';

            // Show typing indicator
            const typingIndicator = document.createElement('div');
            typingIndicator.classList.add('message', 'assistant', 'typing-indicator');
            typingIndicator.innerHTML = `
                <div class="message-bubble">
                    <span class="dot"></span>
                    <span class="dot"></span>
                    <span class="dot"></span>
                </div>
            `;
            chatMessages.appendChild(typingIndicator);
            chatMessages.scrollTop = chatMessages.scrollHeight;

            // Get response and remove typing indicator
            const response = await generateResponse(query);
            chatMessages.removeChild(typingIndicator);

            // Display assistant's message
            appendMessage('assistant', response);
        }

        // Event listeners for sending a message
        sendBtn.addEventListener('click', handleUserInput);
        userInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') {
                handleUserInput();
            }
        });

    </script>
</body>
</html>


