import { useState, useEffect, useRef } from "react";

// ── Fonts via Google Fonts injected in head ──────────────────────────────────
const fontLink = document.createElement("link");
fontLink.rel = "stylesheet";
fontLink.href =
  "https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;1,400&family=DM+Sans:wght@300;400;500&display=swap";
document.head.appendChild(fontLink);

// ── Global styles ────────────────────────────────────────────────────────────
const globalStyle = document.createElement("style");
globalStyle.textContent = `
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --sage:      #7aab8a;
    --sage-light:#b6d8c2;
    --sage-pale: #e8f4ec;
    --warm:      #f5f0e8;
    --cream:     #fdfaf5;
    --clay:      #c8856a;
    --clay-light:#f0d5ca;
    --slate:     #4a5568;
    --dusk:      #2d3748;
    --mist:      #edf2f7;
    --gold:      #d4a94a;
    --lavender:  #9b8ec4;
    --sky:       #6baed6;
    --r: .75rem;
    --R: 1.25rem;
  }
  body { background: var(--cream); font-family: 'DM Sans', sans-serif; color: var(--dusk); }
  h1,h2,h3,h4 { font-family: 'Lora', serif; }
  button { cursor: pointer; border: none; font-family: 'DM Sans', sans-serif; }
  textarea, input { font-family: 'DM Sans', sans-serif; }
  ::-webkit-scrollbar { width: 6px; }
  ::-webkit-scrollbar-track { background: var(--mist); }
  ::-webkit-scrollbar-thumb { background: var(--sage-light); border-radius: 3px; }

  @keyframes fadeUp {
    from { opacity:0; transform:translateY(18px); }
    to   { opacity:1; transform:translateY(0); }
  }
  @keyframes pulse-ring {
    0%   { transform:scale(1);   opacity:.6; }
    50%  { transform:scale(1.15);opacity:.2; }
    100% { transform:scale(1);   opacity:.6; }
  }
  @keyframes breathe-in  { from{transform:scale(1)} to{transform:scale(1.45)} }
  @keyframes breathe-out { from{transform:scale(1.45)} to{transform:scale(1)} }
  @keyframes breathe-hold{ from{transform:scale(1.45)} to{transform:scale(1.45)} }

  .fade-up  { animation: fadeUp .5s ease both; }
  .card {
    background: #fff;
    border-radius: var(--R);
    padding: 1.5rem;
    box-shadow: 0 2px 16px rgba(74,85,104,.07);
    transition: box-shadow .2s;
  }
  .card:hover { box-shadow: 0 4px 24px rgba(74,85,104,.13); }
  .btn {
    padding: .65rem 1.4rem;
    border-radius: var(--r);
    font-size: .9rem;
    font-weight: 500;
    transition: transform .15s, box-shadow .15s, background .15s;
  }
  .btn:hover  { transform: translateY(-1px); box-shadow: 0 4px 12px rgba(0,0,0,.12); }
  .btn:active { transform: translateY(0); }
  .btn-sage  { background: var(--sage);  color:#fff; }
  .btn-clay  { background: var(--clay);  color:#fff; }
  .btn-ghost { background: transparent; color: var(--sage); border: 1.5px solid var(--sage); }
  .tag {
    display:inline-block; padding:.25rem .7rem;
    border-radius:99px; font-size:.75rem; font-weight:500;
  }
`;
document.head.appendChild(globalStyle);

// ── Data ─────────────────────────────────────────────────────────────────────
const MOODS = [
  { emoji: "😔", label: "Very Low",  value: 1, color: "#c0392b" },
  { emoji: "😟", label: "Low",       value: 2, color: "#e67e22" },
  { emoji: "😐", label: "Neutral",   value: 3, color: "#f1c40f" },
  { emoji: "🙂", label: "Good",      value: 4, color: "#27ae60" },
  { emoji: "😄", label: "Great",     value: 5, color: "#2980b9" },
];

const EXERCISES = [
  { id:"breath",  icon:"🌬️", title:"Box Breathing",      duration:"4 min", desc:"Calm your nervous system with rhythmic breath." },
  { id:"gratitude",icon:"🌿",title:"Gratitude Journal",  duration:"5 min", desc:"Notice three things you appreciate right now." },
  { id:"grounding",icon:"🌱",title:"5-4-3-2-1 Grounding",duration:"3 min", desc:"Anchor yourself to the present moment." },
  { id:"affirmation",icon:"✨",title:"Daily Affirmation", duration:"2 min", desc:"Strengthen your inner voice with kind words." },
];

const AFFIRMATIONS = [
  "I am worthy of care and kindness.",
  "My feelings are valid and temporary.",
  "I have the strength to face today.",
  "I choose peace over perfection.",
  "I am enough, exactly as I am.",
  "Small steps still move me forward.",
];

