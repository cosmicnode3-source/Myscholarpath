import { useState, useEffect, useRef } from "react";

const SCHOLARSHIPS = [
  { id: 1, name: "GKS (KGSP)", country: "South Korea", deadline: "2026-09-30", status: "in-progress", tag: "Fully Funded", color: "#FF6B35", gpa: 3.0, ielts: 5.5, docs: ["SOP", "LOR x2", "Transcript", "Passport"] },
  { id: 2, name: "DAAD", country: "Germany", deadline: "2026-10-15", status: "not-started", tag: "Research", color: "#4ECDC4", gpa: 3.2, ielts: 6.0, docs: ["CV", "SOP", "LOR x2", "Language Certificate"] },
  { id: 3, name: "Erasmus Mundus", country: "Europe", deadline: "2026-11-10", status: "not-started", tag: "Masters", color: "#45B7D1", gpa: 3.0, ielts: 6.5, docs: ["SOP", "LOR x3", "Transcript", "CV"] },
  { id: 4, name: "MEXT", country: "Japan", deadline: "2026-06-20", status: "submitted", tag: "Govt.", color: "#F7DC6F", gpa: 2.8, ielts: 0, docs: ["Research Plan", "LOR", "Transcript"] },
  { id: 5, name: "Chevening", country: "UK", deadline: "2026-11-02", status: "not-started", tag: "Leadership", color: "#BB8FCE", gpa: 3.0, ielts: 6.5, docs: ["4 Essays", "LOR x2", "Work Experience"] },
];

const DOCS = [
  { name: "Statement of Purpose (v3)", type: "SOP", date: "May 12", size: "45KB", status: "final" },
  { name: "Transcript (Official)", type: "Academic", date: "Apr 28", size: "2.1MB", status: "final" },
  { name: "Prof. Ahmed LOR", type: "LOR", date: "May 18", size: "120KB", status: "pending" },
  { name: "IELTS Certificate", type: "Language", date: "Jan 15", size: "800KB", status: "final" },
  { name: "CV / Resume (Latest)", type: "CV", date: "May 20", size: "78KB", status: "final" },
];

const PROFILE = { name: "Khyati Vishal", gpa: 3.4, ielts: 7.0, country: "India", degree: "B.Tech CS", year: "Final Year", publications: 1, experience: "2 internships" };

function daysUntil(dateStr) {
  const diff = new Date(dateStr) - new Date();
  return Math.ceil(diff / (1000 * 60 * 60 * 24));
}

function ChanceBar({ scholarship }) {
  const { gpa, ielts } = PROFILE;
  let score = 0;
  if (gpa >= scholarship.gpa) score += 40;
  else score += Math.max(0, 40 - (scholarship.gpa - gpa) * 30);
  if (scholarship.ielts === 0 || ielts >= scholarship.ielts) score += 30;
  else score += Math.max(0, 30 - (scholarship.ielts - ielts) * 20);
  if (PROFILE.publications > 0) score += 15;
  score += 15;
  score = Math.min(Math.round(score), 95);
  const color = score >= 70 ? "#4ECDC4" : score >= 45 ? "#F7DC6F" : "#FF6B35";
  return (
    <div style={{ marginTop: 8 }}>
      <div style={{ display: "flex", justifyContent: "space-between", fontSize: 11, color: "#888", marginBottom: 4 }}>
        <span>Match score</span><span style={{ color, fontWeight: 700 }}>{score}%</span>
      </div>
      <div style={{ background: "#1a1a2e", borderRadius: 4, height: 5, overflow: "hidden" }}>
        <div style={{ width: `${score}%`, height: "100%", background: color, borderRadius: 4, transition: "width 1s ease" }} />
      </div>
    </div>
  );
}

