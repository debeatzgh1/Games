                                                    <!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Debeatzgh Games — Floating Launcher</title>
<style>
  :root{
    --bg:#060606;
    --panel:#0f0f10;
    --card:#111214;
    --accent:#00ff88;
    --muted:#bfcfc0;
    --glass: rgba(255,255,255,0.02);
  }

  /* Basic page reset */
  html,body{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, "Segoe UI", Roboto, Arial;}
  body{background:var(--bg); color:#fff; -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale;}

  /* Floating icon-only button - left center */
  .floating-launcher {
    position:fixed;
    left:18px;
    top:50%;
    transform:translateY(-50%);
    z-index:9998;
    width:62px;
    height:62px;
    border-radius:50%;
    background:linear-gradient(180deg,var(--panel),rgba(0,0,0,.4));
    display:flex;
    align-items:center;
    justify-content:center;
    box-shadow:0 8px 30px rgba(0,0,0,.6), 0 2px 8px rgba(0,0,0,.5);
    border:2px solid rgba(255,255,255,0.04);
    cursor:pointer;
    transition:transform .18s ease, box-shadow .18s;
    backdrop-filter: blur(4px);
  }
  .floating-launcher:active{ transform:translateY(-50%) scale(.98) }
  .floating-launcher .icon { width:28px; height:28px; display:block; fill:var(--accent) }

  /* Slide-out panel */
  .games-panel {
    position:fixed;
    left:0;
    top:0;
    height:100vh;
    width:0;
    max-width:420px;
    background:linear-gradient(180deg,var(--panel),#0c0c0c);
    box-shadow: 4px 0 40px rgba(0,0,0,0.6);
    z-index:9997;
    overflow:hidden;
    transition: width .32s cubic-bezier(.2,.9,.2,1);
    border-right:1px solid rgba(255,255,255,0.03);
  }
  .games-panel.open { width: 92vw; max-width:420px; }

  .panel-inner { padding:18px; height:100%; display:flex; flex-direction:column; gap:12px; box-sizing:border-box; }

  .panel-header {
    display:flex;align-items:center;gap:12px;
  }
  .panel-header h2 { margin:0;font-size:1.05rem;color:var(--accent) }
  .panel-header p { margin:0;color:var(--muted);font-size:.9rem }

  .close-panel {
    margin-left:auto;
    background:transparent;
    border:none;
    color:var(--muted);
    font-size:1.05rem;
    cursor:pointer;
    padding:6px 8px;
    border-radius:8px;
  }
  .close-panel:hover { background:var(--glass) }

  /* Games list */
  .games-list { overflow:auto; padding-right:6px; margin-top:8px; display:grid; gap:12px; grid-auto-rows:min-content; }
  .game-card {
    display:flex; gap:12px; align-items:flex-start;
    background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.00));
    border-radius:10px; padding:12px; border:1px solid rgba(255,255,255,0.03);
  }
  .thumb {
    width:74px; height:56px; border-radius:8px; background:linear-gradient(180deg,#0a0a0a,#121212); flex:0 0 74px;
    display:flex;align-items:center;justify-content:center;font-size:26px;
    color:var(--muted);
    overflow:hidden;
  }
  .game-body { flex:1; min-width:0; display:flex; flex-direction:column; gap:6px; }
  .game-title { font-weight:700; color:var(--accent); font-size:.95rem; margin:0; }
  .game-desc { margin:0; color:var(--muted); font-size:.86rem; line-height:1.1; }
  .game-caption { margin-top:4px; color:rgba(191,207,192,0.6); font-size:.78rem }

  .card-actions { display:flex; gap:8px; margin-top:8px; align-items:center }
  .play-btn {
    background:var(--accent);
    color:#000;
    border:none;
    padding:8px 12px;
    border-radius:8px;
    font-weight:700;
    cursor:pointer;
  }
  .info-btn {
    background:transparent;
    color:var(--muted);
    border:1px solid rgba(255,255,255,0.03);
    padding:6px 10px;
    border-radius:8px;
    cursor:pointer;
    font-size:.85rem;
  }

  /* Fullscreen iframe modal */
  .game-modal {
    position:fixed; inset:0; display:none; z-index:9999;
    background:linear-gradient(180deg, rgba(2,2,2,0.96), rgba(0,0,0,0.98));
    align-items:center; justify-content:center; padding:18px; box-sizing:border-box;
  }
  .game-modal.open { display:flex; }
  .modal-frame {
    width:100%; height:100%; max-width:1100px; max-height:88vh; border-radius:12px; overflow:hidden; border:1px solid rgba(255,255,255,0.04);
    background:#000;
  }
  .modal-top {
    height:56px; display:flex; align-items:center; gap:12px; padding:10px 12px; box-sizing:border-box; background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(0,0,0,0.02));
  }
  .modal-title { color:var(--accent); font-weight:700; font-size:1rem; }
  .modal-close { margin-left:auto; background:transparent; border:none; color:var(--muted); font-size:1rem; cursor:pointer; padding:8px 10px; border-radius:8px }
  .modal-iframe { width:100%; height:calc(100% - 56px); border:none; display:block; }

  /* small screens tuning */
  @media (max-width:560px){
    .floating-launcher { left:12px; width:56px;height:56px }
    .thumb { width:64px;height:48px }
    .panel-inner{padding:14px}
  }