const HOTLINES = [
  { name:"Befrienders KL",       number:"03-7956 8145", color:"var(--sage)"    },
  { name:"Mental Health Hotline",number:"1800-82-0066", color:"var(--sky)"     },
  { name:"MIASA Helpline",       number:"03-2780 6803", color:"var(--lavender)"},
  { name:"Crisis Text (MY)",     number:"SMS: 15999",   color:"var(--clay)"    },
];

const TIPS = [
  "Take 3 slow, deep breaths when you feel overwhelmed.",
  "A 10-minute walk outside can shift your mood.",
  "Stay hydrated — dehydration affects your thinking.",
  "Reach out to one person today, even briefly.",
  "Rest is productive. You don't have to earn it.",
];

// ── Hooks ────────────────────────────────────────────────────────────────────
function useLocalStorage(key, init) {
  const [val, setVal] = useState(() => {
    try { const s = localStorage.getItem(key); return s ? JSON.parse(s) : init; }
    catch { return init; }
  });
  useEffect(() => { localStorage.setItem(key, JSON.stringify(val)); }, [key, val]);
  return [val, setVal];
}

// ══════════════════════════════════════════════════════════════════════════════
// SCREENS
// ══════════════════════════════════════════════════════════════════════════════

// ── 1. Dashboard ─────────────────────────────────────────────────────────────
function Dashboard({ moodLog, onNav }) {
  const today = new Date().toDateString();
  const todayMoods = moodLog.filter(m => new Date(m.date).toDateString() === today);
  const lastMood = moodLog[moodLog.length - 1];
  const streak = calcStreak(moodLog);
  const tip = TIPS[new Date().getDay() % TIPS.length];

  return (
    <div style={{ padding:"1.5rem", display:"flex", flexDirection:"column", gap:"1.2rem" }}>

      {/* Header */}
      <div className="fade-up" style={{ animationDelay:".05s" }}>
        <p style={{ color:"var(--sage)", fontWeight:500, fontSize:".85rem", letterSpacing:".08em", textTransform:"uppercase" }}>
          {new Date().toLocaleDateString("en-MY",{weekday:"long", day:"numeric", month:"long"})}
        </p>
        <h2 style={{ fontSize:"1.7rem", fontStyle:"italic", color:"var(--dusk)" }}>
          Hello, how are you today?
        </h2>
      </div>

      {/* Quick mood */}
      {!todayMoods.length && (
        <div className="card fade-up" style={{ animationDelay:".1s", background:"linear-gradient(135deg,var(--sage-pale),var(--cream))" }}>
          <p style={{ fontSize:".85rem", color:"var(--slate)", marginBottom:".8rem" }}>Start with a quick check-in ✦</p>
          <button className="btn btn-sage" onClick={() => onNav("tracker")}>Log my mood →</button>
        </div>
      )}

      {/* Stats row */}
      <div className="fade-up" style={{ animationDelay:".15s", display:"grid", gridTemplateColumns:"1fr 1fr 1fr", gap:".8rem" }}>
        {[
          { label:"Streak",    val:`${streak}d`,          icon:"🔥" },
          { label:"Entries",   val:moodLog.length,         icon:"📝" },
          { label:"Avg Mood",  val:avgMood(moodLog),       icon:"💚" },
        ].map(s => (
          <div key={s.label} className="card" style={{ textAlign:"center", padding:"1rem .8rem" }}>
            <div style={{ fontSize:"1.4rem" }}>{s.icon}</div>
            <div style={{ fontSize:"1.3rem", fontWeight:600, fontFamily:"Lora,serif", color:"var(--dusk)" }}>{s.val}</div>
            <div style={{ fontSize:".72rem", color:"var(--slate)", marginTop:".15rem" }}>{s.label}</div>
          </div>
        ))}
      </div>

      {/* Daily tip */}
      <div className="card fade-up" style={{ animationDelay:".2s", borderLeft:"4px solid var(--sage)", background:"var(--sage-pale)" }}>
        <p style={{ fontSize:".72rem", color:"var(--sage)", fontWeight:600, textTransform:"uppercase", letterSpacing:".07em", marginBottom:".4rem" }}>Tip of the day</p>
        <p style={{ fontSize:".92rem", color:"var(--dusk)", lineHeight:1.6 }}>{tip}</p>
      </div>

      {/* Last mood snapshot */}
      {lastMood && (
        <div className="card fade-up" style={{ animationDelay:".25s" }}>
          <p style={{ fontSize:".8rem", color:"var(--slate)", marginBottom:".6rem" }}>Last logged mood</p>
          <div style={{ display:"flex", alignItems:"center", gap:"1rem" }}>
            <span style={{ fontSize:"2.2rem" }}>{MOODS.find(m=>m.value===lastMood.mood)?.emoji}</span>
            <div>
              <p style={{ fontWeight:600 }}>{MOODS.find(m=>m.value===lastMood.mood)?.label}</p>
              <p style={{ fontSize:".78rem", color:"var(--slate)" }}>
                {new Date(lastMood.date).toLocaleString("en-MY",{hour:"2-digit",minute:"2-digit",month:"short",day:"numeric"})}
              </p>
              {lastMood.note && <p style={{ fontSize:".82rem", marginTop:".3rem", fontStyle:"italic", color:"var(--slate)" }}>"{lastMood.note.slice(0,60)}{lastMood.note.length>60?"…":""}"</p>}
            </div>
          </div>
        </div>
      )}

      {/* Quick nav */}
      <div className="fade-up" style={{ animationDelay:".3s", display:"grid", gridTemplateColumns:"1fr 1fr", gap:".8rem" }}>
        {[
          { label:"Breathing", icon:"🌬️", screen:"breathing" },
          { label:"Journal",   icon:"📓", screen:"journal"   },
          { label:"Exercises", icon:"🌱", screen:"exercises" },
          { label:"Resources", icon:"🆘", screen:"resources" },
        ].map(n => (
          <button key={n.screen} className="card" onClick={() => onNav(n.screen)}
            style={{ display:"flex", alignItems:"center", gap:".75rem", padding:"1rem", background:"#fff", textAlign:"left", border:"none", cursor:"pointer" }}>
            <span style={{ fontSize:"1.5rem" }}>{n.icon}</span>
            <span style={{ fontWeight:500, fontSize:".9rem" }}>{n.label}</span>
          </button>
        ))}
      </div>
    </div>
  );
}