function SOPReviewer({ onClose }) {
  const [sop, setSop] = useState("");
  const [feedback, setFeedback] = useState(null);
  const [loading, setLoading] = useState(false);

  async function reviewSOP() {
    if (!sop.trim()) return;
    setLoading(true);
    setFeedback(null);
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          system: `You are an expert scholarship application advisor. Review the SOP and return ONLY a JSON object (no markdown, no backticks) with this structure:
{
  "score": <number 1-100>,
  "verdict": "<one sentence overall verdict>",
  "strengths": ["<point>", "<point>", "<point>"],
  "improvements": ["<point>", "<point>", "<point>"],
  "keywords_missing": ["<word>", "<word>"],
  "opening_hook": "<rewritten first sentence that's more compelling>"
}`,
          messages: [{ role: "user", content: `Review this SOP:\n\n${sop}` }]
        })
      });
      const data = await res.json();
      const text = data.content?.map(b => b.text || "").join("");
      const parsed = JSON.parse(text.replace(/```json|```/g, "").trim());
      setFeedback(parsed);
    } catch (e) {
      setFeedback({ score: 0, verdict: "Error analyzing SOP. Please try again.", strengths: [], improvements: ["Could not parse response"], keywords_missing: [], opening_hook: "" });
    }
    setLoading(false);
  }

  return (
    <div style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,0.85)", zIndex: 100, display: "flex", alignItems: "center", justifyContent: "center", padding: 20 }}>
      <div style={{ background: "#0f0f1a", border: "1px solid #2a2a3e", borderRadius: 16, width: "100%", maxWidth: 680, maxHeight: "90vh", overflowY: "auto", padding: 32 }}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 24 }}>
          <div>
            <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 22, fontWeight: 700, color: "#fff" }}>SOP Reviewer</div>
            <div style={{ fontSize: 13, color: "#666", marginTop: 2 }}>AI-powered feedback in seconds</div>
          </div>
          <button onClick={onClose} style={{ background: "none", border: "none", color: "#666", cursor: "pointer", fontSize: 24 }}>×</button>
        </div>

        <textarea
          value={sop}
          onChange={e => setSop(e.target.value)}
          placeholder="Paste your Statement of Purpose here..."
          style={{ width: "100%", height: 180, background: "#1a1a2e", border: "1px solid #2a2a3e", borderRadius: 10, padding: 16, color: "#fff", fontSize: 14, resize: "vertical", fontFamily: "inherit", boxSizing: "border-box", outline: "none" }}
        />
        <button
          onClick={reviewSOP}
          disabled={loading || !sop.trim()}
          style={{ marginTop: 12, width: "100%", background: loading ? "#2a2a3e" : "linear-gradient(135deg, #FF6B35, #FF8C5A)", border: "none", borderRadius: 10, padding: "14px", color: "#fff", fontWeight: 700, fontSize: 15, cursor: loading ? "not-allowed" : "pointer", transition: "all 0.2s" }}
        >
          {loading ? "✦ Analyzing your SOP..." : "✦ Review My SOP"}
        </button>

        {feedback && (
          <div style={{ marginTop: 24 }}>
            <div style={{ display: "flex", alignItems: "center", gap: 16, background: "#1a1a2e", borderRadius: 12, padding: 20, marginBottom: 20 }}>
              <div style={{ textAlign: "center" }}>
                <div style={{ fontSize: 42, fontWeight: 900, color: feedback.score >= 70 ? "#4ECDC4" : feedback.score >= 45 ? "#F7DC6F" : "#FF6B35", fontFamily: "'Playfair Display', serif" }}>{feedback.score}</div>
                <div style={{ fontSize: 11, color: "#666" }}>/ 100</div>
              </div>
              <div style={{ flex: 1 }}>
                <div style={{ fontSize: 14, color: "#ccc", lineHeight: 1.5 }}>{feedback.verdict}</div>
              </div>
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 16, marginBottom: 16 }}>
              <div style={{ background: "#0d1f1e", border: "1px solid #1a3330", borderRadius: 12, padding: 16 }}>
                <div style={{ fontSize: 12, color: "#4ECDC4", fontWeight: 700, marginBottom: 10, textTransform: "uppercase", letterSpacing: 1 }}>✓ Strengths</div>
                {feedback.strengths?.map((s, i) => <div key={i} style={{ fontSize: 13, color: "#aaa", marginBottom: 6, paddingLeft: 12, borderLeft: "2px solid #4ECDC4" }}>{s}</div>)}
              </div>
              <div style={{ background: "#1f1510", border: "1px solid #332010", borderRadius: 12, padding: 16 }}>
                <div style={{ fontSize: 12, color: "#FF6B35", fontWeight: 700, marginBottom: 10, textTransform: "uppercase", letterSpacing: 1 }}>↑ Improve</div>
                {feedback.improvements?.map((s, i) => <div key={i} style={{ fontSize: 13, color: "#aaa", marginBottom: 6, paddingLeft: 12, borderLeft: "2px solid #FF6B35" }}>{s}</div>)}
              </div>
            </div>

            {feedback.opening_hook && (
              <div style={{ background: "#13112a", border: "1px solid #2a2050", borderRadius: 12, padding: 16, marginBottom: 16 }}>
                <div style={{ fontSize: 12, color: "#BB8FCE", fontWeight: 700, marginBottom: 8, textTransform: "uppercase", letterSpacing: 1 }}>✦ Stronger Opening Line</div>
                <div style={{ fontSize: 14, color: "#ddd", fontStyle: "italic", lineHeight: 1.6 }}>"{feedback.opening_hook}"</div>
              </div>
            )}

            {feedback.keywords_missing?.length > 0 && (
              <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16 }}>
                <div style={{ fontSize: 12, color: "#F7DC6F", fontWeight: 700, marginBottom: 10, textTransform: "uppercase", letterSpacing: 1 }}>Missing Keywords</div>
                <div style={{ display: "flex", flexWrap: "wrap", gap: 8 }}>
                  {feedback.keywords_missing.map((k, i) => <span key={i} style={{ background: "#2a2510", border: "1px solid #3a3010", borderRadius: 20, padding: "4px 12px", fontSize: 12, color: "#F7DC6F" }}>{k}</span>)}
                </div>
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}

