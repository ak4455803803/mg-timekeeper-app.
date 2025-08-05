<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MGtimekeeper</title>
    <!-- Tailwind CSS CDN for easy, responsive styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles to ensure the app looks good on all devices */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        #app-container {
            max-width: 420px; /* Mobile-first design */
            width: 100%;
            height: 100%;
            background-color: white;
            display: flex;
            flex-direction: column;
            border-radius: 1rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        .screen {
            display: none;
            flex: 1;
            padding: 1.5rem;
            justify-content: center;
            align-items: center;
            flex-direction: column;
        }
        .screen.active {
            display: flex;
        }
        .btn {
            @apply p-4 rounded-lg font-bold text-white text-lg transition-all duration-300 transform hover:scale-105 shadow-md;
        }
    </style>
</head>
<body>

<div id="app-container">

    <!-- Screen 1: Initial Menu -->
    <div id="initial-screen" class="screen active">
        <h1 class="text-3xl font-bold mb-8">MGtimekeeper</h1>
        <button id="capture-pic-btn" class="btn bg-blue-600 w-full mb-4">Capture Pic</button>
        <button id="exit-btn" class="btn bg-gray-500 w-full">Exit</button>
    </div>

    <!-- Screen 2: Camera View -->
    <div id="camera-screen" class="screen">
        <h2 class="text-2xl font-bold mb-4">Front Camera</h2>
        <div class="relative w-full h-80 rounded-lg overflow-hidden border-4 border-gray-300 mb-8">
            <video id="camera-feed" class="w-full h-full object-cover"></video>
            <canvas id="photo-canvas" class="hidden"></canvas>
            <div id="camera-placeholder" class="absolute inset-0 flex items-center justify-center bg-gray-200">
                <span class="text-gray-500 text-lg">Camera is loading...</span>
            </div>
        </div>
        <button id="take-photo-btn" class="btn bg-green-600 w-full">Take Photo</button>
    </div>

    <!-- Screen 3: Confirm Photo -->
    <div id="confirm-screen" class="screen">
        <h2 class="text-2xl font-bold mb-4">Confirm Photo</h2>
        <img id="captured-photo" class="w-full h-64 object-contain rounded-lg border-4 border-gray-300 mb-8" alt="Captured photo">
        <div class="flex w-full justify-between">
            <button id="confirm-btn" class="btn bg-green-600 w-1/2 mr-2">Confirm</button>
            <button id="retake-btn" class="btn bg-red-600 w-1/2 ml-2">Retake</button>
        </div>
    </div>

    <!-- Screen 4: Message and Status -->
    <div id="message-screen" class="screen">
        <h2 id="status-message" class="text-3xl text-center font-bold mb-8"></h2>
        <button id="back-btn" class="btn bg-gray-500 w-full">Back</button>
    </div>

</div>

