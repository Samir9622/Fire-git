<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Firebase Authentication with GitHub Pages</title>
    
    <!-- Tailwind CSS via jsDelivr -->
    <script src="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.js"></script>
    
    <!-- Firebase App (the core Firebase SDK) -->
    <script src="https://cdn.jsdelivr.net/npm/firebase@9.22.0/firebase-app-compat.js"></script>
    
    <!-- Firebase Auth -->
    <script src="https://cdn.jsdelivr.net/npm/firebase@9.22.0/firebase-auth-compat.js"></script>
    
    <!-- Firebase Firestore (optional, for user data) -->
    <script src="https://cdn.jsdelivr.net/npm/firebase@9.22.0/firebase-firestore-compat.js"></script>
    
    <style>
        .loader {
            border: 5px solid #f3f3f3;
            border-top: 5px solid #3498db;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex flex-col items-center justify-center p-4">
    <div class="bg-white rounded-lg shadow-xl p-8 max-w-md w-full">
        <h1 class="text-2xl font-bold text-center mb-6">Firebase Authentication</h1>
        
        <!-- Auth status container -->
        <div id="auth-status" class="mb-6 text-center">Checking authentication status...</div>
        <div id="loader" class="loader"></div>
        
        <!-- Login container - shown when user is not logged in -->
        <div id="login-container" class="hidden flex flex-col items-center">
            <p class="mb-4 text-gray-700">Please sign in to continue:</p>
            <button id="google-signin" class="bg-white border border-gray-300 rounded-lg px-6 py-3 flex items-center justify-center hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition-colors w-full mb-4">
                <svg class="w-5 h-5 mr-2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                    <path d="M12.24 10.285V14.4h6.806c-.275 1.765-2.056 5.174-6.806 5.174-4.095 0-7.439-3.389-7.439-7.574s3.345-7.574 7.439-7.574c2.33 0 3.891.989 4.785 1.849l3.254-3.138C18.189 1.186 15.479 0 12.24 0c-6.635 0-12 5.365-12 12s5.365 12 12 12c6.926 0 11.52-4.869 11.52-11.726 0-.788-.085-1.39-.189-1.989H12.24z" fill="#4285F4"/>
                    <path d="M12.24 10.285V14.4h6.806c-.275 1.765-2.056 5.174-6.806 5.174-4.095 0-7.439-3.389-7.439-7.574s3.345-7.574 7.439-7.574c2.33 0 3.891.989 4.785 1.849l3.254-3.138C18.189 1.186 15.479 0 12.24 0c-6.635 0-12 5.365-12 12s5.365 12 12 12c6.926 0 11.52-4.869 11.52-11.726 0-.788-.085-1.39-.189-1.989H12.24z" fill="#34A853" transform="translate(0 24) scale(1 -1)"/>
                </svg>
                Sign in with Google
            </button>
        </div>
        
        <!-- User info container - shown when user is logged in -->
        <div id="user-container" class="hidden flex flex-col items-center">
            <div class="mb-4 flex flex-col items-center">
                <img id="user-photo" class="w-20 h-20 rounded-full mb-2" src="" alt="Profile Picture">
                <h2 id="user-name" class="text-xl font-semibold"></h2>
                <p id="user-email" class="text-gray-600"></p>
            </div>
            <button id="sign-out" class="bg-red-500 hover:bg-red-600 text-white rounded-lg px-6 py-3 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 transition-colors">Sign Out</button>
        </div>
        
        <!-- Status messages -->
        <div id="status-messages" class="mt-6 text-sm text-center"></div>
    </div>
    
    <div class="mt-8 text-center text-sm text-gray-500">
        <p>Note: Make sure to configure your Firebase project with the correct authorized domains</p>
    </div>

    <script>
        // Your Firebase configuration object
        // IMPORTANT: Replace with your actual Firebase project config
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "shrinkwrap-tnjfi.firebaseapp.com", // Change this to your GitHub Pages domain when deployed
            projectId: "shrinkwrap-tnjfi",
            storageBucket: "shrinkwrap-tnjfi.appspot.com",
            messagingSenderId: "617549612593",
            appId: "1:617549612593:web:9a705c4e804108bed1b710"
        };
        
        // Function to detect if we're running on GitHub Pages
        function isGitHubPages() {
            return window.location.hostname.includes('github.io');
        }
        
        // Adjust the authDomain if we're on GitHub Pages
        if (isGitHubPages()) {
            // Extract your username and repo from the URL
            const pathParts = window.location.pathname.split('/');
            if (pathParts.length >= 2) {
                const repoName = pathParts[1];
                // Set the authDomain to the GitHub Pages domain
                firebaseConfig.authDomain = window.location.hostname;
                
                // Log for debugging
                logStatus(`Running on GitHub Pages with domain: ${firebaseConfig.authDomain}`);
            }
        }
        
        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const auth = firebase.auth();
        
        // DOM elements
        const authStatus = document.getElementById('auth-status');
        const loader = document.getElementById('loader');
        const loginContainer = document.getElementById('login-container');
        const userContainer = document.getElementById('user-container');
        const userPhoto = document.getElementById('user-photo');
        const userName = document.getElementById('user-name');
        const userEmail = document.getElementById('user-email');
        const googleSignInButton = document.getElementById('google-signin');
        const signOutButton = document.getElementById('sign-out');
        const statusMessages = document.getElementById('status-messages');
        
        // Function to log status messages
        function logStatus(message) {
            const timestamp = new Date().toLocaleTimeString();
            const entry = document.createElement('div');
            entry.textContent = `[${timestamp}] ${message}`;
            statusMessages.prepend(entry);
            console.log(`[${timestamp}] ${message}`);
        }
        
        // Check if we have a pending redirect operation
        function checkRedirectResult() {
            logStatus('Checking for redirect result...');
            
            auth.getRedirectResult()
                .then((result) => {
                    loader.classList.add('hidden');
                    
                    if (result.user) {
                        logStatus('Successfully signed in after redirect!');
                        updateUIForUser(result.user);
                    } else {
                        logStatus('No redirect result or already processed');
                        // Just continue with normal auth state check
                    }
                })
                .catch((error) => {
                    loader.classList.add('hidden');
                    logStatus(`Error after redirect: ${error.message}`);
                    
                    if (error.code === 'auth/account-exists-with-different-credential') {
                        logStatus('You have already signed up with a different auth provider for that email.');
                    } else {
                        console.error('Redirect error:', error);
                    }
                    
                    loginContainer.classList.remove('hidden');
                });
        }
        
        // Update UI for a signed-in user
        function updateUIForUser(user) {
            authStatus.textContent = 'Signed in';
            userContainer.classList.remove('hidden');
            loginContainer.classList.add('hidden');
            
            if (user.photoURL) {
                userPhoto.src = user.photoURL;
            } else {
                userPhoto.src = 'https://cdn.jsdelivr.net/npm/@tailwindcss/custom-forms@0.2.1/dist/user.svg';
            }
            
            userName.textContent = user.displayName || 'User';
            userEmail.textContent = user.email || '';
            
            logStatus(`Signed in as: ${user.email}`);
        }
        
        // Update UI for signed-out user
        function updateUIForSignedOut() {
            authStatus.textContent = 'Signed out';
            userContainer.classList.add('hidden');
            loginContainer.classList.remove('hidden');
            logStatus('User signed out');
        }
        
        // Listen for auth state changes
        auth.onAuthStateChanged((user) => {
            loader.classList.add('hidden');
            
            if (user) {
                updateUIForUser(user);
            } else {
                updateUIForSignedOut();
            }
        });
        
        // Set up sign-in with Google
        googleSignInButton.addEventListener('click', () => {
            logStatus('Starting Google sign-in with redirect...');
            loader.classList.remove('hidden');
            
            const provider = new firebase.auth.GoogleAuthProvider();
            provider.addScope('profile');
            provider.addScope('email');
            
            // Optional: Specify language
            auth.useDeviceLanguage();
            
            // Sign in with redirect, properly configured for GitHub Pages
            auth.signInWithRedirect(provider)
                .catch((error) => {
                    loader.classList.add('hidden');
                    logStatus(`Error starting redirect: ${error.message}`);
                    console.error('Sign-in redirect error:', error);
                });
        });
        
        // Set up sign-out
        signOutButton.addEventListener('click', () => {
            logStatus('Signing out...');
            auth.signOut();
        });
        
        // Check for redirect result on page load
        checkRedirectResult();
        
        // Handle any callback errors that might occur with the GitHub Pages URL
        window.addEventListener('error', (e) => {
            if (e.message && (e.message.includes('auth') || e.message.includes('firebase'))) {
                logStatus(`Error caught: ${e.message}`);
            }
        });
    </script>
</body>
</html>
