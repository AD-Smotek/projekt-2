hraní Šach s kamarádem na webovce

mám tady další kousek mé prace, kterou jsem tentokrát dělal sám.
trvalo mi to asi 4 dny, každý den jsem na tom dělal kousek po kousku.
pořád jsem měl errory v consoly kterým jsem nerozumněl.
tak jsem si nechal pomoct od Chatu jak to dát pryč.
konečně jsem našel problém, vše by mělo fungovat jak normální šachy.
ozkoušel jsem to asi třikrát .
narazil jsem na miliony chyb, jako že pawn nemuže dva bloky do předu, nebo že může do zadu nebo že může sám sebe eliminovat.
taky jsem přidal transformaci na posledním řádku, aby se mohl pawn transformovat na knight,queen,bishop,rook.
konec je když umře král, tak si můžete vybrat play again hrát znovu nebo exit game over.

vs kod:





<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chess with AI</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/3.2.7/tailwind.min.css" rel="stylesheet">
    <style>
        body { background-color: #f0f0f0; display: flex; flex-direction: column; justify-content: center; align-items: center; height: 100vh; gap: 20px; }
        #chessboard { display: grid; grid-template-columns: repeat(8, 60px); grid-template-rows: repeat(8, 60px); }
        .cell { width: 60px; height: 60px; display: flex; justify-content: center; align-items: center; font-size: 24px; cursor: pointer; }
        .white { background-color: #eee; }
        .black { background-color: #555; color: #fff; }
        .selected { background-color: #a0c4ff !important; }
    </style>
</head>
<body>

<div id="chessboard"></div>
<div id="promotion" style="display: none;">
    <p>Select a piece for promotion:</p>
    <button onclick="promotePawn('q')">Queen</button>
    <button onclick="promotePawn('r')">Rook</button>
    <button onclick="promotePawn('b')">Bishop</button>
    <button onclick="promotePawn('n')">Knight</button>
</div>
<div id="controls" style="display: none;">
    <button onclick="restartGame()" class="bg-blue-500 text-white px-4 py-2 rounded">Play Again</button>
    <button onclick="exitGame()" class="bg-red-500 text-white px-4 py-2 rounded">Exit</button>
</div>

<script>
    let board;
    let selectedCell = null;
    let gameEnded = false;
    let promotionPosition = null;

    const initialBoard = [
        ["r", "n", "b", "q", "k", "b", "n", "r"],
        ["p", "p", "p", "p", "p", "p", "p", "p"],
        ["", "", "", "", "", "", "", ""],
        ["", "", "", "", "", "", "", ""],
        ["", "", "", "", "", "", "", ""],
        ["", "", "", "", "", "", "", ""],
        ["P", "P", "P", "P", "P", "P", "P", "P"],
        ["R", "N", "B", "Q", "K", "B", "N", "R"]
    ];

    const pieceSymbols = {
        p: "♟", r: "♜", n: "♞", b: "♝", q: "♛", k: "♚",
        P: "♙", R: "♖", N: "♘", B: "♗", Q: "♕", K: "♔"
    };

    function initBoard() {
        board = JSON.parse(JSON.stringify(initialBoard));
        gameEnded = false;
        promotionPosition = null;
        document.getElementById("promotion").style.display = "none";
        document.getElementById("controls").style.display = "none";
        renderBoard(true);
    }

    function renderBoard(fullRender = false) {
        const chessboard = document.getElementById("chessboard");
        if (fullRender) chessboard.innerHTML = "";

        for (let row = 0; row < 8; row++) {
            for (let col = 0; col < 8; col++) {
                let cell = document.querySelector(`[data-row='${row}'][data-col='${col}']`);
                if (!cell) {
                    cell = document.createElement("div");
                    cell.className = "cell " + ((row + col) % 2 === 0 ? "white" : "black");
                    cell.dataset.row = row;
                    cell.dataset.col = col;
                    cell.addEventListener("click", () => handleCellClick(row, col));
                    chessboard.appendChild(cell);
                }
                const piece = board[row][col];
                cell.textContent = piece ? pieceSymbols[piece] : "";
            }
        }
    }

    function handleCellClick(row, col) {
        if (gameEnded || promotionPosition) return;

        if (selectedCell) {
            if (isValidMove(selectedCell.row, selectedCell.col, row, col)) {
                movePiece(selectedCell.row, selectedCell.col, row, col);
                checkPawnPromotion(row, col);
                checkGameOver();
            }
            selectedCell = null;
        } else if (board[row][col]) {
            selectedCell = { row, col };
        }
        renderBoard();
    }

    function movePiece(fromRow, fromCol, toRow, toCol) {
        board[toRow][toCol] = board[fromRow][fromCol];
        board[fromRow][fromCol] = "";
    }

    function isValidMove(fromRow, fromCol, toRow, toCol) {
        const piece = board[fromRow][fromCol];
        const target = board[toRow][toCol];
        const isWhite = piece === piece.toUpperCase();
        if (target && (target === target.toUpperCase()) === isWhite) return false;

        switch(piece.toLowerCase()) {
            case 'p': return isValidPawnMove(fromRow, fromCol, toRow, toCol, piece);
            case 'r': return isValidRookMove(fromRow, fromCol, toRow, toCol);
            case 'n': return isValidKnightMove(fromRow, fromCol, toRow, toCol);
            case 'b': return isValidBishopMove(fromRow, fromCol, toRow, toCol);
            case 'q': return isValidQueenMove(fromRow, fromCol, toRow, toCol);
            case 'k': return isValidKingMove(fromRow, fromCol, toRow, toCol);
            default: return false;
        }
    }

    function isValidPawnMove(fromRow, fromCol, toRow, toCol, piece) {
        const direction = piece === 'P' ? -1 : 1;
        const startRow = piece === 'P' ? 6 : 1;
        // Single move
        if (fromCol === toCol && board[toRow][toCol] === '' && toRow === fromRow + direction) {
            return true;
        }
        // Double move from start row
        if (fromCol === toCol && board[toRow][toCol] === '' && fromRow === startRow && toRow === fromRow + 2 * direction) {
            // Check path clear
            if (board[fromRow + direction][fromCol] === '') return true;
        }
        // Capture
        if (Math.abs(toCol - fromCol) === 1 && toRow === fromRow + direction && board[toRow][toCol] !== '') {
            return true;
        }
        return false;
    }

    function isValidRookMove(fromRow, fromCol, toRow, toCol) {
        if (fromRow !== toRow && fromCol !== toCol) return false;
        if (fromRow === toRow) {
            for (let c = Math.min(fromCol, toCol) + 1; c < Math.max(fromCol, toCol); c++) {
                if (board[fromRow][c]) return false;
            }
        } else {
            for (let r = Math.min(fromRow, toRow) + 1; r < Math.max(fromRow, toRow); r++) {
                if (board[r][fromCol]) return false;
            }
        }
        return true;
    }

    function isValidKnightMove(fromRow, fromCol, toRow, toCol) {
        const dr = Math.abs(fromRow - toRow);
        const dc = Math.abs(fromCol - toCol);
        return (dr === 2 && dc === 1) || (dr === 1 && dc === 2);
    }

    function isValidBishopMove(fromRow, fromCol, toRow, toCol) {
        if (Math.abs(fromRow - toRow) !== Math.abs(fromCol - toCol)) return false;
        // Check path
        const stepR = toRow > fromRow ? 1 : -1;
        const stepC = toCol > fromCol ? 1 : -1;
        let r = fromRow + stepR, c = fromCol + stepC;
        while (r !== toRow && c !== toCol) {
            if (board[r][c]) return false;
            r += stepR; c += stepC;
        }
        return true;
    }

    function isValidQueenMove(fromRow, fromCol, toRow, toCol) {
        return isValidRookMove(fromRow, fromCol, toRow, toCol) || isValidBishopMove(fromRow, fromCol, toRow, toCol);
    }

    function isValidKingMove(fromRow, fromCol, toRow, toCol) {
        return Math.abs(fromRow - toRow) <= 1 && Math.abs(fromCol - toCol) <= 1;
    }

    function checkGameOver() {
        const pieces = board.flat();
        if (!pieces.includes('K') || !pieces.includes('k')) {
            gameEnded = true;
            document.getElementById("controls").style.display = "block";
        }
    }

    function checkPawnPromotion(row, col) {
        const piece = board[row][col];
        if ((piece === 'P' && row === 0) || (piece === 'p' && row === 7)) {
            promotionPosition = { row, col };
            document.getElementById("promotion").style.display = "block";
        }
    }

    function promotePawn(newPiece) {
        const { row, col } = promotionPosition;
        const isWhite = board[row][col] === 'P';
        board[row][col] = isWhite ? newPiece.toUpperCase() : newPiece;
        promotionPosition = null;
        document.getElementById("promotion").style.display = "none";
        renderBoard();
    }

    function restartGame() { initBoard(); }
    function exitGame() { document.body.innerHTML = "<h1>Game Over</h1>"; }

    initBoard();
</script>

</body>
</html>