// ── 2. Mood Tracker ───────────────────────────────────────────────────────────
function MoodTracker({ moodLog, onLog }) {
  const [selected, setSelected] = useState(null);
  const [note, setNote] = useState("");
  const [done, setDone] = useState(false);

  function submit() {
    if (!selected) return;
    onLog({ mood: selected, note, date: new Date().toISOString() });
    setDone(true);
  }

  if (done) return (
    <div style={{ padding:"2rem", textAlign:"center", display:"flex", flexDirection:"column", alignItems:"center", gap:"1rem" }}>
      <div style={{ fontSize:"3rem" }}>{MOODS.find(m=>m.value===selected)?.emoji}</div>
      <h3 style={{ fontStyle:"italic" }}>Logged! Thank you for checking in.</h3>
      <p style={{ color:"var(--slate)", fontSize:".9rem", maxWidth:"280px", lineHeight:1.6 }}>
        Noticing how you feel is the first step to caring for yourself.
      </p>
      <button className="btn btn-ghost" onClick={() => { setDone(false); setSelected(null); setNote(""); }}>Log another</button>
    </div>
  );

  // History chart (last 7)
  const last7 = moodLog.slice(-7);

  return (
    <div style={{ padding:"1.5rem", display:"flex", flexDirection:"column", gap:"1.4rem" }}>
      <div className="fade-up">
        <h2 style={{ fontStyle:"italic" }}>How are you feeling?</h2>
        <p style={{ color:"var(--slate)", fontSize:".85rem", marginTop:".3rem" }}>Select a mood and add a note if you'd like.</p>
      </div>

      {/* Mood buttons */}
      <div className="fade-up" style={{ animationDelay:".1s", display:"flex", justifyContent:"space-between" }}>
        {MOODS.map(m => (
          <button key={m.value} onClick={() => setSelected(m.value)}
            style={{
              display:"flex", flexDirection:"column", alignItems:"center", gap:".35rem",
              padding:".75rem .5rem", borderRadius:"var(--R)", border:"2.5px solid",
              borderColor: selected===m.value ? m.color : "transparent",
              background: selected===m.value ? m.color+"18" : "var(--mist)",
              transition:"all .2s", flex:1, margin:"0 .2rem",
              transform: selected===m.value ? "translateY(-4px)" : "none",
            }}>
            <span style={{ fontSize:"1.8rem" }}>{m.emoji}</span>
            <span style={{ fontSize:".65rem", color:"var(--slate)", fontWeight:500 }}>{m.label}</span>
          </button>
        ))}
      </div>

      {/* Note */}
      <div className="fade-up" style={{ animationDelay:".15s" }}>
        <label style={{ fontSize:".82rem", color:"var(--slate)", display:"block", marginBottom:".4rem" }}>Add a note (optional)</label>
        <textarea value={note} onChange={e=>setNote(e.target.value)} rows={3}
          placeholder="What's on your mind today…"
          style={{
            width:"100%", padding:".85rem", borderRadius:"var(--r)",
            border:"1.5px solid var(--mist)", background:"var(--cream)",
            fontSize:".9rem", color:"var(--dusk)", resize:"none", lineHeight:1.6,
            outline:"none", transition:"border .2s",
          }}
          onFocus={e=>e.target.style.borderColor="var(--sage)"}
          onBlur={e=>e.target.style.borderColor="var(--mist)"}
        />
      </div>

      <button className="btn btn-sage fade-up" style={{ animationDelay:".2s", opacity: selected?1:.5 }} onClick={submit}>
        Save check-in ✦
      </button>

      {/* 7-day history */}
      {last7.length > 0 && (
        <div className="card fade-up" style={{ animationDelay:".25s" }}>
          <p style={{ fontSize:".8rem", color:"var(--slate)", marginBottom:".8rem", fontWeight:500 }}>Last {last7.length} entries</p>
          <div style={{ display:"flex", alignItems:"flex-end", gap:".5rem", height:"80px" }}>
            {last7.map((entry, i) => {
              const mood = MOODS.find(m=>m.value===entry.mood);
              const h = (entry.mood / 5) * 64 + 10;
              return (
                <div key={i} style={{ flex:1, display:"flex", flexDirection:"column", alignItems:"center", gap:".3rem" }}>
                  <div style={{ width:"100%", height:`${h}px`, borderRadius:"4px 4px 0 0", background: mood?.color+"88", transition:"height .5s" }}/>
                  <span style={{ fontSize:".9rem" }}>{mood?.emoji}</span>
                </div>
              );
            })}
          </div>
        </div>
      )}
    </div>
  );
}

