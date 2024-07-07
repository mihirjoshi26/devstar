<script>
    import { onMount } from 'svelte';
    import { gridStore, wordsStore } from './gridStore';
    import { get } from 'svelte/store';
    import { writable } from 'svelte/store';

    let canvas;
    let ctx;
    let grid = [];
    let words = [];
    let cellSize = 40;
    let padding = 5;
    let selectedCells = [];
    let foundWordsCells = [];
    let isDragging = false;
    let foundWords = writable([]);
    let hintIconColor = 'yellow'; // Reactive variable for icon color

    onMount(() => {
        const link = document.createElement('link');
        link.rel = 'stylesheet';
        link.href = 'https://site-assets.fontawesome.com/releases/v6.4.2/css/all.css';
        document.head.appendChild(link);

        grid = get(gridStore);
        words = get(wordsStore);
        foundWords.set(new Array(words.length).fill(false));

        // Clear any highlighted cells when the play page loads
        grid = grid.map(row => row.map(cell => ({ ...cell, highlight: false })));

        canvas.width = grid[0].length * cellSize;
        canvas.height = grid.length * cellSize;
        ctx = canvas.getContext('2d');
        drawGrid();
    });

    function drawGrid() {
        ctx.fillStyle = '#fff';
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        grid.forEach((row, rowIndex) => {
            row.forEach((cell, colIndex) => {
                const x = colIndex * cellSize;
                const y = rowIndex * cellSize;
                
                // Draw cell background
                ctx.fillStyle = cell.highlight ? 'yellow' : 'white';
                ctx.fillRect(x, y, cellSize, cellSize);
                
                // Draw cell border
                ctx.strokeStyle = '#ccc';
                ctx.strokeRect(x, y, cellSize, cellSize);
                
                // Draw letter
                ctx.fillStyle = '#000';
                ctx.font = '20px Roboto';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(cell.letter, x + cellSize / 2, y + cellSize / 2);
            });
        });
    }

    function getCellFromCoords(x, y) {
        const row = Math.floor(y / cellSize);
        const col = Math.floor(x / cellSize);
        return { row, col };
    }

    function startDrag(event) {
        const rect = canvas.getBoundingClientRect();
        const x = event.clientX - rect.left;
        const y = event.clientY - rect.top;
        const { row, col } = getCellFromCoords(x, y);

        isDragging = true;
        selectedCells = [{ row, col }];
        highlightCells();
    }

    function continueDrag(event) {
        if (isDragging) {
            const rect = canvas.getBoundingClientRect();
            const x = event.clientX - rect.left;
            const y = event.clientY - rect.top;
            const { row, col } = getCellFromCoords(x, y);

            const startCell = selectedCells[0];
            const rowDiff = row - startCell.row;
            const colDiff = col - startCell.col;

            // Check if the movement is diagonal, horizontal, or vertical
            if (Math.abs(rowDiff) === Math.abs(colDiff) || rowDiff === 0 || colDiff === 0) {
                selectedCells = [];
                const steps = Math.max(Math.abs(rowDiff), Math.abs(colDiff));
                for (let i = 0; i <= steps; i++) {
                    const newRow = startCell.row + Math.sign(rowDiff) * i;
                    const newCol = startCell.col + Math.sign(colDiff) * i;
                    selectedCells.push({ row: newRow, col: newCol });
                }
                highlightCells();
            }
        }
    }

    function highlightCells() {
        grid = grid.map((row, rowIndex) => row.map((cell, colIndex) => {
            const isSelected = selectedCells.some(sc => sc.row === rowIndex && sc.col === colIndex);
            const isFound = foundWordsCells.some(fc => fc.row === rowIndex && fc.col === colIndex);
            return { ...cell, highlight: isSelected || isFound };
        }));
        drawGrid();
    }

    function endDrag() {
        isDragging = false;
        const word = selectedCells.map(cell => grid[cell.row][cell.col].letter).join('');
        if (words.includes(word)) {
            markWordAsFound(word);
        }
        selectedCells = [];
        highlightCells();
    }

    function markWordAsFound(word) {
        words = words.map((w, index) => {
            if (w === word) {
                foundWords.update(fw => {
                    fw[index] = true;
                    return fw;
                });
                foundWordsCells = [...foundWordsCells, ...selectedCells];
                return `${w} ✓`;
            }
            return w;
        });
    }

    function hint() {
        let hintWordIndex = -1;
        get(foundWords).some((isFound, index) => {
            if (!isFound) {
                hintWordIndex = index;
                return true;
            }
            return false;
        });

        if (hintWordIndex !== -1) {
            const hintWord = words[hintWordIndex].replace(' ✓', '');
            highlightHintWord(hintWord);
        }
    }

    function highlightHintWord(word) {
        const wordCells = [];

        // Find the starting cell of the hint word
        grid.forEach((row, rowIndex) => {
            row.forEach((cell, colIndex) => {
                if (cell.letter === word[0]) {
                    const cells = checkWordAtPosition(word, rowIndex, colIndex);
                    if (cells.length === word.length) {
                        wordCells.push(cells[0]); // Only highlight the starting cell
                    }
                }
            });
        });

        // Temporarily highlight the starting cell for the hint word
        selectedCells = wordCells;
        highlightCells();

        // Reset the highlight after a short delay
        setTimeout(() => {
            selectedCells = [];
            highlightCells();
        }, 1000); // Highlight for 1 second
    }

    function checkWordAtPosition(word, row, col) {
        const directions = [
            { row: 0, col: 1 },  // Horizontal
            { row: 1, col: 0 },  // Vertical
            { row: 1, col: 1 },  // Diagonal down-right
            { row: 1, col: -1 }  // Diagonal down-left
        ];

        for (const direction of directions) {
            const cells = [];
            for (let i = 0; i < word.length; i++) {
                const newRow = row + i * direction.row;
                const newCol = col + i * direction.col;
                if (grid[newRow] && grid[newRow][newCol] && grid[newRow][newCol].letter === word[i]) {
                    cells.push({ row: newRow, col: newCol });
                } else {
                    break;
                }
            }
            if (cells.length === word.length) {
                return cells;
            }
        }
        return [];
    }