function EmailGenerator({ onClose }) {
  const [type, setType] = useState("cold-email");
  const [profName, setProfName] = useState("");
  const [uniName, setUniName] = useState("");
  const [research, setResearch] = useState("");
  const [email, setEmail] = useState("");
  const [loading, setLoading] = useState(false);

  async function generateEmail() {
    setLoading(true);
    setEmail("");
    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          system: "You are an expert at writing compelling, personalized academic emails for international students applying for scholarships and research positions. Write professional, concise emails that stand out. Return only the email text, no explanations.",
          messages: [{
            role: "user",
            content: `Write a ${type === "cold-email" ? "cold email to a professor for research supervision" : type === "lor-request" ? "polite letter of recommendation request" : "scholarship motivation email"}.
Professor: ${profName || "Professor Smith"}
University: ${uniName || "Target University"}
Research Interest: ${research || "Machine Learning and Computer Vision"}
Student Profile: ${PROFILE.name}, ${PROFILE.degree} from ${PROFILE.country}, GPA ${PROFILE.gpa}, ${PROFILE.experience}.`
          }]
        })
      });
      const data = await res.json();
      setEmail(data.content?.map(b => b.text || "").join(""));
    } catch (e) { setEmail("Error generating email. Please try again."); }
    setLoading(false);
  }

  return (
    <div style={{ position: "fixed", inset: 0, background: "rgba(0,0,0,0.85)", zIndex: 100, display: "flex", alignItems: "center", justifyContent: "center", padding: 20 }}>
      <div style={{ background: "#0f0f1a", border: "1px solid #2a2a3e", borderRadius: 16, width: "100%", maxWidth: 620, maxHeight: "90vh", overflowY: "auto", padding: 32 }}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 24 }}>
          <div>
            <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 22, fontWeight: 700, color: "#fff" }}>Email Generator</div>
            <div style={{ fontSize: 13, color: "#666", marginTop: 2 }}>Personalized academic emails, instantly</div>
          </div>
          <button onClick={onClose} style={{ background: "none", border: "none", color: "#666", cursor: "pointer", fontSize: 24 }}>×</button>
        </div>

        <div style={{ display: "flex", gap: 8, marginBottom: 20 }}>
          {[["cold-email", "Cold Email"], ["lor-request", "LOR Request"], ["motivation", "Motivation"]].map(([v, l]) => (
            <button key={v} onClick={() => setType(v)} style={{ flex: 1, padding: "10px 8px", background: type === v ? "linear-gradient(135deg, #4ECDC4, #2ab5ac)" : "#1a1a2e", border: "none", borderRadius: 8, color: type === v ? "#0f0f1a" : "#666", fontWeight: 700, fontSize: 12, cursor: "pointer" }}>{l}</button>
          ))}
        </div>

        {[["Professor Name", profName, setProfName, "e.g. Dr. Sarah Kim"],
          ["University", uniName, setUniName, "e.g. TU Munich"],
          ["Research Area", research, setResearch, "e.g. NLP, Robotics, Climate Science"]
        ].map(([label, val, setter, ph]) => (
          <div key={label} style={{ marginBottom: 14 }}>
            <label style={{ fontSize: 12, color: "#666", display: "block", marginBottom: 6, textTransform: "uppercase", letterSpacing: 1 }}>{label}</label>
            <input value={val} onChange={e => setter(e.target.value)} placeholder={ph}
              style={{ width: "100%", background: "#1a1a2e", border: "1px solid #2a2a3e", borderRadius: 8, padding: "12px 14px", color: "#fff", fontSize: 14, outline: "none", boxSizing: "border-box" }} />
          </div>
        ))}

        <button onClick={generateEmail} disabled={loading}
          style={{ width: "100%", background: loading ? "#2a2a3e" : "linear-gradient(135deg, #4ECDC4, #2ab5ac)", border: "none", borderRadius: 10, padding: 14, color: loading ? "#666" : "#0f0f1a", fontWeight: 700, fontSize: 15, cursor: loading ? "not-allowed" : "pointer", marginTop: 4 }}>
          {loading ? "✦ Generating..." : "✦ Generate Email"}
        </button>

        {email && (
          <div style={{ marginTop: 20, background: "#1a1a2e", border: "1px solid #2a2a3e", borderRadius: 12, padding: 20 }}>
            <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 12 }}>
              <div style={{ fontSize: 12, color: "#4ECDC4", fontWeight: 700, textTransform: "uppercase", letterSpacing: 1 }}>Generated Email</div>
              <button onClick={() => navigator.clipboard?.writeText(email)} style={{ fontSize: 12, color: "#666", background: "none", border: "1px solid #2a2a3e", borderRadius: 6, padding: "4px 10px", cursor: "pointer" }}>Copy</button>
            </div>
            <pre style={{ fontSize: 13, color: "#ccc", lineHeight: 1.7, whiteSpace: "pre-wrap", fontFamily: "inherit", margin: 0 }}>{email}</pre>
          </div>
        )}
      </div>
    </div>
  );
}

