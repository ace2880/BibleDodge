```jsx name=src/BibleDodge.jsx
import React, { useRef, useEffect, useState, useCallback } from "react";

// Update asset paths to match your project structure.
// Place images in src/assets/ and audio files in public/sfx/
const SFX = {
  collect: "/sfx/collect.mp3",
  hit: "/sfx/hit.mp3",
  shoot: "/sfx/shoot.mp3",
};
const IMAGES = {
  hero: require("./assets/hero.png"),
  sin: require("./assets/sin.png"),
  scripture: require("./assets/scripture.png"),
};

// Sound playback helper
function playSound(filename) {
  const audio = new Audio(filename);
  audio.volume = 0.5;
  audio.play();
}

// Generate a random y coordinate within bounds
function randomY() {
  return Math.floor(Math.random() * 340) + 20;
}

// Rectangle collision detection
function collide(a, b) {
  return (
    a.x < b.x + b.size &&
    a.x + a.size > b.x &&
    a.y < b.y + b.size &&
    a.y + a.size > b.y
  );
}

const GAME_WIDTH = 500;
const GAME_HEIGHT = 400;

export default function BibleDodge() {
  const [score, setScore] = useState(0);
  const [hero, setHero] = useState({ x: 50, y: 180, size: 40 });
  const [sins, setSins] = useState([]);
  const [scriptures, setScriptures] = useState([]);
  const [gameOver, setGameOver] = useState(false);

  const requestRef = useRef();

  // Arrow key movement
  useEffect(() => {
    function onKey(e) {
      setHero(h => {
        let nx = h.x, ny = h.y;
        if (e.key === "ArrowUp") ny = Math.max(0, h.y - 20);
        if (e.key === "ArrowDown") ny = Math.min(GAME_HEIGHT - h.size, h.y + 20);
        if (e.key === "ArrowLeft") nx = Math.max(0, h.x - 20);
        if (e.key === "ArrowRight") nx = Math.min(GAME_WIDTH - h.size, h.x + 20);
        return { ...h, x: nx, y: ny };
      });
    }
    window.addEventListener("keydown", onKey);
    return () => window.removeEventListener("keydown", onKey);
  }, []);

  // Main loop
  const animate = useCallback(() => {
    setSins(sinsOld =>
      sinsOld.map(sin => ({ ...sin, x: sin.x - sin.speed }))
        .filter(sin => sin.x + sin.size > 0)
    );
    setScriptures(scs =>
      scs.map(sc => ({ ...sc, x: sc.x - sc.speed })).filter(sc => sc.x + sc.size > 0)
    );
    setSins(sinsArr => {
      let collided = false;
      const filtered = sinsArr.filter(sin => {
        if (collide({ ...sin }, { ...hero })) {
          collided = true;
          return false;
        }
        return true;
      });
      if (collided) {
        playSound(SFX.hit);
        setGameOver(true);
      }
      return filtered;
    });
    setScriptures(scsArr => {
      let inc = 0;
      const filtered = scsArr.filter(sc => {
        if (collide({ ...sc }, { ...hero })) {
          inc += 1;
          return false;
        }
        return true;
      });
      if (inc > 0) {
        setScore(s => s + inc);
        playSound(SFX.collect);
      }
      return filtered;
    });
    requestRef.current = requestAnimationFrame(animate);
  }, [hero]);

  useEffect(() => {
    if (!gameOver) {
      requestRef.current = requestAnimationFrame(animate);
    }
    return () => cancelAnimationFrame(requestRef.current);
  }, [animate, gameOver]);

  // Spawning logic
  useEffect(() => {
    if (gameOver) return;
    const sinInterval = setInterval(() => {
      setSins(sinsArr => [
        ...sinsArr,
        {
          x: GAME_WIDTH,
          y: randomY(),
          size: 40,
          speed: Math.random() * 2 + 2,
        },
      ]);
    }, 1400);

    const scriptureInterval = setInterval(() => {
      setScriptures(curr => [
        ...curr,
        {
          x: GAME_WIDTH,
          y: randomY(),
          size: 32,
          speed: Math.random() * 1.5 + 1,
        },
      ]);
    }, 1900);

    return () => {
      clearInterval(sinInterval);
      clearInterval(scriptureInterval);
    };
  }, [gameOver]);

  function restart() {
    setGameOver(false);
    setScore(0);
    setSins([]);
    setScriptures([]);
    setHero({ x: 50, y: 180, size: 40 });
    requestRef.current = requestAnimationFrame(animate);
  }

  return (
    <div className="flex flex-col items-center mt-6">
      <h1 className="text-3xl mb-2 font-semibold text-blue-700">BibleDodge</h1>
      <div
        style={{
          width: GAME_WIDTH,
          height: GAME_HEIGHT,
          border: "2px solid #444",
          background: "linear-gradient(to bottom, #f0f5ffd9 80%, #e8effa 100%)",
          position: "relative",
          overflow: "hidden",
        }}
      >
        {/* Hero */}
        <img
          src={IMAGES.hero}
          alt="Hero"
          style={{
            position: "absolute",
            left: hero.x,
            top: hero.y,
            width: hero.size,
            height: hero.size,
            zIndex: 10,
            transition: "left 0.05s, top 0.05s",
            pointerEvents: "none",
          }}
        />
        {/* Sins */}
        {sins.map((sin, i) => (
          <img
            src={IMAGES.sin}
            alt="sin"
            key={i}
            style={{
              position: "absolute",
              left: sin.x,
              top: sin.y,
              width: sin.size,
              height: sin.size,
              zIndex: 7,
              pointerEvents: "none",
            }}
          />
        ))}
        {/* Scriptures */}
        {scriptures.map((sc, i) => (
          <img
            src={IMAGES.scripture}
            alt="scripture"
            key={i}
            style={{
              position: "absolute",
              left: sc.x,
              top: sc.y,
              width: sc.size,
              height: sc.size,
              zIndex: 8,
              pointerEvents: "none",
            }}
          />
        ))}
        {/* Score Board */}
        <div
          className="bg-white bg-opacity-70 px-4 py-1 rounded font-bold text-lg absolute top-3 left-3 border border-yellow-400 text-gray-900 shadow"
          style={{ zIndex: 20 }}
        >
          Score: {score}
        </div>
        {/* Game Over */}
        {gameOver && (
          <div
            className="absolute inset-0 bg-black bg-opacity-60 flex flex-col items-center justify-center z-30"
            style={{ minHeight: GAME_HEIGHT }}
          >
            <h2 className="text-4xl text-white font-bold mb-4">Game Over</h2>
            <button
              onClick={restart}
              className="px-6 py-2 bg-blue-500 text-white rounded shadow font-semibold hover:bg-blue-400"
            >
              Restart
            </button>
          </div>
        )}
      </div>
      <p className="mt-4 text-gray-700">
        Use <strong>arrow keys</strong> to dodge sin and collect scripture!
      </p>
    </div>
  );
}
```
**Instructions:**

- Save this as `src/BibleDodge.jsx` in your React project.
- Save `hero.png`, `sin.png`, and `scripture.png` in `src/assets/`.
- Place `collect.mp3`, `hit.mp3`, and `shoot.mp3` in `public/sfx/`.
- Import and use `<BibleDodge />` in `App.jsx`.
- Run your development server with `npm run dev`.

Let me know if you need any further adjustments or a sample `App.jsx`!