</script>

<style>
    canvas {
        border: 1px solid #ccc;
        margin-top: 28px;
        margin-left: 20px;
    }

    .word-heading {
        background-color: #404A5B;
        color: white;
        padding: 8px 16px;
        margin-bottom: 8px;
        display: inline-block;
        border-radius: 8px;
    }

    .words-list {
        margin-bottom: 250px;
    }

    .line-through {
        text-decoration: line-through;
    }

    .card {
        max-width: 1200px;
        padding: 40px;
        padding-top: 10px; /* Add some padding at the top to separate the canvas from the heading */
    }
    .hint button:hover i {
        transform: scale(1.5); /* Increase size by 20% */
        transition: transform 0.2s; /* Smooth transition */
    }
</style>

<div class="card gap-16 items-center mx-auto max-w-screen-xl lg:grid lg:grid-cols-2 overflow-hidden rounded-lg">
    <canvas 
        bind:this={canvas}
        on:mousedown={startDrag}
        on:mousemove={continueDrag}
        on:mouseup={endDrag}
        on:mouseleave={endDrag}
    ></canvas>
    <div>
        <h2 class="word-heading text-2xl font-semibold">Words to Find:</h2>
        <ul class="words-list list-disc ml-8 text-white">
            {#each words as w, index}
                <li class="{get(foundWords)[index] ? 'line-through' : ''}">{w}</li>
            {/each}
        </ul>
    </div>
    <div class="hint" style="margin-top: -40px; margin-left: 200px;">
        <button on:click={hint}>
              <i class="fa-solid fa-lightbulb-on fa-xl" style="color: yellow; margin-right: 4px; vertical-align: middle;"></i>
        </button>
    </div>
</div>
