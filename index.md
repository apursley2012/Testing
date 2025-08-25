---
layout: default
title: Home
---

<!-- ==============================
     REVIEWS & INSTRUCTOR FEEDBACK
     Cards + 5-star outlines + swipe
     (Hacker theme compatible)
================================= -->
<section id="reviews" class="reviews-carousel" aria-labelledby="reviews-title">
  <h2 id="reviews-title">Reviews &amp; Instructor Feedback</h2>
  <p class="rc-intro">Highlights from course feedback. Stars are based on letter grade (A=★★★★★, B=★★★★, C=★★★, D=★★, F=★).</p>

  <style>
    .reviews-carousel{
      --accent:#39ff14;
      --chip-bg: rgba(57,255,20,.08);
      margin:1.75rem 0 2.25rem;
    }
    .reviews-carousel h2{ margin-bottom:.35rem; }
    .rc-intro{ opacity:.9; margin:.25rem 0 1rem; }

    .rc-shell{
      display:grid;
      grid-template-columns:auto 1fr auto;
      align-items:center;
      gap:.5rem;
    }
    .rc-nav{
      border:1px solid var(--accent);
      background:var(--chip-bg);
      color:var(--accent);
      border-radius:999px;
      padding:.35rem .7rem;
      cursor:pointer;
      user-select:none;
    }
    .rc-nav:disabled{ opacity:.4; cursor:not-allowed; }

    .rc-viewport{
      overflow:hidden;
      border-radius:12px;
    }
    .rc-track{
      display:flex;
      transition:transform .35s ease;
      will-change:transform;
    }

    .rc-card{
      min-width:100%;
      box-sizing:border-box;
      border:1px solid var(--accent);
      border-left-width:6px;
      border-radius:12px;
      background:transparent;
      padding:1rem;
    }
    .rc-top{
      display:flex; align-items:flex-start; justify-content:space-between;
      gap:.75rem; flex-wrap:wrap;
      margin-bottom:.5rem;
    }
    .rc-course{
      color:var(--accent);
      font-weight:700;
      margin-bottom:.15rem;
    }
    .rc-meta{ opacity:.85; font-size:.95rem; }
    blockquote.rc-quote{
      margin:.35rem 0 .6rem; padding:0;
      font-style:italic; line-height:1.5;
    }
    .rc-tags{ display:flex; flex-wrap:wrap; gap:.5rem; }
    .rc-tag{
      border:1px solid var(--accent);
      background:var(--chip-bg);
      color:var(--accent);
      padding:.2rem .55rem;
      border-radius:999px;
      font-size:.85rem;
      white-space:nowrap;
    }

    .rc-stars{ display:flex; gap:.15rem; align-items:center; }
    .rc-star{ width:20px; height:20px; display:inline-block; }
    .rc-star svg{
      width:100%; height:100%;
      stroke:var(--accent); stroke-width:1.3;
      fill:transparent; opacity:.55;
    }
    .rc-star.filled svg{ fill:var(--accent); opacity:1; }

    .rc-dots{
      display:flex; gap:.4rem; justify-content:center;
      margin-top:.6rem;
    }
    .rc-dot{
      width:8px; height:8px; border-radius:50%;
      border:1px solid var(--accent);
      background:transparent; cursor:pointer;
    }
    .rc-dot.active{ background:var(--accent); }

    @media (max-width:420px){
      .rc-card{ padding:.85rem; }
    }
  </style>

  <div class="rc-shell" role="region" aria-label="Reviews carousel">
    <button class="rc-nav rc-prev" aria-label="Previous review">◂</button>
    <div class="rc-viewport">
      <div class="rc-track" id="rc-track"><!-- cards injected by JS --></div>
    </div>
    <button class="rc-nav rc-next" aria-label="Next review">▸</button>
  </div>
  <div class="rc-dots" id="rc-dots" role="tablist" aria-label="Slides"></div>

  <template id="rc-card-tpl">
    <article class="rc-card">
      <div class="rc-top">
        <div>
          <div class="rc-course"></div>
          <div class="rc-meta"></div>
        </div>
        <div class="rc-stars" aria-label="Rating out of 5"></div>
      </div>
      <blockquote class="rc-quote"></blockquote>
      <div class="rc-tags"></div>
    </article>
  </template>

  <script>
    const REVIEWS = [
      {
        course: "CS370: Current/Emerging Trends in Computer Science",
        reviewer: "Dr. Hawk",
        grade: "A",
        quote: "You provided a concise explanation of how neural networks operate, effectively outlining the roles of the input, hidden, and output layers. This was particularly strong in simplifying complex concepts, making them accessible to a non-technical audience. Your identification of legal concerns was on point and relevant.",
        tags: ["Clarity", "AI", "Neural Networks", "Accessible Language"]
      },
      {
        course: "CS465: Full Stack Development I",
        reviewer: "Instructor Benyam Heyi",
        grade: "B",
        quote: "Great work on outlining all the requirements of the rubric—your structure and coverage are well thought out. For future submissions, please make sure to also complete the API endpoints table. Including this information will significantly enhance the quality of your Software Design Document (SDD).",
        tags: ["API", "SDD", "Technical Completeness", "Communication"]
      },
      {
        course: "CS499: Computer Science Capstone",
        reviewer: "Instructor Ramsey Kraya",
        grade: "A",
        quote: "Your Milestone Four submission is an excellent example of a well-executed database integration. You successfully transitioned your Corner Grocer application from in-memory data handling to a persistent SQLite backend, while maintaining original functionality and improving user experience.",
        tags: ["Database Integration", "SQLite", "Persistence", "User Experience"]
      }
    ];

    function gradeToRating(grade){
      const L = String(grade || '').trim().toUpperCase().charAt(0);
      if (L === 'A') return 5;
      if (L === 'B') return 4;
      if (L === 'C') return 3;
      if (L === 'D') return 2;
      return 1;
    }

    function starSVG(filled){
      return `
        <span class="rc-star ${filled ? 'filled' : ''}" aria-hidden="true">
          <svg viewBox="0 0 24 24" focusable="false">
            <path d="M12 17.27l-5.18 3.03 1.39-5.95L3 9.74l6.09-.52L12 3.5l2.91 5.72 6.09.52-5.21 4.61 1.39 5.95z"/>
          </svg>
        </span>`;
    }
    function renderStars(rating){
      let out = '';
      for (let i=1;i<=5;i++) out += starSVG(i <= rating);
      return out;
    }

    const track = document.getElementById('rc-track');
    const dots  = document.getElementById('rc-dots');
    const tpl   = document.getElementById('rc-card-tpl');

    function makeCard(item){
      const node = tpl.content.firstElementChild.cloneNode(true);
      node.querySelector('.rc-course').textContent = item.course;
      node.querySelector('.rc-meta').textContent   = `${item.reviewer} • Grade: ${item.grade}`;
      node.querySelector('.rc-quote').textContent  = `“${item.quote}”`;
      node.querySelector('.rc-stars').innerHTML   = renderStars(gradeToRating(item.grade));
      const tags = node.querySelector('.rc-tags');
      (item.tags || []).forEach(t => {
        const span = document.createElement('span');
        span.className = 'rc-tag';
        span.textContent = t;
        tags.appendChild(span);
      });
      return node;
    }

    function renderAll(){
      track.innerHTML = '';
      REVIEWS.forEach(r => track.appendChild(makeCard(r)));
      dots.innerHTML = REVIEWS.map((_,i)=>`<button class="rc-dot" data-i="${i}" aria-label="Slide ${i+1}"></button>`).join('');
      updateUI();
      dots.querySelectorAll('.rc-dot').forEach(b => b.addEventListener('click', () => goTo(+b.dataset.i)));
    }

    let index = 0;
    const prevBtn = document.querySelector('.rc-prev');
    const nextBtn = document.querySelector('.rc-next');

    function goTo(i){
      const max = REVIEWS.length - 1;
      index = Math.max(0, Math.min(max, i));
      const vw = document.querySelector('.rc-viewport').clientWidth;
      track.style.transform = `translateX(${-index * vw}px)`;
      updateUI();
    }
    function updateUI(){
      const max = REVIEWS.length - 1;
      prevBtn.disabled = index === 0;
      nextBtn.disabled = index === max;
      dots.querySelectorAll('.rc-dot').forEach((d, i) => {
        d.classList.toggle('active', i === index);
      });
    }

    prevBtn.addEventListener('click', () => goTo(index - 1));
    nextBtn.addEventListener('click', () => goTo(index + 1));
    window.addEventListener('resize', () => goTo(index));

    let startX = null, deltaX = 0;
    const viewport = document.querySelector('.rc-viewport');
    viewport.addEventListener('touchstart', (e) => { startX = e.touches[0].clientX; deltaX = 0; }, {passive:true});
    viewport.addEventListener('touchmove',  (e) => { if(startX!==null) deltaX = e.touches[0].clientX - startX; }, {passive:true});
    viewport.addEventListener('touchend',   () => {
      if (startX !== null){
        if (deltaX < -40) goTo(index + 1);
        if (deltaX >  40) goTo(index - 1);
      }
      startX = null; deltaX = 0;
    });

    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft')  goTo(index - 1);
      if (e.key === 'ArrowRight') goTo(index + 1);
    });

    document.addEventListener('DOMContentLoaded', renderAll);
  </script>
</section>
