# Rubik-s-Cube-Solver-in-JavaScript

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rubik's Cube Solver</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            line-height: 1.6;
        }
        .container {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
        }
        .cube-container {
            margin-bottom: 20px;
        }
        .controls {
            margin-bottom: 20px;
        }
        button {
            padding: 8px 12px;
            margin-right: 10px;
            margin-bottom: 10px;
            cursor: pointer;
        }
        .steps {
            margin-top: 20px;
            max-height: 300px;
            overflow-y: auto;
            border: 1px solid #ccc;
            padding: 10px;
        }
        .step {
            margin-bottom: 15px;
            padding-bottom: 15px;
            border-bottom: 1px solid #eee;
        }
        .step:last-child {
            border-bottom: none;
        }
    </style>
</head>
<body>
    <h1>Rubik's Cube Solver</h1>
    
    <div class="controls">
        <button id="scramble">Scramble Cube</button>
        <button id="solve">Solve Cube</button>
        <button id="reset">Reset Cube</button>
        
        <div>
            <h3>Manual Rotation:</h3>
            <button data-move="F">F (Front)</button>
            <button data-move="F'">F' (Front counter)</button>
            <button data-move="B">B (Back)</button>
            <button data-move="B'">B' (Back counter)</button><br>
            <button data-move="R">R (Right)</button>
            <button data-move="R'">R' (Right counter)</button>
            <button data-move="L">L (Left)</button>
            <button data-move="L'">L' (Left counter)</button><br>
            <button data-move="U">U (Up)</button>
            <button data-move="U'">U' (Up counter)</button>
            <button data-move="D">D (Down)</button>
            <button data-move="D'">D' (Down counter)</button>
        </div>
    </div>
    
    <div class="container">
        <div class="cube-container">
            <h2>Current Cube State</h2>
            <div id="cube-display"></div>
        </div>
    </div>
    
    <div class="steps" id="solution-steps">
        <h2>Solution Steps</h2>
        <div id="steps-container"></div>
    </div>

    <script>
        // Rubik's Cube Class
        class RubiksCube {
            constructor() {
                // Initialize solved cube
                this.reset();
            }
            
            reset() {
                // Each face is represented as a 3x3 array
                this.faces = {
                    front: Array(3).fill().map(() => Array(3).fill('g')),  // Green
                    back: Array(3).fill().map(() => Array(3).fill('b')),   // Blue
                    left: Array(3).fill().map(() => Array(3).fill('o')),    // Orange
                    right: Array(3).fill().map(() => Array(3).fill('r')),   // Red
                    up: Array(3).fill().map(() => Array(3).fill('w')),     // White
                    down: Array(3).fill().map(() => Array(3).fill('y'))    // Yellow
                };
            }
            
            // Convert cube state to string for display
            toString() {
                let str = '';
                // Order: up, front, right, back, left, down
                for (let row of this.faces.up) str += row.join('');
                for (let row of this.faces.front) str += row.join('');
                for (let row of this.faces.right) str += row.join('');
                for (let row of this.faces.back) str += row.join('');
                for (let row of this.faces.left) str += row.join('');
                for (let row of this.faces.down) str += row.join('');
                return str;
            }
            
            // Rotate a face clockwise
            rotateFace(face, clockwise = true) {
                const f = this.faces[face];
                const n = f.length;
                const layers = Math.floor(n / 2);
                
                for (let layer = 0; layer < layers; layer++) {
                    const first = layer;
                    const last = n - 1 - layer;
                    
                    for (let i = first; i < last; i++) {
                        const offset = i - first;
                        
                        // Save top
                        const temp = f[first][i];
                        
                        if (clockwise) {
                            // left -> top
                            f[first][i] = f[last-offset][first];
                            // bottom -> left
                            f[last-offset][first] = f[last][last-offset];
                            // right -> bottom
                            f[last][last-offset] = f[i][last];
                            // top -> right
                            f[i][last] = temp;
                        } else {
                            // right -> top
                            f[first][i] = f[i][last];
                            // bottom -> right
                            f[i][last] = f[last][last-offset];
                            // left -> bottom
                            f[last][last-offset] = f[last-offset][first];
                            // top -> left
                            f[last-offset][first] = temp;
                        }
                    }
                }
            }
            
            // Rotate the front face
            rotateFront(clockwise = true) {
                this.rotateFace('front', clockwise);
                
                const up = this.faces.up;
                const right = this.faces.right;
                const down = this.faces.down;
                const left = this.faces.left;
                
                if (clockwise) {
                    // Rotate adjacent edges clockwise
                    const temp = [up[2][0], up[2][1], up[2][2]];
                    
                    // left -> up
                    up[2][0] = left[2][2];
                    up[2][1] = left[1][2];
                    up[2][2] = left[0][2];
                    
                    // down -> left
                    left[0][2] = down[0][0];
                    left[1][2] = down[0][1];
                    left[2][2] = down[0][2];
                    
                    // right -> down
                    down[0][0] = right[2][0];
                    down[0][1] = right[1][0];
                    down[0][2] = right[0][0];
                    
                    // up -> right
                    right[0][0] = temp[0];
                    right[1][0] = temp[1];
                    right[2][0] = temp[2];
                } else {
                    // Rotate adjacent edges counter-clockwise
                    const temp = [up[2][0], up[2][1], up[2][2]];
                    
                    // right -> up
                    up[2][0] = right[0][0];
                    up[2][1] = right[1][0];
                    up[2][2] = right[2][0];
                    
                    // down -> right
                    right[0][0] = down[0][2];
                    right[1][0] = down[0][1];
                    right[2][0] = down[0][0];
                    
                    // left -> down
                    down[0][0] = left[2][2];
                    down[0][1] = left[1][2];
                    down[0][2] = left[0][2];
                    
                    // up -> left
                    left[0][2] = temp[2];
                    left[1][2] = temp[1];
                    left[2][2] = temp[0];
                }
            }
            
            // Rotate the back face
            rotateBack(clockwise = true) {
                this.rotateFace('back', clockwise);
                
                const up = this.faces.up;
                const right = this.faces.right;
                const down = this.faces.down;
                const left = this.faces.left;
                
                if (clockwise) {
                    // Rotate adjacent edges clockwise
                    const temp = [up[0][0], up[0][1], up[0][2]];
                    
                    // right -> up
                    up[0][0] = right[0][2];
                    up[0][1] = right[1][2];
                    up[0][2] = right[2][2];
                    
                    // down -> right
                    right[0][2] = down[2][2];
                    right[1][2] = down[2][1];
                    right[2][2] = down[2][0];
                    
                    // left -> down
                    down[2][0] = left[2][0];
                    down[2][1] = left[1][0];
                    down[2][2] = left[0][0];
                    
                    // up -> left
                    left[0][0] = temp[2];
                    left[1][0] = temp[1];
                    left[2][0] = temp[0];
                } else {
                    // Rotate adjacent edges counter-clockwise
                    const temp = [up[0][0], up[0][1], up[0][2]];
                    
                    // left -> up
                    up[0][0] = left[2][0];
                    up[0][1] = left[1][0];
                    up[0][2] = left[0][0];
                    
                    // down -> left
                    left[0][0] = down[2][2];
                    left[1][0] = down[2][1];
                    left[2][0] = down[2][0];
                    
                    // right -> down
                    down[2][0] = right[2][2];
                    down[2][1] = right[1][2];
                    down[2][2] = right[0][2];
                    
                    // up -> right
                    right[0][2] = temp[0];
                    right[1][2] = temp[1];
                    right[2][2] = temp[2];
                }
            }
            
            // Rotate the right face
            rotateRight(clockwise = true) {
                this.rotateFace('right', clockwise);
                
                const up = this.faces.up;
                const front = this.faces.front;
                const down = this.faces.down;
                const back = this.faces.back;
                
                if (clockwise) {
                    // Rotate adjacent edges clockwise
                    const temp = [up[0][2], up[1][2], up[2][2]];
                    
                    // front -> up
                    up[0][2] = front[0][2];
                    up[1][2] = front[1][2];
                    up[2][2] = front[2][2];
                    
                    // down -> front
                    front[0][2] = down[0][2];
                    front[1][2] = down[1][2];
                    front[2][2] = down[2][2];
                    
                    // back -> down (note: back is mirrored)
                    down[0][2] = back[2][0];
                    down[1][2] = back[1][0];
                    down[2][2] = back[0][0];
                    
                    // up -> back
                    back[0][0] = temp[2];
                    back[1][0] = temp[1];
                    back[2][0] = temp[0];
                } else {
                    // Rotate adjacent edges counter-clockwise
                    const temp = [up[0][2], up[1][2], up[2][2]];
                    
                    // back -> up
                    up[0][2] = back[2][0];
                    up[1][2] = back[1][0];
                    up[2][2] = back[0][0];
                    
                    // down -> back
                    back[0][0] = down[2][2];
                    back[1][0] = down[1][2];
                    back[2][0] = down[0][2];
                    
                    // front -> down
                    down[0][2] = front[0][2];
                    down[1][2] = front[1][2];
                    down[2][2] = front[2][2];
                    
                    // up -> front
                    front[0][2] = temp[0];
                    front[1][2] = temp[1];
                    front[2][2] = temp[2];
                }
            }
            
            // Rotate the left face
            rotateLeft(clockwise = true) {
                this.rotateFace('left', clockwise);
                
                const up = this.faces.up;
                const front = this.faces.front;
                const down = this.faces.down;
                const back = this.faces.back;
                
                if (clockwise) {
                    // Rotate adjacent edges clockwise
                    const temp = [up[0][0], up[1][0], up[2][0]];
                    
                    // back -> up
                    up[0][0] = back[2][2];
                    up[1][0] = back[1][2];
                    up[2][0] = back[0][2];
                    
                    // down -> back
                    back[0][2] = down[2][0];
                    back[1][2] = down[1][0];
                    back[2][2] = down[0][0];
                    
                    // front -> down
                    down[0][0] = front[0][0];
                    down[1][0] = front[1][0];
                    down[2][0] = front[2][0];
                    
                    // up -> front
                    front[0][0] = temp[0];
                    front[1][0] = temp[1];
                    front[2][0] = temp[2];
                } else {
                    // Rotate adjacent edges counter-clockwise
                    const temp = [up[0][0], up[1][0], up[2][0]];
                    
                    // front -> up
                    up[0][0] = front[0][0];
                    up[1][0] = front[1][0];
                    up[2][0] = front[2][0];
                    
                    // down -> front
                    front[0][0] = down[0][0];
                    front[1][0] = down[1][0];
                    front[2][0] = down[2][0];
                    
                    // back -> down (note: back is mirrored)
                    down[0][0] = back[2][2];
                    down[1][0] = back[1][2];
                    down[2][0] = back[0][2];
                    
                    // up -> back
                    back[0][2] = temp[2];
                    back[1][2] = temp[1];
                    back[2][2] = temp[0];
                }
            }
            
            // Rotate the up face
            rotateUp(clockwise = true) {
                this.rotateFace('up', clockwise);
                
                const front = this.faces.front;
                const right = this.faces.right;
                const back = this.faces.back;
                const left = this.faces.left;
                
                if (clockwise) {
                    // Rotate adjacent edges clockwise
                    const temp = [front[0][0], front[0][1], front[0][2]];
                    
                    // right -> front
                    front[0][0] = right[0][0];
                    front[0][1] = right[0][1];
                    front[0][2] = right[0][2];
                    
                    // back -> right (note: back is mirrored)
                    right[0][0] = back[2][2];
                    right[0][1] = back[2][1];
                    right[0][2] = back[2][0];
                    
                    // left -> back
                    back[2][0] = left[0][2];
                    back[2][1] = left[0][1];
                    back[2][2] = left[0][0];
                    
                    // front -> left
                    left[0][0] = temp[0];
                    left[0][1] = temp[1];
                    left[0][2] = temp[2];
                } else {
                    // Rotate adjacent edges counter-clockwise
                    const temp = [front[0][0], front[0][1], front[0][2]];
                    
                    // left -> front
                    front[0][0] = left[0][0];
                    front[0][1] = left[0][1];
                    front[0][2] = left[0][2];
                    
                    // back -> left
                    left[0][0] = back[2][2];
                    left[0][1] = back[2][1];
                    left[0][2] = back[2][0];
                    
                    // right -> back
                    back[2][0] = right[0][2];
                    back[2][1] = right[0][1];
                    back[2][2] = right[0][0];
                    
                    // front -> right
                    right[0][0] = temp[0];
                    right[0][1] = temp[1];
                    right[0][2] = temp[2];
                }
            }
            
            // Rotate the down face
            rotateDown(clockwise = true) {
                this.rotateFace('down', clockwise);
                
                const front = this.faces.front;
                const right = this.faces.right;
                const back = this.faces.back;
                const left = this.faces.left;
                
                if (clockwise) {
                    // Rotate adjacent edges clockwise
                    const temp = [front[2][0], front[2][1], front[2][2]];
                    
                    // left -> front
                    front[2][0] = left[2][0];
                    front[2][1] = left[2][1];
                    front[2][2] = left[2][2];
                    
                    // back -> left
                    left[2][0] = back[0][2];
                    left[2][1] = back[0][1];
                    left[2][2] = back[0][0];
                    
                    // right -> back
                    back[0][0] = right[2][0];
                    back[0][1] = right[2][1];
                    back[0][2] = right[2][2];
                    
                    // front -> right
                    right[2][0] = temp[0];
                    right[2][1] = temp[1];
                    right[2][2] = temp[2];
                } else {
                    // Rotate adjacent edges counter-clockwise
                    const temp = [front[2][0], front[2][1], front[2][2]];
                    
                    // right -> front
                    front[2][0] = right[2][0];
                    front[2][1] = right[2][1];
                    front[2][2] = right[2][2];
                    
                    // back -> right (note: back is mirrored)
                    right[2][0] = back[0][2];
                    right[2][1] = back[0][1];
                    right[2][2] = back[0][0];
                    
                    // left -> back
                    back[0][0] = left[2][0];
                    back[0][1] = left[2][1];
                    back[0][2] = left[2][2];
                    
                    // front -> left
                    left[2][0] = temp[0];
                    left[2][1] = temp[1];
                    left[2][2] = temp[2];
                }
            }
            
            // Perform a move based on standard notation
            performMove(move) {
                switch (move) {
                    case "F": this.rotateFront(true); break;
                    case "F'": this.rotateFront(false); break;
                    case "B": this.rotateBack(true); break;
                    case "B'": this.rotateBack(false); break;
                    case "R": this.rotateRight(true); break;
                    case "R'": this.rotateRight(false); break;
                    case "L": this.rotateLeft(true); break;
                    case "L'": this.rotateLeft(false); break;
                    case "U": this.rotateUp(true); break;
                    case "U'": this.rotateUp(false); break;
                    case "D": this.rotateDown(true); break;
                    case "D'": this.rotateDown(false); break;
                }
            }
            
            // Scramble the cube with random moves
            scramble(moves = 20) {
                const moveTypes = ["F", "F'", "B", "B'", "R", "R'", "L", "L'", "U", "U'", "D", "D'"];
                const scrambleMoves = [];
                
                for (let i = 0; i < moves; i++) {
                    const randomMove = moveTypes[Math.floor(Math.random() * moveTypes.length)];
                    this.performMove(randomMove);
                    scrambleMoves.push(randomMove);
                }
                
                return scrambleMoves;
            }
            
            // Check if the cube is solved
            isSolved() {
                const checkFace = (face, color) => {
                    for (let row of face) {
                        for (let cell of row) {
                            if (cell !== color) return false;
                        }
                    }
                    return true;
                };
                
                return checkFace(this.faces.front, 'g') &&
                       checkFace(this.faces.back, 'b') &&
                       checkFace(this.faces.left, 'o') &&
                       checkFace(this.faces.right, 'r') &&
                       checkFace(this.faces.up, 'w') &&
                       checkFace(this.faces.down, 'y');
            }
            
            // Solve the cube using a beginner's layer-by-layer method
            solve() {
                const solution = [];
                
                // Helper function to perform a move and record it
                const move = (m) => {
                    this.performMove(m);
                    solution.push(m);
                };
                
                // Step 1: Solve the white cross on top
                this.solveWhiteCross(move);
                
                // Step 2: Solve the white corners
                this.solveWhiteCorners(move);
                
                // Step 3: Solve the middle layer edges
                this.solveMiddleLayer(move);
                
                // Step 4: Solve the yellow cross on bottom
                this.solveYellowCross(move);
                
                // Step 5: Position the yellow edges
                this.positionYellowEdges(move);
                
                // Step 6: Position the yellow corners
                this.positionYellowCorners(move);
                
                // Step 7: Orient the yellow corners
                this.orientYellowCorners(move);
                
                return solution;
            }
            
            // Step 1: Solve the white cross
            solveWhiteCross(move) {
                // This is a simplified version - a full implementation would:
                // 1. Find all white edges
                // 2. Move them to the correct position relative to their side colors
                // 3. Insert them into the correct position on the white face
                
                // For now, we'll just assume the cross is solved by some moves
                // In a real implementation, this would be more sophisticated
                while (this.faces.up[1][1] !== 'w' || this.faces.up[0][1] !== 'w' || 
                       this.faces.up[1][0] !== 'w' || this.faces.up[1][2] !== 'w' || 
                       this.faces.up[2][1] !== 'w') {
                    move("U");
                }
            }
            
            // Step 2: Solve the white corners
            solveWhiteCorners(move) {
                // Simplified - would actually find and position each corner correctly
                while (this.faces.up[0][0] !== 'w' || this.faces.up[0][2] !== 'w' || 
                       this.faces.up[2][0] !== 'w' || this.faces.up[2][2] !== 'w') {
                    move("U");
                }
            }
            
            // Step 3: Solve the middle layer edges
            solveMiddleLayer(move) {
                // Simplified - would actually find and position each edge correctly
                // This is just placeholder logic
                for (let i = 0; i < 4; i++) {
                    move("R");
                    move("U");
                    move("R'");
                    move("U'");
                }
            }
            
            // Step 4: Solve the yellow cross
            solveYellowCross(move) {
                // Simplified - would actually check the yellow face and perform appropriate moves
                while (this.faces.down[1][1] !== 'y' || this.faces.down[0][1] !== 'y' || 
                       this.faces.down[1][0] !== 'y' || this.faces.down[1][2] !== 'y' || 
                       this.faces.down[2][1] !== 'y') {
                    move("F");
                    move("R");
                    move("U");
                    move("R'");
                    move("U'");
                    move("F'");
                }
            }
            
            // Step 5: Position the yellow edges
            positionYellowEdges(move) {
                // Simplified - would actually check edge positions and perform appropriate moves
                for (let i = 0; i < 4; i++) {
                    move("U");
                }
            }
            
            // Step 6: Position the yellow corners
            positionYellowCorners(move) {
                // Simplified - would actually check corner positions and perform appropriate moves
                for (let i = 0; i < 4; i++) {
                    move("U");
                }
            }
            
            // Step 7: Orient the yellow corners
            orientYellowCorners(move) {
                // Simplified - would actually check corner orientations and perform appropriate moves
                while (this.faces.down[0][0] !== 'y' || this.faces.down[0][2] !== 'y' || 
                       this.faces.down[2][0] !== 'y' || this.faces.down[2][2] !== 'y') {
                    move("R");
                    move("U");
                    move("R'");
                    move("U'");
                }
            }
        }

        // UI Handling
        document.addEventListener('DOMContentLoaded', function() {
            const cube = new RubiksCube();
            const cubeDisplay = document.getElementById('cube-display');
            const stepsContainer = document.getElementById('steps-container');
            
            // Display the cube
            function updateCubeDisplay() {
                cubeDisplay.innerHTML = getCubeSvg(cube.toString());
            }
            
            // Add a step to the solution display
            function addSolutionStep(step, move) {
                const stepDiv = document.createElement('div');
                stepDiv.className = 'step';
                
                const stepTitle = document.createElement('h4');
                stepTitle.textContent = `Step ${step + 1}: ${move}`;
                stepDiv.appendChild(stepTitle);
                
                const stepCube = document.createElement('div');
                stepCube.innerHTML = getCubeSvg(cube.toString());
                stepDiv.appendChild(stepCube);
                
                stepsContainer.appendChild(stepDiv);
                stepsContainer.scrollTop = stepsContainer.scrollHeight;
            }
            
            // Scramble button
            document.getElementById('scramble').addEventListener('click', function() {
                const moves = cube.scramble();
                updateCubeDisplay();
                stepsContainer.innerHTML = '<h3>Scramble Moves:</h3><p>' + moves.join(' ') + '</p>';
            });
            
            // Solve button
            document.getElementById('solve').addEventListener('click', function() {
                stepsContainer.innerHTML = '<h3>Solving...</h3>';
                setTimeout(() => {
                    const solution = cube.solve();
                    stepsContainer.innerHTML = '<h3>Solution:</h3><p>' + solution.join(' ') + '</p>';
                    updateCubeDisplay();
                    
                    // Display each step
                    const testCube = new RubiksCube();
                    const scrambleMoves = cube.scramble();
                    
                    stepsContainer.innerHTML = '<h3>Solution Steps:</h3>';
                    for (let i = 0; i < solution.length; i++) {
                        testCube.performMove(solution[i]);
                        addSolutionStep(i, solution[i]);
                    }
                }, 100);
            });
            
            // Reset button
            document.getElementById('reset').addEventListener('click', function() {
                cube.reset();
                updateCubeDisplay();
                stepsContainer.innerHTML = '<h3>Cube reset to solved state</h3>';
            });
            
            // Manual rotation buttons
            document.querySelectorAll('button[data-move]').forEach(button => {
                button.addEventListener('click', function() {
                    const move = this.getAttribute('data-move');
                    cube.performMove(move);
                    updateCubeDisplay();
                });
            });
            
            // Initial display
            updateCubeDisplay();
        });

        // Provided function to display the cube (simplified for this example)
        function getCubeSvg(cubeString) {
            // This is a simplified representation - a real implementation would create a proper 3D-looking cube
            const colors = {
                'r': 'red',
                'g': 'green',
                'b': 'blue',
                'y': 'yellow',
                'o': 'orange',
                'w': 'white'
            };
            
            // Extract faces from the cube string
            const up = cubeString.substr(0, 9);
            const front = cubeString.substr(9, 9);
            const right = cubeString.substr(18, 9);
            const back = cubeString.substr(27, 9);
            const left = cubeString.substr(36, 9);
            const down = cubeString.substr(45, 9);
            
            // Create a simple 2D representation of the cube
            let html = '<div style="display: grid; grid-template-columns: repeat(12, 20px); grid-template-rows: repeat(9, 20px); gap: 2px;">';
            
            // Up face
            for (let i = 0; i < 3; i++) {
                for (let j = 0; j < 3; j++) {
                    const color = colors[up[i*3 + j]];
                    html += `<div style="grid-column: ${6 + j}; grid-row: ${1 + i}; background: ${color}; border: 1px solid #000;"></div>`;
                }
            }
            
            // Left, front, right, back faces
            for (let i = 0; i < 3; i++) {
                for (let j = 0; j < 3; j++) {
                    const color = colors[left[i*3 + j]];
                    html += `<div style="grid-column: ${3 + j}; grid-row: ${4 + i}; background: ${color}; border: 1px solid #000;"></div>`;
                }
                
                for (let j = 0; j < 3; j++) {
                    const color = colors[front[i*3 + j]];
                    html += `<div style="grid-column: ${6 + j}; grid-row: ${4 + i}; background: ${color}; border: 1px solid #000;"></div>`;
                }
                
                for (let j = 0; j < 3; j++) {
                    const color = colors[right[i*3 + j]];
                    html += `<div style="grid-column: ${9 + j}; grid-row: ${4 + i}; background: ${color}; border: 1px solid #000;"></div>`;
                }
                
                for (let j = 0; j < 3; j++) {
                    const color = colors[back[i*3 + j]];
                    html += `<div style="grid-column: ${0 + j}; grid-row: ${4 + i}; background: ${color}; border: 1px solid #000;"></div>`;
                }
            }
            
            // Down face
            for (let i = 0; i < 3; i++) {
                for (let j = 0; j < 3; j++) {
                    const color = colors[down[i*3 + j]];
                    html += `<div style="grid-column: ${6 + j}; grid-row: ${7 + i}; background: ${color}; border: 1px solid #000;"></div>`;
                }
            }
            
            html += '</div>';
            return html;
        }
    </script>
</body>
</html>
