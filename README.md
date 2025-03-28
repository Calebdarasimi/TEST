<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>30-Minute Mathematics Test</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
            line-height: 1.6;
        }
        #timer {
            position: fixed;
            top: 10px;
            right: 10px;
            font-size: 24px;
            font-weight: bold;
            color: #d9534f;
            background-color: #f9f9f9;
            padding: 10px 15px;
            border: 2px solid #d9534f;
            border-radius: 5px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }
        .student-info {
            background-color: #f5f5f5;
            padding: 20px;
            border-radius: 5px;
            margin-bottom: 20px;
        }
        .question {
            margin-bottom: 15px;
            padding: 15px;
            background-color: #fff;
            border-left: 4px solid #5bc0de;
            border-radius: 3px;
        }
        #results {
            display: none;
            background-color: #e8f8f5;
            padding: 20px;
            border-radius: 5px;
            margin-top: 20px;
        }
        input[type="text"], input[type="number"], input[type="tel"] {
            padding: 8px;
            margin: 5px 0;
            width: 100%;
            box-sizing: border-box;
        }
        button {
            background-color: #5cb85c;
            color: white;
            border: none;
            padding: 12px 20px;
            font-size: 16px;
            border-radius: 4px;
            cursor: pointer;
            margin-top: 20px;
        }
        button:hover {
            background-color: #4cae4c;
        }
        .warning {
            color: #d9534f;
            font-weight: bold;
        }
        #fullscreen-btn {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background: #337ab7;
        }
    </style>
