<script>
	import { onMount } from "svelte";
    import { onDestroy } from "svelte";
	// apologies to the person who has to sit through this code, uhhh, good luck :D
    
    class Piece {
        /**
		 * @param {string} colour
		 * @param {string} pieceId
		 * @param {{x: any;y: any;}} square // the square the piece is on
		 * @param {HTMLElement | null} element
		 */
        constructor(colour, pieceId, square, element) { // every piece has these 4 attributes
            this.colour = colour;
            this.pieceId = pieceId;
            this.square = square;
            this.element = element;

            this.whitePieces = Array()
            this.blackPieces = Array()
            this.allPieces = [...this.whitePieces, ...this.blackPieces]

            this.lastSquare = {x: null, y: null};
            this.isDragging = false;
            this.mouseOffsetX = 0;
            this.mouseOffsetY = 0;
            
            this.madeValidMove = false // used for validation during the dropPiece function
            this.hasMoved = false; // used by the pawns, the rooks, and the kings

            // @ts-ignore
            this.element.addEventListener("mousedown", this.pickupPiece.bind(this)); // the error "object is possibly null" will not be a problem as the elements will be rendered already
            // @ts-ignore
            this.element.addEventListener("mousemove", this.movePiece.bind(this));
            // @ts-ignore
            this.element.addEventListener("mouseup", this.dropPiece.bind(this));
        }
        get squareId() { 
            return this.square 
        } set squareId(squarePosition) {
            const piece = document.getElementById(this.idOfPiece);
            this.lastSquare = this.square
            let t = "translate("+squarePosition.x*100+"%, "+squarePosition.y*100+"%)"
            // @ts-ignore
            piece.style.left = "0px"
            // @ts-ignore
            piece.style.top = "0px"
            // @ts-ignore
            piece.style.transform = t;
            this.square = squarePosition;
        } get idOfPiece() {
            return this.pieceId
        } get pieceColour() {
            return this.colour
        }

        /**
		 * @param {{ x: number | undefined; y: number | undefined; }} pos
		 */
        snapToSquare(pos) {
            let boardPos = { x: document.getElementById("board")?.getBoundingClientRect().left, y: document.getElementById("board")?.getBoundingClientRect().top }
            // @ts-ignore
            let tileSize = (document.getElementById("board")?.getBoundingClientRect().right - boardPos.x) / 8
            // @ts-ignore
            this.squareId = { x: Math.round((pos.x - boardPos.x)/tileSize), y: Math.round((pos.y - boardPos.y)/tileSize) }
        }

        /**
		 * @param {{ clientX: number; clientY: number; }} event
		 */
        pickupPiece(event) {
            this.isDragging = true;
            // @ts-ignore
            this.mouseOffsetX = event.clientX - this.element.offsetLeft;
            // @ts-ignore
            this.mouseOffsetY = event.clientY - this.element.offsetTop;
            this.element.style.zIndex = "1"
        }
        
        /**
		 * @param {{ clientX: number; clientY: number; }} event
		 */
        movePiece(event) {
            if (this.isDragging) {
                // @ts-ignore
                this.element.style.left = event.clientX - this.mouseOffsetX + "px";
                // @ts-ignore
                this.element.style.top = event.clientY - this.mouseOffsetY + "px";
            }
        }

        /**
		 * @param {any} event
		 */
        dropPiece(event) {
            this.element.style.zIndex = "0"
            this.isDragging = false;
            let position = { x: document.getElementById(this.idOfPiece)?.getBoundingClientRect().left, y: document.getElementById(this.idOfPiece)?.getBoundingClientRect().top }
            this.snapToSquare(position)
            this.validateMove()
            if (this.madeValidMove) {
                this.hasMoved = true
                for ( let i = 0; i < this.allPieces.length; i++) {
                    this.allPieces[i].allPieces = this.allPieces
                    this.allPieces[i].whitePieces = this.whitePieces
                    this.allPieces[i].blackPieces = this.blackPieces
                } for ( let i = 0; i < this.whitePieces.length; i++ ) {
                    this.whitePieces[i].allPieces = this.allPieces
                    this.whitePieces[i].whitePieces = this.whitePieces
                    this.whitePieces[i].blackPieces = this.blackPieces
                } for ( let i = 0; i < this.blackPieces.length; i++ ) {
                    this.blackPieces[i].allPieces = this.allPieces
                    this.blackPieces[i].whitePieces = this.whitePieces
                    this.blackPieces[i].blackPieces = this.blackPieces
                }
                if (this.colour == "white") {
                    colourToMove = "black"

                    let blackHasKing = false
                    for( let i = 0; i < this.blackPieces.length; i++ ) {
                        if (this.blackPieces[i].pieceId == "bKing") {
                            blackHasKing = true
                            break
                        }
                    }
                    if (!blackHasKing) {
                        stop()
                        alert("White wins by king capture")
                    }
                } else {
                    colourToMove = "white"
                    let whiteHasKing = false

                    for( let i = 0; i < this.whitePieces.length; i++ ) {
                        if (this.whitePieces[i].pieceId == "wKing") {
                            whiteHasKing = true
                            break
                        }
                    }
                    if (!whiteHasKing) {
                        stop()
                        alert("Black wins by king capture")
                    }
                }

                let numberOfPieces = 0
                if (this.allPieces.length == 2) {
                    alert("The game is a draw")
                    stop()
                } else if (this.allPieces.length <= 4) {
                    for( let i = 0; i < this.allPieces.length; i++ ) {
                        if (!(this.allPieces[i].pieceId.substr(1,1) == "K" || this.allPieces[i].pieceId.substr(1,1) == "B")) {
                            break
                        } else {
                            numberOfPieces++
                        }
                    }
                }
                if (numberOfPieces == this.allPieces.length) {
                    if (this.blackPieces.length == this.whitePieces.length) {
                        stop()
                        alert("The game is a draw by insufficient material")
                    }
                }
            }
        }

        checkIfPieceIsInSquare() {
            for ( let i = 0; i < this.allPieces.length; i++) {
                if (!(this.pieceId == this.allPieces[i].pieceId)){
                    if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                        return false // there is a piece in the square
                    }
                }
            } return true // there is not a piece in the square
        }

        checkIfOutOfBounds() {
            if (this.squareId.x < 0 || this.squareId.x > 7 || this.squareId.y < 0 || this.squareId.y > 7) {
                this.madeValidMove = false
                this.sqaureId = this.lastSquare
                return false
            } return true
        }

        validateMove() {
            // should be left empty since each move validation is different and will be overridden, but needs to be assigned here to be used in dropPiece()
        }
    }

    class Black_Pawn extends Piece {
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "black") {
                let hasTaken = false
                if (((this.lastSquare.x - this.squareId.x == 1) && (this.lastSquare.y - this.squareId.y == 1)) || ((this.lastSquare.x - this.squareId.x == -1) && (this.lastSquare.y - this.squareId.y == 1))) { // this checks if the pawn is taking a piece
                    for ( let i = 0; i < this.whitePieces.length; i++ ) {
                        if (this.whitePieces[i].squareId.x == this.squareId.x && this.whitePieces[i].squareId.y == this.squareId.y) {
                            this.whitePieces[i].element.remove()
                            this.allPieces.splice(i+this.blackPieces.length, 1); this.whitePieces.splice(i, 1)
                            hasTaken = true
                            this.madeValidMove = true
                            break // you can break because you cannot take multiple pieces
                        }
                    }
                    if (!hasTaken) {
                        this.squareId = this.lastSquare
                    }
                }
                else if (!this.hasMoved) { // this is so that the pawn can move 2 squares forward when it first moves
                    if ((this.lastSquare.x == this.squareId.x) && (this.lastSquare.y - this.squareId.y <= 2) && (this.lastSquare.y - this.squareId.y > 0) && this.checkIfPieceIsInSquare()) { 
                        this.hasMoved = true
                        this.madeValidMove = true
                    }
                    else {
                        this.squareId = this.lastSquare
                    }
                } else {
                    if ((this.lastSquare.x != this.squareId.x) || (this.lastSquare.y - this.squareId.y != 1) || !this.checkIfPieceIsInSquare() || !this.checkIfOutOfBounds()) { this.squareId = this.lastSquare } else { this.madeValidMove = true }
                }
            } else {
                this.squareId = this.lastSquare
            }
        } 
    }
    

    class Black_Rook extends Piece { // my new least favourite piece
        /**
		 * @param {any} currentSquare
		 * @param {any} end
		 * @param {any} inc
		 * @param {any} row
		 */
        horizValidate(currentSquare, end, row, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare && this.allPieces[i].squareId.y == row && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.horizValidate(currentSquare + inc, end, row, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "white") {
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                break
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                break
                            }
                        }
                    }
                }
                
            } else {
                this.madeValidMove = true
            }
        }
        /**
		 * @param {number} currentSquare
		 * @param {number} end
		 * @param {any} col
		 * @param {any} inc
		 */
        vertValidate(currentSquare, end, col, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.y == currentSquare && this.allPieces[i].squareId.x == col && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.vertValidate(currentSquare + inc, end, col, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "white") {
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                break
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                break
                            }
                        }
                    }
                }
            } else {
                this.madeValidMove = true
            }
        }

        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "black" && (this.squareId.x != this.lastSquare.x || this.squareId.y != this.lastSquare.y) && this.checkIfOutOfBounds()) {
                if (this.squareId.y == this.lastSquare.y) { // when it moves horizontally
                    const start = this.lastSquare.x; const end = this.squareId.x
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.y
                    this.horizValidate(start, end, row, increment)
                } else if (this.squareId.x == this.lastSquare.x) { // when it moves vertically
                    const start = this.lastSquare.y; const end = this.squareId.y
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.x
                    this.vertValidate(start, end, row, increment)
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class Black_Knight extends Piece {
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "black") {
                if (((Math.abs(this.squareId.x-this.lastSquare.x) == 2 && Math.abs(this.squareId.y - this.lastSquare.y) == 1) || (Math.abs(this.squareId.x-this.lastSquare.x) == 1 && Math.abs(this.squareId.y - this.lastSquare.y) == 2)) && this.checkIfOutOfBounds()) {
                    if (!this.checkIfPieceIsInSquare()) {
                        loop:
                        for( let i = 0; i < this.allPieces.length; i++ ) {
                            if (this.allPieces[i].pieceId != this.pieceId) {
                                if (this.allPieces[i].squareId.x == this.squareId.x && this.allPieces[i].squareId.y == this.squareId.y) {
                                    if (this.allPieces[i].pieceColour == "white") {
                                        this.allPieces[i].element.remove()
                                        this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                        this.madeValidMove = true
                                        break loop
                                    } else {
                                        this.madeValidMove = false
                                        this.squareId = this.lastSquare
                                        break loop
                                    }
                                }
                            }
                            this.sqaureId = this.lastSquare
                        }
                    } else {
                        this.madeValidMove = true
                    }
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class Black_Bishop extends Piece {
        /**
		 * @param {any[]} currentSquare
		 * @param {any[]} end
		 * @param {number} rowDir
		 * @param {number} colDir
		 */
        diagValidate(currentSquare, end, rowDir, colDir) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare[0] && this.allPieces[i].squareId.y == currentSquare[1] && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare[0]-end[0]) > 1 && Math.abs(currentSquare[1]-end[1]) > 1) {
                this.diagValidate([currentSquare[0] + rowDir, currentSquare[1] + colDir], end, rowDir, colDir) // recursion baybeeeeeeeeeeee
            } else {
                if (!this.checkIfPieceIsInSquare()) {
                    for ( let i = 0; i < this.allPieces.length; i++ ) {
                        if (!(this.pieceId == this.allPieces[i].pieceId)){
                            if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                                if (this.allPieces[i].pieceColour == "white") {
                                    this.madeValidMove = true
                                    this.allPieces[i].element.remove()
                                    this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                    break
                                } else {
                                    this.madeValidMove = false
                                    this.squareId = this.lastSquare
                                    break
                                }
                            }
                        }
                    }
                } else {
                    this.madeValidMove = true
                }
            }
        }

        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "black" && (this.squareId.x != this.lastSquare.x || this.squareId.y != this.lastSquare.y) && this.checkIfOutOfBounds()) {
                if (Math.abs(this.squareId.x - this.lastSquare.x) == Math.abs(this.squareId.y - this.lastSquare.y)) {
                    let start = [this.lastSquare.x, this.lastSquare.y]
                    let end = [this.squareId.x, this.squareId.y]
                    const rowDir = (end[0] > start[0]) ? 1 : -1
                    const colDir = (end[1] > start[1]) ? 1 : -1
                    this.diagValidate(start, end, rowDir, colDir)
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class Black_Queen extends Piece {
        /**
		 * @param {any} currentSquare
		 * @param {any} end
		 * @param {any} inc
		 * @param {any} row
		 */
         horizValidate(currentSquare, end, row, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare && this.allPieces[i].squareId.y == row && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.horizValidate(currentSquare + inc, end, row, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "white") {
                                
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                break
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                break
                            }
                        }
                    }
                }
                
            } else {
                this.madeValidMove = true
            }
        }
        /**
		 * @param {number} currentSquare
		 * @param {number} end
		 * @param {any} col
		 * @param {any} inc
		 */
        vertValidate(currentSquare, end, col, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.y == currentSquare && this.allPieces[i].squareId.x == col && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.vertValidate(currentSquare + inc, end, col, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "white") {
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                return
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                return
                            }
                        }
                    }
                }
            } else {
                this.madeValidMove = true
            }
        }
        /**
		 * @param {any[]} currentSquare
		 * @param {any[]} end
		 * @param {number} rowDir
		 * @param {number} colDir
		 */
         diagValidate(currentSquare, end, rowDir, colDir) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare[0] && this.allPieces[i].squareId.y == currentSquare[1] && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare[0]-end[0]) > 1 && Math.abs(currentSquare[1]-end[1]) > 1) {
                this.diagValidate([currentSquare[0] + rowDir, currentSquare[1] + colDir], end, rowDir, colDir) // recursion baybeeeeeeeeeeee
            } else {
                if (!this.checkIfPieceIsInSquare()) {
                    for ( let i = 0; i < this.allPieces.length; i++ ) {
                        if (!(this.pieceId == this.allPieces[i].pieceId)){
                            if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                                if (this.allPieces[i].pieceColour == "white") {
                                    this.madeValidMove = true
                                    this.allPieces[i].element.remove()
                                    this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                    break
                                } else {
                                    this.madeValidMove = false
                                    this.squareId = this.lastSquare
                                    break
                                }
                            }
                        }
                    }
                } else {
                    this.madeValidMove = true
                }
            }
        }
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "black" && (this.squareId.x != this.lastSquare.x || this.squareId.y != this.lastSquare.y) && this.checkIfOutOfBounds()) {
                if (Math.abs(this.squareId.x - this.lastSquare.x) == Math.abs(this.squareId.y - this.lastSquare.y)) { 
                    let start = [this.lastSquare.x, this.lastSquare.y]
                    let end = [this.squareId.x, this.squareId.y]
                    const rowDir = (end[0] > start[0]) ? 1 : -1
                    const colDir = (end[1] > start[1]) ? 1 : -1
                    this.diagValidate(start, end, rowDir, colDir)
                } else if (this.squareId.y == this.lastSquare.y) { // when it moves horizontally
                    const start = this.lastSquare.x; const end = this.squareId.x
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.y
                    this.horizValidate(start, end, row, increment)
                } else if (this.squareId.x == this.lastSquare.x) { // when it moves vertically
                    const start = this.lastSquare.y; const end = this.squareId.y
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.x
                    this.vertValidate(start, end, row, increment)
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class Black_King extends Piece {
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "black") {
                if (Math.abs(this.squareId.x - this.lastSquare.x) <= 1 &&  Math.abs(this.squareId.y - this.lastSquare.y) <= 1 && this.checkIfOutOfBounds()) {
                    if (!this.checkIfPieceIsInSquare()) {
                        for( let i = 0; i < this.allPieces.length; i++ ) {
                            if (this.allPieces[i].squareId.x == this.squareId.x && this.allPieces[i].squareId.y == this.squareId.y && this.allPieces[i].pieceId != this.pieceId) {
                                if (this.allPieces[i].pieceColour == "white") {
                                    this.madeValidMove = true
                                    this.allPieces[i].element.remove()
                                    this.allPieces.splice(i, 1); this.whitePieces.splice(i-this.blackPieces.length, 1)
                                    break
                                } else {
                                    this.squareId = this.lastSquare
                                    break
                                }
                            }
                        }
                    } else {
                        this.madeValidMove = true
                    }
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class White_Pawn extends Piece {
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "white") {
                let hasTaken = false
                if (((this.lastSquare.x - this.squareId.x == 1) && (this.lastSquare.y - this.squareId.y == -1)) || ((this.lastSquare.x - this.squareId.x == -1) && (this.lastSquare.y - this.squareId.y == -1))) { // this checks if the pawn is taking a piece
                    for ( let i = 0; i < this.blackPieces.length; i++ ) {
                        if (this.blackPieces[i].squareId.x == this.squareId.x && this.blackPieces[i].squareId.y == this.squareId.y) {
                            this.blackPieces[i].element.remove()
                            this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                            hasTaken = true
                            this.madeValidMove = true
                            break // you can break because you cannot take multiple pieces
                        }
                    }
                    if (!hasTaken) {
                        this.squareId = this.lastSquare
                    }
                }
                else if (!this.hasMoved) { // this is so that the pawn can move 2 squares forward when it first moves
                    if ((this.lastSquare.x == this.squareId.x) && (this.squareId.y - this.lastSquare.y <= 2) && (this.squareId.y - this.lastSquare.y > 0) && this.checkIfPieceIsInSquare()) { 
                        this.hasMoved = true
                        this.madeValidMove = true
                    }
                    else {
                        this.squareId = this.lastSquare
                    }
                } else {
                    if ((this.lastSquare.x != this.squareId.x) || (this.squareId.y - this.lastSquare.y != 1) || !this.checkIfPieceIsInSquare() || !this.checkIfOutOfBounds()) { this.squareId = this.lastSquare } else { this.madeValidMove = true }
                }
            } else {
                this.squareId = this.lastSquare
            }
        } 
    }

    class White_Rook extends Piece {
        /**
		 * @param {any} currentSquare
		 * @param {any} end
		 * @param {any} inc
		 * @param {any} row
		 */
         horizValidate(currentSquare, end, row, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare && this.allPieces[i].squareId.y == row && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.horizValidate(currentSquare + inc, end, row, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "black") {
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                break
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                break
                            }
                        }
                    }
                }
                
            } else {
                this.madeValidMove = true
            }
        }
        /**
		 * @param {number} currentSquare
		 * @param {number} end
		 * @param {any} col
		 * @param {any} inc
		 */
        vertValidate(currentSquare, end, col, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.y == currentSquare && this.allPieces[i].squareId.x == col && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.vertValidate(currentSquare + inc, end, col, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "black") {
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                break
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                break
                            }
                        }
                    }
                }
            } else {
                this.madeValidMove = true
            }
        }

        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "white" && (this.squareId.x != this.lastSquare.x || this.squareId.y != this.lastSquare.y) && this.checkIfOutOfBounds()) {
                if (this.squareId.y == this.lastSquare.y) { // when it moves horizontally
                    const start = this.lastSquare.x; const end = this.squareId.x
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.y
                    this.horizValidate(start, end, row, increment)
                } else if (this.squareId.x == this.lastSquare.x) { // when it moves vertically
                    const start = this.lastSquare.y; const end = this.squareId.y
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.x
                    this.vertValidate(start, end, row, increment)
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class White_Knight extends Piece {
        // @ts-ignore
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "white") {
                if (((Math.abs(this.squareId.x-this.lastSquare.x) == 2 && Math.abs(this.squareId.y - this.lastSquare.y) == 1) || (Math.abs(this.squareId.x-this.lastSquare.x) == 1 && Math.abs(this.squareId.y - this.lastSquare.y) == 2)) && this.checkIfOutOfBounds()) {
                    if (!this.checkIfPieceIsInSquare()) {
                        loop:
                        for( let i = 0; i < this.allPieces.length; i++ ) {
                            if (this.allPieces[i].pieceId != this.pieceId) {
                                if (this.allPieces[i].squareId.x == this.squareId.x && this.allPieces[i].squareId.y == this.squareId.y) {
                                    if (this.allPieces[i].pieceColour == "black") {
                                        this.allPieces[i].element.remove()
                                        this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                        this.madeValidMove = true
                                        break loop
                                    } else {
                                        this.madeValidMove = false
                                        break loop
                                    }
                                }
                            }
                            this.sqaureId = this.lastSquare
                        }
                    } else {
                        this.madeValidMove = true
                    }
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class White_Bishop extends Piece {
        /**
		 * @param {any[]} currentSquare
		 * @param {any[]} end
		 * @param {number} rowDir
		 * @param {number} colDir
		 */
        diagValidate(currentSquare, end, rowDir, colDir) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare[0] && this.allPieces[i].squareId.y == currentSquare[1] && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare[0]-end[0]) > 1 && Math.abs(currentSquare[1]-end[1]) > 1) {
                this.diagValidate([currentSquare[0] + rowDir, currentSquare[1] + colDir], end, rowDir, colDir) // recursion baybeeeeeeeeeeee
            } else {
                if (!this.checkIfPieceIsInSquare()) {
                    for ( let i = 0; i < this.allPieces.length; i++ ) {
                        if (!(this.pieceId == this.allPieces[i].pieceId)){
                            if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                                if (this.allPieces[i].pieceColour == "black") {
                                    this.madeValidMove = true
                                    this.allPieces[i].element.remove()
                                    this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                    return
                                } else {
                                    this.madeValidMove = false
                                    this.squareId = this.lastSquare
                                    return
                                }
                            }
                        }
                    }
                } else {
                    this.madeValidMove = true
                    return
                }
            }
        }

        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "white" && (this.squareId.x != this.lastSquare.x || this.squareId.y != this.lastSquare.y) && this.checkIfOutOfBounds()) {
                if (Math.abs(this.squareId.x - this.lastSquare.x) == Math.abs(this.squareId.y - this.lastSquare.y)) {
                    let start = [this.lastSquare.x, this.lastSquare.y]
                    let end = [this.squareId.x, this.squareId.y]
                    const rowDir = (end[0] > start[0]) ? 1 : -1
                    const colDir = (end[1] > start[1]) ? 1 : -1
                    this.diagValidate(start, end, rowDir, colDir)
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class White_Queen extends Piece {
        /**
		 * @param {any} currentSquare
		 * @param {any} end
		 * @param {any} inc
		 * @param {any} row
		 */
         horizValidate(currentSquare, end, row, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare && this.allPieces[i].squareId.y == row && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.horizValidate(currentSquare + inc, end, row, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "black") {
                                
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                return
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                return
                            }
                        }
                    }
                }
                
            } else {
                this.madeValidMove = true
            }
        }
        /**
		 * @param {number} currentSquare
		 * @param {number} end
		 * @param {any} col
		 * @param {any} inc
		 */
        vertValidate(currentSquare, end, col, inc) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.y == currentSquare && this.allPieces[i].squareId.x == col && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare - end) > 1 ) {
                this.vertValidate(currentSquare + inc, end, col, inc)
            } else if (!this.checkIfPieceIsInSquare()) {
                for ( let i = 0; i < this.allPieces.length; i++ ) {
                    if (!(this.pieceId == this.allPieces[i].pieceId)){
                        if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                            if (this.allPieces[i].pieceColour == "black") {
                                this.madeValidMove = true
                                this.allPieces[i].element.remove()
                                this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                return
                            } else {
                                this.madeValidMove = false
                                this.squareId = this.lastSquare
                                return
                            }
                        }
                    }
                }
            } else {
                this.madeValidMove = true
            }
        }
        /**
		 * @param {any[]} currentSquare
		 * @param {any[]} end
		 * @param {number} rowDir
		 * @param {number} colDir
		 */
         diagValidate(currentSquare, end, rowDir, colDir) {
            for( let i = 0; i < this.allPieces.length; i++ ) {
                if (this.allPieces[i].squareId.x == currentSquare[0] && this.allPieces[i].squareId.y == currentSquare[1] && this.allPieces[i].pieceId != this.pieceId) {
                    this.madeValidMove = false
                    this.squareId = this.lastSquare
                    return false
                }
            } if (Math.abs(currentSquare[0]-end[0]) > 1 && Math.abs(currentSquare[1]-end[1]) > 1) {
                this.diagValidate([currentSquare[0] + rowDir, currentSquare[1] + colDir], end, rowDir, colDir) // recursion baybeeeeeeeeeeee
            } else {
                if (!this.checkIfPieceIsInSquare()) {
                    for ( let i = 0; i < this.allPieces.length; i++ ) {
                        if (!(this.pieceId == this.allPieces[i].pieceId)){
                            if ((this.squareId.x == this.allPieces[i].squareId.x) && (this.squareId.y == this.allPieces[i].squareId.y)) {
                                if (this.allPieces[i].pieceColour == "black") {
                                    this.madeValidMove = true
                                    this.allPieces[i].element.remove()
                                    this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                    return
                                } else {
                                    this.madeValidMove = false
                                    this.squareId = this.lastSquare
                                    return
                                }
                            }
                        }
                    }
                } else {
                    this.madeValidMove = true
                    return
                }
            }
        }
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "white" && (this.squareId.x != this.lastSquare.x || this.squareId.y != this.lastSquare.y) && this.checkIfOutOfBounds()) {
                if (Math.abs(this.squareId.x - this.lastSquare.x) == Math.abs(this.squareId.y - this.lastSquare.y)) { 
                    let start = [this.lastSquare.x, this.lastSquare.y]
                    let end = [this.squareId.x, this.squareId.y]
                    const rowDir = (end[0] > start[0]) ? 1 : -1
                    const colDir = (end[1] > start[1]) ? 1 : -1
                    this.diagValidate(start, end, rowDir, colDir)
                } else if (this.squareId.y == this.lastSquare.y) { // when it moves horizontally
                    const start = this.lastSquare.x; const end = this.squareId.x
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.y
                    this.horizValidate(start, end, row, increment)
                } else if (this.squareId.x == this.lastSquare.x) { // when it moves vertically
                    const start = this.lastSquare.y; const end = this.squareId.y
                    let increment = (end > start) ? 1 : -1; let row = this.squareId.x
                    this.vertValidate(start, end, row, increment)
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    class White_King extends Piece {
        validateMove() {
            this.madeValidMove = false
            if (colourToMove == "white") {
                if (Math.abs(this.squareId.x - this.lastSquare.x) <= 1 && Math.abs(this.squareId.y - this.lastSquare.y) <= 1 && this.checkIfOutOfBounds()) {
                    if (!this.checkIfPieceIsInSquare()) {
                        for( let i = 0; i < this.allPieces.length; i++ ) {
                            if (this.allPieces[i].squareId.x == this.squareId.x && this.allPieces[i].squareId.y == this.squareId.y && this.allPieces[i].pieceId != this.pieceId) {
                                if (this.allPieces[i].pieceColour == "black") {
                                    this.madeValidMove = true
                                    this.allPieces[i].element.remove()
                                    this.allPieces.splice(i, 1); this.blackPieces.splice(i, 1);
                                    break
                                } else {
                                    this.squareId = this.lastSquare
                                    break
                                }
                            }
                        }
                    } else {
                        this.madeValidMove = true
                    }
                } else {
                    this.squareId = this.lastSquare
                }
            } else {
                this.squareId = this.lastSquare
            }
        }
    }

    /**
	 * @param {string | any[]} x
     * @param {string | any[]} y
	 */
    function setPositionForBlackPieces(x, y) { //blackPieces, whitePieces
        for (let i = 0; i < x.length; i++) {
            x[i].whitePieces = y; x[i].blackPieces = x; x[i].allPieces = [...y, ...x] // order: whitePieces, blackPieces
            x[i].canMove = false
            if (x[i].idOfPiece.substr(0,2) == "bP") {
                x[i].squareId = { x: i, y: 6 }
            } else {
                x[i].squareId = { x: i-8, y: 7 }
            }
        }
    } 
    
    /**
	 * @param {string | any[]} x
     * @param {string | any[]} y
	 */
    function setPositionForWhitePieces(x, y) { //whitePieces, blackPieces
        for (let i = 0; i < x.length; i++) {
            x[i].whitePieces = x; x[i].blackPieces = y; x[i].allPieces = [...y, ...x] // order: blackPieces, whitePieces
            if (x[i].idOfPiece.substr(0,2) == "wP") {
                x[i].squareId = { x: i, y: 1 }
            } else {
                x[i].squareId = { x: i-8, y: 0 }
            }
        } 
    }

    let time;

    const getTimeValue = () => {
        const params = new URLSearchParams(window.location.search);
        time = Number(params.get('time'));
    }

    getTimeValue()

    var colourToMove = "white";
    
    export let timeLimit = time;
    export let onTimeout;

    let running = false;
    let player1Time = timeLimit;
    let player2Time = timeLimit;
    let intervalId;
    let lastColourToMove = colourToMove;
    
    function start() {
        running = true;
        colourToMove = lastColourToMove
        intervalId = setInterval(() => {
            if (player1Time <= 0 || player2Time <= 0) {
                stop();
                onTimeout = (player1Time <= 0) ? 1 : 2;
                if (onTimeout == 1) {
                    alert("Black wins on time! 🥳")
                } else { alert("White wins on time! 🥳") }
            } else {
                if (running) {
                    if (colourToMove == "white") {
                        player1Time--;
                    } else {
                        player2Time--;
                    }
                }
            }
        }, 1000);
    }

    function stop() {
        running = false;
        lastColourToMove = colourToMove
        colourToMove = ""
        clearInterval(intervalId);
    }

    function toggle() {
        if (running) {
            stop();
        } else {
            start();
        }
    }

    onDestroy(() => {
        stop();
    });

    // Determine which player's turn it is
    function player1Turn() {
        return Math.floor(player1Time / 60) % 2 === Math.floor(player2Time / 60) % 2;
    }

    start()

    onMount(() => { // onMount is called when everything is rendered
        let bPawn1 = new Black_Pawn("black", "bPawn1", {x: 0, y: 0}, document.getElementById("bPawn1"))
        let bPawn2 = new Black_Pawn("black", "bPawn2", {x: 0, y: 0}, document.getElementById("bPawn2"))
        let bPawn3 = new Black_Pawn("black", "bPawn3", {x: 0, y: 0}, document.getElementById("bPawn3"))
        let bPawn4 = new Black_Pawn("black", "bPawn4", {x: 0, y: 0}, document.getElementById("bPawn4"))
        let bPawn5 = new Black_Pawn("black", "bPawn5", {x: 0, y: 0}, document.getElementById("bPawn5"))
        let bPawn6 = new Black_Pawn("black", "bPawn6", {x: 0, y: 0}, document.getElementById("bPawn6"))
        let bPawn7 = new Black_Pawn("black", "bPawn7", {x: 0, y: 0}, document.getElementById("bPawn7"))
        let bPawn8 = new Black_Pawn("black", "bPawn8", {x: 0, y: 0}, document.getElementById("bPawn8"))
        let bRook1 = new Black_Rook("black", "bRook1", {x: 0, y: 0}, document.getElementById("bRook1"))
        let bRook2 = new Black_Rook("black", "bRook2", {x: 0, y: 0}, document.getElementById("bRook2"))
        let bKnight1 = new Black_Knight("black", "bKnight1", {x: 0, y: 0}, document.getElementById("bKnight1"))
        let bKnight2 = new Black_Knight("black", "bKnight2", {x: 0, y: 0}, document.getElementById("bKnight2"))
        let bBishop1 = new Black_Bishop("black", "bBishop1", {x: 0, y: 0}, document.getElementById("bBishop1"))
        let bBishop2 = new Black_Bishop("black", "bBishop2", {x: 0, y: 0}, document.getElementById("bBishop2"))
        let bQueen = new Black_Queen("black", "bQueen", {x: 0, y: 0}, document.getElementById("bQueen"))
        let bKing = new Black_King("black", "bKing", {x: 0, y: 0}, document.getElementById("bKing"))
        let wPawn1 = new White_Pawn("white", "wPawn1", {x: 0, y: 0}, document.getElementById("wPawn1"))
        let wPawn2 = new White_Pawn("white", "wPawn2", {x: 0, y: 0}, document.getElementById("wPawn2"))
        let wPawn3 = new White_Pawn("white", "wPawn3", {x: 0, y: 0}, document.getElementById("wPawn3"))
        let wPawn4 = new White_Pawn("white", "wPawn4", {x: 0, y: 0}, document.getElementById("wPawn4"))
        let wPawn5 = new White_Pawn("white", "wPawn5", {x: 0, y: 0}, document.getElementById("wPawn5"))
        let wPawn6 = new White_Pawn("white", "wPawn6", {x: 0, y: 0}, document.getElementById("wPawn6"))
        let wPawn7 = new White_Pawn("white", "wPawn7", {x: 0, y: 0}, document.getElementById("wPawn7"))
        let wPawn8 = new White_Pawn("white", "wPawn8", {x: 0, y: 0}, document.getElementById("wPawn8"))
        let wRook1 = new White_Rook("white", "wRook1", {x: 0, y: 0}, document.getElementById("wRook1"))
        let wRook2 = new White_Rook("white", "wRook2", {x: 0, y: 0}, document.getElementById("wRook2"))
        let wKnight1 = new White_Knight("white", "wKnight1", {x: 0, y: 0}, document.getElementById("wKnight1"))
        let wKnight2 = new White_Knight("white", "wKnight2", {x: 0, y: 0}, document.getElementById("wKnight2"))
        let wBishop1 = new White_Bishop("white", "wBishop1", {x: 0, y: 0}, document.getElementById("wBishop1"))
        let wBishop2 = new White_Bishop("white", "wBishop2", {x: 0, y: 0}, document.getElementById("wBishop2"))
        let wQueen = new White_Queen("white", "wQueen", {x: 0, y: 0}, document.getElementById("wQueen"))
        let wKing = new White_King("white", "wKing", {x: 0, y: 0}, document.getElementById("wKing"))

        var allPieces = [bPawn1, bPawn2, bPawn3, bPawn4, bPawn5, bPawn6, bPawn7, bPawn8, bRook1, bKnight1, bBishop1, bKing, bQueen, bBishop2, bKnight2, bRook2, wPawn1, wPawn2, wPawn3, wPawn4, wPawn5, wPawn6, wPawn7, wPawn8, wRook1, wKnight1, wBishop1, wKing, wQueen, wBishop2, wKnight2, wRook2]
        
        var blackPieces = allPieces.slice(0, allPieces.length / 2); var whitePieces = allPieces.slice(allPieces.length / 2);

        setPositionForBlackPieces(blackPieces, whitePieces)
        setPositionForWhitePieces(whitePieces, blackPieces)
    })
</script>

<div class="container">
    <chessBoard>
        <div id="board" class="chessBoard">
            <div id="bRook1" class="piece blackRook"></div>
            <div id="bKnight1" class="piece blackKnight"></div>
            <div id="bBishop1" class="piece blackBishop"></div>
            <div id="bQueen" class="piece blackQueen"></div>
            <div id="bKing" class="piece blackKing"></div>
            <div id="bBishop2" class="piece blackBishop"></div>
            <div id="bKnight2" class="piece blackKnight"></div>
            <div id="bRook2" class="piece blackRook"></div>
            <div id="bPawn1" class="piece blackPawn"></div>
            <div id="bPawn2" class="piece blackPawn"></div>
            <div id="bPawn3" class="piece blackPawn"></div>
            <div id="bPawn4" class="piece blackPawn"></div>
            <div id="bPawn5" class="piece blackPawn"></div>
            <div id="bPawn6" class="piece blackPawn"></div>
            <div id="bPawn7" class="piece blackPawn"></div>
            <div id="bPawn8" class="piece blackPawn"></div>
            <div id="wRook1" class="piece whiteRook"></div>
            <div id="wKnight1" class="piece whiteKnight"></div>
            <div id="wBishop1" class="piece whiteBishop"></div>
            <div id="wQueen" class="piece whiteQueen"></div>
            <div id="wKing" class="piece whiteKing"></div>
            <div id="wBishop2" class="piece whiteBishop"></div>
            <div id="wKnight2" class="piece whiteKnight"></div>
            <div id="wRook2" class="piece whiteRook"></div>
            <div id="wPawn1" class="piece whitePawn"></div>
            <div id="wPawn2" class="piece whitePawn"></div>
            <div id="wPawn3" class="piece whitePawn"></div>
            <div id="wPawn4" class="piece whitePawn"></div>
            <div id="wPawn5" class="piece whitePawn"></div>
            <div id="wPawn6" class="piece whitePawn"></div>
            <div id="wPawn7" class="piece whitePawn"></div>
            <div id="wPawn8" class="piece whitePawn"></div>
        </div>
    </chessBoard>

    <div class="stopClock">
        <div class="player1">
            <div class="time">
                { Math.floor(player1Time / 60) }:{ ('0' + (player1Time % 60)).slice(-2) }
            </div>
            <div class="name">White</div>
        </div>
        <div class="player2">
            <div class="time">
                { Math.floor(player2Time / 60) }:{ ('0' + (player2Time % 60)).slice(-2) }
            </div>
            <div class="name">Black</div>
        </div>
        <button on:click={toggle}>
            { running ? 'Stop' : 'Start' }
        </button>
    </div>
</div>
<style>
    @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

    .container {
        display: flex;
        flex-direction: row;
        align-items: center;
    }

    #board {
        background-image: url("Chess_Board2.png");
    } .chessBoard {
        background-size: 100%;
        position: relative;
        margin-top: 5vh;
        margin-bottom: 5vh;
        margin-left: 10vh;
        height: 90vh;
        width: 90vh;
        user-select: none;
    } 
    
    .piece {
        background-size: 100%;
        position: absolute;
        height: 12.5%;
        width: 12.5%;
    } .piece.blackPawn {
        background-image: url("./chessPieces/blackPawn.png");
    } .piece.blackRook {
        background-image: url("./chessPieces/blackRook.png");
    } .piece.blackKnight {
        background-image: url("./chessPieces/blackKnight.png");
    } .piece.blackBishop {
        background-image: url("./chessPieces/blackBishop.png");
    } .piece.blackKing {
        background-image: url("./chessPieces/blackKing.png");
    } .piece.blackQueen {
        background-image: url("./chessPieces/blackQueen.png");
    } .piece.whitePawn {
        background-image: url("./chessPieces/whitePawn.png");
    } .piece.whiteRook {
        background-image: url("./chessPieces/whiteRook.png");
    } .piece.whiteKnight {
        background-image: url("./chessPieces/whiteKnight.png");
    } .piece.whiteBishop {
        background-image: url("./chessPieces/whiteBishop.png");
    } .piece.whiteKing {
        background-image: url("./chessPieces/whiteKing.png");
    } .piece.whiteQueen {
        background-image: url("./chessPieces/whiteQueen.png");
    }

    .stopClock {
        padding: 10px;
        font-family: 'Roboto', sans-serif;
        display: flex;
        justify-content: space-between;
        align-items: center;
        font-size: 24px;
    }

    .player1, .player2 {
        padding: 10px;
        font-family: 'Roboto', sans-serif;
        text-align: center;
    }

    .name {
        font-family: 'Roboto', sans-serif;
        font-size: 18px;
        font-weight: bold;
    }

    button {
        padding: 10px;
        font-family: 'Roboto', sans-serif;
        font-size: 18px;
    }

</style>