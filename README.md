# Reasone.AI
if you tell the situation it will give a good reasons with percentage of working out
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Reasoning App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/js/all.min.js"></script>
    <link rel="stylesheet" href="style.css">
</head>
<body class="flex items-center justify-center h-screen bg-gray-100 dark:bg-gray-900 transition-all">
    <div class="container bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg w-full max-w-md text-center transition-all relative">
        <h2 class="text-3xl font-bold mb-4 text-gray-700 dark:text-gray-200">AI Reasoning Assistant</h2>
        <textarea id="situation" class="w-full p-3 border rounded-lg mb-4 focus:ring-2 focus:ring-blue-500 dark:bg-gray-700 dark:text-white text-lg" placeholder="Describe your situation..." rows="4"></textarea>
        <div class="flex justify-between mb-4">
            <button onclick="startListening()" class="bg-green-500 hover:bg-green-600 text-white p-2 rounded-lg transition-all flex items-center gap-2">
                <i class="fas fa-microphone"></i> Speak
            </button>
            <button onclick="generateResponse()" class="bg-blue-500 hover:bg-blue-600 text-white p-2 rounded-lg font-semibold transition-all flex items-center gap-2">
                <i class="fas fa-magic"></i> Generate
            </button>
        </div>
        <button onclick="toggleDarkMode()" class="bg-gray-600 hover:bg-gray-700 text-white p-2 rounded-lg transition-all mb-4 flex items-center gap-2">
            <i class="fas fa-moon"></i> Toggle Dark Mode
        </button>
        <div id="loading" class="mt-4 text-gray-500 hidden animate-pulse">🤖 Thinking...</div>
        <div id="response" class="mt-4 text-gray-700 dark:text-white p-4 border rounded-lg bg-gray-50 dark:bg-gray-700 shadow-sm hidden text-lg"></div>
        <button onclick="speakResponse()" id="speakButton" class="bg-purple-500 hover:bg-purple-600 text-white p-2 rounded-lg transition-all mt-4 hidden flex items-center gap-2">
            <i class="fas fa-volume-up"></i> Listen
        </button>
    </div>
    <script>
        function toggleDarkMode() {
            document.body.classList.toggle("dark");
        }
        
        function startListening() {
            const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
            recognition.lang = "en-US";
            recognition.start();
            recognition.onresult = (event) => {
                document.getElementById("situation").value = event.results[0][0].transcript;
            };
        }
        
        async function generateResponse() {
            const input = document.getElementById("situation").value.trim();
            const responseDiv = document.getElementById("response");
            const loadingDiv = document.getElementById("loading");
            const speakButton = document.getElementById("speakButton");
            
            if (!input) {
                responseDiv.innerHTML = "<p class='text-red-500 font-semibold'>⚠️ Please enter or speak a situation.</p>";
                responseDiv.classList.remove("hidden");
                return;
            }
            
            loadingDiv.classList.remove("hidden");
            responseDiv.innerHTML = "";
            responseDiv.classList.add("hidden");
            speakButton.classList.add("hidden");
            
            try {
                const response = await fetch("/api/generate-response", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({ input, language: "en" })
                });
                
                if (!response.ok) throw new Error("Server error, try again later.");
                
                const data = await response.json();
                responseDiv.innerHTML = `<p class='font-semibold text-blue-700 text-xl'>💡 AI Response:</p><p>${data.response}</p>`;
                responseDiv.classList.remove("hidden");
                speakButton.classList.remove("hidden");
                speakButton.setAttribute("data-text", data.response);
            } catch (error) {
                responseDiv.innerHTML = "<p class='text-red-500 font-semibold text-lg'>❌ Error generating response. Please try again later.</p>";
                responseDiv.classList.remove("hidden");
            } finally {
                loadingDiv.classList.add("hidden");
            }
        }
        
        function speakResponse() {
            const text = document.getElementById("speakButton").getAttribute("data-text");
            const speech = new SpeechSynthesisUtterance(text);
            speech.lang = "en-US";
            speech.rate = 1;
            window.speechSynthesis.speak(speech);
        }
    </script>
</body>
</html>