// ── 3. Breathing Exercise ─────────────────────────────────────────────────────
function BreathingExercise() {
  const phases = [
    { label:"Breathe In",  duration:4, anim:"breathe-in"  },
    { label:"Hold",        duration:4, anim:"breathe-hold" },
    { label:"Breathe Out", duration:4, anim:"breathe-out"  },
    { label:"Hold",        duration:4, anim:"breathe-hold" },
  ];
  const [active, setActive] = useState(false);
  const [phaseIdx, setPhaseIdx] = useState(0);
  const [count, setCount] = useState(phases[0].duration);
  const [cycles, setCycles] = useState(0);
  const timerRef = useRef(null);

  useEffect(() => {
    if (!active) return;
    timerRef.current = setInterval(() => {
      setCount(c => {
        if (c > 1) return c - 1;
        setPhaseIdx(p => {
          const next = (p + 1) % phases.length;
          if (next === 0) setCycles(cy => cy + 1);
          setCount(phases[next].duration);
          return next;
        });
        return phases[phaseIdx].duration;
      });
    }, 1000);
    return () => clearInterval(timerRef.current);
  }, [active, phaseIdx]);

  function toggle() {
    if (active) { clearInterval(timerRef.current); setActive(false); setPhaseIdx(0); setCount(phases[0].duration); setCycles(0); }
    else { setActive(true); }
  }

  const phase = phases[phaseIdx];
  const colors = ["#7aab8a","#9b8ec4","#6baed6","#9b8ec4"];

  return (
    <div style={{ padding:"1.5rem", display:"flex", flexDirection:"column", alignItems:"center", gap:"1.5rem" }}>
      <div className="fade-up" style={{ textAlign:"center" }}>
        <h2 style={{ fontStyle:"italic" }}>Box Breathing</h2>
        <p style={{ color:"var(--slate)", fontSize:".85rem", marginTop:".3rem" }}>4-4-4-4 rhythm to calm your nervous system</p>
      </div>

      {/* Animated circle */}
      <div className="fade-up" style={{ animationDelay:".1s", position:"relative", width:"200px", height:"200px", display:"flex", alignItems:"center", justifyContent:"center" }}>
        {/* outer ring */}
        <div style={{
          position:"absolute", width:"100%", height:"100%", borderRadius:"50%",
          background: colors[phaseIdx]+"22",
          animation: active ? `pulse-ring 4s ease infinite` : "none",
        }}/>
        {/* breathing circle */}
        <div style={{
          width:"120px", height:"120px", borderRadius:"50%",
          background: `radial-gradient(circle at 40% 40%, ${colors[phaseIdx]}cc, ${colors[phaseIdx]})`,
          display:"flex", flexDirection:"column", alignItems:"center", justifyContent:"center",
          animation: active ? `${phase.anim} ${phase.duration}s ease forwards` : "none",
          boxShadow:`0 0 40px ${colors[phaseIdx]}55`,
          transition:"background .5s",
        }}>
          <span style={{ color:"#fff", fontSize:"2rem", fontWeight:700, fontFamily:"Lora,serif" }}>{count}</span>
        </div>
      </div>

      <div className="fade-up" style={{ animationDelay:".15s", textAlign:"center" }}>
        <p style={{ fontSize:"1.2rem", fontFamily:"Lora,serif", fontStyle:"italic", color: colors[phaseIdx] }}>{phase.label}</p>
        {cycles > 0 && <p style={{ fontSize:".8rem", color:"var(--slate)", marginTop:".3rem" }}>Cycles completed: {cycles}</p>}
      </div>

      <button className="btn fade-up" style={{ animationDelay:".2s", background: active?"var(--clay)":"var(--sage)", color:"#fff", padding:".8rem 2.5rem", fontSize:"1rem" }} onClick={toggle}>
        {active ? "Stop" : "Begin"}
      </button>

      {/* Phase guide */}
      <div className="card fade-up" style={{ animationDelay:".25s", width:"100%", display:"grid", gridTemplateColumns:"1fr 1fr 1fr 1fr", gap:".5rem", padding:"1rem" }}>
        {phases.map((p,i) => (
          <div key={i} style={{ textAlign:"center", padding:".5rem .3rem", borderRadius:"var(--r)", background: active&&phaseIdx===i ? colors[i]+"22" : "var(--mist)", transition:"background .3s" }}>
            <div style={{ fontSize:".9rem", fontWeight:600, color: colors[i] }}>{p.duration}s</div>
            <div style={{ fontSize:".68rem", color:"var(--slate)" }}>{p.label}</div>
          </div>
        ))}
      </div>
    </div>
  );
}

