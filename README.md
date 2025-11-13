# BibleDodge
Dodging The unclean 
BibleDodge/
├── public/
│   ├── sfx/
│   │   ├── collect.mp3
│   │   ├── shoot.mp3
│   │   └── hit.mp3
│   ├── favicon.ico
│   └── index.html
│
├── src/
│   ├── BibleDodge.jsx
│   ├── App.jsx
│   ├── main.jsx
│   ├── index.css
│   └── assets/
│       ├── hero.png
│       ├── sin.png
│       └── scripture.png
│
├── .gitignore
├── package.json
├── tailwind.config.js
├── postcss.config.js
└── README.md
<div className="text-sm text-yellow-400">High Score: {localStorage.getItem('highScore') || 0}</div>
useEffect(() => {
  localStorage.setItem('highScore', Math.max(score, Number(localStorage.getItem('highScore') || 0)));
}, [score]);
npm run dev
npm install
