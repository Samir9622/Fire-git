<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Firebase Authentication</title>
    
    <!-- Tailwind CSS via jsDelivr -->
    <script src="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.js"></script>
    
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
        
        /* Additional styles to ensure buttons are clickable */
        .auth-button {
            cursor: pointer;
            transition: all 0.3s ease;
        }
        
        .auth-button:hover {
            opacity: 0.9;
            transform: translateY(-2px);
        }
        
        .auth-button:active {
            transform: translateY(0);
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex flex-col items-center justify-center p-4">
    <div class="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
        <h1 class="text-2xl font-bold mb-6 text-center">Firebase Authentication</h1>
        
        <div id="loader" class="loader"></div>
        
        <div id="auth-status" class="text-center mb-4">Checking authentication status...</div>
        
        <div id="login-container" class="hidden">
            <p class="text-center mb-4">Please sign in to continue:</p>
            
            <!-- Google Sign In Button -->
            <button id="googleSignIn" class="auth-button w-full flex items-center justify-center bg-white border border-gray-300 rounded-lg py-2 px-4 mb-3 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-blue-500">
                <img src="https://www.gstatic.com/firebasejs/ui/2.0.0/images/auth/google.svg" alt="Google" class="h-5 w-5 mr-2">
                <span>Sign in with Google</span>
            </button>
            
            <!-- Add more auth providers as needed -->
            
            <div id="profilePicture" class="mt-4 hidden">
                <img src="" alt="Profile Picture" class="w-16 h-16 rounded-full mx-auto">
            </div>
            
            <button id="signOut" class="auth-button w-full bg-red-500 text-white py-2 px-4 rounded-lg mt-4 hidden hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500">
                Sign Out
            </button>
        </div>
        
        <div class="mt-6 text-sm text-gray-600">
            <p class="text-center">Note: Make sure to configure your Firebase project with the correct authorized domains</p>
        </div>
    </div>

    <!-- Firebase App (the core Firebase SDK) -->
    <script src="https://cdn.jsdelivr.net/npm/firebase@9.22.0/firebase-app-compat.js"></script>

    <!-- Firebase Auth -->
    <script src="https://cdn.jsdelivr.net/npm/firebase@9.22.0/firebase-auth-compat.js"></script>

    <!-- Firebase Firestore (optional, for user data) -->
    <script src="https://cdn.jsdelivr.net/npm/firebase@9.22.0/firebase-firestore-compat.js"></script>
    
    <script>
        // Firebase configuration
        // REPLACE WITH YOUR FIREBASE CONFIG
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
            projectId: "YOUR_PROJECT_ID",
            storageBucket: "YOUR_PROJECT_ID.appspot.com",
            messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
            appId: "YOUR_APP_ID"
        };
        
        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        
        // Debug mode - check if Firebase is properly initialized
        console.log("Firebase initialized:", firebase.app.name);
        
        // References to elements
        const loader = document.getElementById('loader');
        const authStatus = document.getElementById('auth-status');
        const loginContainer = document.getElementById('login-container');
        const googleSignInBtn = document.getElementById('googleSignIn');
        const signOutBtn = document.getElementById('signOut');
        const profilePicture = document.getElementById('profilePicture');
        const profileImg = profilePicture.querySelector('img');
        
        // Show/hide elements based on auth state
        function updateUI(user) {
            loader.style.display = 'none';
            loginContainer.classList.remove('hidden');
            
            if (user) {
                authStatus.textContent = `Signed in as ${user.displayName || user.email}`;
                googleSignInBtn.classList.add('hidden');
                signOutBtn.classList.remove('hidden');
                
                if (user.photoURL) {
                    profilePicture.classList.remove('hidden');
                    profileImg.src = user.photoURL;
                }
            } else {
                authStatus.textContent = 'Please sign in to continue:';
                googleSignInBtn.classList.remove('hidden');
                signOutBtn.classList.add('hidden');
                profilePicture.classList.add('hidden');
            }
        }
        
        // Auth state change listener
        firebase.auth().onAuthStateChanged(function(user) {
            console.log("Auth state changed:", user ? "User signed in" : "User signed out");
            updateUI(user);
        });
        
        // Google Sign In
        googleSignInBtn.addEventListener('click', function() {
            console.log("Google sign-in button clicked");
            loader.style.display = 'block';
            
            const provider = new firebase.auth.GoogleAuthProvider();
            
            // Add additional OAuth scopes if needed
            // provider.addScope('https://www.googleapis.com/auth/contacts.readonly');
            
            // Sign in with popup (better for most web apps)
            firebase.auth().signInWithPopup(provider)
                .then((result) => {
                    console.log("Sign-in successful:", result.user.displayName);
                })
                .catch((error) => {
                    console.error("Error during sign-in:", error);
                    authStatus.textContent = `Error: ${error.message}`;
                    loader.style.display = 'none';
                });
                
            // Alternative: Sign in with redirect
            // firebase.auth().signInWithRedirect(provider);
        });
        
        // Sign Out
        signOutBtn.addEventListener('click', function() {
            console.log("Sign-out button clicked");
            loader.style.display = 'block';
            
            firebase.auth().signOut()
                .then(() => {
                    console.log("Sign-out successful");
                })
                .catch((error) => {
                    console.error("Error during sign-out:", error);
                    authStatus.textContent = `Error: ${error.message}`;
                    loader.style.display = 'none';
                });
        });
        
        // Debugging: Check if buttons have event listeners
        console.log("Google Sign In button:", googleSignInBtn);
        console.log("Sign Out button:", signOutBtn);
        
        // Force reactivation of buttons (in case they're stuck)
        window.addEventListener('load', function() {
            setTimeout(() => {
                if (googleSignInBtn) {
                    googleSignInBtn.disabled = false;
                    console.log("Google Sign In button reactivated");
                }
                if (signOutBtn) {
                    signOutBtn.disabled = false; 
                    console.log("Sign Out button reactivated");
                }
            }, 2000); // Wait 2 seconds after page load
        });
    </script>
</body>
</html>