</style>
</head>
<body>

<!-- Floating icon-only launcher (Left Center) -->
<button class="floating-launcher" id="launcher" aria-label="Open games panel" title="Games">
  <!-- Gamepad SVG icon -->
  <svg class="icon" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
    <path d="M6 10.5C6 8.57 7.57 7 9.5 7H14.5C16.43 7 18 8.57 18 10.5V12.5C18 14.43 16.43 16 14.5 16H9.5C7.57 16 6 14.43 6 12.5V10.5Z" fill="currentColor"/>
    <circle cx="9.5" cy="10.5" r="0.9" fill="#000"/>
    <circle cx="14.5" cy="13.5" r="0.9" fill="#000"/>
    <path d="M4 12.5C4 9.46 6.46 7 9.5 7H14.5C17.54 7 20 9.46 20 12.5V13.75C20 15.77 18.27 17.5 16.25 17.5H7.75C5.73 17.5 4 15.77 4 13.75V12.5Z" fill="rgba(0,0,0,0.25)"/>
  </svg>
</button>

<!-- Slide-out panel -->
<aside class="games-panel" id="gamesPanel" aria-hidden="true">
  <div class="panel-inner">
    <div class="panel-header">
      <div>
        <h2>🎮 Debeatzgh Games</h2>
        <p>Quick games to relax, practice, and get inspired — open any game in fullscreen.</p>
      </div>
      <button class="close-panel" id="closePanel" aria-label="Close panel">✕</button>
    </div>

    <div class="games-list" id="gamesList" tabindex="0">
      <!-- Card: Music Trivia Quiz -->
      <div class="game-card" data-game="music-quiz.html" data-title="Music Trivia Quiz">
        <div class="thumb" aria-hidden>❓</div>
        <div class="game-body">
          <div class="game-title">Music Trivia Quiz</div>
          <div class="game-desc">20+ curated music questions — test your knowledge and beat the score.</div>
          <div class="game-caption">Category: Trivia • Time: ~5–10 mins</div>
          <div class="card-actions">
            <button class="play-btn" data-src="music-quiz.html">Play Now</button>
            <button class="info-btn" data-info="A 20+ question music quiz about pop, Afrobeats, production, and Ghanaian music.">Info</button>
          </div>
        </div>
      </div>

      <!-- Card: Spin the Wheel -->
      <div class="game-card" data-game="spin-wheel.html" data-title="Spin The Wheel">
        <div class="thumb" aria-hidden>🎛️</div>
        <div class="game-body">
          <div class="game-title">Spin The Wheel</div>
          <div class="game-desc">Beat idea generator — spin for quick prompts to spark a session.</div>
          <div class="game-caption">Category: Creativity • Replayable</div>
          <div class="card-actions">
            <button class="play-btn" data-src="spin-wheel.html">Play Now</button>
            <button class="info-btn" data-info="Spin for beat ideas such as 'Afrobeat drums' or 'lo-fi pad'.">Info</button>
          </div>
        </div>
      </div>

      <!-- Card: Memory Match -->
      <div class="game-card" data-game="memory-match.html" data-title="Memory Match">
        <div class="thumb" aria-hidden>🧠</div>
        <div class="game-body">
          <div class="game-title">Memory Match</div>
          <div class="game-desc">Flip and match music icons — relaxing and good for short sessions.</div>
          <div class="game-caption">Category: Casual • Family-friendly</div>
          <div class="card-actions">
            <button class="play-btn" data-src="memory-match.html">Play Now</button>
            <button class="info-btn" data-info="Flip pairs and match music icons. Tracks moves but no backend.">Info</button>
          </div>
        </div>
      </div>

      <!-- Card: Slide Puzzle -->
      <div class="game-card" data-game="puzzle-game.html" data-title="Slide Puzzle">
        <div class="thumb" aria-hidden>🧩</div>
        <div class="game-body">
          <div class="game-title">Slide Puzzle</div>
          <div class="game-desc">Classic sliding puzzle using the Debeatzgh image — solve the grid.</div>
          <div class="game-caption">Category: Puzzle • Focus</div>
          <div class="card-actions">
            <button class="play-btn" data-src="puzzle-game.html">Play Now</button>
            <button class="info-btn" data-info="4×4 sliding puzzle built with client-side JS.">Info</button>
          </div>
        </div>
      </div>

      <!-- Card: Emoji Slot Machine -->
      <div class="game-card" data-game="slot-machine.html" data-title="Slot Machine">
        <div class="thumb" aria-hidden>🎰</div>
        <div class="game-body">
          <div class="game-title">Emoji Slot Machine</div>
          <div class="game-desc">Spin the reels — music-themed emojis and small rewards of fun.</div>
          <div class="game-caption">Category: Arcade • Quick-play</div>
          <div class="card-actions">
            <button class="play-btn" data-src="slot-machine.html">Play Now</button>
            <button class="info-btn" data-info="Three-reel emoji slot — try for three-of-a-kind.">Info</button>
          </div>
        </div>
      </div>

      <!-- Card: Reaction Speed Test -->
      <div class="game-card" data-game="reaction-test.html" data-title="Reaction Test">
        <div class="thumb" aria-hidden>⚡</div>
        <div class="game-body">
          <div class="game-title">Reaction Speed Test</div>
          <div class="game-desc">Tap when it turns green — measure your reaction time and save best.</div>
          <div class="game-caption">Category: Skill • Score</div>
          <div class="card-actions">
            <button class="play-btn" data-src="reaction-test.html">Play Now</button>
            <button class="info-btn" data-info="Simple reaction timer with local best score saved.">Info</button>
          </div>
        </div>
      </div>

    </div>

    <div style="margin-top:auto; color:var(--muted); font-size:.85rem; text-align:center; padding-top:6px;">
      Tip: Tap a card → Play Now (opens the game fullscreen). Press <strong>Esc</strong> to close.
    </div>
  </div>