</head>
<body>
    <div id="timer">30:00</div>
    <button id="fullscreen-btn" onclick="toggleFullscreen()">Fullscreen</button>
    
    <h1>Mathematics Test</h1>
    <p class="warning">Time Allowed: 30 minutes (Auto-submits when time expires)</p>
    
    <form id="testForm">
        <div class="student-info">
            <h2>Student Information</h2>
            <label for="name">Full Name:</label>
            <input type="text" id="name" name="name" required>
            
            <label for="phone">Phone Number:</label>
            <input type="tel" id="phone" name="phone" pattern="[0-9]{10,11}" required>
            
            <label for="email">Email:</label>
            <input type="email" id="email" name="email" required>
        </div>

        <h2>Test Questions (40 Questions)</h2>
        <div id="questions-container">
            <!-- Questions will be inserted here by JavaScript -->
        </div>

        <button type="submit">Submit Test</button>
    </form>

    <div id="results">
        <h2>Test Results</h2>
        <p id="studentDetails"></p>
        <p id="scoreResult"></p>
        <p id="percentageResult"></p>
    </div>

    <script>
        // ====================== CONFIGURATION ======================
        const TEST_DURATION = 1800; // 30 minutes in seconds
        const STORAGE_URL = 'https://your-server-endpoint.com/submit'; // Replace with your endpoint
        let timeLeft = TEST_DURATION;
        let timerActive = true;
        let startTime = new Date().getTime();
        let warningShown = false;

        // ====================== QUESTIONS & ANSWERS ======================
        const allQuestions = [
            {
                question: "1. Solve for x: 3x + 5 = 20",
                type: "number",
                name: "q1",
                answer: "5",
                pattern: null
            },
            {
                question: "2. Find the roots of x² - 5x + 6 = 0 (format: x,y)",
                type: "text",
                name: "q2",
                answer: "2,3",
                pattern: "\\d+,\\d+"
            },
            {
                question: "3. Calculate the area of a circle with radius 7 (include π)",
                type: "text",
                name: "q3",
                answer: "49π",
                pattern: "\\d+π"
            },
            // Add all 40 questions in this format
            // Sample additional questions:
            {
                question: "11. Simplify: (x³)(x⁴)",
                type: "text",
                name: "q11",
                answer: "x⁷",
                pattern: null
            },
            {
                question: "12. What is the prime factorization of 36?",
                type: "text",
                name: "q12",
                answer: "2²×3²",
                pattern: null
            },
            {
                question: "13. Solve: log₅(25)",
                type: "number",
                name: "q13",
                answer: "2",
                pattern: null
            }
            // Continue adding all 40 questions...
        ];

        // ====================== TEST INITIALIZATION ======================
        document.addEventListener('DOMContentLoaded', function() {
            // Randomize question order
            shuffleArray(allQuestions);
            
            // Display questions
            const container = document.getElementById('questions-container');
            allQuestions.forEach((q, index) => {
                container.innerHTML += `
                    <div class="question">
                        <p>${q.question}</p>
                        <input type="${q.type}" name="${q.name}" 
                               ${q.pattern ? `pattern="${q.pattern}"` : ''}
                               ${q.pattern ? `title="Format: ${q.pattern.replace(/\\/g, '')}"` : ''}
                               required>
                    </div>
                `;
            });

            // Start fullscreen and timer
            toggleFullscreen();
            startTimer();
            startActivityMonitor();
        });

        // ====================== TIMER FUNCTIONS ======================
        function startTimer() {
            const timerElement = document.getElementById('timer');
            const timerId = setInterval(function() {
                if (!timerActive) return;
                
                const minutes = Math.floor(timeLeft / 60);
                let seconds = timeLeft % 60;
                seconds = seconds < 10 ? '0' + seconds : seconds;
                
                timerElement.textContent = `${minutes}:${seconds}`;
                
                // Show warning when 5 minutes left
                if (timeLeft <= 300 && !warningShown) {
                    alert("Warning: Only 5 minutes remaining!");
                    warningShown = true;
                }
                
                if (timeLeft <= 0) {
                    timerActive = false;
                    clearInterval(timerId);
                    submitTest();
                }
                timeLeft--;
            }, 1000);
        }

        // ====================== ANTI-CHEATING MEASURES ======================
        function startActivityMonitor() {
            // Tab change detection
            document.addEventListener('visibilitychange', function() {
                if (document.hidden && timerActive) {
                    alert("Warning: You switched tabs! This action has been recorded.");
                    recordEvent("tab_switch");
                }
            });

            // Copy/paste prevention
            document.addEventListener('copy', function(e) {
                e.preventDefault();
                recordEvent("copy_attempt");
            });

            document.addEventListener('paste', function(e) {
                e.preventDefault();
                recordEvent("paste_attempt");
            });

            // Screenshot prevention (not fully reliable but helps)
            document.addEventListener('keydown', function(e) {
                if ((e.ctrlKey && e.key === 'PrintScreen') || e.key === 'PrintScreen') {
                    e.preventDefault();
                    recordEvent("screenshot_attempt");
                }
            });
        }

        function recordEvent(eventType) {
            // Send to server
            const eventData = {
                student: document.getElementById('name').value || 'unknown',
                event: eventType,
                timestamp: new Date().toISOString(),
                timeLeft: timeLeft
            };
            
            // In a real implementation, you would send this to your server
            console.log("Security Event:", eventData);
        }

        // ====================== TEST SUBMISSION ======================
        async function submitTest() {
            timerActive = false;
            
            // Calculate score
            let score = 0;
            const answers = {};
            
            allQuestions.forEach(q => {
                const answer = document.querySelector(`[name="${q.name}"]`).value.trim();
                answers[q.name] = answer;
                
                // Case-insensitive comparison and whitespace removal
                if (answer.toLowerCase().replace(/\s+/g, '') === 
                    q.answer.toLowerCase().replace(/\s+/g, '')) {
                    score++;
                }
            });

            // Calculate time taken
            const endTime = new Date().getTime();
            const timeTaken = Math.round((endTime - startTime) / 1000); // in seconds
            const percentage = (score / allQuestions.length * 100).toFixed(1);

            // Display results
            document.getElementById('studentDetails').textContent = 
                `Student: ${document.getElementById('name').value} | Email: ${document.getElementById('email').value}`;
            
            document.getElementById('scoreResult').textContent = 
                `Score: ${score}/${allQuestions.length}`;
            
            document.getElementById('percentageResult').textContent = 
                `Percentage: ${percentage}%`;
            
            document.getElementById('results').style.display = 'block';
            window.scrollTo(0, document.body.scrollHeight);

            // Send results to server
            await sendResultsToServer({
                name: document.getElementById('name').value,
                email: document.getElementById('email').value,
                phone: document.getElementById('phone').value,
                answers: answers,
                score: score,
                percentage: percentage,
                timeTaken: timeTaken,
                completionTime: new Date().toISOString()
            });
        }

        async function sendResultsToServer(data) {
            try {
                // In a real implementation, you would use:
                // const response = await fetch(STORAGE_URL, {
                //     method: 'POST',
                //     headers: { 'Content-Type': 'application/json' },
                //     body: JSON.stringify(data)
                // });
                
                // For demonstration:
                console.log("Results submitted:", data);
                return true;
            } catch (error) {
                console.error("Error submitting results:", error);
                return false;
            }
        }

        // ====================== UTILITY FUNCTIONS ======================
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        function toggleFullscreen() {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen().catch(err => {
                    console.error(`Error attempting to enable fullscreen: ${err.message}`);
                });
            } else {
                if (document.exitFullscreen) {
                    document.exitFullscreen();
                }
            }
        }

        // ====================== EVENT LISTENERS ======================
        document.getElementById('testForm').addEventListener('submit', function(e) {
            e.preventDefault();
            submitTest();
        });

        document.addEventListener('contextmenu', function(e) {
            e.preventDefault();
            recordEvent("right_click_attempt");
        });

        window.addEventListener('beforeunload', function(e) {
            if (timerActive) {
                e.preventDefault();
                e.returnValue = 'Your test is still in progress. Are you sure you want to leave?';
                recordEvent("early_exit_attempt");
            }
        });
    </script>
</body>
</html>