<script>
    // Get all screen and button elements
    const initialScreen = document.getElementById('initial-screen');
    const cameraScreen = document.getElementById('camera-screen');
    const confirmScreen = document.getElementById('confirm-screen');
    const messageScreen = document.getElementById('message-screen');

    const capturePicBtn = document.getElementById('capture-pic-btn');
    const exitBtn = document.getElementById('exit-btn');
    const takePhotoBtn = document.getElementById('take-photo-btn');
    const confirmBtn = document.getElementById('confirm-btn');
    const retakeBtn = document.getElementById('retake-btn');
    const backBtn = document.getElementById('back-btn');

    const cameraFeed = document.getElementById('camera-feed');
    const photoCanvas = document.getElementById('photo-canvas');
    const capturedPhotoImg = document.getElementById('captured-photo');
    const statusMessage = document.getElementById('status-message');
    const cameraPlaceholder = document.getElementById('camera-placeholder');

    let capturedPhotoData = null; // Stores the Base64 image data
    let cameraStream = null; // Stores the camera stream to stop it later

    // Function to switch between different screens
    function showScreen(screen) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        screen.classList.add('active');
    }

    // Event listener for the "Capture Pic" button
    capturePicBtn.addEventListener('click', async () => {
        showScreen(cameraScreen);
        await startCamera();
    });

    // Event listener for the "Exit" button
    exitBtn.addEventListener('click', () => {
        alert('Exiting the app.');
        // In a real mobile app, this would close the app.
        // For a web page, we can simply close the window.
        // window.close(); // Note: This may not work in all browsers.
    });

    // Event listener for the "Take Photo" button
    takePhotoBtn.addEventListener('click', () => {
        // Stop the camera stream
        stopCamera();

        // Draw the video frame to the canvas
        photoCanvas.width = cameraFeed.videoWidth;
        photoCanvas.height = cameraFeed.videoHeight;
        const context = photoCanvas.getContext('2d');
        context.drawImage(cameraFeed, 0, 0, photoCanvas.width, photoCanvas.height);

        // Get the image data as a Base64 string
        capturedPhotoData = photoCanvas.toDataURL('image/png');

        // Display the captured image on the confirm screen
        capturedPhotoImg.src = capturedPhotoData;
        showScreen(confirmScreen);
    });

    // Event listener for the "Retake" button
    retakeBtn.addEventListener('click', () => {
        capturedPhotoData = null;
        showScreen(initialScreen);
    });

    // Event listener for the "Confirm" button
    confirmBtn.addEventListener('click', async () => {
        confirmBtn.innerText = 'Sending...';
        confirmBtn.disabled = true;

        // --- Step 1: Check for internet connectivity ---
        if (!navigator.onLine) {
            displayMessage("Connect to internet", "red");
            confirmBtn.innerText = 'Confirm';
            confirmBtn.disabled = false;
            return;
        }

        // --- Step 2: Get GPS location ---
        try {
            const position = await getGpsLocation();
            const location = {
                lat: position.coords.latitude,
                long: position.coords.longitude
            };
            const timestamp = new Date().toLocaleString();

            // --- Step 3: Simulate sending photo, GPS, and timestamp ---
            // IMPORTANT: A real email sending mechanism requires a secure backend server.
            // This part of the code simulates success for demonstration purposes.
            console.log('Simulating attendance marking...');
            console.log('Photo data:', capturedPhotoData ? 'present' : 'missing');
            console.log('GPS Location:', location);
            console.log('Timestamp:', timestamp);

            // Simulate a network delay for a more realistic feel
            await new Promise(resolve => setTimeout(resolve, 1500));

            displayMessage("Attendance marked", "green");

        } catch (error) {
            if (error.message.includes("location services")) {
                displayMessage("Enable location services", "red");
            } else {
                console.error("Error during attendance submission:", error);
                displayMessage("Failed to mark attendance. Please try again.", "red");
            }
        } finally {
            confirmBtn.innerText = 'Confirm';
            confirmBtn.disabled = false;
        }
    });

    // Event listener for the "Back" button
    backBtn.addEventListener('click', () => {
        showScreen(initialScreen);
        statusMessage.textContent = '';
    });

    /**
     * Helper function to start the front camera.
     */
    async function startCamera() {
        try {
            cameraFeed.style.display = 'block';
            cameraPlaceholder.style.display = 'none';
            const constraints = {
                video: { facingMode: 'user' }
            };
            cameraStream = await navigator.mediaDevices.getUserMedia(constraints);
            cameraFeed.srcObject = cameraStream;
            await cameraFeed.play();
        } catch (err) {
            console.error("Error accessing camera:", err);
            let errorMessage = "Could not access camera. Please check permissions.";
            if (err.name === "NotAllowedError") {
                errorMessage = "Camera access denied. Please allow camera access in your browser settings.";
            } else if (err.name === "NotFoundError" || err.name === "DevicesNotFoundError") {
                errorMessage = "No camera found on this device.";
            } else if (err.name === "NotReadableError" || err.name === "TrackStartError") {
                errorMessage = "Camera is already in use or cannot be started.";
            } else if (err.name === "OverconstrainedError") {
                errorMessage = "Camera constraints not supported by your device.";
            } else if (location.protocol !== 'https:') {
                errorMessage += " Camera access often requires a secure (HTTPS) connection. Try opening this page via a web server.";
            }
            cameraPlaceholder.innerHTML = `<span class="text-red-500">${errorMessage}</span>`;
            cameraPlaceholder.style.display = 'flex'; // Ensure placeholder is visible
            cameraFeed.style.display = 'none'; // Hide video element
        }
    }

    /**
     * Helper function to stop the camera stream.
     */
    function stopCamera() {
        if (cameraStream) {
            cameraStream.getTracks().forEach(track => track.stop());
            cameraFeed.srcObject = null;
        }
    }

    /**
     * Helper function to get the current GPS location.
     */
    function getGpsLocation() {
        return new Promise((resolve, reject) => {
            if (!navigator.geolocation) {
                reject(new Error("Geolocation not supported by your browser."));
            }
            navigator.geolocation.getCurrentPosition(resolve, (error) => {
                reject(new Error("Could not get GPS location. Please enable location services."));
            });
        });
    }

    /**
     * Helper function to display a status message.
     */
    function displayMessage(msg, color = "green") {
        statusMessage.textContent = msg;
        statusMessage.className = `text-3xl text-center font-bold mb-8 ${color === "red" ? "text-red-600" : "text-green-600"}`;
        showScreen(messageScreen);
    }

    // Set up a global alert and confirm replacement
    window.alert = (message) => {
        statusMessage.textContent = message;
        statusMessage.className = "text-3xl text-center font-bold mb-8 text-blue-600";
        showScreen(messageScreen);
    };

</script>

</body>
</html>