</aside>

<!-- Fullscreen modal that hosts the game in an iframe -->
<div class="game-modal" id="gameModal" role="dialog" aria-modal="true" aria-hidden="true">
  <div style="width:100%;max-width:1100px;height:88vh;display:flex;flex-direction:column;border-radius:12px;overflow:hidden;">
    <div class="modal-top">
      <div style="display:flex;align-items:center;gap:12px;">
        <svg width="22" height="22" viewBox="0 0 24 24" fill="none" style="opacity:.9"><path d="M6 10.5C6 8.57 7.57 7 9.5 7H14.5C16.43 7 18 8.57 18 10.5V12.5C18 14.43 16.43 16 14.5 16H9.5C7.57 16 6 14.43 6 12.5V10.5Z" fill="currentColor" /></svg>
        <div class="modal-title" id="modalTitle">Loading…</div>
      </div>
      <button class="modal-close" id="modalClose" aria-label="Close game">✕</button>
    </div>

    <div class="modal-frame" style="flex:1;display:flex;flex-direction:column;">
      <iframe id="gameFrame" class="modal-iframe" title="Game frame" src=""></iframe>
    </div>
  </div>
</div>

<script>
  // Elements
  const launcher = document.getElementById('launcher');
  const panel = document.getElementById('gamesPanel');
  const closePanelBtn = document.getElementById('closePanel');
  const playButtons = document.querySelectorAll('.play-btn');
  const infoButtons = document.querySelectorAll('.info-btn');
  const modal = document.getElementById('gameModal');
  const modalClose = document.getElementById('modalClose');
  const frame = document.getElementById('gameFrame');
  const modalTitle = document.getElementById('modalTitle');

  // Toggle panel
  function openPanel(){
    panel.classList.add('open');
    panel.setAttribute('aria-hidden','false');
    // move focus into panel for accessibility
    setTimeout(()=> document.getElementById('gamesList').focus(), 120);
  }
  function closePanel(){
    panel.classList.remove('open');
    panel.setAttribute('aria-hidden','true');
    launcher.focus();
  }

  launcher.addEventListener('click', ()=> {
    if(panel.classList.contains('open')) closePanel();
    else openPanel();
  });
  closePanelBtn.addEventListener('click', closePanel);

  // Play a game in modal (fullscreen iframe)
  function openGame(src, title){
    frame.src = src;
    modalTitle.textContent = title || 'Game';
    modal.classList.add('open');
    modal.setAttribute('aria-hidden','false');
    document.body.style.overflow = 'hidden';
    // Focus close button for accessibility
    setTimeout(()=> modalClose.focus(), 180);
  }
  function closeGame(){
    modal.classList.remove('open');
    modal.setAttribute('aria-hidden','true');
    frame.src = ''; // unload iframe to stop audio/JS
    document.body.style.overflow = '';
  }

  // Attach events for Play Now buttons
  playButtons.forEach(btn=>{
    btn.addEventListener('click', (e)=>{
      const src = btn.getAttribute('data-src');
      // title from card
      const card = btn.closest('.game-card');
      const title = card ? (card.dataset.title || 'Game') : 'Game';
      openGame(src, title);
    });
  });

  // Info pop — small accessible alert (uses simple alert for now)
  infoButtons.forEach(b=>{
    b.addEventListener('click', ()=> {
      const info = b.getAttribute('data-info') || 'No information available';
      // small non-blocking UI: use alert as simple fallback
      alert(info);
    });
  });

  // Modal close
  modalClose.addEventListener('click', closeGame);
  // Close panel when clicking outside (on overlay area), limited behavior: clicking launcher again already toggles
  // Keyboard: ESC closes modal or panel
  document.addEventListener('keydown', (e)=>{
    if(e.key === 'Escape' || e.key === 'Esc'){
      if(modal.classList.contains('open')) closeGame();
      else if(panel.classList.contains('open')) closePanel();
    }
  });

  // Accessibility: allow Enter on launcher to toggle
  launcher.addEventListener('keyup', (e)=> { if(e.key === 'Enter') launcher.click(); });

  // Optional: open sample game if URL contains ?game=slot-machine.html
  (function checkQuery(){
    try {
      const params = new URLSearchParams(location.search);
      const game = params.get('game');
      if(game){
        // find title from card list
        const card = Array.from(document.querySelectorAll('.game-card')).find(c=>c.dataset.game===game);
        const title = card ? card.dataset.title : 'Game';
        openGame(game, title);
        // open panel as well
        openPanel();
      }
    } catch(e){}
  })();

  // Touch-friendly: allow drag to close panel — simple swipe left to close
  let startX = 0, touchActive=false;
  panel.addEventListener('touchstart', (ev)=> {
    if(!panel.classList.contains('open')) return;
    touchActive = true;
    startX = ev.touches[0].clientX;
  });
  panel.addEventListener('touchmove', (ev)=> {
    if(!touchActive) return;
    const dx = ev.touches[0].clientX - startX;
    // if dragging left by more than 60px, close
    if(dx < -60) { touchActive=false; closePanel(); }
  });
  panel.addEventListener('touchend', ()=> { touchActive=false; });

  // Small UX: close modal when iframe posts message { type: 'close' }
  window.addEventListener('message', (ev)=>{
    if(ev && ev.data && ev.data.type === 'close-game') closeGame();
  });