export default function ScholarPath() {
  const [tab, setTab] = useState("dashboard");
  const [showSOP, setShowSOP] = useState(false);
  const [showEmail, setShowEmail] = useState(false);
  const [selectedScholarship, setSelectedScholarship] = useState(null);
  const [filterStatus, setFilterStatus] = useState("all");
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    const link = document.createElement("link");
    link.href = "https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=DM+Sans:wght@300;400;500;600;700&display=swap";
    link.rel = "stylesheet";
    document.head.appendChild(link);
    setTimeout(() => setMounted(true), 100);
  }, []);

  const statusColor = { "not-started": "#444", "in-progress": "#F7DC6F", "submitted": "#4ECDC4", "accepted": "#4CAF50", "rejected": "#FF6B35" };
  const statusLabel = { "not-started": "Not Started", "in-progress": "In Progress", "submitted": "Submitted", "accepted": "Accepted", "rejected": "Rejected" };

  const filtered = filterStatus === "all" ? SCHOLARSHIPS : SCHOLARSHIPS.filter(s => s.status === filterStatus);
  const urgent = SCHOLARSHIPS.filter(s => daysUntil(s.deadline) <= 60 && s.status !== "submitted").sort((a, b) => daysUntil(a.deadline) - daysUntil(b.deadline));

  const styles = {
    app: { fontFamily: "'DM Sans', sans-serif", background: "#080812", minHeight: "100vh", color: "#fff", opacity: mounted ? 1 : 0, transition: "opacity 0.5s ease" },
    sidebar: { width: 220, background: "#0c0c1a", borderRight: "1px solid #1a1a2e", position: "fixed", top: 0, left: 0, bottom: 0, display: "flex", flexDirection: "column", padding: "28px 0" },
    main: { marginLeft: 220, padding: "32px 36px", minHeight: "100vh" },
    card: { background: "#0f0f1f", border: "1px solid #1e1e32", borderRadius: 16, padding: 24 },
    tag: (color) => ({ background: `${color}22`, color, border: `1px solid ${color}44`, borderRadius: 20, padding: "3px 10px", fontSize: 11, fontWeight: 700, letterSpacing: 0.5 }),
  };

  const navItems = [
    { id: "dashboard", icon: "◈", label: "Dashboard" },
    { id: "scholarships", icon: "⊕", label: "Scholarships" },
    { id: "documents", icon: "◻", label: "Documents" },
    { id: "profile", icon: "◉", label: "My Profile" },
  ];

  return (
    <div style={styles.app}>
      {showSOP && <SOPReviewer onClose={() => setShowSOP(false)} />}
      {showEmail && <EmailGenerator onClose={() => setShowEmail(false)} />}

      {/* Sidebar */}
      <div style={styles.sidebar}>
        <div style={{ padding: "0 24px", marginBottom: 36 }}>
          <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 22, fontWeight: 900, color: "#fff", letterSpacing: -0.5 }}>Scholar<span style={{ color: "#FF6B35" }}>Path</span></div>
          <div style={{ fontSize: 11, color: "#444", marginTop: 3, letterSpacing: 1, textTransform: "uppercase" }}>Study Abroad OS</div>
        </div>

        <div style={{ padding: "0 12px", flex: 1 }}>
          {navItems.map(item => (
            <button key={item.id} onClick={() => setTab(item.id)}
              style={{ width: "100%", display: "flex", alignItems: "center", gap: 12, padding: "11px 14px", background: tab === item.id ? "#FF6B3522" : "none", border: "none", borderRadius: 10, color: tab === item.id ? "#FF6B35" : "#555", cursor: "pointer", fontSize: 14, fontWeight: tab === item.id ? 700 : 500, textAlign: "left", marginBottom: 4, transition: "all 0.2s", borderLeft: tab === item.id ? "2px solid #FF6B35" : "2px solid transparent" }}>
              <span style={{ fontSize: 16 }}>{item.icon}</span>{item.label}
            </button>
          ))}
        </div>

        <div style={{ padding: "0 16px" }}>
          <div style={{ background: "linear-gradient(135deg, #FF6B3522, #FF6B3511)", border: "1px solid #FF6B3533", borderRadius: 12, padding: 16 }}>
            <div style={{ fontSize: 12, color: "#FF6B35", fontWeight: 700, marginBottom: 6 }}>✦ AI Tools</div>
            <button onClick={() => setShowSOP(true)} style={{ width: "100%", background: "none", border: "none", color: "#aaa", fontSize: 12, cursor: "pointer", textAlign: "left", padding: "4px 0" }}>→ SOP Reviewer</button>
            <button onClick={() => setShowEmail(true)} style={{ width: "100%", background: "none", border: "none", color: "#aaa", fontSize: 12, cursor: "pointer", textAlign: "left", padding: "4px 0" }}>→ Email Generator</button>
          </div>

          <div style={{ display: "flex", alignItems: "center", gap: 10, padding: "16px 4px 0" }}>
            <div style={{ width: 32, height: 32, borderRadius: "50%", background: "linear-gradient(135deg, #FF6B35, #F7DC6F)", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 13, fontWeight: 700, color: "#000" }}>A</div>
            <div>
              <div style={{ fontSize: 13, fontWeight: 600, color: "#ddd" }}>{PROFILE.name}</div>
              <div style={{ fontSize: 11, color: "#555" }}>{PROFILE.country} · {PROFILE.degree}</div>
            </div>
          </div>
        </div>
      </div>

      {/* Main */}
      <div style={styles.main}>
        {tab === "dashboard" && (
          <div style={{ animation: "fadeIn 0.4s ease" }}>
            <div style={{ marginBottom: 32 }}>
              <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 30, fontWeight: 900, letterSpacing: -1 }}>Good morning, Arjun. <span style={{ opacity: 0.3 }}>👋</span></div>
              <div style={{ color: "#555", marginTop: 6, fontSize: 15 }}>You have {urgent.length} upcoming deadlines. Let's keep moving.</div>
            </div>

            {/* Stats Row */}
            <div style={{ display: "grid", gridTemplateColumns: "repeat(4, 1fr)", gap: 16, marginBottom: 28 }}>
              {[
                { label: "Tracking", value: SCHOLARSHIPS.length, color: "#FF6B35", icon: "◈" },
                { label: "In Progress", value: SCHOLARSHIPS.filter(s => s.status === "in-progress").length, color: "#F7DC6F", icon: "⟳" },
                { label: "Submitted", value: SCHOLARSHIPS.filter(s => s.status === "submitted").length, color: "#4ECDC4", icon: "✓" },
                { label: "Due This Month", value: urgent.length, color: "#BB8FCE", icon: "◷" },
              ].map(stat => (
                <div key={stat.label} style={{ ...styles.card, textAlign: "center" }}>
                  <div style={{ fontSize: 28, color: stat.color, marginBottom: 4 }}>{stat.icon}</div>
                  <div style={{ fontSize: 32, fontWeight: 900, fontFamily: "'Playfair Display', serif", color: stat.color }}>{stat.value}</div>
                  <div style={{ fontSize: 12, color: "#555", marginTop: 2 }}>{stat.label}</div>
                </div>
              ))}
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 20, marginBottom: 20 }}>
              {/* Urgent Deadlines */}
              <div style={styles.card}>
                <div style={{ fontSize: 13, color: "#FF6B35", fontWeight: 700, textTransform: "uppercase", letterSpacing: 1, marginBottom: 16 }}>⚡ Urgent Deadlines</div>
                {urgent.slice(0, 4).map(s => {
                  const days = daysUntil(s.deadline);
                  return (
                    <div key={s.id} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "12px 0", borderBottom: "1px solid #1a1a2e" }}>
                      <div>
                        <div style={{ fontSize: 14, fontWeight: 600, color: "#ddd" }}>{s.name}</div>
                        <div style={{ fontSize: 12, color: "#555", marginTop: 2 }}>{s.country}</div>
                      </div>
                      <div style={{ textAlign: "right" }}>
                        <div style={{ fontSize: 18, fontWeight: 900, color: days <= 30 ? "#FF6B35" : "#F7DC6F", fontFamily: "'Playfair Display', serif" }}>{days}d</div>
                        <div style={{ fontSize: 10, color: "#555" }}>remaining</div>
                      </div>
                    </div>
                  );
                })}
              </div>

              {/* Quick Actions */}
              <div style={styles.card}>
                <div style={{ fontSize: 13, color: "#4ECDC4", fontWeight: 700, textTransform: "uppercase", letterSpacing: 1, marginBottom: 16 }}>✦ AI Quick Actions</div>
                {[
                  { label: "Review my SOP", sub: "Get instant AI feedback", icon: "◻", action: () => setShowSOP(true), color: "#FF6B35" },
                  { label: "Generate Professor Email", sub: "Cold email, LOR request", icon: "✉", action: () => setShowEmail(true), color: "#4ECDC4" },
                  { label: "Check My Eligibility", sub: "See match scores", icon: "◉", action: () => setTab("scholarships"), color: "#BB8FCE" },
                  { label: "Upload Documents", sub: "Secure document vault", icon: "⊕", action: () => setTab("documents"), color: "#F7DC6F" },
                ].map(action => (
                  <button key={action.label} onClick={action.action}
                    style={{ width: "100%", display: "flex", alignItems: "center", gap: 14, padding: "12px 14px", background: `${action.color}11`, border: `1px solid ${action.color}22`, borderRadius: 10, cursor: "pointer", marginBottom: 10, textAlign: "left" }}>
                    <span style={{ fontSize: 20, color: action.color }}>{action.icon}</span>
                    <div>
                      <div style={{ fontSize: 13, fontWeight: 600, color: "#ddd" }}>{action.label}</div>
                      <div style={{ fontSize: 11, color: "#555" }}>{action.sub}</div>
                    </div>
                  </button>
                ))}
              </div>
            </div>

            {/* Progress Timeline */}
            <div style={styles.card}>
              <div style={{ fontSize: 13, color: "#BB8FCE", fontWeight: 700, textTransform: "uppercase", letterSpacing: 1, marginBottom: 20 }}>Application Timeline</div>
              <div style={{ display: "flex", gap: 0, overflowX: "auto" }}>
                {SCHOLARSHIPS.map((s, i) => (
                  <div key={s.id} style={{ flex: 1, minWidth: 120, padding: "0 8px", borderRight: i < SCHOLARSHIPS.length - 1 ? "1px solid #1a1a2e" : "none" }}>
                    <div style={{ fontSize: 12, fontWeight: 700, color: "#ddd", marginBottom: 6 }}>{s.name}</div>
                    <div style={{ ...styles.tag(statusColor[s.status]), display: "inline-block", marginBottom: 8 }}>{statusLabel[s.status]}</div>
                    <div style={{ fontSize: 11, color: "#444" }}>Due {new Date(s.deadline).toLocaleDateString("en", { month: "short", day: "numeric" })}</div>
                    <ChanceBar scholarship={s} />
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {tab === "scholarships" && (
          <div>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 28 }}>
              <div>
                <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 28, fontWeight: 900 }}>Scholarships</div>
                <div style={{ color: "#555", fontSize: 14, marginTop: 4 }}>Track, check eligibility, manage applications</div>
              </div>
              <div style={{ display: "flex", gap: 8 }}>
                {["all", "not-started", "in-progress", "submitted"].map(f => (
                  <button key={f} onClick={() => setFilterStatus(f)}
                    style={{ padding: "8px 14px", background: filterStatus === f ? "#FF6B35" : "#1a1a2e", border: "none", borderRadius: 8, color: filterStatus === f ? "#fff" : "#555", fontSize: 12, fontWeight: 600, cursor: "pointer", textTransform: "capitalize" }}>
                    {f === "all" ? "All" : statusLabel[f]}
                  </button>
                ))}
              </div>
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(300px, 1fr))", gap: 20 }}>
              {filtered.map(s => {
                const days = daysUntil(s.deadline);
                return (
                  <div key={s.id} style={{ ...styles.card, borderTop: `3px solid ${s.color}`, cursor: "pointer", transition: "transform 0.2s, box-shadow 0.2s" }}
                    onMouseEnter={e => { e.currentTarget.style.transform = "translateY(-3px)"; e.currentTarget.style.boxShadow = `0 12px 40px ${s.color}22`; }}
                    onMouseLeave={e => { e.currentTarget.style.transform = ""; e.currentTarget.style.boxShadow = ""; }}>
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 12 }}>
                      <div>
                        <div style={{ fontSize: 16, fontWeight: 700, color: "#fff" }}>{s.name}</div>
                        <div style={{ fontSize: 13, color: "#555", marginTop: 2 }}>🌍 {s.country}</div>
                      </div>
                      <span style={styles.tag(s.color)}>{s.tag}</span>
                    </div>

                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 12 }}>
                      <span style={styles.tag(statusColor[s.status])}>{statusLabel[s.status]}</span>
                      <div style={{ fontSize: 13, color: days <= 30 ? "#FF6B35" : "#888" }}>
                        {days > 0 ? `${days} days left` : "Deadline passed"}
                      </div>
                    </div>

                    <div style={{ marginBottom: 14 }}>
                      <div style={{ fontSize: 11, color: "#555", marginBottom: 6, textTransform: "uppercase", letterSpacing: 1 }}>Requirements</div>
                      <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
                        <span style={{ fontSize: 11, color: PROFILE.gpa >= s.gpa ? "#4ECDC4" : "#FF6B35", background: PROFILE.gpa >= s.gpa ? "#4ECDC422" : "#FF6B3522", padding: "3px 8px", borderRadius: 6 }}>
                          GPA {s.gpa}+ {PROFILE.gpa >= s.gpa ? "✓" : "✗"}
                        </span>
                        {s.ielts > 0 && (
                          <span style={{ fontSize: 11, color: PROFILE.ielts >= s.ielts ? "#4ECDC4" : "#FF6B35", background: PROFILE.ielts >= s.ielts ? "#4ECDC422" : "#FF6B3522", padding: "3px 8px", borderRadius: 6 }}>
                            IELTS {s.ielts}+ {PROFILE.ielts >= s.ielts ? "✓" : "✗"}
                          </span>
                        )}
                      </div>
                    </div>

                    <ChanceBar scholarship={s} />

                    <div style={{ marginTop: 14, paddingTop: 14, borderTop: "1px solid #1a1a2e" }}>
                      <div style={{ fontSize: 11, color: "#555", marginBottom: 8, textTransform: "uppercase", letterSpacing: 1 }}>Documents Needed</div>
                      <div style={{ display: "flex", flexWrap: "wrap", gap: 5 }}>
                        {s.docs.map(d => <span key={d} style={{ fontSize: 11, color: "#888", background: "#1a1a2e", padding: "3px 8px", borderRadius: 6 }}>{d}</span>)}
                      </div>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}

        {tab === "documents" && (
          <div>
            <div style={{ marginBottom: 28 }}>
              <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 28, fontWeight: 900 }}>Document Vault</div>
              <div style={{ color: "#555", fontSize: 14, marginTop: 4 }}>All your application documents, organized</div>
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 16, marginBottom: 28 }}>
              {[["Total Files", DOCS.length, "#4ECDC4"], ["Finalized", DOCS.filter(d => d.status === "final").length, "#FF6B35"], ["Pending", DOCS.filter(d => d.status === "pending").length, "#F7DC6F"]].map(([label, val, color]) => (
                <div key={label} style={{ ...styles.card, textAlign: "center" }}>
                  <div style={{ fontSize: 28, fontWeight: 900, color, fontFamily: "'Playfair Display', serif" }}>{val}</div>
                  <div style={{ fontSize: 12, color: "#555", marginTop: 4 }}>{label}</div>
                </div>
              ))}
            </div>

            <div style={styles.card}>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
                <div style={{ fontSize: 13, color: "#fff", fontWeight: 700 }}>All Documents</div>
                <button style={{ background: "linear-gradient(135deg, #FF6B35, #FF8C5A)", border: "none", borderRadius: 8, padding: "8px 16px", color: "#fff", fontWeight: 700, fontSize: 12, cursor: "pointer" }}>+ Upload</button>
              </div>

              {DOCS.map((doc, i) => (
                <div key={i} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "16px 0", borderBottom: i < DOCS.length - 1 ? "1px solid #1a1a2e" : "none" }}>
                  <div style={{ display: "flex", alignItems: "center", gap: 14 }}>
                    <div style={{ width: 40, height: 40, background: "#1a1a2e", borderRadius: 8, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 18 }}>
                      {doc.type === "SOP" ? "📝" : doc.type === "LOR" ? "✉️" : doc.type === "Academic" ? "🎓" : doc.type === "Language" ? "🌐" : "📄"}
                    </div>
                    <div>
                      <div style={{ fontSize: 14, fontWeight: 600, color: "#ddd" }}>{doc.name}</div>
                      <div style={{ fontSize: 12, color: "#555", marginTop: 2 }}>{doc.type} · {doc.size} · Uploaded {doc.date}</div>
                    </div>
                  </div>
                  <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
                    <span style={styles.tag(doc.status === "final" ? "#4ECDC4" : "#F7DC6F")}>{doc.status === "final" ? "Final" : "Pending"}</span>
                    {doc.type === "SOP" && (
                      <button onClick={() => setShowSOP(true)}
                        style={{ background: "#FF6B3522", border: "1px solid #FF6B3544", borderRadius: 6, padding: "5px 10px", color: "#FF6B35", fontSize: 11, fontWeight: 700, cursor: "pointer" }}>Review →</button>
                    )}
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {tab === "profile" && (
          <div>
            <div style={{ marginBottom: 28 }}>
              <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 28, fontWeight: 900 }}>My Profile</div>
              <div style={{ color: "#555", fontSize: 14, marginTop: 4 }}>Your academic profile powers the eligibility checker</div>
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1fr 2fr", gap: 24 }}>
              <div style={styles.card}>
                <div style={{ textAlign: "center", marginBottom: 20 }}>
                  <div style={{ width: 72, height: 72, background: "linear-gradient(135deg, #FF6B35, #F7DC6F)", borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 28, fontWeight: 700, color: "#000", margin: "0 auto 12px" }}>A</div>
                  <div style={{ fontFamily: "'Playfair Display', serif", fontSize: 20, fontWeight: 700 }}>{PROFILE.name}</div>
                  <div style={{ fontSize: 13, color: "#666", marginTop: 4 }}>{PROFILE.degree} · {PROFILE.country}</div>
                </div>
                <div style={{ borderTop: "1px solid #1a1a2e", paddingTop: 20 }}>
                  {[["GPA", `${PROFILE.gpa} / 4.0`, "#4ECDC4"], ["IELTS", `${PROFILE.ielts} / 9.0`, "#FF6B35"], ["Publications", PROFILE.publications, "#BB8FCE"], ["Experience", PROFILE.experience, "#F7DC6F"]].map(([label, val, color]) => (
                    <div key={label} style={{ display: "flex", justifyContent: "space-between", marginBottom: 12 }}>
                      <span style={{ fontSize: 13, color: "#555" }}>{label}</span>
                      <span style={{ fontSize: 13, fontWeight: 700, color }}>{val}</span>
                    </div>
                  ))}
                </div>
              </div>

              <div>
                <div style={{ ...styles.card, marginBottom: 20 }}>
                  <div style={{ fontSize: 13, color: "#FF6B35", fontWeight: 700, textTransform: "uppercase", letterSpacing: 1, marginBottom: 16 }}>Eligibility Overview</div>
                  {SCHOLARSHIPS.map(s => (
                    <div key={s.id} style={{ display: "flex", alignItems: "center", gap: 16, marginBottom: 16 }}>
                      <div style={{ width: 110, fontSize: 13, fontWeight: 600, color: "#ddd" }}>{s.name}</div>
                      <div style={{ flex: 1 }}>
                        <ChanceBar scholarship={s} />
                      </div>
                    </div>
                  ))}
                </div>

                <div style={styles.card}>
                  <div style={{ fontSize: 13, color: "#4ECDC4", fontWeight: 700, textTransform: "uppercase", letterSpacing: 1, marginBottom: 16 }}>Boost Your Profile</div>
                  {[
                    { tip: "Add a research paper or preprint", impact: "High", color: "#FF6B35" },
                    { tip: "Get a 3rd letter of recommendation", impact: "High", color: "#FF6B35" },
                    { tip: "Improve IELTS to 7.5+ for Chevening", impact: "Medium", color: "#F7DC6F" },
                    { tip: "Document your volunteer or leadership work", impact: "Medium", color: "#F7DC6F" },
                  ].map((tip, i) => (
                    <div key={i} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "10px 0", borderBottom: i < 3 ? "1px solid #1a1a2e" : "none" }}>
                      <div style={{ fontSize: 13, color: "#aaa" }}>→ {tip.tip}</div>
                      <span style={styles.tag(tip.color)}>{tip.impact}</span>
                    </div>
                  ))}
                </div>
              </div>
            </div>
          </div>
        )}
      </div>

      <style>{`
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: none; } }
        * { box-sizing: border-box; }
        ::-webkit-scrollbar { width: 4px; height: 4px; }
        ::-webkit-scrollbar-track { background: #0f0f1a; }
        ::-webkit-scrollbar-thumb { background: #2a2a3e; border-radius: 2px; }
      `}</style>
    </div>
  );
}
