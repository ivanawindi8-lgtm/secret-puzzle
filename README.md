const password = '0404';
const slides = document.querySelectorAll('.slide');
let currentSlide = 0;
let puzzleComplete = false;

// Password logic
document.getElementById('enterBtn').addEventListener('click', checkPassword);
document.getElementById('passwordInput').addEventListener('keypress', function(e) {
    if (e.key === 'Enter') checkPassword();
});

function checkPassword() {
    const input = document.getElementById('passwordInput').value;
    if (input === password) {
        goToSlide(1);
        createFloatingHearts();
    } else {
        document.getElementById('passwordInput').style.borderColor = '#ff6b6b';
        setTimeout(() => {
            document.getElementById('passwordInput').style.borderColor = '#ffeef2';
        }, 500);
    }
}

// Slide navigation
document.getElementById('nextBtn').addEventListener('click', () => goToSlide(2));

// Music player
const music = document.getElementById('bgMusic');
const vinyl = document.getElementById('vinyl');
const musicBtn = document.getElementById('musicToggle');

musicBtn.addEventListener('click', toggleMusic);

function toggleMusic() {
    if (music.paused) {
        music.play();
        vinyl.classList.add('spinning');
        musicBtn.textContent = '⏸️ Pause';
    } else {
        music.pause();
        vinyl.classList.remove('spinning');
        musicBtn.textContent = '▶️ Play';
    }
}

function goToSlide(slideIndex) {
    slides[currentSlide].classList.remove('active');
    slides[slideIndex].classList.add('active');
    currentSlide = slideIndex;
    
    if (slideIndex === 1) {
        setTimeout(() => {
            document.querySelectorAll('.letter-scroll p').forEach(p => {
                p.style.animationPlayState = 'running';
            });
        }, 500);
    }
}

// Puzzle Logic
const puzzleImage = 'images/puzzle.jpg';
const pieces = 16; // 4x4 grid
let draggedPiece = null;
let dropZone = null;

function initPuzzle() {
    const board = document.getElementById('puzzleBoard');
    board.innerHTML = '';
    
    // Shuffle pieces
    const positions = Array.from({length: pieces}, (_, i) => i);
    shuffleArray(positions);
    
    for (let i = 0; i < pieces; i++) {
        const piece = document.createElement('div');
        piece.className = 'puzzle-piece';
        piece.style.backgroundImage = `url(${puzzleImage})`;
        piece.style.backgroundPosition = `-${(positions[i] % 4) * 140}px -${Math.floor(positions[i] / 4) * 140}px`;
        
        piece.draggable = true;
        piece.dataset.position = positions[i];
        
        piece.addEventListener('dragstart', dragStart);
        piece.addEventListener('dragend', dragEnd);
        piece.addEventListener('dragover', dragOver);
        piece.addEventListener('drop', drop);
        
        board.appendChild(piece);
    }
}

function shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
    }
}

function dragStart(e) {
    draggedPiece = this;
    setTimeout(() => this.style.display = 'none', 0);
}

function dragEnd(e) {
    setTimeout(() => {
        this.style.display = 'block';
        draggedPiece = null;
    }, 0);
}

function dragOver(e) {
    e.preventDefault();
}

function drop(e) {
    e.preventDefault();
    if (draggedPiece) {
        const target = e.currentTarget;
        const draggedPos = parseInt(draggedPiece.dataset.position);
        const targetPos = parseInt(target.dataset.position);
        
        if (draggedPos === targetPos) {
            draggedPiece.classList.add('correct');
            checkPuzzleComplete();
        }
        
        // Swap positions visually
        const draggedStyle = draggedPiece.style.backgroundPosition;
        const targetStyle = target.style.backgroundPosition;
        draggedPiece.style.backgroundPosition = targetStyle;
        target.style.backgroundPosition = draggedStyle;
        
        draggedPiece.dataset.position = targetPos;
        target.dataset.position = draggedPos;
    }
}

function checkPuzzleComplete() {
    const pieces = document.querySelectorAll('.puzzle-piece');
    const allCorrect = Array.from(pieces).every(piece => 
        parseInt(piece.dataset.position) === parseInt(piece.dataset.position)
    );
    
    if (allCorrect) {
        setTimeout(() => {
            document.getElementById('puzzleBoard').style.display = 'none';
            document.getElementById('puzzleComplete').style.display = 'block';
            createHeartExplosion();
        }, 1000);
    }
}

function createFloatingHearts() {
    for (let i = 0; i < 20; i++) {
        setTimeout(() => {
            const heart = document.createElement('div');
            heart.className = 'heart-float';
            heart.innerHTML = ['💕', '💖', '💗', '💓', '💞'][Math.floor(Math.random() * 5)];
            heart.style.left = Math.random() * 100 + '%';
            heart.style.animationDuration = (Math.random() * 3 + 3) + 's';
            document.body.appendChild(heart);
            
            setTimeout(() => heart.remove(), 6000);
        }, i * 200);
    }
}

function createHeartExplosion() {
    for (let i = 0; i < 50; i++) {
        setTimeout(() => {
            const heart = document.createElement('div');
            heart.innerHTML = ['💕', '💖', '💗', '💓', '💞'][Math.floor(Math.random() * 5)];
            heart.style.position = 'fixed';
            heart.style.left = '50%';
            heart.style.top = '50%';
            heart.style.fontSize = '2rem';
            heart.style.pointerEvents = 'none';
            heart.style.animation = 'explodeHearts 2s ease-out forwards';
            document.body.appendChild(heart);
            
            setTimeout(() => heart.remove(), 2000);
        }, i * 50);
    }
}

function resetAll() {
    currentSlide = 0;
    document.getElementById('passwordInput').value = '';
    slides.forEach(slide => slide.classList.remove('active'));
    slides[0].classList.add('active');
    document.getElementById('puzzleBoard').style.display = 'grid';
    document.getElementById('puzzleComplete').style.display = 'none';
    document.querySelectorAll('.puzzle-piece').forEach(p => p.classList.remove('correct'));
    music.pause();
    vinyl.classList.remove('spinning');
    musicBtn.textContent = '▶️ Play';
}

// Initialize
document.addEventListener('DOMContentLoaded', () => {
    initPuzzle();
});