// ── 4. Journal ────────────────────────────────────────────────────────────────
const PROMPTS = [
  "What's weighing on your mind today?",
  "Describe one small thing that brought you comfort.",
  "What do you need to hear right now?",
  "What would you tell a friend feeling the way you do?",
  "Write about a moment of calm this week.",
];

function Journal() {
  const [entries, setEntries] = useLocalStorage("mcs_journal", []);
  const [text, setText] = useState("");
  const [prompt] = useState(PROMPTS[new Date().getDay() % PROMPTS.length]);
  const [view, setView] = useState("write");

  function save() {
    if (!text.trim()) return;
    setEntries(e => [{ id:Date.now(), text, date:new Date().toISOString() }, ...e]);
    setText("");
  }

  return (
    <div style={{ padding:"1.5rem", display:"flex", flexDirection:"column", gap:"1.2rem" }}>
      <div className="fade-up">
        <h2 style={{ fontStyle:"italic" }}>Journal</h2>
        <div style={{ display:"flex", gap:".5rem", marginTop:".8rem" }}>
          {["write","entries"].map(t => (
            <button key={t} className="btn" onClick={() => setView(t)}
              style={{ background: view===t ? "var(--sage)" : "var(--mist)", color: view===t?"#fff":"var(--slate)", padding:".5rem 1rem", fontSize:".82rem" }}>
              {t === "write" ? "✏️ Write" : `📖 Entries (${entries.length})`}
            </button>
          ))}
        </div>
      </div>

      {view === "write" && (
        <>
          <div className="card fade-up" style={{ animationDelay:".1s", background:"var(--sage-pale)", borderLeft:"4px solid var(--sage)" }}>
            <p style={{ fontSize:".75rem", color:"var(--sage)", fontWeight:600, marginBottom:".3rem" }}>Today's prompt</p>
            <p style={{ fontStyle:"italic", color:"var(--dusk)", fontSize:".92rem", lineHeight:1.6 }}>{prompt}</p>
          </div>
          <div className="fade-up" style={{ animationDelay:".15s" }}>
            <textarea value={text} onChange={e=>setText(e.target.value)} rows={8}
              placeholder="Begin writing here… this space is just for you."
              style={{ width:"100%", padding:"1rem", borderRadius:"var(--r)", border:"1.5px solid var(--mist)", background:"var(--cream)", fontSize:".92rem", lineHeight:1.8, resize:"none", outline:"none", color:"var(--dusk)" }}
              onFocus={e=>e.target.style.borderColor="var(--sage)"}
              onBlur={e=>e.target.style.borderColor="var(--mist)"}
            />
            <p style={{ fontSize:".75rem", color:"var(--slate)", textAlign:"right", marginTop:".3rem" }}>{text.length} chars</p>
          </div>
          <button className="btn btn-sage fade-up" style={{ animationDelay:".2s", opacity:text.trim()?1:.5 }} onClick={save}>Save entry ✦</button>
        </>
      )}

      {view === "entries" && (
        <div className="fade-up" style={{ animationDelay:".1s", display:"flex", flexDirection:"column", gap:".8rem" }}>
          {entries.length === 0 && <p style={{ color:"var(--slate)", textAlign:"center", padding:"2rem", fontStyle:"italic" }}>No entries yet. Start writing ✦</p>}
          {entries.map(e => (
            <div key={e.id} className="card" style={{ borderLeft:"3px solid var(--sage-light)" }}>
              <p style={{ fontSize:".75rem", color:"var(--slate)", marginBottom:".5rem" }}>
                {new Date(e.date).toLocaleString("en-MY",{weekday:"short",day:"numeric",month:"short",hour:"2-digit",minute:"2-digit"})}
              </p>
              <p style={{ fontSize:".9rem", lineHeight:1.7, color:"var(--dusk)", whiteSpace:"pre-wrap" }}>{e.text}</p>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

// ── 5. Exercises ──────────────────────────────────────────────────────────────
function Exercises({ onNav }) {
  const [active, setActive] = useState(null);
  const affIdx = new Date().getDate() % AFFIRMATIONS.length;

  if (active === "grounding") return <Grounding onBack={() => setActive(null)} />;

  return (
    <div style={{ padding:"1.5rem", display:"flex", flexDirection:"column", gap:"1.2rem" }}>
      <div className="fade-up">
        <h2 style={{ fontStyle:"italic" }}>Wellness Exercises</h2>
        <p style={{ color:"var(--slate)", fontSize:".85rem", marginTop:".3rem" }}>Small practices for a calmer mind.</p>
      </div>

      {/* Affirmation of the day */}
      <div className="card fade-up" style={{ animationDelay:".1s", background:"linear-gradient(135deg,#e8f4ec,#f5f0e8)", textAlign:"center", padding:"2rem 1.5rem" }}>
        <p style={{ fontSize:".75rem", color:"var(--sage)", fontWeight:600, textTransform:"uppercase", letterSpacing:".08em", marginBottom:".8rem" }}>✨ Affirmation of the day</p>
        <p style={{ fontFamily:"Lora,serif", fontStyle:"italic", fontSize:"1.1rem", lineHeight:1.7, color:"var(--dusk)" }}>
          "{AFFIRMATIONS[affIdx]}"
        </p>
      </div>

      {/* Exercise cards */}
      {EXERCISES.map((ex, i) => (
        <button key={ex.id} className="card fade-up" onClick={() => ex.id==="breathing" ? onNav("breathing") : setActive(ex.id)}
          style={{ animationDelay:`${.15 + i*.05}s`, textAlign:"left", width:"100%", display:"flex", alignItems:"center", gap:"1rem", cursor:"pointer" }}>
          <span style={{ fontSize:"2rem", background:"var(--sage-pale)", width:"52px", height:"52px", borderRadius:"50%", display:"flex", alignItems:"center", justifyContent:"center", flexShrink:0 }}>{ex.icon}</span>
          <div style={{ flex:1 }}>
            <div style={{ fontWeight:600, fontSize:".95rem" }}>{ex.title}</div>
            <div style={{ fontSize:".8rem", color:"var(--slate)", marginTop:".2rem" }}>{ex.desc}</div>
          </div>
          <span style={{ fontSize:".75rem", color:"var(--sage)", whiteSpace:"nowrap", background:"var(--sage-pale)", padding:".3rem .6rem", borderRadius:"99px" }}>{ex.duration}</span>
        </button>
      ))}

      {/* Gratitude inline */}
      {active === "gratitude" && <GratitudeExercise onBack={() => setActive(null)} />}
      {active === "affirmation" && <AffirmationExercise onBack={() => setActive(null)} />}
    </div>
  );
}

function GratitudeExercise({ onBack }) {
  const [items, setItems] = useState(["","",""]);
  const [saved, setSaved] = useState(false);
  return (
    <div className="card" style={{ background:"var(--sage-pale)", border:"2px solid var(--sage-light)" }}>
      <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:"1rem" }}>
        <h4 style={{ fontStyle:"italic" }}>🌿 Gratitude Practice</h4>
        <button onClick={onBack} style={{ background:"none", color:"var(--slate)", fontSize:"1.2rem" }}>✕</button>
      </div>
      <p style={{ fontSize:".85rem", color:"var(--slate)", marginBottom:"1rem" }}>Name three things you're grateful for today:</p>
      {items.map((v,i) => (
        <input key={i} value={v} onChange={e => { const a=[...items]; a[i]=e.target.value; setItems(a); }}
          placeholder={`Gratitude ${i+1}…`}
          style={{ display:"block", width:"100%", padding:".65rem .9rem", marginBottom:".6rem", borderRadius:"var(--r)", border:"1.5px solid var(--sage-light)", background:"#fff", fontSize:".9rem", color:"var(--dusk)", outline:"none" }}
        />
      ))}
      {!saved
        ? <button className="btn btn-sage" onClick={() => setSaved(true)} style={{ marginTop:".4rem" }}>Save ✦</button>
        : <p style={{ color:"var(--sage)", fontStyle:"italic", textAlign:"center", padding:".5rem" }}>Beautiful. 🌿 Carry these with you today.</p>
      }
    </div>
  );
}

function AffirmationExercise({ onBack }) {
  const [idx, setIdx] = useState(0);
  return (
    <div className="card" style={{ background:"linear-gradient(135deg,#e8f4ec,#fdfaf5)", border:"2px solid var(--sage-light)", textAlign:"center" }}>
      <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:"1rem" }}>
        <h4 style={{ fontStyle:"italic" }}>✨ Affirmations</h4>
        <button onClick={onBack} style={{ background:"none", color:"var(--slate)", fontSize:"1.2rem" }}>✕</button>
      </div>
      <p style={{ fontFamily:"Lora,serif", fontStyle:"italic", fontSize:"1.15rem", lineHeight:1.7, padding:"1rem", color:"var(--dusk)" }}>
        "{AFFIRMATIONS[idx]}"
      </p>
      <button className="btn btn-ghost" onClick={() => setIdx((idx+1)%AFFIRMATIONS.length)} style={{ marginTop:".5rem" }}>Next →</button>
    </div>
  );
}

function Grounding({ onBack }) {
  const steps = [
    { num:5, sense:"See",   prompt:"Name 5 things you can see right now", color:"var(--sage)" },
    { num:4, sense:"Touch", prompt:"Notice 4 things you can touch or feel", color:"var(--sky)" },
    { num:3, sense:"Hear",  prompt:"Listen for 3 sounds around you", color:"var(--lavender)" },
    { num:2, sense:"Smell", prompt:"Find 2 things you can smell", color:"var(--gold)" },
    { num:1, sense:"Taste", prompt:"Notice 1 thing you can taste", color:"var(--clay)" },
  ];
  const [step, setStep] = useState(0);
  const [done, setDone] = useState(false);
  const s = steps[step];
  if (done) return (
    <div style={{ padding:"2rem", textAlign:"center", display:"flex", flexDirection:"column", gap:"1rem", alignItems:"center" }}>
      <span style={{ fontSize:"3rem" }}>🌱</span>
      <h3 style={{ fontStyle:"italic" }}>You're grounded.</h3>
      <p style={{ color:"var(--slate)", fontSize:".9rem", maxWidth:"260px", lineHeight:1.6 }}>Well done. Take a moment to notice how you feel now.</p>
      <button className="btn btn-ghost" onClick={onBack}>Back to Exercises</button>
    </div>
  );
  return (
    <div style={{ padding:"1.5rem", display:"flex", flexDirection:"column", gap:"1.5rem", alignItems:"center" }}>
      <button onClick={onBack} style={{ alignSelf:"flex-start", background:"none", color:"var(--slate)", fontSize:".85rem", display:"flex", alignItems:"center", gap:".3rem" }}>← Back</button>
      <div style={{ width:"100px", height:"100px", borderRadius:"50%", background:s.color+"22", border:`3px solid ${s.color}`, display:"flex", alignItems:"center", justifyContent:"center" }}>
        <span style={{ fontSize:"2.5rem", fontFamily:"Lora,serif", fontWeight:700, color:s.color }}>{s.num}</span>
      </div>
      <div style={{ textAlign:"center" }}>
        <p style={{ color:s.color, fontWeight:600, fontSize:"1.1rem" }}>{s.sense}</p>
        <p style={{ color:"var(--slate)", fontSize:".92rem", marginTop:".4rem", lineHeight:1.6 }}>{s.prompt}</p>
      </div>
      <div style={{ display:"flex", gap:".5rem" }}>
        {steps.map((_,i) => <div key={i} style={{ width:"10px", height:"10px", borderRadius:"50%", background: i<=step ? s.color : "var(--mist)", transition:"background .3s" }}/>)}
      </div>
      <button className="btn btn-sage" style={{ padding:".8rem 2rem" }} onClick={() => step < steps.length-1 ? setStep(step+1) : setDone(true)}>
        {step < steps.length-1 ? "Next sense →" : "Finish ✦"}
      </button>
    </div>
  );
}

// ── 6. Resources ──────────────────────────────────────────────────────────────
function Resources() {
  return (
    <div style={{ padding:"1.5rem", display:"flex", flexDirection:"column", gap:"1.2rem" }}>
      <div className="fade-up">
        <h2 style={{ fontStyle:"italic" }}>Crisis & Resources</h2>
        <p style={{ color:"var(--slate)", fontSize:".85rem", marginTop:".3rem" }}>You are not alone. Help is a call away.</p>
      </div>

      <div className="card fade-up" style={{ animationDelay:".1s", background:"#fff5f5", border:"2px solid #feb2b2", padding:"1rem 1.2rem" }}>
        <p style={{ color:"#c53030", fontWeight:600, fontSize:".85rem" }}>🆘 If you are in immediate danger, call <strong>999</strong> or go to your nearest Emergency Department.</p>
      </div>

      <div className="fade-up" style={{ animationDelay:".15s", display:"flex", flexDirection:"column", gap:".8rem" }}>
        {HOTLINES.map(h => (
          <div key={h.name} className="card" style={{ display:"flex", alignItems:"center", gap:"1rem", borderLeft:`4px solid ${h.color}` }}>
            <div style={{ flex:1 }}>
              <p style={{ fontWeight:600, fontSize:".92rem" }}>{h.name}</p>
              <p style={{ color:"var(--slate)", fontSize:".82rem", marginTop:".2rem" }}>{h.number}</p>
            </div>
            <a href={`tel:${h.number.replace(/[^0-9]/g,"")}`}
              style={{ background:h.color, color:"#fff", padding:".5rem 1rem", borderRadius:"var(--r)", fontSize:".82rem", textDecoration:"none", fontWeight:500 }}>
              Call
            </a>
          </div>
        ))}
      </div>

      <div className="card fade-up" style={{ animationDelay:".3s" }}>
        <h4 style={{ marginBottom:".8rem", fontStyle:"italic" }}>Self-care reminders</h4>
        <ul style={{ listStyle:"none", display:"flex", flexDirection:"column", gap:".6rem" }}>
          {["Drink a glass of water","Step outside for 5 minutes","Reach out to someone you trust","Rest without guilt","You don't have to figure everything out today"].map(r => (
            <li key={r} style={{ display:"flex", gap:".6rem", alignItems:"flex-start", fontSize:".88rem", color:"var(--slate)", lineHeight:1.5 }}>
              <span style={{ color:"var(--sage)", marginTop:".1rem" }}>✦</span>{r}
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}

// ── Helpers ───────────────────────────────────────────────────────────────────
function avgMood(log) {
  if (!log.length) return "—";
  const avg = log.reduce((s,m) => s + m.mood, 0) / log.length;
  return MOODS.find(m => m.value === Math.round(avg))?.emoji || "—";
}
function calcStreak(log) {
  if (!log.length) return 0;
  const days = [...new Set(log.map(m => new Date(m.date).toDateString()))].reverse();
  let streak = 0;
  let d = new Date();
  for (const day of days) {
    if (new Date(day).toDateString() === d.toDateString()) {
      streak++;
      d.setDate(d.getDate() - 1);
    } else break;
  }
  return streak;
}

// ── Nav bar ───────────────────────────────────────────────────────────────────
const NAV = [
  { id:"dashboard", icon:"🏠", label:"Home"      },
  { id:"tracker",   icon:"💚", label:"Mood"      },
  { id:"breathing", icon:"🌬️", label:"Breathe"  },
  { id:"exercises", icon:"🌱", label:"Exercises" },
  { id:"journal",   icon:"📓", label:"Journal"   },
  { id:"resources", icon:"🆘", label:"Help"      },
];

// ── App ───────────────────────────────────────────────────────────────────────
export default function App() {
  const [screen, setScreen] = useState("dashboard");
  const [moodLog, setMoodLog] = useLocalStorage("mcs_moods", []);

  function logMood(entry) { setMoodLog(l => [...l, entry]); }

  const screens = {
    dashboard: <Dashboard moodLog={moodLog} onNav={setScreen} />,
    tracker:   <MoodTracker moodLog={moodLog} onLog={logMood} />,
    breathing: <BreathingExercise />,
    exercises: <Exercises onNav={setScreen} />,
    journal:   <Journal />,
    resources: <Resources />,
  };

  return (
    <div style={{ maxWidth:"430px", margin:"0 auto", minHeight:"100vh", display:"flex", flexDirection:"column", background:"var(--cream)", position:"relative" }}>

      {/* Top bar */}
      <div style={{ padding:"1rem 1.5rem .8rem", borderBottom:"1px solid var(--mist)", background:"var(--cream)", position:"sticky", top:0, zIndex:10, display:"flex", alignItems:"center", justifyContent:"space-between" }}>
        <div style={{ display:"flex", alignItems:"center", gap:".6rem" }}>
          <span style={{ fontSize:"1.3rem" }}>🌿</span>
          <span style={{ fontFamily:"Lora,serif", fontWeight:600, fontSize:"1.05rem", color:"var(--dusk)" }}>MindSpace</span>
        </div>
        <span style={{ fontSize:".75rem", color:"var(--sage)", background:"var(--sage-pale)", padding:".25rem .7rem", borderRadius:"99px" }}>
          {NAV.find(n=>n.id===screen)?.label}
        </span>
      </div>

      {/* Screen */}
      <div style={{ flex:1, overflowY:"auto", paddingBottom:"5rem" }}>
        {screens[screen]}
      </div>

      {/* Bottom nav */}
      <nav style={{ position:"fixed", bottom:0, left:"50%", transform:"translateX(-50%)", width:"100%", maxWidth:"430px", background:"rgba(253,250,245,.96)", backdropFilter:"blur(12px)", borderTop:"1px solid var(--mist)", display:"flex", zIndex:20 }}>
        {NAV.map(n => (
          <button key={n.id} onClick={() => setScreen(n.id)}
            style={{
              flex:1, padding:".6rem .2rem .5rem", background:"none", display:"flex", flexDirection:"column", alignItems:"center", gap:".18rem",
              borderTop: screen===n.id ? "2.5px solid var(--sage)" : "2.5px solid transparent",
              transition:"all .2s",
            }}>
            <span style={{ fontSize:"1.2rem", filter: screen===n.id ? "none" : "grayscale(.4) opacity(.7)" }}>{n.icon}</span>
            <span style={{ fontSize:".6rem", color: screen===n.id ? "var(--sage)" : "var(--slate)", fontWeight: screen===n.id ? 600 : 400 }}>{n.label}</span>
          </button>
        ))}
      </nav>
    </div>
  );
}
