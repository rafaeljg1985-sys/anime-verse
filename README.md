<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Animé-Web</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <style>
        body {
            font-family: 'Poppins', sans-serif;
            background-color: #1a202c;
            color: #e2e8f0;
        }
        .container-wrapper {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        .container {
            width: 100%;
            max-width: 1200px;
            padding: 1rem;
        }
        .hero-section {
            background: linear-gradient(rgba(0,0,0,0.5), rgba(0,0,0,0.5)), url('https://placehold.co/1200x600/1a202c/e2e8f0?text=Anime+Background') no-repeat center center/cover;
            height: 400px;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
        }
        .hero-title {
            font-size: 3rem;
            font-weight: 700;
            color: #fff;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
        .forum-container {
            background-color: #2d3748;
            border-radius: 1rem;
            padding: 1.5rem;
        }
        .forum-input {
            border-radius: 0.5rem;
            padding: 0.5rem;
            border: none;
            background-color: #4a5568;
            color: #e2e8f0;
        }
        .forum-post {
            background-color: #4a5568;
            border-radius: 0.75rem;
            padding: 1rem;
            margin-bottom: 1rem;
        }
        .forum-user-id {
            font-size: 0.75rem;
            color: #a0aec0;
            margin-bottom: 0.5rem;
        }
        .video-container {
            position: relative;
            padding-bottom: 56.25%; /* 16:9 Aspect Ratio */
            height: 0;
            overflow: hidden;
            border-radius: 1rem;
        }
        .video-container iframe {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body class="bg-gray-900 text-gray-200">

    <div id="loading" class="fixed inset-0 bg-gray-900 flex items-center justify-center z-50">
        <div class="text-white text-xl">Cargando...</div>
    </div>

    <div class="container-wrapper">
        <div class="container mx-auto p-4">

            <!-- Hero Section -->
            <header class="hero-section rounded-2xl mb-8">
                <h1 class="hero-title">Explora el mundo del anime</h1>
            </header>

            <!-- Video Player Section -->
            <section id="video-section" class="mb-8">
                <h2 class="text-3xl font-semibold mb-4">Ver Episodios</h2>
                <div class="video-container">
                    <iframe id="anime-player" src="https://www.youtube.com/embed/dQw4w9WgXcQ" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
                </div>
                <div class="flex justify-center mt-4 space-x-4">
                    <button onclick="playEpisode('dQw4w9WgXcQ')" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-xl shadow-lg">Episodio 1</button>
                    <button onclick="playEpisode('iMv1N7wS-tM')" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-xl shadow-lg">Episodio 2</button>
                    <button onclick="playEpisode('ZkE047W-3Qc')" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-xl shadow-lg">Episodio 3</button>
                </div>
            </section>

            <!-- Forum Section -->
            <section id="forum-section" class="forum-container mb-8">
                <h2 class="text-3xl font-semibold mb-4">Foro de Discusión</h2>
                <div id="user-info" class="mb-4 text-center">
                    <p class="text-sm">ID de Usuario: <span id="user-id" class="font-mono bg-gray-700 rounded-lg px-2 py-1">Cargando...</span></p>
                </div>

                <div class="flex items-center mb-4 space-x-2">
                    <input type="text" id="forum-input" class="flex-grow forum-input focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-300" placeholder="Escribe tu comentario...">
                    <button id="post-button" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-xl shadow-lg transition duration-300">
                        Publicar
                    </button>
                </div>

                <div id="posts-container" class="mt-4 max-h-96 overflow-y-auto">
                    <!-- Posts will be loaded here by JavaScript -->
                </div>
            </section>

        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithCustomToken, signInAnonymously } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, orderBy, serverTimestamp, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables for Firebase
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Initialize Firebase
        let app;
        let auth;
        let db;
        let userId;

        setLogLevel('debug'); // To view firebase logs in the console

        async function initFirebase() {
            try {
                if (Object.keys(firebaseConfig).length === 0) {
                    console.error("Firebase config is not available. Some features may not work.");
                    return;
                }
                app = initializeApp(firebaseConfig);
                auth = getAuth(app);
                db = getFirestore(app);

                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                userId = auth.currentUser.uid;
                document.getElementById('user-id').textContent = userId;

                // Hide loading screen after authentication
                document.getElementById('loading').style.display = 'none';

                setupForum();

            } catch (error) {
                console.error("Firebase initialization failed:", error);
                document.getElementById('loading').style.display = 'none';
            }
        }

        function playEpisode(youtubeId) {
            const player = document.getElementById('anime-player');
            player.src = `https://www.youtube.com/embed/${youtubeId}`;
        }

        function setupForum() {
            if (!db) {
                console.error("Firestore is not initialized.");
                return;
            }

            const forumInput = document.getElementById('forum-input');
            const postButton = document.getElementById('post-button');
            const postsContainer = document.getElementById('posts-container');
            const collectionPath = `artifacts/${appId}/public/data/forum_posts`;

            postButton.addEventListener('click', async () => {
                const text = forumInput.value.trim();
                if (text) {
                    try {
                        const newPost = {
                            userId: userId,
                            text: text,
                            timestamp: serverTimestamp()
                        };
                        await addDoc(collection(db, collectionPath), newPost);
                        forumInput.value = '';
                    } catch (error) {
                        console.error("Error al publicar el mensaje:", error);
                    }
                }
            });

            // Listen for new posts
            onSnapshot(collection(db, collectionPath), (snapshot) => {
                const posts = [];
                snapshot.forEach(doc => {
                    const data = doc.data();
                    posts.push({ id: doc.id, ...data });
                });
                
                // Sort posts in memory by timestamp in descending order
                posts.sort((a, b) => b.timestamp - a.timestamp);

                postsContainer.innerHTML = '';
                posts.forEach(post => {
                    const postElement = document.createElement('div');
                    postElement.classList.add('forum-post');
                    postElement.innerHTML = `
                        <p class="forum-user-id">${post.userId}</p>
                        <p>${post.text}</p>
                    `;
                    postsContainer.appendChild(postElement);
                });
            });
        }

        window.onload = initFirebase;
        window.playEpisode = playEpisode;

    </script>
</body>
</html>
