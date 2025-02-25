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
    <link rel="stylesheet" href="style.css">
</head>
<body class="flex items-center justify-center h-screen bg-gray-100 dark:bg-gray-900 transition-all">
    <div class="container bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg w-96 text-center transition-all">
        <h2 class="text-2xl font-bold mb-4 text-gray-700 dark:text-gray-200">AI Reasoning App</h2>
        <textarea id="situation" class="w-full p-2 border rounded-lg mb-4 focus:ring-2 focus:ring-blue-300 dark:bg-gray-700 dark:text-white" placeholder="Describe your situation..." rows="4"></textarea>
        <div class="flex justify-between mb-4">
            <button onclick="startListening()" class="bg-green-500 hover:bg-green-600 text-white p-2 rounded-lg transition-all">🎤 Speak</button>
            <button onclick="generateResponse()" class="bg-blue-500 hover:bg-blue-600 text-white p-2 rounded-lg font-semibold transition-all">Generate</button>
        </div>
        <button onclick="toggleDarkMode()" class="bg-gray-600 hover:bg-gray-700 text-white p-2 rounded-lg transition-all mb-4">🌙 Toggle Dark Mode</button>
        <div id="loading" class="mt-4 text-gray-500 hidden animate-pulse">🤖 Thinking...</div>
        <div id="response" class="mt-4 text-gray-700 dark:text-white p-3 border rounded-lg bg-gray-50 dark:bg-gray-700 shadow-sm hidden"></div>
        <button onclick="speakResponse()" id="speakButton" class="bg-purple-500 hover:bg-purple-600 text-white p-2 rounded-lg transition-all mt-4 hidden">🔊 Listen</button>
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
        
        function getExcuseProbability(excuse) {
            const excuseRatings = {
                "My pet tore up my assignment.": 21,
                "I accidentally brought the wrong book.": 73,
                "I left my book in another classroom.": 65,
                "I misunderstood the assignment requirements and wanted to clarify them first.": 80,
                "I had a power outage last night and couldn’t complete my work.": 50,
                "I was rushing this morning and forgot to pack it.": 60
            };
            
            return excuseRatings[excuse] || Math.floor(Math.random() * 50) + 30;
        }
        
        async function generateResponse() {
            const input = document.getElementById("situation").value;
            const responseDiv = document.getElementById("response");
            const loadingDiv = document.getElementById("loading");
            const speakButton = document.getElementById("speakButton");
            
            if (!input.trim()) {
                responseDiv.innerHTML = "<p class='text-red-500 font-semibold'>⚠️ Please enter or speak a situation.</p>";
                responseDiv.classList.remove("hidden");
                return;
            }
            
            loadingDiv.classList.remove("hidden");
            responseDiv.innerHTML = "";
            responseDiv.classList.add("hidden");
            speakButton.classList.add("hidden");
            
            try {
                const response = await axios.post('https://api.openai.com/v1/chat/completions', {
                    model: "gpt-4",
                    messages: [{role: "system", content: "You are an AI reasoning assistant that provides insightful responses."},
                               {role: "user", content: input}],
                    max_tokens: 250
                }, {
                    headers: {
                        'Authorization': `Bearer YOUR_OPENAI_API_KEY`,
                        'Content-Type': 'application/json'
                    }
                });
                
                const outputText = response.data.choices[0].message.content.trim();
                const probability = getExcuseProbability(outputText);
                responseDiv.innerHTML = `<p class='font-semibold text-blue-700'>💡 AI Response:</p><p>${outputText}</p><p class='font-bold text-green-600'>📊 Likelihood of Acceptance: ${probability}%</p>`;
                responseDiv.classList.remove("hidden");
                speakButton.classList.remove("hidden");
                speakButton.setAttribute("data-text", outputText);
            } catch (error) {
                responseDiv.innerHTML = "<p class='text-red-500 font-semibold'>❌ Error generating response. Please try again later.</p>";
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
