<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Image Definition Match</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom styles for Inter font */
        body {
            font-family: "Inter", sans-serif;
        }
        /* Style for draggable definitions */
        .definition-card {
            cursor: grab;
            transition: transform 0.2s ease-in-out, background-color 0.2s ease-in-out;
        }
        .definition-card:active {
            cursor: grabbing;
        }
        /* Style for drop targets */
        .image-card.drag-over {
            border: 3px dashed #6366f1; /* Indigo 500 */
            background-color: #e0e7ff; /* Indigo 100 */
        }
        /* Style for matched elements */
        .matched {
            pointer-events: none; /* Disable interaction after match */
            opacity: 0.7; /* Slightly dim matched items */
        }
        .matched .definition-card {
            background-color: #d1fae5; /* Green 100 */
            border-color: #34d399; /* Green 400 */
            color: #065f46; /* Green 800 */
        }
        .matched .image-card {
            border-color: #34d399; /* Green 400 */
        }
    </style>
</head>
<body class="bg-gradient-to-br from-indigo-50 to-purple-100 min-h-screen flex flex-col items-center justify-center p-4">

    <!-- Main Game Container -->
    <div class="bg-white rounded-2xl shadow-xl p-8 max-w-4xl w-full space-y-8">
        <h1 class="text-4xl font-extrabold text-center text-indigo-700 mb-8">
            Match Definitions to Images
        </h1>

        <!-- Message Box -->
        <div id="message-box" class="p-4 rounded-xl text-center font-medium transition-all duration-300 ease-in-out hidden"
             role="alert">
            <!-- Messages will be displayed here -->
        </div>

        <!-- Image Container (Drop Targets) -->
        <div id="image-container" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 mb-8">
            <!-- Images will be dynamically loaded here -->
        </div>

        <!-- Definition Container (Draggable Items) -->
        <div id="definition-container" class="flex flex-wrap justify-center gap-4 mb-8">
            <!-- Definitions will be dynamically loaded here -->
        </div>

        <!-- Reset Button -->
        <div class="text-center">
            <button id="reset-button"
                    class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-8 rounded-full shadow-lg transform transition duration-300 ease-in-out hover:scale-105 active:scale-95 focus:outline-none focus:ring-4 focus:ring-indigo-300">
                Reset Game
            </button>
        </div>
    </div>

    <script type="module">
        // JavaScript for the drag-and-drop game logic

        // Data for the game: objects with id, imageUrl, and definition
        const gameData = [
            { id: '1', imageUrl: '"C:\Users\yossi\Downloads\acanthodactylus-scutellatus_image-2_6gPVmzA.png"', definition: 'שנונית החולות' },
            { id: '2', imageUrl: 'https://placehold.co/150x150/33FF57/FFFFFF?text=Tree', definition: 'A tall plant with a woody stem, branches, and leaves, typically perennial.' },
            { id: '3', imageUrl: 'https://placehold.co/150x150/5733FF/FFFFFF?text=Book', definition: 'A written or printed work consisting of pages glued or sewn together along one side and bound in covers.' },
            { id: '4', imageUrl: 'https://placehold.co/150x150/FF33A8/FFFFFF?text=Sun', definition: 'The star that the Earth orbits, providing light and heat to our planet.' },
            { id: '5', imageUrl: 'https://placehold.co/150x150/33A8FF/FFFFFF?text=Cloud', definition: 'A visible mass of condensed water vapor floating in the atmosphere, typically white or gray.' },
            { id: '6', imageUrl: 'https://placehold.co/150x150/A8FF33/FFFFFF?text=Mountain', definition: 'A large natural elevation of the Earth\'s surface rising abruptly from the surrounding level; a large steep hill.' },
        ];

        // DOM elements
        const imageContainer = document.getElementById('image-container');
        const definitionContainer = document.getElementById('definition-container');
        const messageBox = document.getElementById('message-box');
        const resetButton = document.getElementById('reset-button');

        // Store the ID of the definition being dragged
        let draggedDefinitionId = null;

        // Keep track of successful matches to disable further interaction
        let matchedPairs = {}; // { imageId: definitionId }

        /**
         * Shuffles an array in place (Fisher-Yates shuffle).
         * @param {Array} array The array to shuffle.
         * @returns {Array} The shuffled array.
         */
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]]; // Swap elements
            }
            return array;
        }

        /**
         * Displays a message in the message box.
         * @param {string} message The message to display.
         * @param {'success'|'error'|'info'} type The type of message for styling.
         */
        function showMessage(message, type) {
            messageBox.textContent = message;
            messageBox.classList.remove('hidden', 'bg-green-100', 'text-green-800', 'bg-red-100', 'text-red-800', 'bg-blue-100', 'text-blue-800', 'border-green-400', 'border-red-400', 'border-blue-400');
            messageBox.classList.add('border'); // Add border for all message types

            if (type === 'success') {
                messageBox.classList.add('bg-green-100', 'text-green-800', 'border-green-400');
            } else if (type === 'error') {
                messageBox.classList.add('bg-red-100', 'text-red-800', 'border-red-400');
            } else { // info
                messageBox.classList.add('bg-blue-100', 'text-blue-800', 'border-blue-400');
            }

            // Hide message after a few seconds
            setTimeout(() => {
                messageBox.classList.add('hidden');
            }, 3000);
        }

        /**
         * Handles the start of a drag operation.
         * Stores the ID of the dragged definition.
         * @param {DragEvent} event
         */
        function dragStart(event) {
            draggedDefinitionId = event.target.dataset.id;
            event.dataTransfer.setData('text/plain', draggedDefinitionId); // Required for Firefox
            event.target.classList.add('opacity-50'); // Visual feedback for dragging
        }

        /**
         * Handles the end of a drag operation.
         * Cleans up visual feedback.
         * @param {DragEvent} event
         */
        function dragEnd(event) {
            event.target.classList.remove('opacity-50');
        }

        /**
         * Handles dragover event on drop targets.
         * Prevents default to allow dropping.
         * @param {DragEvent} event
         */
        function dragOver(event) {
            event.preventDefault();
            // Add visual feedback to the drop target
            if (event.target.closest('.image-card') && !event.target.closest('.image-card').classList.contains('matched')) {
                event.target.closest('.image-card').classList.add('drag-over');
            }
        }

        /**
         * Handles dragleave event on drop targets.
         * Removes visual feedback.
         * @param {DragEvent} event
         */
        function dragLeave(event) {
            if (event.target.closest('.image-card')) {
                event.target.closest('.image-card').classList.remove('drag-over');
            }
        }

        /**
         * Handles the drop event on image cards.
         * Checks if the dropped definition matches the image.
         * @param {DragEvent} event
         */
        function drop(event) {
            event.preventDefault();
            const imageCard = event.target.closest('.image-card');

            // Remove drag-over class regardless of drop validity
            if (imageCard) {
                imageCard.classList.remove('drag-over');
            }

            if (!imageCard || imageCard.classList.contains('matched')) {
                // If not dropped on an image card or if already matched
                showMessage('That\'s not a valid place to drop!', 'error');
                return;
            }

            const droppedDefinitionId = event.dataTransfer.getData('text/plain');
            const targetImageId = imageCard.dataset.id;

            if (droppedDefinitionId === targetImageId) {
                // Correct match
                showMessage('Correct! Great job!', 'success');

                const definitionCard = document.querySelector(`.definition-card[data-id="${droppedDefinitionId}"]`);

                // Move the definition card into the image card
                imageCard.appendChild(definitionCard);
                definitionCard.classList.add('absolute', 'bottom-2', 'left-2', 'right-2', 'p-2', 'text-xs', 'bg-white', 'rounded-md', 'shadow-md', 'z-10'); // Ensure definition is above other elements
                definitionCard.classList.remove('definition-card', 'flex', 'items-center', 'justify-center', 'text-center', 'p-4', 'bg-purple-50', 'text-purple-800', 'border', 'border-purple-200', 'rounded-xl', 'shadow-sm', 'hover:shadow-md');
                definitionCard.draggable = false; // Make it non-draggable once matched

                // Mark both as matched
                imageCard.classList.add('matched');
                definitionCard.classList.add('matched');

                // Store the match
                matchedPairs[targetImageId] = droppedDefinitionId;

                checkAllMatched();

            } else {
                // Incorrect match
                showMessage('Incorrect match, try again!', 'error');
            }
        }

        /**
         * Checks if all pairs have been matched.
         */
        function checkAllMatched() {
            if (Object.keys(matchedPairs).length === gameData.length) {
                showMessage('Congratulations! You\'ve matched all the pairs!', 'success');
            }
        }

        /**
         * Initializes or resets the game.
         */
        function initGame() {
            // Clear containers
            imageContainer.innerHTML = '';
            definitionContainer.innerHTML = '';
            messageBox.classList.add('hidden');
            matchedPairs = {}; // Reset matched pairs

            // Shuffle data for random order of images and definitions
            const shuffledImages = shuffleArray([...gameData]);
            const shuffledDefinitions = shuffleArray([...gameData]);

            // Create and append image cards
            shuffledImages.forEach(item => {
                const imageCard = document.createElement('div');
                imageCard.dataset.id = item.id;
                imageCard.classList.add(
                    'image-card', 'relative', 'bg-white', 'rounded-xl', 'shadow-md', 'flex', 'flex-col', 'items-center', 'justify-center', 'p-4', 'border-2', 'border-gray-200', 'transition-all', 'duration-200', 'ease-in-out', 'min-h-[200px]', 'max-h-[250px]', 'overflow-hidden'
                );

                // Add drop event listeners to image cards
                imageCard.addEventListener('dragover', dragOver);
                imageCard.addEventListener('dragleave', dragLeave);
                imageCard.addEventListener('drop', drop);

                const img = document.createElement('img');
                img.src = item.imageUrl;
                img.alt = `Image for ${item.definition.substring(0, 20)}...`;
                img.classList.add('w-full', 'h-32', 'object-contain', 'mb-2', 'rounded-lg');
                img.onerror = () => {
                    img.src = `https://placehold.co/150x150/CCCCCC/000000?text=Image+Error`; // Fallback image
                    showMessage(`Could not load image for ID ${item.id}. Using a placeholder.`, 'info');
                };

                // Add image to the card
                imageCard.appendChild(img);
                imageContainer.appendChild(imageCard);
            });

            // Create and append definition cards
            shuffledDefinitions.forEach(item => {
                const definitionCard = document.createElement('div');
                definitionCard.dataset.id = item.id;
                definitionCard.draggable = true; // Make draggable
                definitionCard.classList.add(
                    'definition-card', 'flex', 'items-center', 'justify-center', 'text-center',
                    'p-4', 'bg-purple-50', 'text-purple-800', 'border', 'border-purple-200',
                    'rounded-xl', 'shadow-sm', 'hover:shadow-md', 'cursor-grab',
                    'transition-all', 'duration-200', 'ease-in-out',
                    'min-w-[150px]', 'max-w-[200px]'
                );
                definitionCard.textContent = item.definition;

                // Add drag event listeners to definition cards
                definitionCard.addEventListener('dragstart', dragStart);
                definitionCard.addEventListener('dragend', dragEnd);

                definitionContainer.appendChild(definitionCard);
            });
        }

        // Initialize the game when the window loads
        window.onload = initGame;

        // Add event listener for the reset button
        resetButton.addEventListener('click', initGame);
    </script>
</body>
</html>