</script>

</body>
</html>
                                                                                                                                                     | Project                                                                                                                                                      | Description                                                                                                         | Action                                                                                                           |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| ![Custom Blogger Theme](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/generateamobile-firstresponsivebloggertemplatewithcustomizablecolorsfontsandsections1576324612066211977.jpg)                                                            | **[Custom Blogger Theme with Dynamic Post Loading and Logo](https://debeatzgh1.github.io/Custom-Blogger-Theme-for-with-Dynamic-Post-Loading-and-Logo-/)**    | A responsive, mobile-first Blogger theme with customizable sections, dynamic post loading, and custom logo support. | [🔗 View Demo](https://debeatzgh1.github.io/Custom-Blogger-Theme-for-with-Dynamic-Post-Loading-and-Logo-/)       |
| ![Popup Generator](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/createatoolthatgeneratesiframeorcard-styleembedsforindividualbloggerpostscompletewiththumbnailtitleandreadmorebuttonforcross-blogpromotion754077096311972631.jpg)            | **[Popup HTML Page Generator](https://debeatzgh1.github.io/popup-html-page-generator-blogger/)**                                                             | Easily generate iframe or card-style embeds for individual Blogger posts with thumbnails and read-more buttons.     | [🔗 View Demo](https://debeatzgh1.github.io/popup-html-page-generator-blogger/)                                  |
| ![Newsletter Widget](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/amodernflat-styleillustrationofanotificationbellwithglowinghighlightssurroundedbyabstractshapespaperenvelopesanddigitalicons28emailmessageupdate293314991682681990671.jpg) | **[Sliding Newsletter Signup Widget](https://debeatzgh1.github.io/Sliding-Newsletter-Signup-Widget-with-Pulse-Animation/)**                                  | A modern newsletter signup widget with sliding animation and pulsing notification highlights.                       | [🔗 View Demo](https://debeatzgh1.github.io/Sliding-Newsletter-Signup-Widget-with-Pulse-Animation/)              |
| ![Product Carousel](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/designacleanmodernthumbnailforabloggerproductscarouseltool1711994558720457535.jpg)                                                                                          | **[Blogger Product Carousel with WhatsApp Floating Button](https://debeatzgh1.github.io/Blogger-Product-Carousel-with-WhatsApp-Floating-Button/)**           | Showcase Blogger products in a responsive carousel with a built-in WhatsApp floating button for instant contact.    | [🔗 View Demo](https://debeatzgh1.github.io/Blogger-Product-Carousel-with-WhatsApp-Floating-Button/)             |
| ![Floating Dock](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/amodernminimallayoutwithafloatingdockofcolorfulroundicons28patreonbloggergithub29ontherightsideofacleanwebpagemockup6676994054500999142.jpg)                                   | **[Floating Dock Smart Iframe Modal](https://debeatzgh1.github.io/-Floating-Dock-Smart-Iframe-Modal/)**                                                      | A modern floating dock with colorful round icons and iframe modal integration.                                      | [🔗 View Demo](https://debeatzgh1.github.io/-Floating-Dock-Smart-Iframe-Modal/)                                  |
| ![Tailwind Homepage](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/createamodernandcleanthumbnailforawebdevelopmentproducttitledmodernhomepagestylingtemplatewithtailwindcss3420170625469385526.jpg)                                          | **[Modern Homepage Styling with TailwindCSS](https://debeatzgh1.github.io/Modern-homepage-styling-with-TailwindCSS-/)**                                      | A clean and modern homepage layout template styled with TailwindCSS.                                                | [🔗 View Demo](https://debeatzgh1.github.io/Modern-homepage-styling-with-TailwindCSS-/)                          |
| ![TechAdapt Solutions](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/wp-17550417188267308669484942620808.jpg)                                                                                                                                 | **[TechAdapt Solutions – Strategies for Modern Startups](https://debeatzgh1.github.io/TechAdapt-Solutions-Strategies-for-Modern-Startups-and-Individuals/)** | Practical strategies, resources, and tools for startups and individuals adapting to the modern digital world.       | [🔗 View Demo](https://debeatzgh1.github.io/TechAdapt-Solutions-Strategies-for-Modern-Startups-and-Individuals/) |

---

## 🛠️ Features

* ✅ Fully responsive designs
* ✅ Blogger-friendly & easy to embed
* ✅ Lightweight, no heavy dependencies
* ✅ Includes floating buttons, modals, and animations
* ✅ Perfect for enhancing engagement & monetization

---

## 📌 Explore More Scripts

👉 Check out the full curated collection here:
[**Firebase Curated Front-End Components**](https://github.com/debeatzgh1/firebase-front-end-components)

---

## 💡 Contribution

Want to suggest improvements or add your own scripts?

* Open an **issue**
* Submit a **pull request**
* Share your ideas in the discussions

---

## 📜 License

This project is released under the **MIT License** – free to use, modify, and share with attribution.

---

✨ Designed for creators, bloggers, and developers who want to level up their Blogger sites with modern UI components.

---


# 🌐 Personal Portfolio Website

A simple and responsive personal portfolio website built with **HTML, CSS, and JavaScript**.  
This project is perfect for showcasing your skills, projects, and contact information.  

Live Demo 👉 [View Portfolio](https://debeatzgh.github.io/portfolio-site/)

---

## ✨ Features
- Responsive design (works on mobile & desktop)  
- Navigation bar with smooth scrolling  
- Hero section with intro & call-to-action button  
- About section with bio details  
- Projects section with cards  
- Contact section with email & social links  
- **Dark mode toggle** 🌙☀️  

---

## 🛠️ Technologies Used
- **HTML5** – Structure  
- **CSS3** – Styling & Responsive design  
- **JavaScript (Vanilla)** – Interactivity (dark mode toggle)  
- **GitHub Pages** – Free hosting & deployment  

---

## 🚀 Getting Started
To run this project locally:  

1. Clone the repository:
   ```bash
   git clone https://github.com/debeatzgh1/portfolio-site.git

# 🚀 DeBeatzGH – AI Tools & Side Hustle Hub  

![DeBeatzGH Thumbnail](https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/designamodernminimalisticdesignfeaturinganai-themedicon28likeabraincircuitorrobot29overlaidwithdebeatzghoraitoolshustles6089986211026037047.jpg)  

## 🌟 About  
Welcome to **[DeBeatzGH](https://debeatzgh.wordpress.com/)** — your go-to hub for **AI tools, side hustle strategies, blogging resources, and digital growth guides**.  

Our platform is built to help **students, creators, startups, and professionals** unlock the power of AI, monetize their skills, and thrive in today’s digital economy.  

### ✨ What You’ll Find  
- 💡 Explore **AI prompts, tools, and hacks**  
- 📈 Discover **side hustle strategies & online income ideas**  
- ✍️ Access **blogging & digital business guides**  
- 🚀 Stay ahead with **regular updates and fresh insights**  

---

## 👉 Get Started  
🔥 **Start your journey today → [Visit DeBeatzGH]([https://debeatzgh.wordpress.com/](https://www.patreon.com/debeatzgh/collections))**  

---


<!-- README: DebeatzGH Digital Store (HTML-friendly for GitHub) -->
<div align="center">
  <a href="https://www.socialcreator.com/debeatzgh" target="_blank" rel="noopener">
    <img
      src="https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/designadigitalproductse-commerceonlinedeals3545265155247625100.jpg"
      alt="DebeatzGH Digital Store"
      style="max-width:100%; border-radius:16px;"
    />
  </a>

  <h1 style="margin-top: 14px;">DebeatzGH Digital Store</h1>
  <p style="max-width:780px;">
    Your hub for AI insights, tech tutorials, side-hustle playbooks, and productivity tools.
    Learn, build, and launch digital projects faster.
  </p>

  <!-- CTAs -->
  <p>
    <a href="https://www.socialcreator.com/debeatzgh" target="_blank" rel="noopener"
       style="display:inline-block; padding:10px 16px; margin:4px; border-radius:999px; text-decoration:none; font-weight:600; border:1px solid #2563eb;">
      🚀 View Live App
    </a>
    <a href="https://github.com/debeatzgh1/Personal-Portfolio-site-" target="_blank" rel="noopener"
       style="display:inline-block; padding:10px 16px; margin:4px; border-radius:999px; text-decoration:none; font-weight:600; border:1px solid #111827;">
      ⭐ Star this Repo
    </a>
  </p>
</div>

<hr/>

<h2>Overview</h2>
<p>
  <strong>DebeatzGH</strong> helps beginners and creators build profitable digital assets:
  blogs, affiliate funnels, AI-assisted content, and more. Explore tutorials, tools, and
  ready-to-use components to speed up your workflow.
</p>

<h2>Features</h2>
<ul>
  <li><strong>AI & Tech Learning:</strong> Bite-sized guides for modern tools and workflows.</li>
  <li><strong>Side-Hustle Playbooks:</strong> Practical steps to validate and launch ideas.</li>
  <li><strong>Productivity Toolkit:</strong> Reusable widgets, templates, and scripts.</li>
  <li><strong>Beginner-Friendly:</strong> Clear explanations, curated resources, and examples.</li>
</ul>

<h2>Quick Start</h2>
<ol>
  <li>Clone:
    <pre><code>git clone https://github.com/debeatzgh1/Personal-Portfolio-site-</code></pre>
  </li>
  <li>Enter folder:
    <pre><code>cd debeatzgh</code></pre>
  </li>
  <li>Install deps (adjust to your stack):
    <pre><code># Node
npm install
npm run dev

# or Python
pip install -r requirements.txt
python app.py</code></pre>
  </li>
  <li>Open in browser:
    <pre><code>http://localhost:3000</code></pre>
  </li>
</ol>

<h2>Project Links</h2>
<ul>
  <li>🌐 Live App: <a href="https://www.socialcreator.com/debeatzgh" target="_blank" rel="noopener">socialcreator.com/debeatzgh</a></li>
  <li>🖼️ Thumbnail: <a href="https://debeatzgh.wordpress.com/wp-content/uploads/2025/08/designadigitalproductse-commerceonlinedeals3545265155247625100.jpg" target="_blank" rel="noopener">View image</a></li>
</ul>

<h2>Contributing</h2>
<p>
  Contributions are welcome! Open an issue for bugs or ideas. For changes, fork the repo,
  create a feature branch, and submit a pull request.
</p>

<h2>License</h2>
<p>
  Released under the <a href="./LICENSE">MIT License</a>.
</p>

<hr/>

<div align="center">
  <p><em>If this project helps you, consider giving it a star. It really helps! ⭐</em></p>
  <p>
    <a href="https://www.socialcreator.com/debeatzgh/?s=314768" target="_blank" rel="noopener"
       style="display:inline-block; padding:10px 16px; margin-top:6px; border-radius:10px; text-decoration:none; font-weight:600; border:1px solid #2563eb;">
      PRODUCTS →
    </a>
  </p>
</div>
