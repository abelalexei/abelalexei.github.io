<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Adivina el Cuento del Tesoro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Luckiest+Guy&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .font-luckiest {
            font-family: 'Luckiest Guy', cursive;
        }
        .treasure-chest {
            cursor: pointer;
            transition: transform 0.3s ease-in-out;
        }
        .treasure-chest:hover {
            transform: scale(1.1) rotate(5deg);
        }
        .treasure-chest:active {
            transform: scale(1.05) rotate(0deg);
        }
        .fragment-modal {
            transition: opacity 0.5s ease-in-out, transform 0.5s ease-in-out;
        }
        .hidden-modal {
            opacity: 0;
            transform: scale(0.9);
            pointer-events: none;
        }
        .visible-modal {
            opacity: 1;
            transform: scale(1);
            pointer-events: auto;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-cyan-300 to-blue-500 min-h-screen flex items-center justify-center p-4">

    <main id="game-container" class="w-full max-w-2xl text-center">
        <!-- Título principal -->
        <h1 class="text-5xl md:text-7xl font-luckiest text-white text-shadow-lg drop-shadow-[0_4px_4px_rgba(0,0,0,0.25)] mb-4">
            Adivina el Cuento
        </h1>
        <p id="instruction" class="text-white text-xl mb-8 drop-shadow-md">
            ¡Haz clic en el cofre para descubrir un tesoro de palabras!
        </p>

        <!-- Cofre del Tesoro (Botín) -->
        <div id="treasure-chest-container" class="relative">
            <div id="treasure-chest" class="treasure-chest text-8xl md:text-9xl inline-block">
                <!-- Usamos un emoji como SVG para mejor control y estilo -->
                <svg xmlns="http://www.w3.org/2000/svg" width="150" height="150" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round" class="drop-shadow-2xl">
                    <path d="M20 12H4" style="fill: #a0522d; stroke: #8b4513;"></path>
                    <path d="M18 6.5V18a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2V6.5" style="fill: #d2691e; stroke: #8b4513;"></path>
                    <path d="M22 6.5H2" style="fill: #a0522d; stroke: #8b4513;"></path>
                    <path d="M12 2v4.5" style="fill: #ffd700; stroke: #daa520;"></path>
                    <path d="M10 16h4" style="fill: none; stroke: #8b4513;"></path>
                    <path d="M8 6.5C8 4.5 9.5 3 12 3s4 1.5 4 3.5" style="fill: #a0522d; stroke: #8b4513;"></path>
                </svg>
            </div>
        </div>

        <!-- Modal con el fragmento del cuento -->
        <div id="fragment-modal" class="fragment-modal hidden-modal fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4">
            <div class="bg-yellow-100 border-4 border-yellow-300 rounded-2xl p-6 md:p-8 max-w-xl w-full shadow-2xl relative text-gray-800">
                <button id="close-modal" class="absolute top-2 right-4 text-3xl font-bold text-yellow-600 hover:text-yellow-800">&times;</button>
                <h2 class="text-3xl font-luckiest text-yellow-700 mb-4">Fragmento Misterioso</h2>
                <p id="story-fragment" class="text-lg md:text-xl mb-6 leading-relaxed">
                    Aquí aparecerá un trocito de un cuento muy famoso...
                </p>
                <div class="flex flex-col sm:flex-row gap-4 items-center justify-center">
                    <input type="text" id="guess-input" placeholder="¿Qué cuento es?" class="w-full sm:w-auto flex-grow px-4 py-2 border-2 border-yellow-400 rounded-full focus:outline-none focus:ring-2 focus:ring-yellow-500">
                    <button id="submit-guess" class="w-full sm:w-auto bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-6 rounded-full transition-transform transform hover:scale-105">
                        ¡Adivinar!
                    </button>
                </div>
                <p id="feedback-message" class="mt-4 text-lg font-bold h-6"></p>
            </div>
        </div>

    </main>

    <script>
        const stories = [
            {
                fragment: "En un país muy lejano, vivía una hermosa joven con su madrastra y dos hermanastras que la obligaban a hacer las tareas más duras de la casa, y por eso la llamaban...",
                answer: "cenicienta"
            },
            {
                fragment: "Había una vez una niña muy bonita. Su madre le había hecho una capa roja y la muchachita la la llevaba tan a menudo que todo el mundo la llamaba...",
                answer: "caperucita roja"
            },
            {
                fragment: "Érase una vez tres hermanos cerditos que vivían en el corazón del bosque. El lobo siempre andaba persiguiéndolos para comérselos y tuvieron que construir casas para protegerse.",
                answer: "los tres cerditos"
            },
            {
                fragment: "Una joven princesa es maldecida por un hada malvada: el día de su decimoquinto cumpleaños, se pinchará el dedo con el huso de una rueca y caerá en un profundo sueño...",
                answer: "la bella durmiente"
            },
            {
                fragment: "Era un muñeco de madera, pero soñaba con ser un niño de verdad. Cada vez que decía una mentira, su nariz crecía y crecía.",
                answer: "pinocho"
            },
            {
                fragment: "Tras huir de su malvada madrastra, una joven princesa encontró refugio en una casita en el bosque donde vivían siete hombrecitos que trabajaban en una mina.",
                answer: "blanca nieves"
            }
        ];

        let currentStory = null;

        const treasureChest = document.getElementById('treasure-chest');
        const modal = document.getElementById('fragment-modal');
        const closeModalBtn = document.getElementById('close-modal');
        const storyFragmentEl = document.getElementById('story-fragment');
        const guessInput = document.getElementById('guess-input');
        const submitGuessBtn = document.getElementById('submit-guess');
        const feedbackMessage = document.getElementById('feedback-message');
        const instruction = document.getElementById('instruction');

        function getRandomStory() {
            const randomIndex = Math.floor(Math.random() * stories.length);
            return stories[randomIndex];
        }

        function openModal() {
            currentStory = getRandomStory();
            storyFragmentEl.textContent = currentStory.fragment;
            guessInput.value = '';
            feedbackMessage.textContent = '';
            feedbackMessage.classList.remove('text-green-600', 'text-red-600');
            modal.classList.remove('hidden-modal');
            modal.classList.add('visible-modal');
            instruction.textContent = "¡Lee con atención y adivina el cuento!";
        }

        function closeModal() {
            modal.classList.add('hidden-modal');
            modal.classList.remove('visible-modal');
            instruction.textContent = "¡Haz clic en el cofre para descubrir otro tesoro!";
        }

        function checkGuess() {
            const userGuess = guessInput.value.trim().toLowerCase();
            if (!userGuess) return;

            if (userGuess === currentStory.answer) {
                feedbackMessage.textContent = "¡Correcto! ¡Felicidades!";
                feedbackMessage.classList.add('text-green-600');
                feedbackMessage.classList.remove('text-red-600');
                setTimeout(() => {
                    closeModal();
                }, 2000); // Cierra el modal después de 2 segundos de éxito
            } else {
                feedbackMessage.textContent = "¡Casi! Inténtalo de nuevo.";
                feedbackMessage.classList.add('text-red-600');
                feedbackMessage.classList.remove('text-green-600');
                guessInput.focus();
            }
        }

        treasureChest.addEventListener('click', openModal);
        closeModalBtn.addEventListener('click', closeModal);
        submitGuessBtn.addEventListener('click', checkGuess);

        // Permitir adivinar con la tecla Enter
        guessInput.addEventListener('keyup', (event) => {
            if (event.key === 'Enter') {
                checkGuess();
            }
        });
        
        // Cerrar el modal si se hace clic fuera del contenido
        modal.addEventListener('click', (event) => {
            if (event.target === modal) {
                closeModal();
            }
        });

    </script>
</body>
</html>
